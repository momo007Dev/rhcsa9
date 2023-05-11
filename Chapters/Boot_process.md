### Boot Process

* The boot process is split into 4 phases :
  * Firmware phase => Either BIOS or UEFI
  * Bootloader phase => Loads GRUB2
  * Kernel phase => initrd > systemd > /sbin/init
  * Initialization phase => Goes in default boot target/ run level

**Am I using BIOS or UEFI ?**

* Option 1 : `lsblk`
  * "/boot" = BIOS (Basic Input Ouput System)
  * "/boot/efi" = UEFI (Unified Extensible Firmware Interface)

* Option 2 : `sudo dmesg | grep BIOS`
  * If it does NOT return anything => UEFI
  * If it returns something => BIOS

**Modify Grub settings**

* **BIOS**
  * ***/boot/grub2/grub.cfg***
  * `grub2-mkconfig -o (above_file)` => To apply Grub2 changes

* **UEFI**
  * ***/boot/efi/EFI/redhat/grub.efi***
  * `grub2-mkconfig -o (above_file)` => To apply Grub2 changes
  * (Or could also be ***/boot/efi***)

### GRUB2 - Root Password Recovery

* When the system boots and prompts you to select a boot option (GRUB2 menu), you can press "e" key to edit or "c" to go to the "grub>" command prompt.
* Edit mode ("e") :
  * When pressing the "e" key.
  * Loads the configuration from /boot/grub2/grub.cfg
  * Modifying the configuration file from edit mode is only a 1 time temporary change !

* **Changing root password** :
  * GRUB2 => "e"
  * Go to the line that starts with "linux"
  * Append to the end "rd.break" => The system will break just before moving initrd to system.
    * If you want to boot into another target/run-level : change "rd.break" to "emergency" (for example).
  * `CTRL+x` => To boot with the changes
  * `mount -o remount,rw /sysroot` => Mounts in read/write (instead or read only)
  * `chroot /sysroot` => Switch into a "chroot jail" where "/sysroot" is treated AS root.
  * `passwd root` => Change root password
  * `touch /.autorelable` => Tells the OS to run SELinux to relabel all files including the shadow file.

---

### Units

Units are ***systemd*** objects used for organizing boot and maintenance tasks, such as hardware initialization, socket creation, file system mounts and service startups. Units are in one of several operational states, including active, inactive, in the process of being activated or deactivated, and failed. Units can be enabled or disabled. An enabled unit can be started to an active state. A disabled unit can NOT be started.

Units have a name and a type, and they are encoded in files with names in the form ***unitname.type***. Some examples are `tmp.mount`, `sshd.service`, `syslog.socket`, `umount.target`,...

There are 2 types of unit configuration files : (1) system unit files that are distributed with installed packages and located in the `/usr/lib/systemd/system` directory, and (2) user unit files that are user-defined and saved in the `/etc/systemd/user` directory.

Example :

```bash
[test@fedora ~]$ cat /usr/lib/systemd/system/sshd.service 
[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.target
Wants=sshd-keygen.target

[Service]
Type=notify
EnvironmentFile=-/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```

---

### Targets (or Run Levels)

Targets are simply logical collections of units. They are a special systemd unit type with the ".target" file extension.

| RUN LEVELS     | COMMENT                            |
| -------------- | ---------------------------------- |
| 0 - Poweroff   | Shuts down the system              |
| 1 - rescue     | Single-user mode                   |
| 2 - emergency  | Multi-user mode without networking |
| 3 - multi-user | Multi-user mode with networking    |
| 4              | User-definable                     |
| 5 - graphical  | Multi-user mode with networking    |
| 6 - reboot     | Reboots the system to restart it   |

---

### Systemctl command

The ***systemctl*** command performs administrative functions and supports plentiful subcommands and flags.

* `systemctl daemon-reload`
* `systemctl (enable/disable)` => Activate/desactivate start on boot for a service
* `systemctl (get-default/set-default)` => Get/sets default boot target (or run-level) for NEXT BOOT
* `systemctl (get-property/set-property)` => Get/sets the value of a property
* `systemctl (is-active/is-enabled/is-failed)` => Quick check if service is running, autostart, failed.
* `systemctl isolate (multi-user/rescue)` => Changes the current boot target NOW
* `systemctl kill` => Terminates all processes for a unit
* `systemctl (list-depencies/list-sockets/list-units)`
* `systemctl (mask/unmask)` => Deny or allow manual or automatic launch of a service
* `systemctl (reload/restart/start/stop/status)`
* `systemctl show`
* `systemctl` => List all units that are currently loaded in memory along with their status and description
* `systemctl -t (target/socket)`
