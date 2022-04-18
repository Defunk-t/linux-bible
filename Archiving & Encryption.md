# Archiving

## tar

When creating archives such as backups on a Linux system, `tar` is generally used as it preserves the Linux filesystem structure and permissions.

To create a tar archive:

```sh
tar -cvf output.tar <files...>
```

Add the `-z` flag to enable `gzip` compression:

```sh
tar -cvzf output.tar.gz <files...>
```

**Explanation:**

- `-c`: create archive
- `-v`: verbose mode (list files)
- `-z`: gzip compress
- `-f`: output to file

### Excluding files

You can pass one or many `--exclude=PATTERN` options.

You can also pass an exclude file location with `--exclude-from=FILE` or `-X <FILE>`. This file can have multiple lines of exclude patterns.

When excluding a directory, **don't** include a trailing slash in the pattern.

# Encryption

GPG can be used to symmetrically encrypt files:

```sh
gpg -c <file>
```

**Additional options:**

- `--cipher-algo`: encryption algorithm e.g. `aes256`.
- `--compress-algo`: compression algorithm e.g. `zip`.

# Encrypted tar

The `stdout` of the `tar` command (or any command) can be piped into `gpg`, which can then encrypt the stream before saving it to a file.

To output to `stdout`, `-` is specified as the output file.

Example:

```sh
tar -cvzf - /home | gpg -o mybackup.tar.gz.gpg -c
```

Decrypt and de-archive:

```
gpg -d mybackup.tar.gz.gpg | tar -xvzf -
```

## Archive, Encrypt, Send Over SSH

This is an example of how I usually run a backup. This command will:

- Create a compressed `tar` archive of the `/home` directory, excluding patterns in the `backup_exclude` file in the current directory.
- Pipe it into `gpg` to encrypt it with the `AES256` cipher.
- Pipe the encrypted stream to an `ssh` command, where the remote machine writes the stream to a file with today's date in the file name (formatted as `YYYY-MM-DD`).

```sh
tar -cvzf - -X backup_exclude /home | gpg --cipher-algo aes256 -o - -c | ssh admin@backup-server "cat > /backups/$(date '+%F')_backup.tar.gz.gpg"
```
