---
layout: post
title: "RollerCoaster Tycoon (original) is still one of my all-time favorite games"
tags: [Tech, Cloud, DigitalOcean, Hosting]
hidden: true
---

RollerCoaster Tycoon (the original) released in 1999, is easily one of my all-time favorite games. I used to be completely obsessed by it when I was a kid. Every now and then I still get a craving for it.

In 2016, a remastered version of RCT was released, called RollerCoaster Tycoon Classic. While it retains a lot of the original game, it still just doesn't feel quite the same.

Thankfully you can still buy the original game on Steam, [RollerCoaster Tycoon: Deluxe](https://store.steampowered.com/app/285310/RollerCoaster_Tycoon_Deluxe/), which includes the original game, and its expansion packs, Corkskrew Follies and Loopy Landscapes.

While doing 2 weeks of hotel quarantine after flying back to Taiwan, I ended putting in nearly 40 hours of play time. It definitely helped make the days pass a little quicker! 😅

![](/assets/img/rct/steam.png)

The game only runs on Windows. So, if you're on a Mac, you'll have to dual-boot, run it in a VM using ([Parallels](https://www.parallels.com/) or [VirtualBox](https://www.virtualbox.org/)), or perhaps try [Porting Kit](https://portingkit.com/game/285). Since I already use Parallels for doing Windows development on my Mac, I figured I would give it a shot.

The game starts up just fine on Windows 10. However, I did run into two issues, which after some troubleshooting, I managed to work around.

## Resolution Issue

For some reason, the default resolution is way too small, too small that I couldn't even access the resolution setting, which is near the bottom of the options window. I'm guessing maybe this has something to do with display scaling in Windows.

![](/assets/img/rct/fullscreen.png)

Fortunately, I found a way to work around the issue by editing the game config file to change the game to windowed mode.

Right-click the game in Steam, go to Properties, Local Files, and then click "Browse Local Files". You can find the configuration file format here. Essentially it's just a binary file format storing all of the game settings.

![](/assets/img/rct/browse.png)

Next, open the Data folder where you'll find a file called "Game.cfg"

![](/assets/img/rct/data.png)

You'll need to open this file in a hex editor. Since this is a binary file format, you can't just open it in a normal text editor. I used Visual Studio Code with the Hex Editor extension, but any Hex Editor would work just as well.

![](/assets/img/rct/editor.png)

The config file format is [well documented](http://tid.rctspace.com/gamecfg1.html). In order to change the screen resolution setting, we need to update the value stored at offset `0x17` (row `1` column `7`).

This controls the screen resolution game setting, and can take the following values.

- `00`: Windowed
- `01`: 640x480 (default)
- `02`: 800x600
- `03`: 1204x760

I changed the value from `01` to `00` to set it to windowed mode. Save the file and relaunch the game!

![](/assets/img/rct/windowed.png)

Now, the game might look very small if you have a large display or high-resolution display. So I found the following to work quite well.

1. Put Parallels in Fullscreen mode.
2. Change your Windows display resolution to 1280x1024
3. Run the game in windowed to fill the screen

![](/assets/img/rct/cover.png)

## Right-Click Issue

For some reason it seems right-click causes problems in the game (I'm guessing this is a compatibility issue between the game and the virtual machine).

When I try to right-click the game seems to think I'm holding down the right mouse button, which causes the map to jump around like crazy. Right-clicking is quite important in RCT. You need it in order to remove paths and trees. Luckily, I found a workaround.

In Parallels you can also perform a right-click by holding down Shift and Control while left clicking (a compatibility feature for old mac mice). You still need to make sure you perform your click fast without moving the mouse, as otherwise it might cause the map to move anyways.

> By default, Parallels Desktop is set to mimic a right-click when you press Shift+Control (see below) and click the mouse.

## Misc Tips

I've developed a pretty solid strategy for playing RCT by now. However, every time I play RCT I still learn something new. Like, how apparently the excitement rating for Go Karts gets impacted negatively if you go underground. I've also started charging $0.20 for all restrooms. Also, understanding the path finding rules will definitely help you design better park layouts.

Check out the RCT Fandom website for more [Hints and Tips](https://rct.fandom.com/wiki/Hints_and_Tips).
