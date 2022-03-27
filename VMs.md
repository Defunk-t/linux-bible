This page will discuss management of VMs over CLI. If you have a GUI based system then use `virt-manager`. As long as you are familiar with virtual machine concepts, you should find it straightforward.

# Set up your host environment

You need somewhere to store VM data such as disk images. My preference is to create folders in `/var` as root and give it public, sticky bit permissions.

```sh
# Make directories
sudo mkdir /var/vm /var/vm/images /var/vm/disks

# Read/write with sticky bit
sudo chmod 1777 /var/vm /var/vm/images /var/vm/disks
```

The sticky bit prevents users from deleting or renaming each other's files. The permissions of each individual file can limit read/write access.

In `images`, we can store images for operating systems and driver install disks, etc.

In `disks`, we can store our VM hard disks.

# Install QEMU

On Arch Linux, install `qemu-headless`. This allows emulation of `x86_64` systems and doesn't require a GUI.

```sh
sudo pacman -Sy qemu-headless
```

Make sure virtualization features are actually enabled for your machine in the BIOS/UEFI settings.

# VM Management

## Create a disk image

First, decide which kind of disk you'd like for your virtual machine.

- `raw`: exactly as it sounds, this kind of disk image file is just raw data with a fixed size.

- `qcow2`: a disk image file of this format only grows in size as the guest writes data to it. The guest will see it as the full size, but in reality it only uses a fraction of the space on the host's machine. There is a penalty to performance, however.

Command:

```sh
qemu-img create -f <raw|qcow2> <file> <size>
```

Example:

```sh
qemu-img create -f raw /var/vm/disks/test-vm.raw 16G
```

## Resize a disk image

Command:

```sh
qemu-img resize <file> <resize>
```

Example:

```sh
qemu-img resize /var/vm/disks/test-vm.raw +8G
```

## Set up a VM

You can use `curl` to directly download an installation image if you know where it's hosted. For instance, here I download an image for [Ubuntu Server](https://ubuntu.com/):

```sh
# Enter the images folder
cd /var/vm/images

# Download the image into here
curl https://releases.ubuntu.com/20.04.4/ubuntu-20.04.4-live-server-amd64.iso
```

You'll need to boot a VM from the installation media while a disk image is attached so that you can install the operating system onto it.

```sh
qemu-system-x86_64 -cdrom /var/vm/images/ubuntu-20.04.4-live-server-amd64.iso -boot order=d -drive file=/var/vm/disks/test-vm.raw,format=raw
```

QEMU will create a VNC display server on port `5900` which you can use a VNC viewer to connect to from the host machine. If you need to connect from another machine, add `-vnc :0` to the command.
