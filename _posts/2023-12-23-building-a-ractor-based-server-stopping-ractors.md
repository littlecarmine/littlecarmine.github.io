---
layout: post
title:  "Building a Ractor-based server: Stopping Ractors"
date:   2023-12-23 12:15:00 +0900
categories: programming
---
Recently, I published [Mooro](https://github.com/Forthoney/mooro), a non-toy, parallel server powered by Ractors with logging and graceful stopping. Many rubyists dismiss Ractors as theoretically appealing but practically limited. I wish to dispel this notion in this blog series where I will walk through how Mooro redesigns the standard features of a practical Ruby web server like logging, stopping, and more for Ractor-based parallelism. Ractors may not have the same methods as Threads, Processes, and Fibers, but they are far from practically limited - they just require different design patterns.


## Background: Ractors do their own thing
You have two ways of talking to a running Ractor. You can send a message to their figurative mailbox which the other Ractor will read whenever they please or you can leave a message on your figurative porch which the other Ractor will take whenever they please. The key phrase here is "whenever they please". You have no way of enforcing your will upon Ractors unlike Threads which have `Thread#exit` and `Thread#raise`. And according to this [rejected feature proposal](https://bugs.ruby-lang.org/issues/18139) Ractors will keep on doing their own thing for the foreseeable future.

## Stopping workers
Stopping a worker in a thread based server is a question with a straightforward solution: use `Thread#raise` to raise an error somewhere in the execution of the thread. For Ractors, I needed to figure out a couple different ways of handling the stopping of workers. The first is a more naive but straighforward approach which assumes that whatever server logic run inside the worker does not hang.


```ruby
worker = Ractor.new(supervisor) do |supervisor|
  until (client = supervisor.take) == :terminate
    do_something(client)
  end
end
```


Here, the supervisor will yield one of two objects: a `TCPSocket` or the symbol literal `:terminate`. Upon the worker taking a `:terminate` symbol, it will, surprise surprise, terminate! This enables workers to stop without interrupting whatever non-hanging operation they were working on.

Ideally, the logic run by Mooro users inside a worker is somewhat brief and does not hang, in which case strategy 1 will do the job no problem. The reality unfortunately is that we are _not_ all responsible adults here, and some operations may potentially hang. Thus, any serious server should provide some way to "ungracefully stop" a worker. This is where my second strategy comes in. The second strategy is less elegant than the first, but it successfully stops a worker even when an worker operation may hang.


```ruby
worker = Ractor.new(supervisor) do |supervisor|
  clients = Thread::Queue.new
  runner = Thread.new do
    while (current_client = clients.pop)
      do_something(current_client)
    end
  end
  until (client = supervisor.take) == :terminate
    clients.push(client)
  end
  runner.raise(TerminateServer)
  runner.join
end
```


A `Thread` inside a `Ractor`? Sacrilege... right?
Well, not really. Ractors and threads are not at all mutually exclusive. I would argue they actually work pretty well in conjunction here, where the Ractor serves as a standin for where a `Process` would have been used traditionally. My bigger concern is the client queue.  The current setup means that the worker will keep adding to the queue regardless of whether the runner is able to keep up. This is necessary for interrupting the runner, since the worker needs to be able to keep on receiving messages regardless of the runner's state. If something like a `SizedQueue` is used, the following could happen.


```ruby
worker = Ractor.new(supervisor) do |supervisor|
  clients = Thread::SizedQueue.new(5)
  runner = Thread.new do
    while (current_client = clients.pop)
      do_something(current_client) # runner is hanging here
    end
  end
  until (client = supervisor.take) == :terminate
    clients.push(client) # This operation blocks because the runner is not popping any clients off the queue
  end
  runner.raise(TerminateServer)
  runner.join
end
```


I'm not yet fully satisfied with this strategy. So, the vanilla `Mooro::Server` instead opts for strategy 1. If you need strategy 2, you can Opt in by `include Mooro::Plugin::InterruptableServer`.

## Stopping the supervisor
The problem is further complicated by the supervisor. Despite both using Ractors, a supervisor Ractor is very different in behavior to a worker Ractor. A worker Ractor takes a message from an outsider then does something; it is triggered by external stimulus. It is the recipient of messages. On the other hand, the supervisor should indefinitely loop and accept clients from the socket without receiving messages. It is the producer of messages, but not a recipient. In code, it would look something like


```ruby
supervisor = Ractor.new(host, port) do |host, port|
  socket = TCPServer.new(host, port)
  loop do
    Ractor.yield(socket.accept)
  end
end
```


How do you, an external factor, stop a Ractor that doesn't receive messages? You don't. There is no means to stop this Ractor with vanilla Ruby. What if I make it receive the `:terminate` message like the worker?


```ruby
supervisor = Ractor.new(host, port) do |host, port|
  socket = TCPServer.new(host, port)
  until Ractor.receive == :terminate do
    Ractor.yield(socket.accept)
  end
end
```


Because Ractor.receive is a blocking operation, the supervisor will serve just one client and block on `Ractor.receive`, waiting for someone to send it a message. So, something needs to continuously send the supervisor a non-`:terminate` message. Perhaps the main Ractor?


```ruby
class Server
  def start
    Signal.trap 'INT' do
      supervisor = make_supervisor
      worker = make_worker(supervisor)
      loop { supervisor.send(:okay) } # This is an infinite loop
    end
    supervisor.send(:terminate)
  end

  def stop
  end
end
```


Here, the main Ractor being permanently occupied sending `:okay` messages to the supervisor in the start method, resulting in a never-terminating start method. Not only is this somewhat of a waste of resources for the main Ractor, it's also very unergonomic. This is not what I want for `Mooro`. One reasonable solution I found is for the supervisor to modify the supervisor to consume messages from two sources: the main Ractor and itself. The supervisor will indefinitely send itself the `:continue` message and the main Ractor will send the supervisor a `:terminate` message when `Server#stop` is called.


```ruby
class Server
  def start
    @supervisor = Ractor.new(host, port) do |host, port|
      socket = TCPServer.new(host, port)
      Ractor.current.send(:continue)
      
      until Ractor.receive == :terminate do
        Ractor.yield(socket.accept)
        Ractor.current.send(:continue)
      end
    end
  end

  def stop
    @supervisor.send(:terminate)
  end
end
```


This solution is fully functional, but Mooro instead opts for a Thread based solution. The supervisor is run on a separate thread and `Server#stop` raises an error on the supervisor thread. I found it doesn't make much sense to dedicate a whole Ractor to the supervisor considering the main Ractor will be sitting idle the vast majority of the time. I would much rather have an additional worker Ractor than a Ractor dedicated just to `server.start` and `server.stop`.
