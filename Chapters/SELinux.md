### SELinux

**Security Enhanced Linux - Principle**

*  If the HTTP service process is compromised, the attacker can only damage the files the hacked process will have access to, and not the other processes running on the system, or the objects the other processes will have access to.

**Terminology**

* **Subject**
  * A subject is any user or process that accesses an object.
  * Examples include **system_u** for the SELinux system user, and **unconfined_u** for subjects that are not bound by the SELinux policy.
* Object
  * An object is a resource, such as a file, directory, hardware device, network interface/connection, port, pipe, or socket, that a subject accesses.
  * Examples include **object_r** for general objects, **system_r** for system-owned objects, and **unconfined_r** for objects that are not bound by the SELinux policy.
* Access
  * An access is an action performed by the subject on an object.
* Policy
  * A policy is a defined ruleset that is enforced system-wide, and is used to analyze security attributes assigned to subjects and objects.
  * The default behavior of SELinux in the absence of a rule is to **deny the access**. Two standard preconfigured policies are ***targeted*** and ***mls*** with **targeted being the default.**
  * Targeted policy
    * The targeted policy dictates that any process that is targeted runs in a confined domain, and any process that is not targeted runs in an unconfined domain.
    * Example : logged-in users are in the ***unconfined*** domain and the httpd process is in a ***confined*** domain.
  * MLS policy
    * The mls policy places tight security controls at deeper levels.
    * A third preconfigured policy called minimum is a light version of the targeted policy, and it is designed to protect only selected processes.
* Context
  * A context (a.k.a. label) is a tag to store security attributes for subjects and objects.
  * In SELinux, every subject and object has a context assigned, which consists of a SELinux user, role, type (or domain), and sensitivity level.
* Labeling
  * Labeling is the mapping of files with their stored contexts.
* SELinux User
  * SELinux policy has several predefined SELinux user identities that are authorized for a particular set of roles.
  * This controls what roles and levels a process (with a particular SELinux user identity) can enter. 
* Role
  *  It classifies who (subject) is allowed to access what (domains or types).
  * Each subject has an associated role to ensure that the system and user processes are separated.
* Type Enforcement
  * Type enforcement (TE) identifies and limits a subject’s ability to access domains for processes, and types for files. 
* Type and Domain
  * A type is an attribute of type enforcement. It is a group of objects based on uniformity in their security requirements. Objects such as files and directories with common security requirements, are grouped within a specific type.
  * A domain determines the type of access that a process has. Processes with common security requirements are grouped within a specific domain type, and they run confined within that domain.
* Level
  * A level is an attribute of Multi-Level Security (MLS) and Multi-Category Security (MCS). It is a pair of sensitivity:category values that defines the level of security in the context.

---

### SELinux Contexts for Users

* ```bash
  id -Z # Get context for current user
  unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
  ```

* `unconfined_u` => Means that there's no SELinux restrictions placed on this user.

---

### SELinux Contexts for Processes

* ```bash
  ps -eZ # Get context for processes
  system_u:system_r:init_t:s0
  ```

---

### SELinux Contexts for Files

* ```bash
  ls -lZ /etc/passwd
  system_u:object_r:passwd_file_t:s0
  ```

---

### SELinux Contexts for Ports

* ```bash
  sudo semanage port -l
  SELinux Port Type | Proto | Port Number
  chronyd_port_t      udp     323
  dns_port_t          tcp     53, 853
  (...)
  ```

---

### Copying, Moving and Archiving Files with SELinux Contexts

* **If a file is copied to a different directory, the destination file will receive the destination directory’s context**, unless the `--preserve=context` switch is specified with the cp command to retain the source file’s original context.
* **If a copy operation overwrites the destination file in the same or different directory, the file being copied will receive the context of the overwritten file**, unless the --preserve=context switch is specified with the cp command to preserve the source file’s original context.
* If a file is archived with the tar command, use the `--selinux` option to preserve the context.

---

### Domain Transitioning

* SELinux allows a process running in one domain to enter another domain to execute an application that is restricted to run in that domain only, provided a rule exists in the policy to support such transition. SELinux defines a permission setting called entrypoint in its policy to control processes that can transition into another domain.
* To understand how this works, a basic example is provided below that shows what happens when a Linux user attempts to change their password using the `/usr/bin/passwd` command.
* The passwd command requires access to the `/etc/shadow` file in order to modify a user password. The shadow file has a different type set on it (shadow_t).
* The SELinux policy has rules that specifically allow processes running in domain `passwd_t` to read and modify the files with type `shadow_t`, and allow them entrypoint permission into domain `passwd_exec_t`. This rule enables the user’s shell process executing the `passwd` command to switch into the `passwd_t` domain and update the shadow file.

---

### SELiux Booleans

