### Superuser and su

* `su user1` => Switch to "user1"

* `su` or `sudo su` => Switchs to root user (second option only requires your own account password)

* > PS : the **root user** does NOT need to provide a password to login as other users

* The `/etc/sudoers` File

  * ```bash
    user1	ALL=(ALL)	ALL
    %wheels	ALL=(ALL)	ALL
    # "%" means it's a group
    # WHO WHERE = (AS_WHO=AS_WHICH_GROUP) WHAT
    # WHO WHERE = ALL WHAT (Simplified)
    ```

  * ```bash
    # If you want "user200" to run "tcpdump" (with sudo rights of course)
    user200 ALL=(ALL) /usr/bin/tcpdump
    ```

  * ```bash
    # Allow multiple users to run multiple commands
    Cmd_Alias  ALLCMD   = /usr/bin/yum,/usr/bin/rpm
    User_Alias ALLUSERS = user1,user100,user200
    ALLUSERS   ALL      = ALLCMD
    ```

  * ```bash
    # Allow user2 to run "sudo" WITHOUT password prompt
    user2 ALL=(ALL) NOPASSWD:ALL
    ```

  * Log all commands executed as **sudo**

    * ```bash
      (...) # Got in the "Default" part of the file...
      Defaults    env_keep =  "COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS"
      Defaults 	logfile  =	/var/log/sudo # Add this to log SUDO commands
      (...)
      ```

---

### Permissions

* `chmod a=rwx -v test` => Gives permissions for ALL users (a=U+G+O) to file test

`chmod ugo +rwx test.txt` => U(User) G(Group) O(Other) | Read Write Execute
`chmod 777 test.txt` => 7=4(r)+2(w)+1(e)

**Change owner of file/directory**

`chown (new_owner) (file_or_dir)`

**Change Group Owner of file/directory**

`chgrp (group_name) (file_or_dir)`

---

### Default permissions - Umask

  * For **root** user => 0022

    * For Files => 666-022=644 (RW-R--R--)
    * For Folders => 777-022=755 (RWX-R-X-R-X)

  * For normal users => 0002

    * For Files => 666-002=664 (RW-RW-R--)
    * For Folders => 777-002=775 (RWX-RWX-R-X)

  * `umask` or `umask -S` => Displays the default permissions

  * `umask 027` => To change default permissions

  * > The "umask" value will be lost as soon as you log off !

* **File Maximum Permissions** => 666 (NEVER Executable)

* **Folder Maximum Permissions** => 777

---

### Special Permissions

* **SUID** => Allows a **file** to be executed AS THE **OWNER**.
  * `chmod u+s` or `chmod 4000`
* **SGID** => Allows a **file** to be executed AS THE **GROUP** OWNER.
  * `chmod g+s` or `chmod 2000`
* **Sticky** => Allows ONLY the owner to delete the **directory**.
  * `chmod o+s` or `chmod 1000`

---

### ACL

* **ACL Format**
  
  * `u:(user_name):(permissions) (file_name)`
  * `u:toto:r file1`
  * `g:group1:rw file2`
  * `o:r`
  * `m:rw` => "m" is the ACL mask (=Maximum allowed permissions for NAMED USERS or GROUPS)
  
* `getfacl (file_name)` => Returns ACL details about a specific file

* > PS: if `ls -l` displays a file with '-rw-rw-r--+' (the "+" sign) => ACL are applied on that file.

*  **SETFACL**

  * Options :

    * `-b` => Removes ALL ALCs
    * `-m` => Modifies ACL permissions for a file
    * `-x` => Removes specific ALC for a file

  * Examples :

    * `setfacl -m u:toto:rw,m:r test1`
    * `setfacl -m u:test:rwx,g:group1:rw,m:r`
    * `setfacl -x u:user2 file10`
    * `setfacl -b file20`

  * Test case :

    * ```bash
      useradd toto # Create a new user toto
      passwd toto # set new password for user toto
      cd /home/
      chmod 755 test/ # gives read access to all users
      cd Documents/
      touch test1
      chmod 600 test1 # Only access for owner
      
      setfacl -m u:toto:rw test1 # Gives RW to toto
      su toto
      echo "test" > test1 # WORKS !
      
      # Back to user "test"
      setfacl -m u:toto:rw,m:r test1 # Add a ACL mask of READ
      su toto
      echo "test" > test1 # Does NOT Work anymore
      ```
      
    * The user "toto" was not able to write anymore, because the ACL mask was set to READ only as maximum permission.

---

### Default ACL

* On sub-directory

  * **Receives Default ACLs**
  * **Receives Default ACLs AS ACCESS ACLs**

* On Files

  * Receives Default ACLs AS ACCESS ACLs

  * > PS : Files are NEVER Executable automatically

* Example :

  * ```bash
    [test@fedora Documents]$ mkdir a # Create a test dir "a"
    [test@fedora Documents]$ setfacl -dm u:test:7,u:toto:7 a/ # apply default ACL
    [test@fedora Documents]$ getfacl a
    (...)
    user::rwx
    default:user:test:rwx
    default:user:toto:rwx
    default:mask::rwx
    ```

  * ```bash
    # Default ACL with Subdir
    [test@fedora Documents]$ mkdir a/aa # Create a subdir "aa"
    [test@fedora Documents]$ getfacl a/aa/
    (...)
    user::rwx
    user:test:rwx # Default ACL added as Access ACL
    user:toto:rwx # Default ACL added as Access ACL
    default:user:test:rwx
    default:user:toto:rwx
    default:mask::rwx
    ```

  * ```bash
    # Default ACL with file
    [test@fedora Documents]$ touch a/aa/aa.txt
    [test@fedora Documents]$ getfacl a/aa/aa.txt 
    (...)
    user::rw- # "X" removed as File are NEVER Automatically Executable
    user:test:rwx # Default ACL added as Access ACL
    user:toto:rwx # Default ACL added as Access ACL
    mask::rw-
    # NO DEFAULT ACL
    ```
