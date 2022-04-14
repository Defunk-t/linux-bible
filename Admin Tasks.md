This page covers assorted administrative tasks.

# Change a User ID

```sh
# Change user's ID
usermod  -u [newUserID]  [username]

# Change group's ID
groupmod -g [newGroupID] [username]

# Set user owned files to use new ID
find / -user  [oldUserID]  -exec chown -h [username] {} \;

# Set user group associated files to use new ID
find / -group [oldGroupID] -exec chgrp -h [username] {} \;
```
