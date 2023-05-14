### NFS - Network File System

* Network File System (NFS) is a networking protocol that allows file sharing over the network.
* The Network File System service is based upon the client/server architecture whereby users on one system access files, directories, and file systems (collectively called “shares”) that reside on a remote system as if they are mounted locally on their system.

* **Export Share on NFS SERVER**

  * In this exercise, you will create a directory called /common and export it to server10 in read/write mode. You will ensure that NFS traffic is allowed through the firewall. You will confirm the export.

  * ```bash
    sudo dnf -y install nfs-utils
    sudo mkdir /common
    sudo chmod 777 /common
    
    sudo firewall-cmd --permanent --add-service nfs
    sudo firewall-cmd --reload
    sudo systemctl --now enable nfs-server
    
    sudo nano /etc/exports
    /common server10(rw)
    # could also be /common 192.168.0.0/24(rw) - to allow all network
    # "server10" is the target who is allowed to access the share - NOT THE SERVER ITSELF !!!
    
    sudo exportfs -av
    # sudo exportfs -u server10:/common => To unexport share
    ```

* **Mount Share on NFS CLIENT**

  * In this exercise, you will mount the /common share exported. You will create a mount point called /local, mount the remote share manually, and confirm the mount. You will add the remote share to the file system table for persistence. You will remount the share and confirm the mount. You will create a test file in the mount point and confirm the file creation on the NFS server.

  * ```bash
    sudo mkdir /local
    sudo mount server20:/common /local # manual mount
    # here "server 20" matches the NFS server
    
    sudo nano /etc/fstab
    server20:/common /local nfs _netdev 0 0 # persistence
    
    sudo unmount /local
    sudo mount -a # auto mount
    ```

---

### Auto File System - AutoFS

* The AutoFS service automatically mounts a share upon detecting an activity in its mount point with a command such as ls or cd.

* In the same manner, AutoFS unmounts the share automatically if it has not been accessed for a predefined period of time.

* > :warning: To avoid inconsistencies, mounts managed with AutoFS should **not be mounted or unmounted manually or via the /etc/fstab file**.

* Benefits of AutoFS

  * Does NOT require root privileges.
  * All shares are defined in text configuration files called **maps**.
  * AutoFS prevents an NFS client from hanging if an NFS server is down or inaccessible.
  * With AutoFS, a **share is unmounted automatically** if it is not accessed for 5 minutes by default.

* The configuration file for the AutoFS service is ***/etc/autofs.conf***.

  * ```bash
    master_map_name = auto.master # name of the master map
    timeout = 300 # time in seconds after share is automatically unmounted
    negative_timeout = 60 # Timout value failed mount attempts
    mount_nfs_default_protocol = 4 # Default NFS version to used
    logging = none # Log level - verbose, debug or none
    ```

  * Generally, those parameters are left to default values.

* **AutoFS Maps**

  * The AutoFS service needs to know the NFS shares to be mounted and their locations.
  * **The Master Map**
    * The auto.master file located in the /etc directory is the default master map.
    * However, it is recommended to store user-defined map files in the ***/etc/auto.master.d*** directory, which the AutoFS service automatically parses at startup.
  * **The Direct Map**
    * The direct map is used to mount shares automatically on any number of unrelated mount points.
    * Each direct map entry places a separate share entry to the ***/etc/mtab*** file, which maintains a list of all mounted file systems whether they are local or remote.
  * **The Indirect Map**
    * The indirect map is preferred over the direct map if you want to mount all of the shares under one common parent directory.
    * Indirect mounted shares become visible only after they have been accessed.

* **Access NFS Share Using Direct Map**

  * In this exercise, you will configure a direct map to automount the NFS share **/common** that is available from server20. You will install the relevant software, create a local mount point **/autodir**, and set up AutoFS maps to support the automatic mounting.

  * ```bash
    sudo dnf install -y autofs
    sudo mkdir /autodir
    
    sudo nano /etc/auto.master
    /- /etc/auto.master.d/auto.dir
    
    sudo touch /etc/auto.master.d/auto.dir
    sudo nano /etc/auto.master.d/auto.dir
    /autodir server20:/common # server20 is the NFS server
    
    sudo systemctl enable --now autofs
    ```

* **Access NFS Share Using Indirect Map**

  * In this exercise, you will configure an indirect map to automount the NFS share **/common** that is available from server20. You will install the relevant software and set up AutoFS maps to support the automatic mounting. You will observe that the specified mount point **“autoindir” is created automatically under /misc**.

  * ```bash
    sudo dnf install -y autofs
    sudo nano /etc/auto.misc
    autoindir server20:/common
    
    sudo systemctl enable --now autofs
    ```

* **Automount User Home Directories using Indirect Map**

  * In the first portion (NFS Server), you will create a user account called user30 with UID 3000. You will add the /home directory to the list of NFS shares so that it becomes available for remote mount.

  * In the second portion (NFS CLIENT), you will create a user account called user30 with UID 3000, base directory /nfshome, and no user home directory. You will create an umbrella mount point called /nfshome for mounting the user home directory from the NFS server. You will install the relevant software and establish an indirect map to automount the remote home directory of user30 under /nfshome. You will observe that the home directory of user30 is automounted under /nfshome when you sign in as user30.

  * ```bash
    # NFS SERVER (server20)
    sudo useradd -u 3000 user30
    sudo passwd user30 # set password
    
    sudo nano /etc/exports
    /home server10(rw)
    
    sudo exportfs -avr
    ```

  * ```bash
    # NFS CLIENT (server10)
    sudo useradd -u 3000 -b /nfshome -M user30
    sudo mkdir /nfshome
    
    sudo nano /etc/auto.master
    /nfshome /etc/auto.master.d/auto.home
    
    sudo touch /etc/auto.master.d/auto.home
    sudo nano /etc/auto.master.d/auto.home
    * -rw server20:/home/user30 # can replace "user30" with "&" for all
    sudo systemctl enable --now autofs
    ```
