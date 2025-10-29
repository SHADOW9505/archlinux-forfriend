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

Inside the GUI, DELETE ALL of the partitions inside, by selecting the partition and pressing delete. 
Once you have deleted everything, you may create your first and most important partition.
Press “new” (or “create”, I forgot)
Type in 2G (for 2 gigabytes of your EFI/Boot)
Once you have created that, press “type” and scroll all the way up and choose “EFI system” (or whatever, it will say EFI)

For the rest of your storage, just press “new” again and press enter this time, assuming you want ONE main partition only. 
You do not have to change the filesystem for this. 

Press WRITE, type “yes”, and press exit.


Now, confirm that you are happy with your partitions by running 

    lsblk

(sda1 should be 2G, and sda2 should be around 900+ or something)
You may start formatting your partitions. 

TO FORMAT EFI:

    mkfs.vfat -F32 /dev/sda1

FORMAT ROOT:

    mkfs.ext4 /dev/sda2

now, mount your ROOT partition FIRST (RULE OF THUMB)

    mount /dev/sda2 /mnt

make a directory AND mount your boot at the same time:

    mount —mkdir /dev/sda1 /mnt/boot


INSTALLATION 1

to explain what happened right now, is that you essentially prepared your drives for the installation of linux itself. Everything before now was just the hardware part, now we’re getting into installing Linux itself. 
REMEMBER: never shut down your system at this point, since your drives are mounted. Whenever your drives are mounted, they are in use. If you force shutdown when they are mounted you WILL corrupt the drives (personal experience). ALWAYS umount before doing something with the drives such as servicing them or shutting down. In the actual complete system itself, the drives will umount automatically, whenever you run “shutdown” or “reboot”. It is heavily advised against to force shutdown.

I advise you to change your pacman mirrorlist, as the automatically generated ones are DOGSHIT.

Firstly install nano (text editor)

    pacstrap -K /mnt nano

Then once it’s done run this command:

    nano /etc/pacman.d/mirrorlist

In the file, DELETE EVERYTHING, YES EVERYTHING. And inside the empty file, put 

    Server = https://in-mirror.garudalinux.org/archlinux/$repo/os/$arch

I made you use only mirror, as it is Garuda linux and it can be trusted. 

Once you have done that, press 

CTRL+O 
ENTER (to save changes)

CTRL+X (to exit)


Now you can install the essential files needed for your bare-bones system

    pacstrap -K /mnt linux linux-firmware linux-headers base base-devel grub nano vim networkmanager iwd efibootmgr intel-ucode os-prober bash-completion pwgen

After that finishes, run

    genfstab -U /mnt > /mnt/etc/fstab


INSTALLATION 2

Now that you have installed your bare bones operating system, you will run it in the most super user mode ever. Although it is also ran in “safe mode”, aka programs and stuff do NOT work in chroot, apart from text editors. 

    arch-chroot /mnt


Set up time:

    ln -sf /usr/share/zoneinfo/Asia/Bengaluru /etc/localtime 

DISCLAIMER: I am unsure whether it is Bengaluru or Bangalore, to check which one it is, run # ls /usr/share/zoneinfo and find the correct spelling.

    hwclock —systohc


Set locale (important)

    nano /etc/locale.gen

FIND “eng_US.UTF-8” and UNCOMMENT IT by deleting “# “

    locale-gen

    nano /etc/locale.conf

INSIDE THE FILE PUT:

    LANG=en_US.UTF-8


hostname, users and passwords

set your hostname, its basically the name of your machine (such as DESKTOP-93746 on windows)

    echo YourHostNameOfChoice > /etc/hostname


MOVING ON TO PASSWORDS: I installed pwgen along with your pacstrap. I highly highly highly suggest you run it right now. You MUST use a good and strong password for the ROOT account, which you will not be using at all (hopefully). Try not to use the root account at ALL. Which is why we have “sudo”. 
This is how I ran pwgen:

    pwgen 24 -y -s 

(y=special characters 
s=secure/pure randomness)

After you have picked your password, run 

    passwd 

This will set your ROOT ACCOUNT PASSWORD. 
Note: you cannot copy and paste in the live ISO.


After you have set up root password, move on to your user account.

    useradd -d -m -G wheel -s /bin/bash Arnav214

set a password (it can be any password you like, choose something simple but normal, you’ll be typing your password a LOT)

    passwd Arnav214


The next few steps are a bit tricky.

    EDITOR=nano visudo
VERY IMPORTANT: uncomment (remove # ) %wheel ALL=(ALL:ALL) ALL 


The next steps are going to be regarding installing and configuring software, boot loader, and drivers.

Configure for the future

n    ano /etc/environment

Add these lines somewhere in the file:

QT_QPA_PLATFORMTHEME="wayland;xcb"
GBM_BACKEND=nvidia-drm
__GLX_VENDOR_LIBRARY_NAME=nvidia
ENABLE_VKBASALT=1
LIBVA_DRIVER_NAME=nvidia

Next:

    nano /etc/mkinitcpio.conf

you will see MODULES=(). Inside the brackets, insert 

    nvidia nvidia_uvm nvidia_drm nvidia_modeset

    mkinitcpio -P


install and configure bootloader (GRUB)

    grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB --recheck

    grub-mkconfig -o /boot/grub/grub.cfg

    nano /etc/default/grub

In this file, you want to find the GRUB_CMDLINE_LINUX_DEFAULT=“”

If there is something already inside the “”, keep it, and add 
nvidia_drm.modeset=1

    grub-mkconfig -o /boot/grub/grub.cfg


Enable the installation of multilib packages (such as steam)

    nano /etc/pacman.conf

scroll down until you find 

    [multilib]
    Include = /etc/pacman.d/mirrorlist

UNCOMMENT THOSE TWO LINES BY REMOVING # BEFORE BOTH OF THE LINES.

then run

    pacman -Syu


INSTALLATION 3

We have now finished installing and configuring system settings. Now you can safely start installing and enabling packages. Here, I will list all of the packages that you will want (you yourself wanted all kde-applications and the full plasma experience)


    pacman -S plasma kde-applications sddm Firefox discord steam libreoffice-fresh nvidia nvidia-settings nvidia-utils networkmanager iwd git noto-fonts-cjk vlc cups gimp obs jdk-openjdk vlc-plugin-ffmpeg 

now you will be met with a shit ton of choices to install. (I don’t know what order it is in, but I do know you MUST press enter for the first 2)

1. press enter
2. press enter
3. press 2 for pipewire jack
4. press 1 for ffmpeg
5. press 1 for pyside6
6. press 30 for the eng option
7. press 1 for lib32nvidia-utils (or something)


after everything installs, run these two commands:

    systemctl enable sddm

    systemctl enable NetworkManager


FINALIZATION

    exit
    umount -R /mnt
    reboot

congratulations, you have installed arch! now you will reboot into your main system (hopefully everything works and the linux god will bless you
