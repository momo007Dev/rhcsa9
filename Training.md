## EX200 - Exam Dump

* **Q1 : Configure your Host Name, IP Adress, Gateway and DNS**

  * Host name: station.domain40.example.com
  
  * IP Address:172.24.40.40/24
  
  * Gateway172.24.40
  
  * DNS:172.24.40.1
  
  * ```bash
    # GUI Option
    sudo nmtui
    
    # CMD Option
    nmcli connection modify ipv4.method manual ipv4.addresses 172.24.40.40/24 ipv4.gateway 172.24.40.1 ipv4.dns 172.24.40.1
    
    hostnamectl set-hostname station.domain40.example.com
    sudo nmcli connection down enp0s3 && sudo nmcli connection up enp0s3
    ```

* **Q2 : Add 3 users : Harry, Natasha, Tom.**

  * Harry and Natasha in the admin group.

  * Tom's login shell should be non-interactive.

  * ```bash
    useradd -G admin harry
    useradd -G admin natasha
    useradd -s /sbin/nologin tom
    ```

* **Q3 : Create a catalog under /home named admins.**

  * Its respective group is requested to be the admin group.

  * The group users could read and write, while other users are not allowed to access it.

  * The files created by users from the same group should also be the admin group.

  * ```bash
    mkdir /home/admins
    groupadd admin
    chgrp admin admins/
    chmod 770 admins/
    chmod g+s admins/ # SGID
    ```

* **Q4 : Configure a task: plan to run echo hello command at 14:23 every day.**

  * ```bash
    which echo # To find the absolute path of the command "echo"
    crontab -e # Edit cron jobs
    23 14 * * * /bin/echo hello # Min-Hours-Day-Month-Week
    :wq! # To save file
    ```

* **Q5 : Find the files owned by harry, and copy it to catalog: /opt/dir**

  * ```bash
    mkdir /opt/dir
    find / -user harry -exec cp -rfp {} /opt/dir/ \;
    ```

* **Q6 : Find the rows that contain abcde from file /etc/testfile, and write it to the file/tmp/testfile, and the sequence is requested as the same as /etc/testfile.**

  * ```bash
    grep abcde /etc/testfile > /tmp/testfile
    ```

* **Q7 : Create a 2G swap partition which take effect automatically at boot-start, and it should not affect the original swap partition.**

  * ```bash
    #create a partition for given size(2G)
    fdisk /dev/sdb
    n
    +2G
    w
    partprobe # Updates partitions information
    
    #check the present free memory
    free -h
    
    mkswap /dev/sdb5
    swapon /dev/sdb5
    
    nano /etc/fstab
    /dev/sdb5 swap swap defaults 0 0
    
    swapon -a # Auto mounts Swap
    ```

* **Q8 : Create a user named alex, and the user id should be 1234, and the password should be alex111.**

  * ```bash
    useradd -u 1234 alex
    passwd alex
    ```

* **Q9 : Configure yum repo from /var/ftp/pub catalog.**

  * ```bash
    cd /etc/yum.repos.d
    vim local.repo
    [local] # can be any name in brackets
    name=local.repo # any name you want
    baseurl=/var/ftp/pub # local repo's network location)
    enabled=1
    gpgcheck=0
    ```

* **Q12 : Configure autofs to make sure after login successfully, it has the home directory autofs, which is shared as /rhome/ldapuser40 at the ip: 172.24.40.10 and it also requires that, other ldap users can use the home directory normally.**

  * ```bash
    yum install autofs
    showmount -e 172.24.40.10
    ##Create user without a home directory
    useradd -M ldapuser40
    ##Create file
    vi /etc/auto.master.d/autohome.autofs
    ##add this line
    /- /etc/auto.home
    ##create this file
    vi /etc/auto.home
    ##Add this line and save file
    /home/ldapuser40 -rw,sync 172.24.40.10:/rhome/ldapuser40
    
    systemctl enable --now autofs.service
    ```

* **Q13 : Configure the system synchronous as 172.24.40.10.**

  * ```bash
    System-->Administration-->Date & Time # # GUI
    ```