* Booleans activate or deactivate certain rule in the SELinux policy immediately and without the need to recompile or reload the policy.
* Boolean values are stored in virtual files located in the ***/sys/fs/selinux/booleans*** directory. 
* `man -K (selinux_boolean)` => To view man page for SELinux boolean.
* Temporary changes are stored as a “1” or “0” in the corresponding Boolean file in the `/sys/fs/selinux/booleans` directory and permanent changes are saved in the policy database.

---

### SELinux Administration

* Managing SELinux involves plentiful tasks, including controlling the activation mode, checking operational status, setting security contexts on subjects and objects, and switching Boolean values.
* Packages :
  * `libselinux-utils`
    * `getenforce` / `setenforce` and `getsebool`.
  * `policycoreutils`
    * `sestatus`, `setsebool` and `restorecon`.
  * `policycoreutils-python-utils`
    * `semanage`
  * `setools-console`
    * `seinfo` and `sesearch`

* > For viewing alerts and debugging SELinux issues, a graphical tool called **SELinux Alert Browser** is available, which is part of the **setroubleshoot-server** package.

---

### Management Commands

* **Mode Management**
  * `getenforce` / `setenforce` => Display/switch mode of operation.
  * `sestatus` => Shows runtime status and boolean values.
* **Context Management**
  * `chcon` => Changes context on file (temp)
  * `restorecon` => Restore default context
  * `semanage` => Changes context on files (permanent)
* **Policy Management**
  * `seinfo` => Provides information on policy components.
  * `semanage` => Manages policy database.
  * `sesearch` => Searches rules in the policy database.
* **Boolean Management**
  * `getsebool` => Displays booleans and their current settings
  * `setsebool` => Modifies booleans (temp)
  * `semanage` => Modifies booleans (perm)
* **Troubleshooting**
  * `sealert` => Graphical troubleshooting tool

---

### SELinux Operational State

* One of the key configuration files that controls the SELinux operational state, and sets its default type is the config file located in the ***/etc/selinux*** directory.

* ```bash
  cat /etc/selinux/config
  (...)
  SELINUX=enforcing # enforcing=ON, permissive=LOG only and disabled=OFF
  (...)
  SELINUXTYPE=targeted # Could also be mls (Multi Level Security protection)
  ```

* `sudo setenforce permisive` => Temp way of changing operational state of SELinux.

---

### Exercices

* **Modify SELinux File Context**

  * In this exercise, you will create a directory **sedir1 under /tmp** and a **file sefile1 under sedir1**. You will check the context on the directory and file. **You will change the SELinux user and type to user_u and public_content_t on both and verify.**

  * ```bash
    cd /temp
    mkdir sedir1
    touch sedir1/sefile1
    
    ls -ldZ sedir1
    ls -lZ sedir1/sefile1
    
    sudo chcon -vu user_u -t public_content_t sedir1 -R
    ```

* **Add and Apply File Context**

  * In this exercise, you will **add the current context on sedir1 to the SELinux policy database** to ensure a relabeling will not reset it to its previous value. Next, you will **change the context on the directory to some random values**. **You will restore the default context from the policy database** back to the directory **recursively**.

  * ```bash
    sudo semanage fcontext -a -s user_u -t public_content_t '/tmp/sedir1(/.*)?'
    sudo semanage fcontext -Cl | grep sedir
    sudo chcon -vu staff_u -t etc_t sedir1 -R
    sudo restorecon -Rv sedir1
    
    
    # '/tmp/sedir1(/.*)?' => Regex to include all files and subdir
    ```

* **Add and Delete Network Ports**

  * In this exercise, you will add a non-standard network port **8010** to the SELinux policy database for the httpd service and confirm the addition. You will then remove the port from the policy and verify the deletion.

  * ```bash
    sudo semanage port -l |grep ^http_port
    sudo semanage port -at http_port_t -p tcp 8010
    sudo semanage port -d -p tcp 8010
    ```

* **View and Toggle SELinux Boolean Values**

  * In this exercise, you will display the current state of the Boolean ***nfs_export_all_rw***. You will toggle its value temporarily, and reboot the system. You will flip its value persistently after the system has been back up.

  * ```bash
    sudo getsebool -a | grep nfs_export_all_rw
    # or sudo semanage boolean -l | grep nfs_export_all_rw
    sudo setsebool nfs_export_all_rw 0 # Temp
    sudo setsebool -P nfs_export_all_rw off # Permanent
    ```

---

### Monitoring and Analyzing SELinux Violations

* It writes the alerts to the ***/var/log/audit/audit.log*** file if the **auditd** daemon is running, or to the ***/var/log/messages*** file via the **rsyslog** daemon in the absence of **auditd**. 
* You can also use the **sealert** command to analyze (-a) all AVC records in the audit.log file. This command produces a formatted report with all relevant details.
