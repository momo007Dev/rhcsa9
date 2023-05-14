### The Linux Firewall - firewalld

* Ports are defined in the ***/etc/services*** file for common network services that are standardized across all network operating systems, including RHEL.
* One of the major advantages is its ability to add, modify, or delete firewall rules immediately without disrupting current network connections or restarting the service process.
* firewalld stores the default rules in files located in the ***/usr/lib/firewalld*** directory.
* firewalld uses the concept of zones for easier and transparent traffic management.
  * Zones
    * public => Reject all except what is allowed. Default.
    * home, internal, trusted,...

**Zone Configuration Files**

* firewalld stores zone rules in XML format at two locations: the **system-defined** rules in the ***/usr/lib/firewalld/zones*** directory, and the **user-defined** rules in the ***/etc/firewalld/zones*** directory.

  * Public zone example : (`cat /usr/lib/firewalld/zones/public.xml`)

    * ```xml
      <?xml version="1.0" encoding="utf-8"?>
      <zone>
        <short>Public</short>
        <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
        <service name="ssh"/>
        <service name="mdns"/>
        <service name="dhcpv6-client"/>
        <forward/>
      </zone>
      ```

**Service Configuration Files**

* firewalld stores service rules in XML format at two locations: the system-defined rules in the ***/usr/lib/firewalld/services*** directory, and the user-defined rules in the ***/etc/firewalld/services*** directory. 

  * SSH service example : (`cat /usr/lib/firewalld/services/ssh.xml`)

    * ```xml
      <?xml version="1.0" encoding="utf-8"?>
      <service>
        <short>SSH</short>
        <description>Secure Shell (SSH) is a protocol for logging into and executing commands on remote machines. It provides secure encrypted communications. If you plan on accessing your machine remotely via SSH over a firewalled interface, enable this option. You need the openssh-server package installed for this option to be useful.</description>
        <port protocol="tcp" port="22"/>
      </service>
      ```

---

### Firewall Management

* The `firewall-cmd` command is used to manage the firewalld service.

* Commands :

  * General
    * `firewall-cmd --state`
    * `firewall-cmd --reload`
    * `firewall-cmd --permanent` => State will remain after reboot
  * Zones
    * `firewall-cmd (--get-default-zone/--set-default-zone)`
    * `firewall-cmd --get-zones` => List available zones
    * `firewall-cmd --get-active-zones`
    * `firewall-cmd --list-all` => List all settings for a zone
    * `firewall-cmd --list-all-zones`
    * `firewall-cmd --zone` => Specifies the name of the zone to work on.
  * Services
    * `firewall-cmd --get-services`
    * `firewall-cmd --list-services` => List services for a zone
    * `firewall-cmd --add-service`
    * `firewall-cmd --remove-service`
    * `firewall-cmd --query-service` => Query for the presence of a service
  * Ports
    * `firewall-cmd --list-ports`
    * `firewall-cmd --add-port`
    * `firewall-cmd --remove-port`
    * `firewall-cmd --query-port`
  * Network Connections
    * `firewall-cmd --list-interfaces`
    * `firewall-cmd --add-interface`
    * `firewall-cmd --change-interface`
    * `firewall-cmd --remove-interface`
  * IP Sources
    * `firewall-cmd --list-sources`
    * `firewall-cmd --add-source`
    * `firewall-cmd --change-source`
    * `firewall-cmd --remove-source`

* **Exercice : Add Services and Ports and Manage Zones**

  * In this exercise, you will determine the **current active zone**. You will **add and activate a permanent rule to allow HTTP traffic on port 80**, and then **add a runtime rule for traffic intended for TCP port 443** (the **HTTPS service**). You will **add a permanent rule to the internal zone for TCP port range 5901 to 5910**. You will confirm the changes and display the contents of the affected zone files. Lastly, you will **switch the default zone to the internal zone and activate it**.

  * ```bash
    sudo firewall-cmd --get-default-zone
    sudo firewall-cmd --permanent --add-service http
    sudo firewall-cmd --add-port 443/tcp
    
    sudo firewall-cmd --add-port 5901-5910/tcp --permanent --zone internal
    sudo cat /etc/firewalld/zones/internal.xml
    
    sudo firewall-cmd --set-default-zone internal
    sudo firewall-cmd --get-default-zone
    
    sudo firewall-cmd --reload
    ```

* **Exercice : Remove Services and Ports, and Manage Zones**

  * In this exercise, you will **remove the two permanent rules** that were added in the previous exercise. You will **switch the public zone back as the default zone**, and confirm the changes

  * ```bash
    sudo firewall-cmd --remove-service=http --zone public --permanent
    sudo firewall-cmd --remove-port 5901-5910/tcp --permanent
    sudo firewall-cmd --set-default-zone=public
    sudo firewall-cmd --reload
    ```

---

### SSH

