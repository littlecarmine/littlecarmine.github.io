---
layout: post
title: "Moving to Neovim"
date: 2023-02-05 17:20:00 +0500
categories: programming
---
We all have our go-to procrastination activities, whether that's cleaning your room or watchig tiktok. For many programmers, that activity is changing their code editor theme. 

I too loved changing my editor theme whenever I had an impending deadline or a project I just didn't want to go back to. But, changing editor themes is a particularly malicious type of procrastination. At least Cleaning your room gets a task checked off of your to-do list - it may not be the most urgent task, but it _is_ a task. Tiktok can probably get a laugh or two out of you. But changing editor themes rarely ends well. Most times, you find something you like, use it, realize it doesn't have the highlighting features you want for a certain languages, and you go back to your original.

So, last week, I decided to procrastinate with something more productive and educational. I was going to learn to use vim.

## Entering the rabbithole
I only follow a couple programming content creators, those being [mCoding](https://www.youtube.com/@mCoding) and [ThePrimeagen](https://www.youtube.com/@ThePrimeagen). mCoding is clearly the more serious, straightforward creator, delving deep into the technicals of python straightup, whereas ThePrimeagen is more edutainment oriented. But perhaps thanks to the edgy, comedic approach taken by ThePrimeagen, I found ThePrimeagen offers more interesting tips, one of them being "use Vim."

I passed it off as a meme most of the times, but I can't deny the premise did sound pretty sweet to me. I'm pretty good with touch typing and I genuinely enjoy typing (I have an 1580 races on [typeracer](https://play.typeracer.com)). My favorite productivity has been the ThinkPad largely thanks to the trackpoint which kept my hands on the keyboard at all times. So not having to move my hands around much while writing code would be very convenient to me.

The only big barrier to me seriously switching over was how barebones the Vim editor is. Pulling up the preinstalled vim on my terminal, it feels barren and empty. Then, I found an interesting [tutorial](https://youtu.be/stqUbv-5u2s) on setting up your Neovim to be like an IDE. At first, I didn't follow along, skeptical that something like that is possible, but as I saw the end result, I was convinced I wanted to move to Vim.

## Setting up Neovim
At this point, I knew nothing about the difference between Vim and Neovim. All I knew was that he used Neovim over Vim and I was going to follow exactly the steps. And the steps were pretty simple actually.

Firstly, I downloaded neovim from the latest stable release on github - simple enough. Then was _supposed_ to be the hard part, but thanks to [kickstart.lua](https://github.com/nvim-lua/kickstart.nvim), it was as simple as running a few lines of commands.

```bash
cd ~/.config
mkdir nvim
wget https://raw.githubusercontent.com/nvim-lua/kickstart.nvim/master/init.lua
```

Essentially, I create a nvim directory under my config directory, then downloaded the init.lua file from kickstart into ~/.confing/nvim.

kickstart.nvim came with so many conveniences preconfigured that I was almost felt bad about using it. It came with easy ways to install LSPs, packer to manage plugins, Telescope for searching files, and the tried-and-true Atom One Dark theme. Once I installed the LSPs for the languages I need, I was pretty much set.

## Tinkering with Customization
I won't lie. I'll write this section in a coherent manner, but in no way were my first attempts at customization straightforward. The first thing I tried to customize myself was, of course, change my theme. Atom One Dark is a nice theme, but it has a toy-like color scheme which I've grown out of, and I want something more unique. So, I went with [rose-pine](https://github.com/rose-pine/neovim) (still using it). After installing the theme, I left customization alone because I first wanted to get used to actually using the editor and learn what I need. Having used for a week or so, there were certain things I knew I really wanted to fix.
1. Run a terminal in the background so that I can have a server reflect changes live
2. Line length indicator
3. Improve line visibility
4. MAKE IT RECOGNIZE ERB AS A HTML TEMPLATE LANGUAGE (I use Rails)

### Improving the Terminal
What I pictured as a solution for 1 was something like a VSCode terminal where you can simply click on the code section to start coding again. But the solution I found was even better. [termtoggle](https://github.com/akinsho/toggleterm.nvim) allowed me to simply toggle the terminal on and off. When I wanted to start up my server, I'd toggle my terminal on and run `bin/dev`. Then, I'd just toggle it off, make whatever changes I need to make, and revisit the locally hosted site. I didn't have a terminal clogging up my screen all the time.

### Configurating the Editor
After some digging around, I found pretty simple solutions to 2 and 3.
```lua
vim.opt.scrolloff = 8 -- leave at least 8 lines above and below in the window when scrolling
vim.opt.colorcolumn = "80" -- put a colored bar at the 80th character to tell you to start wrapping
```
One problem I did encounter was translating over stackoverflow answers written in vimscript into the neovim lua. Little did I know this would come haunt me...

### Adding HTML Tags on ERB files 
This took me a while straightup. Many of the "solutions" I found for this were 10, 15 years old and didn't translate over to neovim well. In fact, even now, I'm not entirely sure which plugin exactly is duing the heavy lifting or if all of them collaborate. I tried figuring out in the readmes but [tpope](https://github.com/tpope)'s readmes are not the most detailed.

In the end, I had downloaded [vim-rails](https://github.com/tpope/vim-rails), [vim-surround](https://github.com/tpope/vim-surround), [ragtag](https://github.com/tpope/ragtag). But I was stuck between [nvim-ts-autotag](https://github.com/windwp/nvim-ts-autotag) and [vim-closetag](https://github.com/alvan/vim-closetag). I naturally started out with the nvim-ts-autotag, a neovim specific plugin. Because it was newer and specifically catering to neovim, I thought it would work better. But after wrestling with it for hours trying to figure out how to get it to recognize ERB, I found this [issue](https://github.com/windwp/nvim-ts-autotag/issues/85), specifically stating getting the plugin to recognize ERB is challenging.

Thus, I moved on instead to the older vim-closetag plugin. The most intimidating part about this plugin was how it asked the user to set up some stuff in `.vimrc`.
I didn't have `.vimrc`, I was on neovim!
After searching tutorials and docs online, I finally figured out how to translate the setup mentioned in vim-closetag into init.lua. Ironically, it was as simple as changing
```
let g:closetag_filetypes = 'eruby'
```
to
```lua
vim.g.closetag_filetypes = 'eruby'
```
And with this, my setup was complete for now.

## Takeaway and plans
The process of setting up neovim was full of surprises, pleasant and unpleasant. The existence of kickstart.nvim was truly a lifesaver, and a good number of things I thought would be difficult turned out pretty easy. But there were also parts that I anticipated to be trivial that were actually pretty difficult. I didn't mention this too much in the post but I was confused by how different the syntax was between different package managers like packer and plug.

While I really do owe my ass to kickstart, I do hope to move away from it slowly as I become better acquainted with neovim. Having one init.lua file makes it very difficult to find plugins and settings, so that would be my top priority of things to do to move away from kickstart. But for the time being, I want to settle down in this editor more, practice the motions, and chill. It's already been a pretty drastic transition so I think I can use a breather.
And yes, I do intend on staying with neovim permanently.
