---
layout: post
title: Configuring Raspberry PI Zero + LCD Display + Tmux
category: Raspberry-Pi, Linux, Tmux
---

I needed a display for a new project that I am working on and saw that the 3.5 RPI Display Board was on sale and decided to pick one up.  I've previously used mini OLED displays before, but they're pretty limited by its size and the colors that it can display. This is a 480x320 resolution device that is designed to affix right onto the Raspberry Pi (RPi) GPIO pins. The installation is simple as you'd imagine:

![Connection](/images/posts/RpiLcdTmux/10_final_form.jpg)

![Connection](/images/posts/RpiLcdTmux/20_connection.jpg)

For the project I had in mind, I do not need a fancy GUI nor the use of the touch controller. The display will be used to show console statistics and accessing the device using SSH. 

<!--excerpt-->

## Install and Configure LCD Show

I am using a vanilla Raspbian lite and no additional drivers were required to get this working. All we need to do is configure some boot scripts and introduce some new configuration files. It's possible to do this [manually](https://www.raspberrypi.org/forums/viewtopic.php?t=143581), but thankfully [LCD-Show](https://github.com/goodtft/LCD-show) automates the process for us.

Installation of LCD-Show and switching between displays is pretty straight forward:

1. Clone the repository. `git clone https://github.com/goodtft/LCD-show.git`
1. Give the new folder execute permission. `chmod -R 755 LCD-show`
1. For my version of the display I need to use the LCD 35 version of the file. Run the `LCD35-show` script.
1. Reboot the Pi using `sudo shutdown -r now` or `sudo reboot`.

Upon reboot the LCD panel should be your primary display. 

It would have been nice if I could have mirrored the HDMI output and the LCD panel at the same time, but I could not figure out how to do this or if it was possible at all. 

In order to flip back to HDMI as the primary, execute the `LCD-hdmi` script and reboot to take effect.

![Connection](/images/posts/RpiLcdTmux/30_panel.jpg)

## Automatic Login

In order to show console of my program on screen we need to login as a user and I want this to happen  automatically at boot time. 

I followed the instructions in [this post](https://unix.stackexchange.com/a/401798) in order to configure auto login on boot:

1. Edit the `/etc/systemd/logind.conf` file and uncomment the `#NAutoVTs=6` line to `NAutoVTs=1`
1. Create a new file at `/etc/systemd/system/getty@tty1.service.d/override.conf`.
1. Add the following content:

        [Service]
        ExecStart=
        ExecStart=-/sbin/agetty --autologin appsvcuser --noclear %I 38400 linux`

Replace the `autologin` parameter value with the user you would like to login. In my scenario there is no possibility to physically access and compromise the device, even then I would recommend to use an account with the least amount of privileged to accomplish your tasks.

1. Start the `getty@tty1.service` service using `systemctl enable getty@tty1.service` command.
1. Reboot using `sudo shutdown -r now` or `sudo reboot`.

## Configure Tmux

I initially attempted to configure it using [this approach](https://kerpanic.wordpress.com/2017/03/30/loading-tmux-on-boot-in-linux/), but quickly realised that it would not work for my requirement as there is no user logged into the console. If there's no one logged in then there's shown on screen.

My solution is to use terminal multiplexer like [Tmux](https://medium.com/@tholex/what-is-tmux-and-why-would-you-want-it-for-frontend-development-e43e8f370ef2). Among its many useful features, there are a couple of that I am interested in. The first being the fact that a SSH session initiated via Tmux does not terminate upon user disconnection and all your processes continue to run in the background. 

Secondly, it allows a remote user to connect to an existing SSH session. If I were to kick off a Tmux session on user log on, I am be able to connect to the same session from a remote computer. This means that I should connect to the session that is being shown on the physical display and interact with it as if I was seated in front of it.

I updated the`~/.bashrc` of the `appsvcuser` in order to [launch Tmux on logon](https://wiki.archlinux.org/index.php/tmux#Autostart_tmux_with_default_tmux_layout). I also used the technique described on [this post](https://superuser.com/a/863940)  to make sure that only one instance of Tmux always running at a given time.

1. Login as the execution account - `appsvcuser` in my example.
1. `sudo nano ~/.bashrc`
1. Scroll all the way to the bottom of the file and paste the following block of text:

        if [[ `tty` == "/dev/tty1" ]] && [[ -z "$TMUX" ]];then 
                tmux new -s "nfx"                              
        fi                                                     

Connecting to the Tmux session:

1. Remote into the RPi via SSH.
1. Use the `tmux ls` command to get a list of all the sessions running in the background.
1. The connect to the running session `tmux a -t nfx`.

![Connection](/images/posts/RpiLcdTmux/40_ssh_connectivity.png)


## Final Thoughts

Right now the display is on all the time, I would need to figure out a way to timeout or turn it off manually.

The LCD is compatible with both the Raspberry PI Zero and its big brother variants so these same instructions can be applied to get them both running.

## References

- [GitHub - goodtft/LCD-show: 2.4" 2.8"3.2" 3.5" 5.0" 7.0" TFT LCD driver for the Raspberry PI 3B+/A/A+/B/B+/PI2/ PI3/ZERO/ZERO W](https://github.com/goodtft/LCD-show)
- [Automatically Login on Debian 9.2.1 Command Line - Unix & Linux Stack Exchange](https://unix.stackexchange.com/a/401798)
- [Loading tmux on Boot in Linux – Kernel Panic](https://kerpanic.wordpress.com/2017/03/30/loading-tmux-on-boot-in-linux/)
- [tmux - ArchWiki](https://wiki.archlinux.org/index.php/tmux#Autostart_tmux_with_default_tmux_layout)
- [getty - ArchWiki](https://wiki.archlinux.org/index.php/getty)
- [tmux cheat sheet](https://gist.githubusercontent.com/afair/3489752/raw/e7106ac93c8f9602d3843696692a87cfb43c2d21/tmux.cheat)
- [XPT2046 Touch Screen instructions for Raspbery Pi 3 - Raspberry Pi Forums](https://www.raspberrypi.org/forums/viewtopic.php?t=143581)
- [Calibirating the touch screen](https://www.jeffgeerling.com/blog/2016/review-elecrow-hdmi-5-800x480-tft-display-xpt2046-touch-controller)