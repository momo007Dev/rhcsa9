### Process States

* Daemons = Background processes that are critical to system operation.
* Process = It's a running instance of a launched executable program.
* **Process states** :
  * **Running/Runnable (R)** :
    * The process is running or waiting for its turn.
  * **Interruptable_sleep (S)** :
    * The process is waiting for some condition :
      * Hardware request, system resource access or ***signal***.
      * When the condition is met, the process goes back to Running/Runnable.
  * **Uninterruptable_sleep (D)** :
    * Same as "Interruptable_sleep" but DOES NOT respond to ***signals***.
  * **Stopped (T)** :
    * The process is stopped and is waiting for a signal.
  * **Zombie (Z)** :
    * A zombie process exists in the process table alongside other process entries, but it does not take any resources.
    * Its entry is retained until its parent process permits it to die.

---

### Process tools

* PS = Process State

* TOP = Table of Processes

* ```bash
  ps -ef
  UID  PID PPID C STIME  TTY TIME      CMD
  root 1   0    0 sep14  ?   00:00:06  /usr/lib/...
  # UID => User ID/Name of the process owner
  # C => CPU utilization for the process
  # STIME => Process start date/time
  # TTY => "?" = Deamon process
  # CMD => The command or program name
  ```

* The "ps" command can be customized :

  * `ps -o comm,pid,ppid,user` => COMMAND PID PID USER
  * `ps -c sshd` => List ALL process that match the specified command name
  * `ps -U user1` => List process by user
  * `ps -G root` => List process by group

* > PS : **pidof** ssh or **pgrep** ssh returns the PID of the command.

---

### Process Niceness and Priority

* -20 = Highest priority
* +20 = Lowest priority
* 0 = Default priority
* `nice` is used to start a program with a different priority.
* `renice` is used to CHANGE a program with a different priority.
* Examples :
  * `nice -n 2 top`
  * `sudo nice -n -10 top`
  * `sudo renice -n -5 $(pidof top)`

---

### Signals

| Signal Number | Name    | Definition        | Purpose                                                      |
| ------------- | ------- | ----------------- | ------------------------------------------------------------ |
| 1             | SIGHUP  | Hangup            | Request process to reload                                    |
| 2             | SIGINT  | Keyboard interupt | Causes program to terminate. Can be blocked. (`Ctrl+c`)      |
| 3             | SIGQUIT | Keyboard quit     | Same as SIGINT but producs a process dump at termination (`Ctrl+\`) |
| 9             | SIGKILL | KILL UNBLOCKABLE  | Cause program to stop IMMEDIATLY. **Can NOT be stopped**.    |
| 15 (default)  | SIGTERM | Terminate         | SAME as SIGKILL, but can be stopped. It's the "polite" way of terminating a program. |
| 18            | SIGCONT | Continue          | Always resumes the process. **Can NOT be blocked**.          |
| 19            | SIGSTOP | Stop              | Suspends the process. **Can NOT be blocked**.                |
| 20            | SIGTSTP | Keyboard stop     | SAME as SIGSTOP, but can be blocked (`Ctrl+z`)               |

* `sudo pkill crond`
* `sudo kill $(pidof crond)`
* `sudo pkill -9 crond)` or `sudo pkill -s SIGKILL crond`
* `sudo killall crond`

---

### Job Scheduling

* One time jobs = "at"

* Recurrent jobs = "cron"

* User Access :

  * By default EVERYONE is allowed to schedule jobs.
  * This can be changed to restrict users.
  * Files : at.allow, at.deny, cron.allow, cron.deny.
  * By default, deny files exists but are empty and allow files does NOT exist.

* **The "at" command**

  * All submitted jobs are spooled in the `/var/spool/at` directory and executed by the atd daemon at the specified time.
  * Commands :
    * `at -l` => List all planned jobs
    * `at -c (job_number)` => Shows command that will run
    * `atrm (job_number)` => Removes a specific job
    * `at 1:15am ` | `at noon` | `at 23:45` | `at 17:05 tomorow` | `at now + 5 hours` | `at 3:00 10/15/20`
    * `echo "date >> /home/student/myjob.txt" | at now + 3min`

* **Contab**

  * Crontables = contab files

  * Conrtables are located in the /var/spool/cron directory

  * Each authorized user with a scheduled job has a file matching their login name in this directory

    * Example : user 1 will have /var/spool/cron/user1

  * System crontables

    * /etc/crontab
    * /etc/cron.d
    * Logs in /var/log/cron file

  * Commands

    * `crontab -e` => Edit

    * `crontab -l` => List the jobs for the current user

    * `crontab -r` => Removes all jobs for the current user

    * > **root** can manage jobs for other users with the **crontab -u** command

  * Crontab file structure

    * ```bash
      # Example of job definition:
      # .---------------- minute (0 - 59)
      # | .------------- hour (0 - 23)
      # | | .---------- day of month (1 - 31)
      # | | | .------- month (1 - 12) OR jan,feb,mar,apr ...
      # | | | | .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue ...
      # | | | | |
      # * * * * * user-name command to be executed
      ```

    * `crontab -e` and then `21 16 * 10 * echo "Crontab test"`

      * 16h21 every day of the month, in October, every day of the week.

---

### Anacron

It's a service that runs after every system reboot. It checks for any "cron" and "at" jobs that were scheduled for execution during the time the system was down and were missed as a result.