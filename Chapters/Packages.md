### RPM commands

* The ***rpm*** utility is a low-level tool that can get information about the content of package files and installed packages. By default, it gets information from the local database of installed packages. However, you can use the ***-p*** option to specify that you want to get information about a downloaded package file. You might want to do this in order to inspect the contents of the package file before installing it.
* Package database => /var/lib/rpm
* **RPM Commands - Query**
  * `rpm -q` => (Query) Queries and display information about a specific package
  * `rpm -qa` => (Query All) List ALL installed packages
  * `rpm -qc` => (Config Files) List just the configuration files
  * `rpm -qd` => (Document Files) List just the documentation files
  * `rpm -qf (file_name)` => Finds out which package provides "file_name"
  * `rpm -qi` => (Information) => Finds information about a package
  * `rpm -ql` => (List) List all files included in a package
  * `rpm -qR` => (Requirements) List package requirements
* **RPM Commands - Installation**
  * `rpm -ivh (package_name)` => Installs a package
    * "i" = Install | "v" = verbose | "h" = shows progress
  * `rpm -Uvh (package_name)` => Upgrades a package or installs it if missing
  * `rpm -Fvh (package_name)` => Freshen a package (=Upgrade Only) - NOT USED
  * `rpm -ivh --replacepkgs (package_name)` => Overwrites Package
  * `rpm (package_name) -ve` => Removes a package
* RPM Commands - Extraction
  * `rpm2cpio (package_name) | cpio -imd`
    * "i" = All files | "d" = create a directory
  * `rpm -K (package_name)` => Validates the signature + package integrity
  * `rpmkeys --import (package_key)` => Import GPG key file
  * `rpm -q gpg-pubkey` => Displays imported GPG Keys

---

### Define Yum/DNF repo

* :warning: ***yum*** is simply a SOFT LINK to the new ***dnf*** utility.

* **Repo Definition** => ***/etc/yum.repos.d***

  * ```bash
    [repo_name]
    name=Test repo
    baseurl=file:///mnt/test # Path
    enabled=1
    gpgcheck=0
    ```

* `dnf repolist` => To check if new repo has correctly been added.

---

### YUM/DNF commands

* `dnf check-update`
* `dnf list` => `dnf list (package_name)` this version checks if a package is already installed
* `dnf install (package_name)`
* `dnf remove`
* `dnf upgrade`
* `dnf clean` => Clear cached data
* `dnf history` => Display previous yum/dnf activity (/var/lib/dnf/history)
* `dnf info (package_name)`
* `dnf provides (file_name)` => Search for a package that contains a specific file
* `dnf reinstall`
* `dnf repolist` => List enabled repositories
* `dnf repoquery` => Runs query on available packages
* `dnf search (string)` => Searches packages meta-data for the specific "string"
* `dnf group install/info/list/remove`
* `dnf module enable/disable/install/remove`