* **GSSAPI-Basec Authentication**
  * GSSAPI provides a standard interface that allows security mechanisms, such as Kerberos, to be plugged in.
* **Host-Based Authentication**
  *  For each user that requires an automatic entry on the server, a ***~/.shosts*** file is set up containing the client name or IP address, and, optionally, a different username.
  * The same rule applies to a group of users or all users on the client that require access to the server. In that case, the setup is done in the ***/etc/ssh/shosts.equiv*** file on the server.
* Private/Public Key-Based Authentication
* Challenge-Response Authentication
* Password-Based Authentication

* OpenSSH commands

  * `ssh-copy-id` => Copies public key to a remote system
  * `ssh-keygen` => Generates and manages private/public keys

* SSH configuration files

  * Server configuration file => `/etc/ssh/sshd_config`
    * **ListenAddress** => Default is to listen on ALL local addresses
    * **UsePAM** => If enabled, onyl root will be able to run sshd daemon. Default is YES.
    * **X11Forwarding** => Allows or disallows remote access to graphical applications. Default is yes.
    * **PermitRootLogin no** => To remove direct access to root
    * **PasswordAuthentication no** => To use key-based auth
  * Server log file => `/var/log/secure`
  * Client configuration file => `/etc/ssh/ssh_config`
    * IdentityFile => Defines the name and location of a file that store a user's private key.

* **Configuring SSH Key-Based Authentication**

  * 1. Generating SSH Keys

    - `ssh-keygen (-t key_type)` => Will generate a pair of public and private key in the ***~/.ssh/***
    - Permissions must be 600 and 644 on the private and public key.

  * 2. Sharing the Public Key

    - `ssh-copy-id -i .ssh/my_public_key.pub user@remote_host`

  * 3. Using a private key (yours on someone else) to access the server

    - `ssh -i .ssh/my_private_key user@remote_host`

---

### Secure SSH

* Configure Idle Timeout Interval (avoids having an unattended SSH session)

  - Edit "/etc/ssh/sshd_config" and add this at the END of file.

  - ```bash
    ClientAliveInterval 600 # 600 sec = 10min
    ClientAliveCountMax 0
    ```

  - `systemctl restart sshd`

* Disable root login

  - Edit "/etc/ssh/sshd_config" and find `PermitRootLogin`
  - Set to "no" and uncomment it.
  - `systemctl restart sshd`

* Disable Empty Passwords

  - Edit "/etc/ssh/sshd_config" and find `PermitEmptyPasswords`
  - Set to "no" and uncomment it.
  - `systemctl restart sshd`

* Limit Users' SSH Access

  - Edit "/etc/ssh/sshd_config" and add at the END of file.

  - ```
    AllowUsers (user_1) (user_2)
    ```

  - `systemctl restart sshd`

* Use a different port

* Use SSH-Keys

---

### SFTP - SCP - RSYNC

* `scp` => Secure Copy - to transfer files with the ssh protocol
  * `scp file1 file2 remote_user@remote_host:/home/remote_dir` To copy a file TO remote user
  * `scp remote_user@remote_host:/etc/hostname /home/my_dir` To copy a file FROM remote user
  * `scp -r remote_user:/var/log /tmp` To copy a directory FROM remote user


* `sftp` => Secure FTP - to transfer files with the ssh protocol


  * `sftp server20`
  * Commands

    * `get /usr/bin/gzip` => Downloads file
    * `put /etc/group` => Sends file

* `rsync`


  * The ***rsync*** tool synchronizes ONLY the portions of files that have been changed. Unlike ***scp*** which will need to copy everything.

    One of the most important options of ***rsync*** is the **-n** option to perform a ***dry run***. It's a simulation of what happens when the command gets executed. It helps to ensure that no important files get overwritten or deleted.

    Options :

    - `-r` => Synchronize recursively the whole directory tree
    - `-l` => Synchronize symbolic links
    - `-p` => Preserve permissions
    - `-t` => Preserve time stamps
    - `-g` => Preserve group ownership
    - `-o` => Preserve the owner of the file
    - `-D` => Synchronize device file
    - All of the above options are included all together with the `rsync -a` (Archive Mode).
    - `-v` => Verbose / Shows more details
    - Archive mode does NOT preserve hard links (`-H`), ACLs (`-A`) or SELinux contexts (`-X`)

    **Example** :

    - `rsync -av /var/log /tmp` Synchronize contents of the ***/var/log*** directory to the ***/tmp*** folder.
      - Synchronizes the directory ***/var/log***
    - `rsync -av /var/log/ /tmp` Does NOT create the ***log*** directory in the ***/tmp*** output directory
      - Synchronizes the content of the directory (not the directory) ***/var/log***
    - `rsync -av /var/log remote_host:/tmp` Sync the local directory to the remote directory
    - `rsync -av remote_host:/var/log /tmp` Sync the remote directory with the local directory
