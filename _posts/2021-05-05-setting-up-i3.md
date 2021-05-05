---
layout: post
title: Setting up the i3 Window Manager
readtime: true
tags: [linux, guide, i3]
---

A few weeks ago, I started running Linux full time again (specifically, Ubuntu 20.04). I also decided to use the [i3 window manager](https://i3wm.org/). I haven't run this setup since college, so I had to re-learn (read: re-google) a lot of the solutions, tips, and tricks I found before that made my experience more pleasant. In this post, I will outline some of the more advanced configuration changes I made and point to some good documentation for getting started. I did all of the following configuration on Ubuntu 20.04, so I cannot speak to other versions/flavors of Linux. I hope that this post consolidates a lot of the information out there and makes it easier for someone to fully setup i3 in one go :)

## Why Use i3?

Before we dive in, I think it's prudent for me to explain why I chose to use i3. The biggest difference between i3 and the default GNOME desktop environment on Ubuntu 20.04 is that i3 is a tiling window manager. If you haven't heard of tiling window managers, you should check them out. In short, it means that your windows are automatically laid out in such a way that your screen is always full. Open one window, and it's full screen. Open another window, and each window now takes up half of the screen.

I much prefer this style as opposed to dragging and dropping windows around to arrange my desktop how I like. I know there are keyboard shortcuts that snap applications to the side on Windows, but it's not nearly as powerful. i3 is also very configurable so I have a lot of control over exactly how my applications and workspaces are laid out. This allows me to move much faster via keeping my hands on the keyboard. For me, it's also a chance to deal more intimately with a lot of the things we take for granted on more polished desktop environments like battery management, monitor management, etc. Don't get me wrong, this is definitely a blessing and a curse, but it's fun to learn more about these processes. With that being said, let's dive into some of the configurations available to us in i3!

## Getting Started

I actually don't have much to add for setup and installation. There are a few pretty good tutorials for getting started with i3 out of the box. Rather than repeat everything, I thought I'd point to some guides that have helped me along the way.

- [i3 Configuration Screencast](https://www.youtube.com/watch?v=j1I63wGcvU4): I think this screen-cast is relatively well-known at this point. Youtube says it was posted around 5.5 years ago, but pretty much all of it is still relevant. If you're totally brand new to i3 or window managers, this is the place to go as these videos will walk you through the basic mechanics of using a window manager.
- [Install and Setup i3](https://kifarunix.com/install-and-setup-i3-windows-manager-on-ubuntu-20-04/): This tutorial is more recent and concise than the screen casts, but still provides a lot of information. I actually went through it myself when I installed i3 and it was great!
- [i3 User Guide](https://i3wm.org/docs/userguide.html): Lastly, the user guide is quite detailed and a great read for anyone looking to find some advanced configurations. 

Note that there is a popular fork of i3, [i3-gaps](https://github.com/Airblader/i3), which as the name suggests allows for gaps between each window if you desire. I am currently using i3-gaps, but it's really easy to switch between them. There's a concise guide [here](https://gist.github.com/boreycutts/6417980039760d9d9dac0dd2148d4783) that walks you through setting up i3-gaps.

### Status Bar

I highly recommend installing [polybar](https://github.com/polybar/polybar). In my opinion it looks much better than the default i3status bar that comes with i3, especially after you do some configuration. I took inspiration from [this](https://github.com/nicomazz/i3-polybar-config/blob/master/polybar/config) config, but I was able to tweak it easily by checking out the options on the [polybar configuration wiki](https://github.com/polybar/polybar/wiki/Configuration). You can check out my config [here](https://github.com/bradsherman/dotfiles/blob/master/polybar/config). I used i3status or i3blocks for a while before finding polybar and now that I've found it, I can't go back! 

### Locking and Suspending

One great way to improve your i3 workflow is to configure the lock screen and suspend behavior. By default i3 uses `i3lock` to lock your screen, which is just an ugly white screen. You can add an image, but beyond that the configuration is not straightforward. I highly recommend using [betterlockscreen](https://github.com/pavanjadhaw/betterlockscreen) to handle locking your screen. The `README` in the github repo is easy to follow, and once you have betterlockscreen installed you just need to find a picture for your lock screen. betterlockscreen also provides an easy way for you to automatically lock your screen when you suspend your machine.

Lastly, the [article](https://kifarunix.com/install-and-setup-i3-windows-manager-on-ubuntu-20-04/) I mentioned above has a great script for dealing with logout, locking the screen, and suspending. I tweaked it a bit [here](https://github.com/bradsherman/dotfiles/blob/master/i3/bin/display).

### Rofi

When I ran i3 in college, I used [rofi](https://github.com/davatorium/rofi) but didn't realize how easy it is to make it look much nicer! Below is the script I run to launch apps:

```shell
#!/usr/bin/env bash

rofi -combi-modi window,drun,ssh -show combi -icon-theme "Papirus" -show-icons
```

If you put that in a file at `~/rofi_launcher.sh`, you can add this line to your i3 config to quickly launch apps:

```shell
bindsym $mod+d exec --no-startup-id ~/rofi_launcher.sh
```

This allows me to quickly switch between apps, launch new instances of apps, as well as ssh into machines I have configured in `~/.ssh/config`. It also shows icons for any apps that have them. This script is very much inspired from [this](https://wiki.archlinux.org/index.php/Rofi) arch wiki page.

I also ended up making my own solarized light theme for rofi, which you can see [here](https://github.com/bradsherman/dotfiles/blob/master/solarized-light.rasi). Just put that into `~/.local/share/rofi/themes` and run `rofi-theme-selector` to use it! You can also do this with [other rofi themes](https://github.com/adi1090x/rofi).

### Notifications

Another aspect of i3 that I did not experiment with before was notifications. In my opinion they are pretty bland by default, but with [dunst](https://dunst-project.org/) we can [add icons](https://wiki.archlinux.org/index.php/Dunst#Icon_sets), make notifications transparent, and much more! I found it easy to get started with [this](https://linuxconfig.org/get-better-notifications-in-your-wm-with-dunst) tutorial. Once I had dunst up and running it was a matter of playing around with the config to get it just right. [Here](https://github.com/bradsherman/dotfiles/blob/master/dunst/dunstrc) is my `dunstrc`, which is modeled off of the solarized light theme as well.

### Multiple Monitors

Probably the most difficult thing I had to do was get a better workflow for multiple monitors. I use a tool called `xrandr` to deal with switching between setups (laptop only or laptop + external monitor). I found a handy script [here](https://gist.github.com/amanusk/6b79d407945ca79caa945ce2658fd987) that I tweaked a bit for my specific setup, so now I can switch between setups with just a few keystrokes. 

The most time-consuming aspect was trying to get the picture to match up on my laptop and my external monitor which have different resolutions. I have a satisfactory setup right now, but I'm sure I'll continue to tweak this as time goes on. Unfortunately I do not have a magic command that will work for everyone since all setups are different. However, you can check out the `xrandr` commands I use [here](https://github.com/bradsherman/dotfiles/blob/master/i3/bin/display#L57), and here's a few links I used to figure out the best command to run: [1](https://blog.summercat.com/configuring-mixed-dpi-monitors-with-xrandr.html), [2](https://wiki.archlinux.org/index.php/HiDPI), [3](https://fedoramagazine.org/how-to-get-firefox-looking-right-on-a-high-dpi-display-and-fedora/), and [4](https://techknowfile.dev/using-a-hidpi-monitor-with-a-low-resolution-external-monitor-ubuntu-i3/).

The last piece of my multi-monitor puzzle was automatically running my xrandr commands when I plug/unplug my external monitor. Originally, I'd have to manually run a script to get output on my external monitor after plugging it in. No one wants to do that! Thankfully I found [srandrd](https://github.com/jceb/srandrd) which was easy to setup, and now my screen output is automatically updated when my monitor is connected or disconnected.

{: .box-note}
I first tried [this](https://tyler.vc/auto-monitor-detection-on-linux) article which describes a solution using `udev` rules, but I could not get that to work on my setup. `srandrd` worked for me essentially out of the box, but I thought I'd add this other solution in case it works for others.

## Wrapping Up

The main issue I have with my setup right now is battery life. Right now my laptop dies if it sits idle on battery power for a while. Currently the OS is not configured to hibernate, and when I tried to turn it on I ran into some errors about not having enough swap space. I have a few possible solutions to try, but just haven't gotten around to it yet. It'll be really nice to have this so I am not so reliant on being plugged in. Other than that it's been smooth sailing!

This ended up being somewhat of a brain dump from all of my research over the past few weeks. There were many times where I was going down a rabbit hole trying to find a solution and I remembered that I had traversed the same rabbit hole my first time setting up i3. In the hopes of avoiding doing that a third time, I wrote this post for myself and others. I tried to aggregate all of the useful links I found, as well as highlight some configuration options that I wasn't aware of during my first stint using i3. Lastly, I want to give a huge shout-out to all of the people who created the articles/configurations I've linked in this post. They drastically reduced the learning curve for me, and hopefully this article does for others as well. If you have any other cool configurations that I missed please drop them in the comments! Thanks for reading!

