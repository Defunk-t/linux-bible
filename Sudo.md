`sudo` is a program that temporarily grants superuser privileges to users in a specific group as they request it.

It is generally accepted as a more secure alternative to logging in as or switching to the `root` user, which would leave the system vulnerable for a longer period of time and may even lead to a system administrator forgetting that they are no longer using their account and doing something to damage it. Furthermore, with `sudo` configured, the `root` user can be disabled altogether, which eliminates the threat of it being compromised by a hacker.

# Installation

**Ubuntu:**

```sh
apt-get install sudo
```

**Arch:**

```sh
pacman -Sy sudo
```

# Configuration

## Using `visudo`

`visudo` is a program that lets you edit the configuration for `sudo` and also checks to make sure syntax is valid. You can run it in the terminal:

```sh
visudo
```

If you are the `root` user you can set the `EDITOR` environment variable to override the editor program it uses. The default is `vi`, although this may vary between distributions.

```sh
EDITOR=nano visudo
```

## Enabling the `wheel` group

If you're installing `sudo` to the system for the first time, you'll need to configure whom to allow to use `sudo`.

Near the end of the file will be the following:

```
...

##
## User privilege specification
##
root ALL=(ALL) ALL

## Uncomment to allow members of group wheel to execute any command
# %wheel ALL=(ALL) ALL

...
```

Simply uncomment the line:

```
## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL) ALL
```

Now users any users in the `wheel` group can use `sudo`. We will add users to the group in a following section.

## Changing 'Defaults specification'

Mid-way through the file, there is a section on 'Defaults specification'. You can add a defaults directive of your own to change certain settings and tell `sudo` which environment variables to preserve.

They are separated by comma, e.g.:

```
Defaults env_keep += "EDITOR" , timestamp_timeout=5
```

### Keeping Environment Variables

If you were to try `EDITOR=nano sudo visudo`, it would not work. `visudo` would still try to use `vi` instead of `nano` because the environment variable is not carried through into the `sudo` command.

`env_keep` can be used to tell `sudo` to keep certain environment variables:

```
Defaults env_keep += "EDITOR"
```

### Changing the Timeout

By default, `sudo` grants superuser privileges for 15 minutes. This can be changed with `timestamp_timeout`:

```
Defaults timestamp_timeout=5
```

The above reduces the timeout to 5 minutes.

## Adding users to the `wheel` group

> **Note:** some distributions are configured to use the `sudo` group instead of the `wheel` group.

To add a user to the `wheel` group and grant them `sudo` privileges:

```sh
usermod -aG wheel [username]
```

To remove them from the group:

```sh
gpasswd -d [username] wheel
```

## Disabling `root`

With `sudo` configured and at least one user able to use it, you can now go ahead and disable the `root` user entirely. If you are not going to use `root` anymore, it's good to disable it as it prevents it from being leveraged by hackers.

Open the `/etc/passwd` file:

```sh
sudo nano /etc/passwd
```

You'll see a list of users on the system including a line that looks like this:

```
root:x:0:0::/root:/bin/bash
```

The last part, `/bin/bash`, indicates the shell program that the user has access to when they log in. Change the shell for the `root` user to `/bin/nologin` instead:

```
root:x:0:0::/root:/bin/nologin
```

Save and quit. The `root` user can no longer log in.
