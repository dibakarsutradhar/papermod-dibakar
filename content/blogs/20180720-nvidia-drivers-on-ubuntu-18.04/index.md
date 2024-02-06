---
title: How I Fixed My Display Resolution by Installing NVIDIA Drivers on Ubuntu 18.04 Bionic Beaver Linux
slug: how-i-fixed-my-display-resolution-by-installing-nvidia-drivers-on-ubuntu-18.04-bionic-beaver-linux
date: 2018-07-20T11:22:46+08:00
description: A step-by-step look at troubleshooting low-resolution display
draft: false
tags:
  - linux
  - nvidia
ShowToc: false
cover:
  image:
---

Last night I installed _Ubuntu 18.04 Bionic Beaver Linux_ on my desktop PC by completely whipping out Windows 7 OS.

So after proper Linux installation, the first problem I faced was a low-resolution display, which was totally annoying me given that I have a **NVIDIA Geforce GTX 750 TI** graphics card installed in the machine. My monitor’s resolution is _1368x768_, but Ubuntu defaulted it to 960x540. So I searched over Google and found some solutions and started applying all until I got my result. Now I’m writing all the methods that worked for me to fix this problem.

I assume that you’ve updated the system after OS installation. If not, then write this command in the terminal. Use `CTRL + ALT + T` to open Ubuntu Terminal.

```
$ sudo apt update && sudo apt upgrade
```

First we need to detect the model of NVIDIA graphic card and the recommended driver. To detect itk, execute this:

```
$ ubuntu-drivers devices
```

The result will be like this:

```
== /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0 ==
modalias : pci:v000010DEd00001380sv00001043sd000084BEbc03sc00i00
vendor : NVIDIA Corporation
model : GM107 [GeForce GTX 750 Ti]
manual_install: True
driver : nvidia-340 - distro non-free
driver : nvidia-driver-390 - distro non-free recommended
driver : xserver-xorg-video-nouveau - distro free builtin
```

So the above output says that I have NVIDIA Geforce GTX 750 Ti graphic card installed, and the recommended drive to install is `nvidia-390`.

Now run this command in the Ubuntu Terminal to install the NVIDIA driver automatically:

```
$ sudo ubuntu-drivers autoinstall
```

Or alternatively install the desired driver selectively:

```bash
$ sudo apt install nvidia-390
```

After completing the installation, reboot your system.

> _The above method didn’t work for me. But I put it here thinking that it may work for someone else since it’s the easiest way out there. But I did everything manually. All the steps are mentioned below._

---

## Manual Installation Steps

First, let’s download the desired NVIDIA Driver from their [official website](http://www.nvidia.com/Download/index.aspx). I already know my desired driver, **NVIDIA-390**. Save the file into your home directory or downloads directory. Mine was in the downloads directory. To move to downloads directory from home directory, just use this command in your terminal:

```bash
$ cd Downloads/
```

Then type ls command in terminal:

```bash
$ ls
```

And you’ll see your NVIDIA driver in the terminal.
![Nvidia driver in Download director](images/Display%20Resolution%20Fix.webp)
_Nvidia driver in Download directory_

Now let’s install the driver. To do so, write these commands and execute the following:

```bash
$ sudo dpkg --add-architecture i386
$ sudo apt update
$ sudo apt install build-essential libc6:i386
```

These commands will require your system password.

Cool! We’re almost halfway done. Now you need to disable your Nouveau kernel driver. In the terminal, enter the following commands:

```bash
$ sudo bash -c "echo blacklist nouveau > /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
$ sudo bash -c "echo options nouveau modeset=0 >> /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
```

After executing, now let’s confirm the content of new modprobe config file…

```bash
$ cat /etc/modprobe.d/blacklist-nvidia-nouveau.conf
```

and it will return this result:

```
blacklist nouveau
options nouveau modeset=0
```

![Result of the above command](images/Result%20of%20the%20above%20command.webp)
_Result of the above command_

Then you need to update the kernel initramfs.

```bash
$ sudo update-initramfs -u
```

Once done, reboot the system.

```bash
$ sudo reboot
```

After rebooting, I panicked. Don’t panic because it’s normal. Let me explain what happened.

We just disabled the Nouveau kernel display server, so currently there is no UI to work with. But now we’ll stop this current display server so that we can install the NVIDIA driver.

Just try hitting `CTRL + ALT + F1` to `F12`. Mine worked at `F2`. You’ll see a login panel; enter your username and password to open a new TTY1 session. After login, enter this command to proceed:

```bash
$ sudo telinit 3
```

Cool! We’re almost done. Now it’s time to install the NVIDIA Driver. To start the installation, first navigate to the Downloads directory or wherever your NVIDIA driver is currently. Remember you’ll need root access to install this driver. First execute this command, which will give you root access:

```bash
$ sudo su
```

Now execute this:

```bash
$ bash NVIDIA-Linux-x86_64-390.77.run
```

Now you need to follow these steps:

```
1. Accept License
2. The distribution-provide pre-install script failed! Are you sure you want to continue? -> CONTINUE INSTALLATION
3. Install all NVIDIA's 32-bit compatibility libraries? -> YES
4. Would you like to run the nvidia-xconfig utility? -> YES
```

Bingo! The NVIDIA driver is now installed. Reboot your system. After rebooting you should be able to start the NVIDIA X Server Settings app from the Activities menu.

![NVIDIA X Server Settings App](images/NVIDIA%20X%20Server%20Settings%20App.webp)
_NVIDIA X Server Settings App_

From that app you’ll be able to change your display resolution.

I hope this fixed your problem too.
