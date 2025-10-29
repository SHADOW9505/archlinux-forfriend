# archlinux-forfriend

# Notes beforehand

PLEASE READ EVERYTHING, 
I CANNOT STRESS THIS ENOUGH.
PLEASE.

This guide also assumes
that you have already booted into
the live ISO (of course you did)


# PRE-INSTALLATION

To check your internet connection,
ping the archlinux website.

    ping ping.archlinux.org

To stop the ping, or stop any commands
at all in the future, press `Ctrl+C`

To ensure your 
time synchronization works fine, run

    timedatectl


# PARTITIONING AND WORKING WITH DISKS

To list all of the available disks
connected to your system, please run

    lsblk

ENSURE THAT THE DISK YOU ARE
INSTALLING ON IS CORRECT!!
FOR THE SAKE OF THIS GUIDE,
I WILL BE USING SDA.

to start partitioning the desired disk, 
you may either run `fdisk`, 
or `cfdisk`.
For the sake of simplicity, run

    cfdisk /dev/sda