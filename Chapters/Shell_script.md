### Shell and Variables

* **Bash Shell**

  * "$" = Normal Users
  * "#" = root user
  * Location => `/bin/bash`

* **System-wide Shell Startup Files**

  * `/etc/bashrc` => Defines functions, aliases, sets umasks,...
  * `/etc/profile` => Sets common environment variables such as PATH, MAIL,...
  * `/etc/profile.d` => Contains scripts that are executed in the /etc/profile file

* **User Shell Startup Files**

  * bashrc
  * bash_profile
  * gnome2/

* **Types of variables**

  * Local variable => Private to the shell
  * Environment variable => Accessible to programs as well as any sub-programs that it spawns during its lifecycle.

* **Environment variables** :

  * To list ALL Env Var => `printenv` or `env`
  * `DISPLAY` => Stores hostname/IP for GUI sessions
  * `HISTFILE `=> File used to store the history of executed commands
    * Default file is ".bash_history"
  * `HISTSIZE `=> Max size of "HISTFILE"
  * `HOME` => Home directory path
  * `LOGNAME `=> Login name
  * `MAIL `=> Paht to user mail directory
  * `PATH `=> This variable removes the need to specify the absolute path of a command to run it.0
  * `PPID `=> Holds the value of the Parent Process ID
  * `PS1 `=> Primary command prompt
  * `PS2 `=> Secondary command prompt
  * `PWD `=> Present Working Directory
  * `SHELL `=> Path to current shell
  * `TERM `=> Stores the terminal type value
  * `UID `=> Stores the logged-in user's UID
  * `USER `=> Name of the logged-in user

* **Local Variables** :

  * ```bash
    VR1=Test # Defines a local var "VR1" with value "Test"
    echo $VR1 # TEST
    export VR1 # VR1 is now an environment variable
    
    export VR1="I love Linux!" # One Liner (creates the variables and exports it)
    ```

  * ```bash
    [test@fedora ~]$ echo $PS1
    # [\u@\h \W]\$
    # \u = Logged-in username | \W = Current working directory
    # \h = Hostname of the system | \$ = End of command prompt
    
    # This can be changed :
    [test@fedora ~]$ export PS1="<$LOGNAME on $(hostname) in \$PWD>"
    # <test on fedora in /home/test>
    ```

---

### Shell tricks

* `CTRL+a` or `Home` => Moves cursor to beginning
* `CTRL+e` or `End` => Moves cursor to end
* `CTRL + u` => Erase everything you've typed on the command line

---

### Alias

* `alias ls="ls -al"`
* User Defined alias => "/home/user/.bashrc" (in ubuntu => .bashrc_alias)
* Global Alias => "/etc/bashrc" (in ubuntu => /etc/bash.bashrc)
* To run a command WITHOUT using it's alias => `\(command)`
* `unalias test` => Removes alias "test"

---

### Shell Scripting

* You can store your scripts in the ***/usr/local/bin*** directory, which is included in the PATH of all users by default.

* **Script 1 : Displaying System Information**

  * ```bash
    #!/bin/bash
    echo "Display Basic System Information"
    echo "================================"
    echo
    echo "The hostname, hardware and OS information is:"
    /usr/bin/hostnamectl
    echo
    echo "The following users are currently logged in:"
    /usr/bin/who
    ```

* `bash -x script.sh` => To debug the script

* **Script 2 : Using Local Variables**

  * ```bash
    SYSNAME=server10.example.com
    echo "The hostname of this system is $SYSNAME"
    ```

* **Script 3 : Using Pre-Defined Environment Variables**

  * ```bash
    echo $SHELL
    echo "I am logged in as $LOGNAME"
    ```

* **Script 4 : Using Command Substitution**

  * ```bash
    SYSNAME=$(hostname)
    KERNVER=`uname -r`
    echo "The hostname is $SYSNAME"
    echo "The kernel version is $KERNVER"
    ```

* Shell parameters

  * `$0` => Script itself
  * `$@` or `$*` => Count number of arguments given
  * `$#` => All arguments
  * `$$` => PID of the process
  * `$1` or `$2` => argument 1 or argument 2
  * `$?` => Exit code (0 is ok)

* **Script 5 : Using Special and Positional Parameters**

  * ```bash
    echo "There are $# arguments specified at the command line"
    echo "The arguments supplied are: $*"
    echo "The first argument is: $1"
    echo "The process ID of the script is: $$"
    ```

* **Script 6 : If-then-fi Construct**

  * ```bash
    #!/bin/bash
    if [ $# -ne 2 ]
    then
    	echo "Error: Invalid number of arguments supplied"
    	echo "Usage: $0 source_file destination_file"
    	exit 2
    fi
    ```

* **Script 7 : If-then-else-fi Construct**

  * ```bash
    #!/bin/bash
    if [ $1 -gt 0 ]
    then
    	echo "$1 is a positive number"
    else
    	echo "$1 is a negative number"
    fi
    ```

* **Script 8 : If-then-elif-fi Construct**

  * ```bash
    #!/bin/bash
    if [ "$1" = ex200 ]
    then
    	echo "RHCSA"
    elif [ "$1" = ex294 ]
    then
    	echo "RHCE"
    else
    	echo "Usage: Acceptable values are ex200 and ex294"
    fi
    ```

* **Script 9 : Print Alphabets using for loop**

  * ```bash
    #!/bin/bash
    COUNT=0
    for LETTER in {A..Z}
    do
    	COUNT=`/usr/bin/expr $COUNT +1`
    	echo "Letter $COUNT is [$LETTER]"
    done
    ```

---

### Test Conditions

* **Operation on numbers**
  * `num1 (-eq/-ne) num2` => Equal / not equal
  * `num1 (-lt/-gt) num2` => Less than / Greater than
  * `num1 (-le/-ge) num2` => Less or equal / Greater or equal
* **Operation on characters**
  * `string1 (=/!=) string2` => Test if string is equal/not equal.
  * `-l string1` or `-z string1` => Test if string is empty.
* **Operation on file**
  * `-b (-c) (-d) file1` => File is a block (-b) or character (-c) or directory (-d)
  * `-e file1` => Test if file exists
  * `(-r/-w/-x) file1` => Test if file is readable, writable or executable
  * `file1 (-nt/-ot) file1` => Newer than / older than
* **Logical operator**
  * `!` => NOT
  * `-a` or `&&` => AND
  * `-o` or `||` => OR
