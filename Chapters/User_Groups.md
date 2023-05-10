### Managing Users

| Command       | Option | Comment                                |
| ------------- | ------ | -------------------------------------- |
| **`useradd`** | `-c`   | Sets the "comment"                     |
|               | `-d`   | Home directory                         |
|               | `-s`   | Path to shell (default shell for user) |
|               | `-u`   | UID (User ID)                          |

* Example :
  * `useradd -u 1010 -d /usr/toto -s /bin/bash toto`


| Command       | Option | Comment                                                      |
| ------------- | ------ | ------------------------------------------------------------ |
| **`usermod`** | `-ag`  | Adds ("-a") PRIMARY ("-g") groups to user                    |
|               | `-aG`  | Adds ("-a") SUPPLEMENTARY ("-G") group to user               |
|               | `-l`   | Specify a new login name                                     |
|               | `-m`   | Creates a new home directory + moves old content to new directory |
|               | `-c`   | Modifies the "comment" of the user                           |
|               | `-d`   | Specify the home directory                                   |
|               | `-s`   | Specify the shell                                            |
|               | `-u`   | New UID                                                      |
|               | `-L`   | Locks user account                                           |
|               | `-U`   | Unlocks user account                                         |

* Notes :
  * `-g` => Replaces (sets the PRIMARY group to)
  
  * `-G` => Replaces (sets the SUPPLEMENTARU group to)

  * Locked user accounts have a "!" in the `/etc/passwd` file
  
* Examples :
  
    * ```bash
      # Lock user account
      [root@fedora Documents]# usermod -L toto
      [root@fedora Documents]# grep toto /etc/shadow
      toto:!$y$j9T(...):19370:0:99999:7::: # Notice the "!" => DISABLED
      [root@fedora Documents]# usermod -U toto
      [root@fedora Documents]# grep toto /etc/shadow
      toto:$y$j9T(...):19370:0:99999:7::: # Account ENABLED
      ```
    
    * `usermod -l user2new -m -d /home/user2new -s /sbin/nologin -u 2000 user2`
    
    * `usermod -ag g1 user1` => Adds a PRIMARY group "g1" to user1
    
    * > PS : ALL USERS MUST HAVE AT LEAST 1 PRIMARY GROUP
      >
      > PPS : Max 15 SUPPLEMENTARY groups for a user

| Command       | Option | Comment                                                    |
| ------------- | ------ | ---------------------------------------------------------- |
| **`userdel`** | `-r`   | Also removes the home directory and data owned by the user |

---

### Managing Groups

* Groupadd
  * Options
    * `-g` => Sets GID
    *  `-o` => Same GID as another group
    * `-r` => GID below 1000 (for services groups)
  * Example :
    * `groupadd -g 5000 g1` => Creates a new group "g1" with GID 5000
* Groupmod
  * Options
    * `-g` => Change the GID
    * `-n` => Renames the group
  * Examples :
    * `groupmod -n new g1` => Changes the group name of "g1" to "new"
    * `groupmod -g 6000 new` => Changes the GID of "new" group to 6000
* Groupdel
  * `groupdel (group_name)`
* Get group information
  * `groups user1` => Display groups that user1 is a member of

---

### Modify Password and Account Information

| Command     | Option | Comment                                                      |
| ----------- | ------ | ------------------------------------------------------------ |
| **`chage`** | `-d`   | Last day that the password was changed (`-d 0` = force password change at next login) |
|             | `-E`   | Date on which the account will be DISABLED                   |
|             | `-I`   | Number of days AFTER password has expired to disable account |
|             | `-m`   | Minimum days before user can change password again           |
|             | `-M`   | Maximum days that the password is valid                      |
|             | `-W`   | Number of days for which user gets alert to change password  |
|             | `-l`   | List password aging attributes set on a user account         |

* Example :
  * `chage -m 7 -M 28 -W 5 user100`
  * `chage -E 2023-01-31 user 200` => Set password to expire on the 31/01/2023

| Command      | Option | Comment                                                      |
| ------------ | ------ | ------------------------------------------------------------ |
| **`passwd`** | `-d`   | Deletes a user password WITHOUT expiring the account         |
|              | `-e`   | Forces a user to change password at next login (=`chage -d 0 user1`) |
|              | `-l`   | Locks a user account (=`usermod -L user1`)                   |
|              | `-u`   | Unlocks a user account (=`usermod -U user1`)                 |
|              | `-S`   | Displays status information for a specific user              |

* Example :
  * `passwd -e user200` => Forces user200 to change his password

---

### User Information File - "/etc/passwd"

* ***File => /etc/passwd***

* ```bash
  [test@fedora Documents]$ ls -l /etc/passwd
  -rw-r--r--. 1 root root 2669 Jan 13 18:03 /etc/passwd
  # Readable by all users
  ```

* `user1:x:1000:1000:test_user:/home/user1:/bin/bash`

  * `user1` => Login name
  * `x` => Password is encrypted
  * `1000` => UID
  * `1000` => GID
  * `test_user` => Comment
  * `/home/user1` => Home directory
  * `/bin/bash` => Shell

---

### Group Information File - "/etc/group"

* ***File => /etc/group***

* ```bash
  [test@fedora Documents]$ ls -l /etc/group
  -rw-r--r--. 1 root root 1042 Jan 13 18:03 /etc/group
  # Readable by all users
  ```

* `group1:x:1000:group1`

  * `group1` => Group name
  * `x` => Password is encrypted
  * `1000` => GID
  * `group1` => Group members

---

### User Password Information File - "/etc/shadow"

* ***File => /etc/shadow***

* ```bash
  [test@fedora Documents]$ ls -l /etc/shadow
  ----------. 1 root root 1334 Jan 13 18:03 /etc/shadow
  # NO PERMISSIONS !
  # But root user can still edit this file
  ```

* `test:$y$j9T$w63H0Ahc5qX0h9KNMZGbv1$7(...):19345:0:99999:7:::`

  * (1) `test` => Login name
  * (2) `$y$...$...$...` => Encrypted Password
    * "$y" = Format | Round | Salt | Hash
  * (3) `19345` => Last Password Changed days
  * (4) `0` => Minimum days before the user can change he's password
  * (5) `99999` => Maximum days that the password is valid (after, must be changed)
  * (6) `7` => Warn days (7 days before the end of password validity, you'll get a warning)
  * (7) Number of days (after the password has been expired) before account is DISABLED
  * (8) Date which the account will be DISABLED

---

### User file management

**`/etc/login.defs`**: Used for default settings like UID settings, passwd default settings, and other things.

**`/etc/profile`**: Used for default settings for all users when starting a login shell.

**`/etc/bashrc`**: Used to define defaults for all users when starting a subshell.

**`~/.profile`**: Specific settings for one user applied when starting a login shell.

**`~/.bashrc`**: Specific settings for one user applied when starting a subshell.