# Raspberry Pi File Bridge Howto

## What this is

I have set up a Raspberry Pi Zero W to serve as a file
bridge to a Babylock embroidery machine. This is a how-to
guide to help others who wish to do something similar.
You'll need to have some familiarity with installing
and configuring Linux to be successful.

## Background

The motivation was to provide a convenient way to get
design files transfered to the embroidery machine. The
usual way is to put the design file(s) onto an SDcard
or USB thumb drive, walk that over to the machine and
plug it in. You can also use a USB cable, but that's
inconvenient if your computer is not right next to
the machine.

It turns out that when you connect a USB cable from
a PC to the machine, the machine presents itself as a
mass-storage device, and you just copy your design file
to it. That's something an RPi Zero W is perfectly
suited for. The Pi mounts the machine and shares it
using Samba; meaning you can access it over the network
and drop files from your PC right into the machine.

## Required Parts

* Raspberry Pi Zero W or Raspberry Pi 3 Model B
    * Older Pi models will work, too, but only the above two have built-in WiFi.
* Power supply for the Pi
* (optional) a case for the Pi
* MicroSD card (8GB or larger)
    * A smaller capacity card can be used if you don't use NOOBS and install the "light" version of Raspbian.
* USB-2 A to B cable (your typical printer cable)
* If using the Pi Zero W, you also need a USB OTG adaptor like this one: https://www.adafruit.com/product/1099
* For initial setup, you'll probably want a keyboard and monitor to connect to the Pi. Once it's set up, the Pi can run "headless" (especially if you enable ssh so you can connect to it remotely).

## Setup

1. Start by installing Raspbian on the MicroSD card. Follow one of the guides here: https://www.raspberrypi.org/documentation/installation/

2. Get the Pi connected to your WiFi; use the excellent documentation at raspberrypi.org and/or google for help.

3. Use the ``raspi-config`` tool to:

    1. Change the password from the default (optional but recommended).
    2. Set the hostname if you want it to be something other than ``rasbperrypi``.
    3. Enable SSH (under ``Interfacing Options``) so you can access the Pi over the network.

4. Install samba:

        sudo apt install samba

5. Place the ``mount_machine`` and ``umount_machine`` scripts found in this repository into ``/home/pi`` on the Pi and make sure they are executable (``chmod 755 /home/pi/*_machine``).

6. Create the directory for the samba share:

        sudo mkdir /mnt/machine

7. Place the ``smb.conf`` file in this repo into ``/etc/samba/smb.conf`` on the Pi. The given smb.conf is the same as the default, except I commented out the printer shares and added the share for the machine. Feel free to change the name of the share from "machine" to something more descriptive. The name of the share is the part between the square brackets.

8. Restart samba:

        sudo systemctl restart smbd

## Try it out

1. Connect the USB A-to-B cable from the Pi to the emboidery machine using the "B" port on the machine, and make sure the machine is turned on.

2. Go to your PC (or Mac) and you should see your Pi appear in the "Network" section of an Explorer window (look for the hostname you set in step 3 above). On MacOS, look in the "Shared" section on the left side of a Finder window.

3. Click the icon for the Pi and it should show your machine share. Click that to connect to the share and you should be able to copy a file into it. Connecting to the share will fail if the embroidery machine is off or the USB cable is not connected.

## How it works

The Babylock embroidery machine, when connected via its USB "B" port,
presents itself as a mass-storage device, without a partition
table, formatted as FAT. This appears on the pi as ``/dev/sda``.

The given ``smb.conf`` uses preexec and postexec hooks to invoke
the ``mount_machine`` and ``umount_machine`` scripts, so that the
machine is only mounted when someone is connected to the share.
``mount_machine`` includes a check to see if the volume is already
mounted, that way there can be muliple samba sessions at the same
time (the unmount will silently fail if there are other active
samba connections).

## Notes

This will also work with Brother embroidery machines, as they share
technology.

The mount script assumes the only USB storage device will be the
machine. Connecting a thumb drive to the Pi, for example, may confuse
things.

Since Pi's don't have built-in ways to power themselves off, we
just leave ours running when we're not using it. They don't use
much electricity anyway. But you could always pull the plug when
you're not using it. Some will argue that it's bad to cut power to
a Linux system without a proper shutdown, to which I say: "that's
true, but meh." For this application, the risk is pretty
small, especially considering the root filesystem is journaled.

You don't have to use a Raspberry Pi for this application, other types
of computers would work. But the Pi's are cheap, compact, and well-supported.
