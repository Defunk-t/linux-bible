# Installation

Arch Linux:

```sh
sudo pacman -Sy openssh
```

# SSH Server

## Enable service

```sh
# Enable the SSH server daemon on boot
sudo systemctl enable sshd

# Start the SSH server daemon now
sudo systemctl start sshd
```

## Configuration

```sh
# Open server-side configuration file
sudo nano /etc/ssh/sshd_config
```

Some notable options here:

- **Port:** Which port(s) to accept connections on.  
  Multiple of this type is allowed.  
  The default port for SSH is `22`.


- **ListenAddress:** Which local IP addess(es) to accept connections on.  
  Multiple of this type is allowed.  
  Default behaviour is to accept connections on all addresses.


- **PermitRootLogin:** Controls whether or not the root account can be accessed over SSH.  
  Valid options are `yes`, `no`, `prohibit-password` and `forced-commands-only`.  
  Default is `prohibit-password`.


- **StrictModes:** Only allows login if the local user has the correct file permissions set.  
  Requires `$HOME` to be writable ONLY by the user, e.g. `rwxr-xr-x` or `755`.  
  Requires `$HOME/.ssh` to be `rwx------` or `700`.  
  Valid options are `yes` or `no`. Default is `yes`.


- **PubkeyAuthentication:** Controls whether authentication by public key is allowed.  
  Valid options are `yes` or `no`. Default is `yes`.


- **PasswordAuthentication:** Controls whether authentication by password is allowed.  
  Valid options are `yes` or `no`. Default is `yes`.

# SSH Client

## Generate keys

You only need to do this once per user.
By default, `id_rsa` and `id_rsa.pub` will be added to your `$HOME/.ssh` folder.

```sh
ssh-keygen
```

## Upload your key to an SSH server

```sh
# Copy your public key to authorized_keys on server
ssh-copy-id admin@192.168.10.2
```

Now you can access without a password prompt, so long as the server allows public key authentication.

## Add a remote server to hosts file

Open the hosts file as root:

```sh
sudo nano /etc/hosts
```

Add the server's IP address along with a name:

```
192.168.10.2   server-name
```

Now `192.168.10.2` can be accessed by `server-name` instead.  
For example:

```sh
# Login over SSH
ssh admin@server-name
```