* **Q14 : Change the logical volume capacity named vo from 190M to 300M. and the size of the floating range should set between 280 and 320. (This logical volume has been mounted in advance.)**

  * ```bash
    lvextend -r -L +110M /dev/vg2/lv2
    # -r => To resize underlaying File System
    ```
  
* **Q15 : Create a volume group, and set 16M as a extends. And divided a volume group containing 50 extends on volume group lv, make it as ext4 file system, and mounted automatically under /mnt/data.**

  * ```bash
    pvcreate /dev/sda1 /dev/sda2
    vgcreate -s 16M vg01 /dev/sda1 /dev/sda2
    lvcreate -l 50 -n lv01 vg01
    mkfs.ext4 /dev/vg01/lv01
    mkdir -p /mnt/data
    blkid /dev/vg01/lv01
    
    vim /etc/fstab
    UUID=UUID /mnt/data ext4 defaults 0 0
    mount -a
    ```

* **Q16 : Upgrading the kernel as 2.6.36.7.1, and configure the system to Start the default kernel, keep the old kernel available.**

  * ```bash
    rpm -ivh [kernel.rmp] # Install a rpm package in verbose mode
    yum install kernel # If you're using repositories
    
    grubby --info=ALL | grep ^kernel # List all kernel paths
    grubby --set-default=#kernel path obtained from previous command
    ```

* **Q17 : Create a 512M partition, make it as ext4 file system, mounted automatically under /mnt/data and which take effect automatically at boot-start.**

  * ```bash
    fdisk /dev/vda
    n
    +512M
    w
    partprobe /dev/vda
    mkfs -t ext4 /dev/vda5
    
    mkdir -p /data
    vim /etc/fstab
    /dev/vda5 /data ext4 defaults 0 0
    mount -a
    ```

* **Q18 : Create a volume group, and set 8M as a extends. Divided a volume group containing 50 extends on volume group lv (lvshare), make it as ext4 file system, and mounted automatically under /mnt/data. And the size of the floating range should set between 380M and 400M.**

  * ```bash
    vgcreate -s 8M vg1 /dev/sda3
    lvcreate -l 50 -n lvshare
    mkfs.ext4 /dev/vg1/lvshare
    
    mkdir /mnt/data3
    lsblk -pf (UUID=…457f-446bbb805753)
    echo “UUID=…457f-446bbb805753 /mnt/data3 ext4 defaults 0 0” >> /etc/fstab
    ```

* **Q19 : Download ftp://192.168.0.254/pub/boot.iso to /root, and mounted automatically under /media/cdrom and which take effect automatically at boot-start.**

  * ```bash
    cd /root
    wget ftp://192.168.0.254/pub/boot.iso
    mkdir -p /media/cdrom
    vim /etc/fstab
    /root/boot.iso /media/cdrom iso9660 defaults,loop 0 0
    mount -a
    ```

* **Q20 : Add admin group and set gid=600.**

  * ```bash
    groupadd -g 600 admin
    ```

* **Q21 : Add user: user1, set uid=601.**

  * Password is redhat.

  * The user's login shell should be non-interactive.

  * ```bash
    useradd -u 601 -s /sbin/nologin user1
    passwd user1
    ```

* **Q23 : Copy /etc/fstab to /var/tmp name admin, the user1 could read, write and modify it, while user2 without any permission.**

  * ```bash
    cp /etc/fstab /var/tmp/admin
    setfacl -m u:user1:rw- /var/tmp/admin
    setfacl -m u:user2:--- /var/tmp/admin
    ```

* Q25 : Configure a default software repository for your system.

  * One YUM has already provided to configure your system on http://server.domain11.example.com/pub/ x86_64/Server, and can be used normally.

  * ```bash
    dnf config-manager --add-repo=http://server.domain11.example.com/pub/ x86_64/Server/BaseOs
    
    vim /etc/yum.repos.d
    name="My Repo"
    baseurl="http://server.domain11.example.com/pub/ x86_64/Server/BaseOs"
    enable=1
    gpgcheck=0
    ```

  * 
