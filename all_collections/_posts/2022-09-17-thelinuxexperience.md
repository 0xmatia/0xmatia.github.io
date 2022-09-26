---
layout: post
title: My experience migrating to Linux
date: 2022-09-26
categories: ["linux", "first post"]
---

Today I'd like to share with you my experience using linux as my main operating system at home.
I will go over the installation process and then I'll cover some of linux
so-called "weaknesses" (gaming, nvidia drivers etc...).

To clarify, this is the experience of a guy who used linux for couple of years in a VM for research
and development, but never as a day to day OS. I once dual-booted Windows and Ubuntu on my old
laptop but haven't used it much for things other than development.
The real challenge is to be able to perform the day to day tasks with linux. Examples are: printing,
playing DRM content, gaming, peripheral devices compatibility and more.

- [Picking the distro](#picking-the-distro)
- [Installing Manjaro](#installing-manjaro)
- [Display](#display)
- [Importing config](#importing-config)
  - [Fixing the shell](#fixing-the-shell)
  - [Missing fonts](#missing-fonts)
  - [Fixing Polybar](#fixing-polybar)
  - [i3 config tide-up](#i3-config-tide-up)
  - [NVIM](#nvim)
- [Sound](#sound)
- [Authentication Agent](#authentication-agent)
- [Lock screen and Power settings](#lock-screen-and-power-settings)
- [What's the time?](#whats-the-time)
- [Keyboard layout](#keyboard-layout)
- [Mounting my HDD](#mounting-my-hdd)
  - [Persistent Mount](#persistent-mount)
- [Games](#games)
  - [Xbox one controller](#xbox-one-controller)
- [Themes](#themes)
- [Final thoughts](#final-thoughts)

## Picking the distro

I have used many distros in my VMs, but Manjaro i3 was my favorite. Not because of its features
or aesthetics (it actually looks pretty bad by default) but because it is a good starting point:

- It is an Arch Linux derivative, which means AUR support (once you try this you can't go back)
and other Arch goodies.
- It takes care of installing all the necessary packages, including drivers and libraries.
- It looks bad, but works out of the box.

<sub> Manjaro i3 out-of-the-box:</sub>
![Default Manjaro i3 installation](https://miro.medium.com/max/1400/1*zqS8DwYWB-GKulnJ3Yxsig.png)

I downloaded the latest Manjaro i3 ISO (you can too from [Manjaro download page](https://manjaro.org/download/)),
created a bootable flash drive using rufus, turned off my machine, said goodbye Windows one last time,
and started the journey.

## Installing Manjaro

The installation itself went pretty smoothly; I chose the correct disk (I have one SSD and one HDD),
added swap of around 18gb, chose libreoffice as my office suite and clicked install.
However, before I could boot into the live environment and start the installer, I encountered two
"problems":

- Secure Boot: Trying to boot Manjaro caused my UEFI firmware to display a very angry red message
telling me something about firmware integrity and tampering. I knew it must be secure boot so I went
to the UEFI settings to disable secure boot but was surprised to find out that the option to change
the setting was grayed out. Instead I had the option to change the OS type from "Microsoft UEFI" to "Other OS."
This is actually great because it means only some of the protections granted by Secure Boot will be
disabled but not and of them (At least this is what it says beneath the setting).
- I have an Nvidia GPU (Nvidia GTX 1060 3GB to be exact). Upon boot I had to choose "Proprietary
drivers" or open source drivers in grub. Apparently Nvidia's closed source drivers work better than
the open source ones so I chose Proprietary drivers.

After the installation process finished, I restarted my machine, unplugged the flash drive and
successfully booted Manjaro i3!

Below you will find everything I did in detail to get the machine to the point where I am happy
with it.

## Display

I was surprised actually that both monitors worked without me doing anything (Nvidia drivers on linux are
notoriously bad). The problem was that my secondary monitor (the smaller one) was actually the
primary one and its "virtual location" was wrong. I knew `xrandr` is the command that can change
monitor settings but I didn't know how to use it to change secondary to primary and how to switch the
monitors' location. I installed `arandr`, a graphical tool used to "generate" `xrandr` commands.
I switched the monitors' location and set the big one to be the primary display, pressed apply and
it worked like magic - until restart. `xrandr` stuff aren't actually saved between sessions, but
`arandr` is able to create a bash script with the `xrandr` command to run to apply the wanted settings.
What can we do with this?

Manjaro uses `lightdm` as its display manager. The display manager is essentially the login screen where
a user can choose a WM of his choice and login. `lightdm` can actually be used to run a script at startup
(`lightdm` is started when the boot process is completed, so the display drivers should already be up).
I modified the `/etc/lightdm/lightdm.conf` file and under `display-startup-script` I added the path
to the script. Make sure you put this under `[Seat:*]`. I accidentally put the line somewhere else and
couldn't boot so I had to enter live environment, type `manjaro-chroot -a` to enter the installation
context (fancy chroot) and fix the config. After all that, I restarted my machine and the display
settings were correct and reapplied themselves automatically each reboot.

## Importing config

At this point I decided to import my configuration (dotfiles for various programs). I git cloned
the repository from my [Github account](https://github.com/0xmatia/dotfiles) and immediately noticed
some bugs:

- Alacritty (my terminal of choice) is missing. This was a simple `yay -S` and then I could open up terminals.
- Polybar showed two bars on my main monitor and none on the secondary one.
- Missing icons in Polybar (missing fonts probably).
- No Hebrew keyboard layout.
- No fzf. (Simple `yay -S` fixed it).

### Fixing the shell

I use ZSH as my shell, so obviously some packages were missing:

- zsh
- oh-my-zsh
- zsh-autosuggestions
- zsh-syntax-highlighting
- neofetch

I installed them all and everything looked great. I just needed to log out and back in order for
the default shell to change, but `arcolinux-logout` was missing (this is a binary I use for logging
out and shutting down the machine). It didn't work and then I remembered `libwnck3` is required for
it to work. After I installed the library, I logged out, logged back it and everything worked as expected.

### Missing fonts

This was actually pretty easy - All I needed was `Jetbrains Mono Nerd` (nerd-fonts-jetbrains-mono in
the AUR) and `Font Awesome` for Polybar and then both Polybar and Alacritty looked OK.

### Fixing Polybar

The script that launches Polybar searches for displays using `xrandr` and starts Polybar for each display.
It passes the `MONITOR` environment variable to Polybar but in the config itself I used a hard-coded value,
so it explains why it didn't work. I changed the config to respect the `MONITOR` environment
variable and it fixed the double bar problem - now each monitor has its own bar.

I added the `pin-workspaces` options to show only the workspaces that are relevant to the current
monitor. Then I noticed that the tray appears on both monitors. I created a secondary bar that
inherits the first one and overrides `tray-position` to be null. Then in the `launch.sh` script
I check for each monitor - if it is the secondary display I launch the secondary bar (without the tray)
and if it is the main one I launch the main bar.

### i3 config tide-up

I went over the i3 config and deleted / added / tweaked some settings:

- Discord is now on workspace 9
- I installed autotiling (it wasn't installed so it did noting)
- Deleted lots of random unused stuff
- Changed the font in the window title to Jetbrains Mono (I don't actually use window titles much, but
when I do I'd like to have a better font)
- I commented out `vmware-user` (useful inside a VMWare guest, not so much on a physical machine)
- I commented out `polkit-agent` because I forgot what it was doing there (we'll get back to that)
- I added `exec --no-startup-id setxkbmap -model pc104 -layout us,il -option grp:alt_shift_toggle`
in order to have Hebrew input.
- Noticed that both `xfce4-taskmanager` and `Flameshot` were missing so I installed them.

### NVIM

My main text editor is nvim (you can check out the config [here](https://github.com/0xmatia/dotfiles/tree/main/.config/nvim)).
I am using `packer` as nvim package manager so I had to install it using yay. I ran `:packer sync`
to download all plugins and reopened nvim. It worked OK mostly, except I had to change some plugin configurations
that changed in recent updates and install rust, lua and python language servers but other than that
everything worked.

## Sound

One example of something you never pay attention to while running linux inside a VM is sound - me
personally never got sound to work in linux (I never had to).
Out of the box my headset didn't work (HyperX Cloud Alpha S). I am not 100% sure how the sound "system"
in linux works (I read about it a bit in the Arch wiki) but I knew I needed pulseaudio and alsa
so I installed: `pulseaudio, pulseaudio-alsa, pavucontrol, pa-applet`. I enabled the pulseaudio service
(`systemctl --user enable --now pulseaudio.service`) and then in pavucontrol I could see all me devices.
Then I changed the i3 config to automatically execute `pa-applet` on startup so I would have an easy
way controlling the sound settings of the system from the tray.

Sadly I couldn't find suitable drivers for HyperX Cloud Alpha S for linux, so a lot of its features
(dual speakers for instance) simply don't work. 

## Authentication Agent

Remember when I went over the i3 config I deleted a line with `polkit-agent` because I didn't know
what it was? Apparently when you run as user and need to do something as superuser NOT in a terminal
context but in GUI applications, such as pamac, the app will "ask" the agent for a password and it's the
agent's job to prompt the user for password.

<small> polkit-agent window example </small>
![polkit-agent example](/assets/posts/yearofthelinux/polkit.png)

All I had to do was to install `polkit-gnome` and execute the agent on startup (simple exec in
i3 config).

This was all for one weekend. At this point I felt the system was usable, but still had a lot of
things to fix and tweak.

## Lock screen and Power settings

I noticed that Winkey+L doesn't work on my system (I shouldn't be surprised, it's not Windows).
I installed `betterlockscreen` which lets you pick a photo and then generates a beautiful lock screen.
I added a shortcut to i3 config (Alt+L) to lock the machine and that was it!

Then I played a bit with the power settings in xfce4-power-manager. I chose when to the PC should enter
sleep mode and when to dim the displays. This is another thing I had never had to do inside
a VM because I didn't care until now if my PC draws power or not.

## What's the time?

It's been a week since I installed linux and only know I noticed that the time wasn't even correct.
I fixed it in Manjaro Settings.

## Keyboard layout

I added a Polybar module that shows me the current keyboard layout.

## Mounting my HDD

I decided it was time to mount my HDD (contains games, movies and old backups). `lsblk` and `fdisk -l`
revealed the partition I needed to mount:

![lsblk output](/assets/posts/yearofthelinux/lsblk.png)

I mounted the partition: `mount /dev/sda1 /mnt/storage`, but received an error telling me something
is wrong with my NTFS drive (it mounted but as RO).

<small> Trying to mount the NTFS drive: </small>
![mounting NTFS](/assets/posts/yearofthelinux/ntfsmount.png)

From a quick google search I managed to fix it using `ntfsfix` (which was already installed).

![ntfs fix](/assets/posts/yearofthelinux/ntfsfix.png)

Next I wanted to automatically auto-mount the HDD on startup, but when reading about SSDs in the Arch
wiki I encountered a section about [TRIM](https://wiki.archlinux.org/title/Solid_state_drive#TRIM).
In short, it's an operation you can do in order to improve the health of your SSD (Windows does this
operation once a week by default). In `linux-utils` (was installed on my machine by default) comes
a service called `fstrim.timer` that calls `fstrim.service` every week. The `fstrim.service` TRIMs all
connected SSDs that support TRIM. I enabled the service by running: `systemctl enable,start fstrim.timer`.

### Persistent Mount

First I needed the UUID of the partition I wanted to mount on boot. I ran `sudo blkid /dev/sda1` and
got it. Then I added the following line to `/etc/fstab`:

![fstab line](/assets/posts/yearofthelinux/fstab.png)

I had to specify:

- Device UUID
- Mount point
- Filesystem type
- options

The options include `defaults` (mount as RW, respect SUID bit and more) and `noatime` (don't update
access time). The two numbers at the end tell fstab to not dump the FS on boot and to do a FS check
at boot only after the main (SSD) drive.

## Games

After a week I decided I wanted to make games work on my machine. I installed steam (which has a native
app) and download Assassin's Creed Odyssey to my HDD. I enabled Proton Experimental for all games, but
no matter what I did I couldn't get the to game launch. After some debugging (PROTON_LOG=1 in Steam launch options)
I found out that the game installation folder was not "owned by user". I could just fix it by specifying UID and
GID in the fstab file (see [here](https://github.com/ValveSoftware/Proton/wiki/Using-a-NTFS-disk-with-Linux-and-Windows)),
but I decided to split my drive to two partitions:

- The game was really slow, I suspected it was because NTFS on linux is actually a FUSION FS.
- As I will learn later, some games / game launchers simply don't work when being run from NTFS partitions.

Using Gparted I created one partition for games (ext4 partition) and kept the NTFS partition for movies
and other media files.

### Xbox one controller

Getting the Xbox controller to work was really easy - I just installed `xboxdrv`.

## Themes

At this point I felt quite comfortable with my setup. 
While adding some final touches I deleted [accidentally] `manjaro-i3-settings` (which actually deleted quite a bit
of junk) and it caused the dark theme for some apps (firefox, pcmanfm) to reset. After a bit for
googling, I found out that there are two main "GUI frameworks" in linux: GTK and QT. For example,
Wireshark respects QT themes while firefox respects GTK themes.
But instead of tinkering with configuration files I used `qt5ct` (For QT themes) and `lxapearence`
(for GTK themes) to set my themes (I chose `Orchis-Dark` for GTK themes, `Breeze` for QT).
The result was very pleasing: The dark mode on GTK apps looks WAY better than the default one that ships with Manjaro i3.

## Final thoughts

I actually really enjoyed working with linux and discovering new things about it I didn't know existed.
Some things I take away with me:

- The amount of packages that make up linux is pretty big. Sometimes I forget that some functionality
I take for granted is actually a package I need to install (and that it doesn't come by default in
minimal distros).
- There are a lot of concepts the simply don't "exist" on Windows, or exist but quietly and don't
require the interaction of the user in order to work. It was really high-opening to learn about them:
GUI Frameworks, auto-mounting partitions on boot, TRIM and more.
- There are things in linux you only pay attention to if you run it as host (Sound, games, drivers
for your peripheral devices and more).
- It's a bottomless pit of config files and options - I could keep improving and tweaking my config
forever, but I am quit pleased with how it turned out. I am sure I will learn about the existance of
more config files and hidden "systems" that lurk around waiting to be tinkered with.
- Linux for Desktop is really mature - I expected games and devices to not work well, but I am really
surprised with how things work (even if it's not all of them). I can wholeheartedly recommend you
try linux on your personal computer.
- It is really fun. I really feel like linux brought the fun back to computing. My machine looks and
behaves exactly how I expect it too.

Here are some pictures from my setup. All my config files are synced to [Dotfiles github](https://github.com/0xmatia/dotfiles),
feel free to use them however you please.
Hope you enjoyed!

![Config example 1](/assets/posts/yearofthelinux/pic1.png)

![Config example 2](/assets/posts/yearofthelinux/pic2.png)
