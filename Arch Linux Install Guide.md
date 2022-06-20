The official installation guide for Arch Linux can be found [here](https://wiki.archlinux.org/title/Installation_guide).

However, I have my own preferences and tweaks that I've picked up. I also seem to run into a new problem every time I
go through the process on a new machine, so hopefully this guide will be a little more detailed and cover any issues
that you may encounter during the process.

## 1. Boot into an installation environment

First, flash a USB drive with an image from [archlinux.org/download](https://archlinux.org/download/).

> **Make sure you use a blank USB drive, or that you've copied all of your data off of it first!**

If you are using Windows or Mac, I would recommend [Balena Etcher](https://www.balena.io/etcher/) for this.

If you are on Linux, you can use the `dd` command, but **be careful to choose the right output device!**  
Please don't overwrite your boot drive, and if you do, don't yell at me.

```sh
# List drives to find your USB drive
sudo fdisk -l

# Directly write image to USB drive
#  - not a partition
sudo dd if=archlinux-20XX.XX.XX-x86_64.iso of=/dev/sdX status=progress
```

While you're waiting for the process to complete, you could dig out your motherboard's manual (or consult the
manufacturer website) to find out what the button is that you need to spam like a madman when the computer
starts to get into the BIOS or UEFI menu - just in case you need to change the boot order to get your machine
to boot from the USB drive.

Once your bootable USB is ready, connect it to the system and boot into it.

## 2. Load your keyboard layout

By default, a US layout is loaded, but you can load a different one if you need to.

```sh
# List available layouts
ls /usr/share/kbd/keymaps/**/*.map.gz

# Load the UK keyboard layout
loadkeys uk
```

## 3. Connect to the internet

Wired connections should automatically work without any extra configuration. If that's what you have, you can skip to
the next step.

If you need to set up Wi-Fi, or if a wired connection isn't working, read on.

### Enable the network interface

Wired interfaces might begin with `eth` or `eno`. Wireless interfaces usually begin with `wlan`.

```sh
# List network interfaces
ip link

# Make sure interface is enabled
# Replace 'wlan0' with the desired interface
ip link set wlan0 up
```

### Connecting to a wireless network

You can connect to a wireless network using `iwd`.

```sh
# Enter iwd shell
iwctl
```

Then, in the `iwd` shell:

```sh
# Replace 'wlan0' with the desired interface
station wlan0 scan
station wlan0 get-networks

# Replace 'ssid' with the wireless network name
station wlan0 connect ssid

quit
```

## 4. Refresh the package database

```
pacman -Syy
```

Yup, that's it. Done. If this fails, go back to [step 3](#3-connect-to-the-internet).

## 5. Adjust time settings

> **Hint:** you can start typing and press `tab` to auto-complete things like paths and commands.
> You can use this when setting the timezone.

```sh
# List all timezones (there's a lot, press 'q' to exit)
timedatectl list-timezones

# List timezones and filter to those beginning with 'Europe/'
timedatectl list-timezones | grep Europe/

# Set the timezone
timedatectl set-timezone Europe/London

# Enable network time protocol
timedatectl set-ntp true

# Sanity check
timedatectl status
```

## 6. Set up the file system

### Identify the drive(s)

Your drive is most likely `/dev/sdX` where `X` is a letter,  
or `/dev/nvmeXnY` where `X` and `Y` are numbers.

```sh
# List all disk devices and partitions
fdisk -l
```

### (Optional) Zero-fill

If you have a brand-spanking-new drive or if you just want to wipe the thing clean, it's a good idea to write zeros
across the whole thing.

> **Warning: This will overwrite the entire drive!**  
> Make sure you've typed the right drive name.  
> This process will take a while.

```sh
# Zero-fill the drive
dd if=/dev/zero of=/dev/sdX bs=8M status=progress
```

### Partitioning

```sh
# Enter fdisk shell to work on the given drive
fdisk /dev/sdX
```

In the `fdisk` shell:

```sh
# Show the partition table
p

# Quit without saving changes
# Remember this in case you mess up
q
```

#### Partition table

> **Warning: The following will wipe any existing partition table!**  
> Pay close attention if you already have an OS installed that you are planning
> to dual boot, or if you have another partition that you need to keep intact!

If the disk is empty, give it a GPT partition table.

```sh
# Create a GPT partition table
g
```

#### EFI partition

If you already have an OS installed that you're planning on dual-booting, you probably already have an EFI partition,
perhaps with something like Windows Boot Manager on it. It would have been listed when you showed the partition table
earlier.

In this case, skip this bit. We'll use the existing EFI partition.

Otherwise:

```sh
# Create a partition
# When it asks for the size, enter: +500M
n 

# Set the type to EFI
# When it asks for the type, enter: 1
t
```

#### Boot partition

If we want to encrypt our file system, we're going to create a special unencrypted partition just for `/boot`.

This is where the GRUB bootloader and our Linux boot files will live. While you can set it up so that you can decrypt
the drive prior to running GRUB, that setup is painfully slow, and it's not worth it.

```sh
# Create a partition
# When it asks for the size, enter: +500M
n

# Type should automatically be set to Linux filesystem
```

#### File system partition(s)

We can create any number of partitions across multiple drives if we want to. The important thing is that we
have, at minimum, one partition for our root filesystem. Additional partitions could even be made for directories
like `/home`. This might be useful if you want the Linux root on one drive, and all of your user files on another,
maybe for portability.

##### Swap

It's worth considering making a swap space too. This is a space on the disk that is reserved kind of like spare
memory, and can be used if memory overflows to prevent crashes. The system will absolutely crawl to a halt if this
happens, but it might be better than losing data.

Most sources you'll find online suggest allocating a swap space of half your system's RAM, though this rule-of-thumb
originates from times when RAM wasn't as abundant as it is today. It's common now to have systems with 32GB RAM, or
even more, and you probably don't need or want to allocate 16GB for swap.

Personally, I'd suggest that you can probably get away with 2-4GB of swap space. Maybe 8GB to be on the safe side. It's
up to you.

##### Encryption & LVM

If you need multiple partitions on a single drive, say a root partition and a swap, and you want to use encryption
(you do), then it's a good idea to create one LVM partition and create logical volumes within it so that everything
gets decrypted at the same time.

##### I'm confused?

If all of this has gone over your head, I recommend just creating a single partition with the default options, which
should fill up the rest of the drive, then set its type to LVM.

We'll encrypt it and create two volumes inside: the root filesystem and a small swap space.

We'll end up with something that looks like this:

```
sda
 |-- sda1 (EFI)
 |-- sda2 (/boot)
 |-- sda3 (LUKS encrypted LVM)
      |-- vg0/root
      |-- vg0/swap
```

Create partition(s):

```sh
# Create a partition
n

# Set the type
# 20 for Linux filesystem
# 30 for Linux LVM
t
```

Once you're done modifying the partition table:

```sh
# Print the partition table to double check everything
p

# Write changes to the disk and close
w

# OR quit without altering anything (in case you messed up)
q
```

### Set up encryption

If you want to set up encryption, you'll need to do it before formatting.

> **Warning: Don't try encrypting an existing file system. You will end up losing your data.**

> **Warning: Zero-filling will overwrite the entire drive or partition! Make sure you type the command & partition name
> correctly!**

```sh
# List drives and partition tables
# Identify the partition that you want to be encrypted e.g. /dev/sda3
fdisk -l

# Zero fill the partition (may take a while)
# (Skip this if you already zero filled the drive earlier)
dd if=/dev/zero of=/dev/sdXX bs=8M status=progress

# Encrypt the partition
cryptsetup luksFormat /dev/sdXX

# Open the newly encrypted partition
# Replace 'luks-main' with any unique name you like
cryptsetup luksOpen /dev/sdXX luks-main
```

Each decrypted volume will be found in `/dev/mapper/[name]`.

### Set up LVM

In this example, we'll set up the LVM partition that we just encrypted. We'll allocate a small swap space and leave the
rest of the partition for our root file system.

```sh
# Create physical volume
# Can pass a decrypted partition from /dev/mapper/
# or a non-encrypted partition from /dev/ 
pvcreate /dev/mapper/luks-main

# Create volume group
# Replace vg0 with any name you like
vgcreate vg0 /dev/mapper/luks-main

# Create logical volumes
# Use 4GB, call it swap
lvcreate -L 4GB vg0 -n swap
# Use the rest of free space, call it root
lvcreate -l 100%FREE vg0 -n root
```

Logical volumes will be found at `/dev/vg0/root` etc.

### Create file systems

Here we format our partitions and/or logical volumes with new file systems.

```sh
# Format the EFI partition (only if we had to make a new one)
mkfs.fat -F32 /dev/sdXX

# Format the /boot partition and root LVM volume
# with ext4 file systems
mkfs.ext4 /dev/sdXX
mkfs.ext4 /dev/vg0/root

# Format the swap space
mkswap /dev/vg0/swap
```

## 7. Mount file systems

Now we will mount everything we just made so that we can begin installing files and working on the system.

In this example, we'll mount the root LVM volume, the `/boot` partition, and enable the swap space. If you have other
partitions, you should mount those too.

```sh
# Mount root file system
mount /dev/vg0/root /mnt

# Mount /boot partition
mkdir /mnt/boot
mount /dev/sda2 /mnt/boot
# Repeat this for /mnt/home etc. if you made separate partitions for them

# Enable swap volume
swapon /dev/vg0/swap
```

## 8. Install packages

`pacstrap` will install the Linux kernel, and any other packages we tell it to, onto the new file system.

Be sure to read this entire section to ensure your system gets all the packages it needs. Might as well install everything now in one go so that you can make a cup of tea while your 64K modem works its [magic](https://youtu.be/aV8DEJ8ydJQ?t=4).

```sh
# Install the essentials
pacstrap /mnt base linux linux-firmware base-devel grub efibootmgr sudo
```

### EFI

If you're not using EFI you don't need `efibootmgr`.

### LTS kernel

Substitute `linux` for `linux-lts` if you want the LTS kernel.

### Microcode

Include `intel-ucode` if you have an Intel CPU, or `amd-ucode` if you have an AMD CPU.

### LVM

If you're using LVM, include `lvm2`.

### Networking

Your new system won't have any networking configured whatsoever, so be sure to install packages for networking now, while you can. Just remember that enabling multiple networking services can cause conflicts!

Most people should use `networkmanager`, especially if you will be using a graphical desktop environment and multiple WiFi networks. `network-manager-applet` will install a system tray applet for managing connections.

If your system uses a wired connection only, or if you're willing to spend some time configuring, you can use `dhcpcd` instead for a minimal set up.

### Text editor

You'll probably need something to edit configuration files in the terminal. You can use `vim`, `nano`, or any editor of your choosing. If you're unsure of which one to pick, go for `nano`.

### Desktop Environment

I've only really used XFCE desktop environment, so I recommend that. It's the most minimal desktop environment that I know of. include `xorg-server`, `xfce4` and `xfce4-goodies`.

### Other goodies

- `openssh` - If you plan on using SSH.
- `os-prober` - Will help GRUB find other OS you have installed to display as an option on the GRUB boot menu.

## 9. Generate & tweak `fstab`

The `fstab` file gives Linux instructions for mounting drives automatically.

### Generate

If you have everything mounted, you can automatically generate the `fstab` file:

```sh
# Generate the file based on mounted devices
genfstab -U /mnt >> /mnt/etc/fstab

# Open the file to check/tweak stuff
nano /mnt/etc/fstab
```

### Tweak: `relatime` or `noatime`

To prolong SSD life and improve performance, you may want to add a `relatime` or `noatime` mount option to intelligently limit or fully prevent updating the access time every time any file is accessed. Your `fstab` was likely generated with `relatime` enabled.

```
# /dev/mapper/vg0-root
UUID=12345678-1234-1234-1234-123456789abc  /  ext4  rw,relatime  0 1
```

### Tweak: `tmpfs`

If you have a good amount of RAM available, consider creating a `tmpfs` for `/tmp`. This will store temporary files in
memory rather than on the disk. It will dynamically resize as it needs to, only using the space it needs. You can limit
it to any size, but here I've given it 8GB.

```
# tmpfs
tmpfs  /tmp  tmpfs  rw,nosuid,noatime,nodev,size=8G,mode=1777  0 0
```

### Removable drives

Add the `noauto` mount option to prevent automatically mounting the volume. This should reduce errors if a removable
drive isn't present.

Of course, you could just not include them in the file, and instead mount them manually after boot, either in console or
through a graphical file manager.

```
# /dev/sdXX
UUID=12345678-1234-1234-1234-123456789abc  /  ext4  rw,relatime,noauto  0 2
```

## 10. Configure Arch Linux

```sh
# Enter shell as root of the newly installed system
arch-chroot /mnt

# Set the root user's password
passwd
```

### Hostname

This is the name that your computer will have on the network.

```sh
# Set hostname
echo "my-pc" >> /etc/hostname
```

### Hosts file

```sh
nano /etc/hosts
```
Add the following, substituting `my-pc` for the hostname you chose:

```
127.0.0.1   localhost
::1         localhost
127.0.1.1   my-pc.localdomain  my-pc
```

### Network

Now configure any networking packages you installed earlier.  
*(See: [Install packages/Networking](#networking))*

#### Network Manager

```sh
systemctl enable NetworkManager
```

#### DHCPCD

```sh
# List network interfaces
ip link

# Enable DHCPCD service for the network interface
systemctl enable dhcpcd@eno1.service
```

If you want to set up a static IP address:

```sh
# Edit config
nano /etc/dhcpcd.conf
```

Then add the following but with your own IP addresses:

```
# Static IP
interface eno1
static ip_address=192.168.10.2/24
static routers=192.168.10.1
static domain_name_servers=192.168.10.1

# Disable arp probing
noarp
```

### mkinitcpio

This is the configuration that is used to make the script that actually initialises Linux from our bootloader.
We may need to add extra hooks and modules depending on our setup.

```sh
# Edit config
nano /etc/mkinitcpio.conf
```

Add hooks:

- `encrypt` if using LUKS encryption.
- `lvm2` if using LVM

In my case, I use both:

```
...
HOOKS=(base udev autodetect modconf block encrypt lvm2 filesystems keyboard fsck)
...
```

If you have an Intel SSD, you may need to add `vmd` to modules:

```
...
MODULES=(vmd)
...
```

Save the file, then:

```sh
# Generate init script
# Substitute 'linux' for 'linux-lts' if you chose the LTS kernel
mkinitcpio -p linux
```

### Adjust time settings

```sh
# Set the timezone (again)
ln -sf /usr/share/zoneinfo/Europe/London /etc/localtime

# Enable network time protocol (again)
timedatectl set-ntp true

# Generate /etc/adjtime
hwclock --systohc
```

### Generate locale

Open `/etc/locale.gen` and uncomment your locale. In my case, I uncomment `en_GB.UTF-8`.

Once you've done that, generate the locale:

```sh
locale-gen
```

You also need to set the locale in `/etc/locale.conf`:

```sh
echo "LANG=en_GB.UTF-8" >> /etc/locale.conf
```

Also set the keyboard layout in `etc/vconsole.conf`:

```sh
echo "KEYMAP=uk" >> /etc/vconsole.conf
```

## 11. Create a sudo user

It's best to never use the root user after the installation process. Instead, it is recommended to create a new user to
be an admin and give them superuser privileges.

This command will create a new user, create a home directory for them in `/home`, create a group with the same name to
be their main group, and also add them to the 'wheel' group, which will grant them superuser privileges. Substitute
'username' for any username you like.

```sh
# Create the user
useradd -m -U -G wheel username

# Set their password
passwd username
```

Now, to enable superuser privileges for the 'wheel' group:

```sh
# Run visudo using Nano for text editing
EDITOR=nano visudo
```

```
...
## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL) ALL
...
```

## 12. Configure GRUB

```sh
# Mount EFI partition
mkdir /boot/EFI
mount /dev/sda1 /boot/EFI

# Edit GRUB config file
nano /etc/default/grub
```

Add encrypted devices to `GRUB_CMDLINE_LINUX_DEFAULT` and uncomment or add `GRUB_ENABLE_CRYPTODISK=Y`:

```
...
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 cryptdevice=/dev/sda3:cryptroot:allow-discards quiet"
...
# Uncomment to enable booting from LUKS encrypted devices
GRUB_ENABLE_CRYPTODISK=y
...
```

If you installed `os-prober`, also uncomment or add:

```
GRUB_DISABLE_OS_PROBER=false
```

Now install the GRUB bootloader. The `--bootloader-id` argument is the name of the bootloader. You could name it
something like GRUB or Arch. It's up to you.

```sh
# Install GRUB
grub-install --target=x86_64-efi --bootloader-id=GRUB

# Generate config file
grub-mkconfig -o /boot/grub/grub.cfg
```

## 13. Restart

You're now ready to exit chroot and restart. You may need to fiddle with boot order in the BIOS/UEFI settings.

```sh
# Exit chroot environment
exit

# Reboot
reboot

# or shutdown
shutdown now
```

## More Config

Here is some additional configuration you can perform on your Arch system once you've rebooted (or before you exit `arch-chroot`.)

### Install an AUR helper

You may wish to install software that isn't available via `pacman`, but that can be found on the [Arch User Repository (AUR)](https://aur.archlinux.org/packages). These packages are a little more complicated to set up, but you can install an AUR helper to automate the process.

`yay` doubles as a wrapper for `pacman`, so you can use it in place of `pacman` to manage both your Arch and AUR packages at the same time.

To install `yay`, first make sure you are logged in as the sudo user you [set up earlier](#11-create-a-sudo-user), then:

```sh
# Install base-devel & Git, if you haven't already
sudo pacman -S base-devel git

# Clone the yay repository
git clone https://aur.archlinux.org/yay.git

# Enter the yay cloned repo
cd yay

# Finally, build and install yay
makepkg -si
```

This is actually the process for other AUR packages, but it's a pain doing this manually every time. Besides, now you can use `yay` to update everything!

```sh
# Installs an AUR or pacman package
yay -S [package]

# Updates all AUR and pacman packages
yay -Syu

# Remove an AUR or pacman package and any no longer needed dependencies
yay -Rs [package]
```

### Set a console screen timeout

```sh
# Screen turns off after 1 minute of inactivity in console
setterm -blank 1
```

### Change laptop lid close behaviour

```sh
# Edit systemd-logind settings
sudo nano /etc/systemd/logind.conf
```

Uncomment and and edit `HandleLidSwitch` and `HandleLidSwitchExternalPower`.

"The specified action for each event can be one of `ignore`, `poweroff`, `reboot`, `halt`, `suspend`, `hibernate`, `hybrid-sleep`, `suspend-then-hibernate`, `lock` or `kexec`."

Default is `suspend`, but for example you could set it to `lock` to simply turn off the screen:

```
...
HandleLidSwitch=lock
HandleLidSwitchExternalPower=lock
...
```

### Set a console auto-logout timeout

```sh
# Create a file
sudo nano /etc/profile.d/autologout.sh
```

Set the `TMOUT` variable to the number of seconds before an auto-logout.  
e.g. 5 minutes:

```
TMOUT=300
readonly TMOUT
export TMOUT
```

```sh
# Set execute permission
sudo chmod +x /etc/profile.d/autologout.sh
```

### Enable `pacman` `VerbosePkgLists`

Shows more information about package upgrades, including old and new versions and sizes.

1. Edit `/etc/pacman.conf`
2. Uncomment `VerbosePkgLists`
