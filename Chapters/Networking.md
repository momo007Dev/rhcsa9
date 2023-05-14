### Network files

* Protocols are defined in the ***/etc/protocols*** file.

* ```bash
  cat /etc/protocols
  ip   0 IP   # internet protocol, pseudo protocol number
  icmp 1 ICMP # internet control message protocol
  (...)
  ```

* Well-known ports are defined in the ***/etc/services*** file.

* ```bash
  cat /etc/services
  ftp  21/tcp
  ssh  22/tcp # The Secure Shell (SSH) Protocol
  (...)
  ```

---

### Modify Network Connection with nmtui (GUI)

This method is recommended as it will save a LOT of time.
* `nmtui`

![](../images/nmtui.png)

---

### Modify Network Connection by the configuration file

* Each network connection has a configuration file that defines IP assignments and other relevant parameters for it.

* Connection configuration files (or connection profiles) are stored under :

  * ***/etc/sysconfig/network-scripts/***
    * The filenames begin with "`ifcfg-`" and are followed by the name of the connection.
  * ***/etc/NetworkManager/system-connections/*** => NEW RHEL 9

* Example :

  * ```bash
    cat /etc/sysconfig/network-scripts/ifcfg-enp0s3 | grep -vi ipv6
    TYPE=Ethernet
    PROXY_METHOD=none
    BROWSER_ONLY=no
    BOOTPROTO=none # YES = DHCP
    DEFROUTE=yes # Use this connection as the default path to internet
    IPV4_FAILURE_FATAL=no # Disable connection if no IP is received
    NAME=enp0s3
    UUID=5ece88f2...
    DEVICE=enp0s3
    ONBOOT=yes # Auto-activates on boot
    IPADDR=192.168.0.110
    NETMASK=255.255.255.0
    GATEWAY=192.168.0.1
    ```

  * More details can be found with the `man nm-settings` file.

---

### Modify Network Connection with nmcli

* `sudo (ifdown/ifup) (interface_name)` => Brings interface down/up

* `nmcli` is a NetworkManager command line tool that is employed to create, view, modify, remove, activate and deactivate network connections and to control and report network device status.

* Nmcli commands :

  * Connections (c)

    * `show` => Lists connections

    * `up / down` => Activates / deactivates a connection

    * `add` => Adds a connection

    * `edit` => Edits an existing connection or adds a new one

    * `modify` => Modifies one or more properties of a connection

    * `delete` => Deletes a connection

    * `reload` => Reloads all connections

    * `load` => Reloads a specific connection

    * ```bash
      nmcli c s # nmcli connection show
      sudo nmcli c (up/down) enp0s3
      ```

  * Device

    * `status` => Displays device status

    * `show` => Displays information about a specific interface

    * ```bash
      nmcli d s # nmcli device show
      ```

* **Configure New Network Connection Using nmcli**

  * In this exercise, you will create a connection profile using the nmcli command for the new network interface **enp0s8**.. You will assign the IP **172.10.10.120/24 with gateway 172.10.10.1**, and set it to **autoactivate at system reboots**. You will **deactivate and reactivate this interface** at the command prompt.

  * ```bash
    sudo nmcli con add type Ethernet ifname enp0s8 con-name enp0s8 ip4 172.10.10.120/24 gv4 172.10.10.1
    # The ONBOOT is automatically set to yes
    
    sudo nmcli c down enp0s8
    sudo nmcli c up enp0s8
    ```

---

### NTP - Network Time Protocol

* RHEL version 8 introduces a new implementation of NTP called Chrony.

* Chrony => UDP/123

* Chrony configuration file => ***/etc/chrony.conf***

* Chrony service runs as a daemon program called ***chronyd***. The chrony service has a command line program called ***chronyc***.

* **Configure NTP Client**

  * ```bash
    sudo dnf install -y chrony
    grep -E 'pool|server' /etc/chrony.conf | gep -v ^#
    pool 2.rhel.pool.ntp.org iburst
    # Just to check the current time pool in the configuration file
    
    sudo systemctl --nom enable chronyd
    chronyc sources
    chronyc tracking
    ```

---

### System Date and Time

* `timedatectl` => Gives information about time, date and NTP service
* `sudo timedatectl set-ntp false` => Disable the NTP service
* `sudo timedatectl set-time 2020-1-1`
* `sudo timedatectl set-time "2019-11-18 23:00"`
* `sudo date --set "2019-11-22 13:00"`
* `date` => Basic date and time information

---

### DNS

* Resolver configuration files
  * The ***/etc/resolv.conf*** file is the DNS resolver configuration file where information to support hostname lookups is defined.
  * Contains "domain", "search" and "nameserver"
* Name Resolution sources
  * The ***/etc/nsswitch.conf*** file directs the lookup utilities to the correct source to get hostname information.
* Dig
  * `dig (domaine_name)`
  * `dig -x (ip)` => Reverse search
* host
  * `host (domain_name)`
  * `host -v (ip)` => Reverse search
* nslookup
  * `nslookup (domain_name) (ip_of_dns_to_query)`
    * `nslookup redhat.com 8.8.8.8`
  * `nslookup (ip)` => Reverse search

---

### Hostname

* The hostname is stored in the `/etc/hostname` file.
* The hostname can be viewed by several commands such as :
  * `hostname`, `hostnamctl`, `uname` and `nmcli`
* Change hostname :
  * Modify ***/etc/hostname*** and`sudo systemctl restart systemd-hostnamed`
  * `sudo hostnamectl set-hostname (new_name)`

---

### Hosts Table

* For small, internal networks, the use of a local hosts table (the ***/etc/hosts file***) is also common. This table is used to maintain hostname to IP mapping for systems on the local network, allowing us to access a system by simply employing its hostname.

* Example :

  * ```bash
    192.168.0.110  server10.example.com  server 10
    192.168.0.120  server20.example.com  server 20
    # IP           FQDN                  ALIAS
    ```

