---
layout: post
title: How I use (and abuse) macOS
tags: [Tech, Apple, MacBook Pro, macOS]
---

Many years ago I switched from Windows to macOS. During this transition I really struggled with the lack of some basic "power user" features in macOS. Here are's a list of some of my favorite tools and tweaks I use to make my macOS usable.

## Window Management: Rectangle

[Rectangle](https://rectangleapp.com/) is great for easy and efficient window management. Honestly, I can't believe macOS still doesn't have something like this built-in, just like Windows `Win+Left`, `Win+Right`.

Rectangle has *a lot* of actions, but I really only ever use these:
- `Opt+Cmd+F` properly maximize the current window to fill the screen. I use this *all the time*. I really don't understand the purpose of the macOS "fullscreen" feature, rather than the "maximize" behavior found in Windows.
- `Opt+Cmd+Left` / `Opt+Cmd+Right` to place windows side-by-side. Pro tip: pressing the shortcut multiple times cycles the window between half, third, and two thirds.

![](/assets/img/using-macos/rectangle.png)

## Window Switching: Witch

[Witch](https://manytricks.com/witch/) is the best way to get proper window switching in macOS. I don't think I could use macOS without Witch. Personally I really can't comprehend why the vanilla macOS `Cmd+Tab` functionality works the way it does. It makes no sense to me.

Of all the options I tested, Witch is by far the best, and it is easily worth the $14 one-time fee they charge (I bought it years ago and still receive free updates). Also their customer support is very responsive.

Witch essentially gives me something as close as possible to Windows style `Alt+Tab` window switching behavior (but you can customize to do many other things). The main benefits for me are:

1. Witch lets you cycle windows, rather than just applications. So if I have, for example, two Chrome windows then I can cycle between them using Cmd+Tab.

2. Witch lets you cycle to minimized windows. Cycling to a minimized window restores its focus.

There are some caveats though.

- You'll occasionally run it some issues where you can't cycle to a window. For example, some dialogs in Java applications or some custom Photoshop windows.

- I've also noticed an issue that if I cycle *away* from Slack using Witch, then Slack doesn't realize that it has lost focus and will no longer give me notifications. I guess this is probably a bug in Slack, rather than Witch though...

For reference, here is what my Witch settings look like. I can't quite remember what changes I have made:

![](/assets/img/using-macos/witch1.png)
![](/assets/img/using-macos/witch2.png)
![](/assets/img/using-macos/witch3.png)

## Keyboard Modifications: Karabiner-Elements

[Karabiner-Elements](https://karabiner-elements.pqrs.org/) is another must-have for me. It lets you completely customize macOS keyboard mappings.

1. I map `fn` to `left_command` so that I can use my pinky finger when performing various keyboard shortcuts like `Cmd+A`, `Cmd+C`, `Cmd+V`, etc. I'm so used to using my left pinky for key-strokes, and since I never really use the `fn` key it doesn't really make any difference matter. I guess having 3 Cmd buttons is better than just 2!

2. I hate the TouchBar. Especially not having a physical escape key, which I use all the time (like to close Spotlight!) I'm glad Apple have finally realized their mistake with newer Macs. For me, fortunately, the top left physical key on a Swedish keyboard is *completely and utterly* useless. Does *anyone ever* type `°` or `§` enough to warrant a dedicated physical key? Instead, I can map that key to escape. Problem solved.

![](/assets/img/using-macos/swedishKeyboard.png)

![](/assets/img/using-macos/karabiner1.png)

You can also find some useful advanced modifications online.

1. I use "Change eject to cmd+ctrl+q" so that I can easily lock my computer by pressing the "Eject" key when I use my (older) Apple wireless bluetooth keyboard with my computer. I really don't know what else you'd use the Eject key for...

2. Some programs will happily accept non-breaking spaces if you accidentally hit Alt+Space. Thankfully you can easily disable this behavior using Karabiner.

![](/assets/img/using-macos/karabiner2.png)

## macOS Settings

Ok, on to some more basic settings I like to change in macOS. I'm sure I have missed some, but this is what I could think of off the top of my head.

### Theme

I like having my macOS system theme step to Dark. However, for most other programs, like Chrome and email, I still prefer to use Light. Most programs have an option to force a specific theme different from the system theme. However, in some cases you'll have to use a command to override the theme instead.

[https://apple.stackexchange.com/a/347101/243609](https://apple.stackexchange.com/a/347101/243609)

### Finder

- I really hate that macOS mixes Folders and Files by default when sorting directories. Luckily you can disable this using the `Keep folders on top` preference.

- Most sensible people will of course also enable `Show all filename extensions`.

![](/assets/img/using-macos/finder.png)

### Dock

- Minimize windows using `Scale effect`
- Auto-hide for extra screen real-estate

### Mouse

- The default TrackPad speed is infuriatingly slow. I always max it out (and honestly wish it could be even faster).
- I also disable Natural scrolling. I like having my fingers move in the same direction I'm scrolling. Down is down. Up is up.

### Updates

I tend to disable automatic updates in macOS, to avoid unexpected surprises. Things always break between updates, so I'd rather control it myself. If you delay installing major updates, make sure you click "More info..." to find minor updates! Otherwise you might miss these.

![](/assets/img/using-macos/updates1.png)
![](/assets/img/using-macos/updates2.png)

## Some other useful tools

[iTerm2](https://iterm2.com/) a must-have for any developer. A great terminal replacement for macOS.

[Amphetamine](https://apps.apple.com/us/app/amphetamine/id937984704?mt=12) is a very useful utility to force your mac to stay awake. For example, say you have some long-running process on-going but you don't want it to be interrupted. The nice thing is that you can use the predefined keep-awake times, so you don't accidentally forget to cancel the wake lock.

[Parallels](https://www.parallels.com/) is something I use extensively for running virtual machines on my mac. From time to time I have to do some dev work in Windows or Linux, and Parallels gives me a great way to do that. Bonus: Windows runs great in Parallels, good enough to even play games.

[Brew](https://brew.sh/) is useful for installing packages, libraries, and other CLI utilities. Some of the things I've used Brew for include: awscli, cmake, htop, mongodb, node, postgres, rclone, ssh-copy-id, etc.

[Macs Fan Control](https://crystalidea.com/macs-fan-control) let's you force the fan speeds on your mac to help cool it down quicker when running CPU-intensive tasks.

Other:

- [VS Code](https://code.visualstudio.com/)
- [GitHub Desktop](https://desktop.github.com/)
- [TablePlus](https://tableplus.com/)
- [Robo 3T](https://robomongo.org/)
- [Spark](https://sparkmailapp.com/)
- [Simplenote](https://simplenote.com/)
- [1Password](https://1password.com/)
- [TickTick](https://ticktick.com/)
- [Clocker](https://apps.apple.com/us/app/clocker/id1056643111?mt=12)
- [Disk Inventory X](http://www.derlien.com/)
- [GIF Brewery 3](https://apps.apple.com/us/app/gif-brewery-3-by-gfycat/id1081413713mt=12)
- [NordVPN](https://nordvpn.com/)
- Chrome, Dropbox, Slack, Spotify, Google Drive, etc...
