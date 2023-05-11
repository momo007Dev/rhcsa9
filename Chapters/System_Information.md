### System Commands

* **"tree" command**
  * Information :
    * Display the file system in a tree view ?
  * Options :
    * `-a` = Hidden
    * `-h` => Display sizes in human style
    * `-f` => Prints the full path for each file
    * `-p` => Print file permissions
    * `-d` => Exclude file from the output (Only shows directories)
    * `-L` (Level) => Max display depth of the directory tree
  * Example :
    * `tree -L 1`

* `tty` => Identifies the current terminal session
* `uptime` => System uptime + process load.

* **Finding command paths** :
  * `which (command)`
  * `whereis (command)`
  * `type (command)`
* System information
  * `uname -a`
* `lscpu` => CPU Information
* `free` => RAM information

---

### Kernel

* `rpm -ivh (kernel.rpm)` => Install new kernel (does not replace current one)
* `grubby --default-kernel` => Displays the default kernel
* `uname -a` => Displays the currebt kernel
* `grubby --set-default="kernel_path"` => Sets the new default kernel

* `uname -m` => Reveals the architecture of the system.
* `uname -r` => 4.18.0-80.el8.x86_64
  * 4 => Major Version
  * 18 => Major Review
  * 0 => Kernel Patch Version
  * 80 => Red Hat Version
  * el8 => Enterprise Linux 8
  * x86_64 => Processor Architecture

---

### User Login Information

* `who`
* `w` => `who` with more details
  * Column "From" is either :
    * ":0" => GUI Session
    * IP Address (SSH for example)
    * Textual on console (tty2, tty3,...)
* `whoami` => User current identity (who is he logged as)
* `logname` => Identity of the user who logged in (who is he)

---

### Reboot information

* `last` => History of successful user login attempts and system reboots
* `last reboot` => Only reboots
* `lastb` => Unseccessfull login attempts
* `lastlog` => Most recent login information
* `id` => Display user information
