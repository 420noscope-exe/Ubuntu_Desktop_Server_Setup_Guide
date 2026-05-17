## Make install media

Using Ubuntu Desktop instead of Ubuntu server gives you some additional packages and utilities by default.  [Download the iso](https://ubuntu.com/download/desktop)

Use dd to make a bootable drive.  You will need the path to the .iso, and the path to the USB drive. 

```
sudo dd if=/path/to/iso of=/path/to/thumbdrive status=progress bs=16M
```

If you are on windows, use rufus.  If you are on mac, you can use the disk utility to restore the .iso onto the drive.
## Install the OS

- Connect the device to the network via Ethernet.
- Erase the whole drive.
- Select standard installation with minimal applications
- Enable Third party driver and hardware support
- Enable Additional packages for media formats
## Create Local Admin Account

Create a local account called ClientAdmin.

Generate a new password to use, and document it!

The hostname should follow this scheme {organization}{xyz}{number}

Example:  PASAPP01
### Enable OpenSSH

If openssh is not installed/enabled, you can do so with the following commands:

```
sudo apt-get install openssh-server
sudo systemctl enable --now ssh
sudo ufw allow ssh
```
## Install Remote Access Solution

Install remote access solution of your choice.  You will not be able to use it correctly until Xorg and an appropriate desktop environment is installed.
## Switch from Wayland to Xorg

Many remote desktop applications do not support Wayland yet.  Many of the ones that do still do not have the lock screen working correctly.  For a server, it is just easier to use X11/Xorg.

Make sure the system is up to date first.

```
sudo apt-get update
sudo apt-get upgrade
```

Install XFCE and lightdm.

```
sudo apt install xfce4 xfce4-goodies lightdm
```

A dialog will appear asking you to choose the default display manager. Select lightdm and press enter.

Reboot the System

```
sudo reboot
```

There is no need to have two display managers, and we don't need the Wayland session anymore, so uninstall them both and remove any dependencies.

```
sudo apt remove --purge gdm3 ubuntu-session
sudo apt autoremove
```

We should also delete the Wayland session for XFCE.

```
sudo rm /usr/share/wayland-sessions/xfce-wayland.desktop
```

Reboot the system again, and now the only option is for the system to use the XFCE session with Xorg.
## Run Headless

If there is no monitor connected when the session is started, only a black screen will show.

The easiest way to run a Linux machine headless is with an HDMI dummy plug, they are less than $10 and are also the most reliable way to have a headless session.  This creates a virtual monitor with no additional configuration.

You can also install the video dummy package and create a configuration file for it (Debian/Ubuntu):

```
sudo apt-get install xserver-xorg-video-dummy
```

```
sudo nano /usr/share/X11/xorg.conf.d/xorg.conf
```

```
Section "Device"
    Identifier  "Configured Video Device"
    Driver      "dummy"
    VideoRam 1024000
EndSection

Section "Screen"
    Identifier  "Default Screen"
    Monitor     "Configured Monitor"
    Device      "Configured Video Device"
    DefaultDepth 24
    SubSection "Display"
	    Depth 24
	    Modes "1920x1080_60.00"
    EndSubSection
EndSection

Section "Monitor"
    Identifier  "Configured Monitor"
    HorizSync 30-200
    VertRefresh 50-1000
    Modeline "1920x1080_60.00"  173.00  1920 2048 2248 2576  1080 1083 1088 1120 -hsync +vsync
EndSection
```

If the 1080p 60 option does not show up in settings, [Ubuntu checks several places for a .conf file](https://manpages.ubuntu.com/manpages/focal/man5/xorg.conf.5.html).  Try placing this config in `/etc/X11/`, and check to see if a .conf file exists somewhere else.

This adds a 1080p 60 virtual monitor to the system.  You may have to go into display setting and switch to this new option.
## Disable Sleep/Suspend/Hibernate

```
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```
