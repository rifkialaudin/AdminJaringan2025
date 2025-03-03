<h1 align=center>
    Chapter 4: Process Control
</h1>

terdiri dari dua bagian utama:

- **Address Space**: Sekumpulan halaman memori yang digunakan untuk menyimpan kode, data, dan stack dari proses.
- **Kernel Data Structures**: Struktur data dalam kernel yang menyimpan informasi tentang status proses, prioritas, sumber daya yang digunakan, dan lainnya

## Komponen dalam Proses

- **Thread**: bagian dari proses yang berbagi ruang alamat dan sumber daya yang sama. Contohnya, server web dapat memiliki banyak thread untuk menangani banyak permintaan secara bersamaan.
- **PID (Process ID)**: ID unik untuk setiap proses yang digunakan dalam berbagai sistem operasi.
- **PPID (Parent Process ID)**: ID dari proses induk yang membuat proses baru.
- **UID (User ID) & EUID (Effective User ID)**: UID menunjukkan siapa pemilik proses, sedangkan EUID menentukan hak akses proses terhadap sumber daya sistem.

## Siklus Hidup Proses 

Proses baru dibuat menggunakan sistem call fork(), yang membuat salinan dari proses induknya. Dalam sistem Linux modern, fork() dipanggil melalui clone(), yang mendukung fitur tambahan seperti thread.

Proses pertama yang dibuat oleh kernel saat sistem menyala adalah init atau systemd (PID 1), yang bertanggung jawab untuk menjalankan skrip startup dan mengelola proses lainnya.

### Sinyal

Digunakan untuk komunikasi antar proses, mengontrol proses (seperti menghentikan atau menjeda), dan memberi tahu proses tentang peristiwa tertentu. Ada sekitar 30 jenis signal, digunakan untuk berbagai tujuan

![Signals](https://liujunming.top/images/2018/12/71.png)

Beberapa sinyal penting:
- KILL: Menghentikan proses secara paksa.
- INT: Dikirim saat pengguna menekan <Control-C> untuk menginterupsi proses.
- TERM: Meminta proses untuk menghentikan eksekusi.
- HUP: Sering digunakan untuk meminta proses (seperti daemon) untuk restart.
- QUIT: Mirip dengan TERM, tetapi menghasilkan core dump jika tidak ditangkap.

**kill**: mengirim sinyal
Digunakan untuk mengirim sinyal ke proses berdasarkan PID (Process ID).

```bash
kill [-signal] pid
```
Contoh:
```bash
kill 1234  # Mengirim sinyal TERM (default) ke proses dengan PID 1234
kill -9 5678  # Mengirim sinyal KILL ke proses dengan PID 5678 (tidak bisa dicegah)
```
Penjelasan:
- kill 1234 → Mengirim sinyal TERM ke proses dengan PID 1234, meminta proses untuk berhenti dengan cara yang bersih. Namun, proses bisa menangkap dan mengabaikannya.
- kill -9 5678 → Mengirim sinyal KILL ke proses dengan PID 5678. Sinyal ini tidak bisa ditangkap atau diabaikan, sehingga proses pasti akan dihentikan secara paksa.

**killall**: Menghentikan semua proses yang memiliki nama yang sama.

Contoh:
```bash
killall firefox  # Menghentikan semua proses dengan nama "firefox"
```

**pkill**: Mirip dengan killall, tetapi lebih fleksibel karena mendukung pemfilteran berdasarkan pola atau pengguna.

```bash
pkill -u abdoufermat  # Menghentikan semua proses yang dijalankan oleh user 'abdoufermat'
```

## PS: Monitoring Processes

The **ps** command is the system administrator’s main tool for monitoring processes. Although versions of **ps** differ in their arguments and display, they all deliver essentially the same information.

**ps** can show the PID, UID, priority, and control terminal of processes. It also informs you how much memory a process is using, how much CPU time it has consumed, and what its current status is (running, stopped, sleeping, and so on).

You can obtain a useful overview of the system by running **ps aux**. The **a** option tells **ps** to show the processes of all users, and the **u** option tells it to provide detailed information about each process. The **x** option tells **ps** to show processes that are not associated with a terminal.

```bash
$ ps aux | head -8
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  22556  2584 ?        Ss   2019   0:02 /sbin/init
root         2  0.0  0.0      0     0 ?        S    2019   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        I<   2019   0:00 [rcu_gp]
root         4  0.0  0.0      0     0 ?        I<   2019   0:00 [rcu_par_gp]
root         6  0.0  0.0      0     0 ?        I<   2019   0:00 [kworker/0:0H-kblockd]
root         8  0.0  0.0      0     0 ?        I<   2019   0:00 [mm_percpu_wq]
root         9  0.0  0.0      0     0 ?        S    2019   0:00 [ksoftirqd/0]
```

![process-explanation](./data/process-explanation.png)

ANother useful set of arguments is **lax**, which gives more technical informations about the processes. **lax** is slightly faster than **aux** because it doesn't need to resolve user and group names.

```bash
$ ps lax | head -8
F   UID   PID  PPID PRI  NI    VSZ   RSS WCHAN  STAT TTY        TIME COMMAND
4     0     1     0  20   0  22556  2584 -      Ss   ?          0:02 /sbin/init
1     0     2     0  20   0      0     0 -      S    ?          0:00 [kthreadd]
1     0     3     2  20   0      0     0 -      I<   ?          0:00 [rcu_gp]
1     0     4     2  20   0      0     0 -      I<   ?          0:00 [rcu_par_gp]
1     0     6     2  20   0      0     0 -      I<   ?          0:00 [kworker/0:0H-kblockd]
1     0     8     2  20   0      0     0 -      I<   ?          0:00 [mm_percpu_wq]
1     0     9     2  20   0      0     0 -      S    ?          0:00 [ksoftirqd/0]
```

To look for a specific process, you can use **grep** to filter the output of **ps**.

```bash
$ ps aux | grep -v grep | grep firefox
```

We can determine the PID of a process by using **pgrep**.

```bash
$ pgrep firefox
```

or **pidof**.

```bash
$ pidof /usr/bin/firefox
```

## Interactive monitoring with top

The **top** command provides a dynamic real-time view of a running system. It can display system summary information as well as a list of processes or threads currently being managed by the Linux kernel. The types of system summary information shown and the types, order, and size of information displayed for processes are all user configurable and that configuration can be made persistent across restarts.

By default, the display update every 1-2 seconds, depending on the system.

There's also a **htop** command, which is an interactive process viewer for Unix systems. It is a text-mode application (for console or X terminals) and requires ncurses. It is similar to top, but allows you to scroll vertically and horizontally, so you can see all the processes running on the system, along with their full command lines. **htop** also has a better user interface and more options for operations.

## Nice and renice: changing process priority

The **niceness** is a numeric hint to the kernel about how the process should be treated in relation to other processes contending for the CPU.

A high niceness means a low priority for your process: you are going to be nice. A low or negative value means high priority: you are not very nice!

The range of allowable niceness values varies among systems. In Linux the range is -20 to +19, and in FreeBSD it's -20 to +20.

A low priority process is one that is not very important. It will get less CPU time than a high priority process. A high priority process is one that is important and should be given more CPU time than a low priority process.

For example, if you are running a CPU-intensive job that you want to run in the background, you can start it with a high niceness value. This will allow other processes to run without being slowed down by your job.

The **nice** command is used to start a process with a given niceness value. The syntax is:

```bash
nice -n nice_val [command]

# Example
nice -n 10 sh infinite.sh &
```

The **renice** command is used to change the niceness value of a running process. The syntax is:

```bash
renice -n nice_val -p pid

# Example
renice -n 10 -p 1234
```

**The priority value** is the process’s actual priority which is used by the Linux kernel to schedule a task.
In Linux system priorities are 0 to 139 in which 0 to 99 for real-time and 100 to 139 for users.

The relation between nice value and priority is as follows:

> priority_value = 20 + nice_value

The default nice value is 0. The lower the nice value, the higher the priority of the process.

## The /proc filesystem

The Linux versions of **ps** and **top** read their process status information from the **/proc** directory, a pseudo-filesystem in which the kernel exposes a variety of interesting information about the system's state.

Despite the name, **/proc** contains other information than just processes (stats generated by the system, etc).

Processes are represented by directories in **/proc**, and each process has a directory named after its PID. The **/proc** directory contains a variety of files that provide information about the process, such as the command line, environment variables, file descriptors, and so on.

![process-information](./data/process-information.png)

## Strace and truss

To figure out what a process is doing, you can use **strace** on Linux or **truss** on FreeBSD. These commands trace system calls and signals. They can be used to debug a program or to understand what a program is doing.

For example, the following log was produced by strace run against an active copy of top (which was running as PID 5810):

```bash
$ strace -p 5810

gettimeofday({1197646605,  123456}, {300, 0}) = 0
open("/proc", O_RDONLY|O_NONBLOCK|O_LARGEFILE|O_DIRECTORY) = 7
fstat64(7, {st_mode=S_IFDIR|0555, st_size=0, ...}) = 0
fcntl64(7, F_SETFD, FD_CLOEXEC)          = 0
getdents64(7, /* 3 entries */, 32768)   = 72
getdents64(7, /* 0 entries */, 32768)   = 0
stat64("/proc/1", {st_mode=S_IFDIR|0555, st_size=0, ...}) = 0
open("/proc/1/stat", O_RDONLY)           = 8
read(8, "1 (init) S 0 1 1 0 -1 4202752"..., 1023) = 168
close(8)                                = 0

[...]
```

**top** starts by checking the current time. It then opens and stats the **/proc** directory, and reads the **/proc/1/stat** file to get information about the **init** process.

## Runaway processes

Occasionally a process will stop responding to the system and run wild. These processes ignore their scheduling priority and insist on taking up 100% of the CPU. Because other processes can only get limited access to the CPU, the machine begins to run very slowly. This is called a runaway process.

The **kill** command can be used to terminate a runaway process. If the process is not responding to a TERM signal, you can use the KILL signal to terminate it.

```bash
kill -9 pid

or

kill -KILL pid
```

We can investigate the cause of the runaway process by using **strace** or **truss**. Runaway processes that produce output can fill up an entire filesystem. 

You can run a **df -h** to check the filesystem usage. If the filesystem is full, you can use the **du** command to find the largest files and directories.


You can also use the **lsof** command to find out which files are open by the runaway process.

```bash
lsof -p pid
```

## Periodic processes

### cron: schedule command

The cron (crond on RedHat: yeah weirddos!!) daemon is the traditional tool for running commands on a predtermined schedule. It starts when the system boots and runs as long as the system is up.

**cron** reads configuration files containing lists of command lines and times at which they are to be invoked. The command lines are executed by **sh**, so almost anything you can do by hand from the shell can also be done with **cron**.

A cron configuration file is called a “crontab,” short for “cron table.” Crontabs for individual users are stored under **/var/spool/cron** (Linux) or **/var/cron/tabs** (FreeBSD).

### format of crontab

A crontab file has five fields for specifying day, date and time followed by the command to be run at that interval.

```bash
*     *     *     *     *  command to be executed
-     -     -     -     -
|     |     |     |     |
|     |     |     |     +----- day of week (0 - 6) (Sunday=0)
|     |     |     +------- month (1 - 12)
|     |     +--------- day of month (1 - 31)
|     +----------- hour (0 - 23)
+------------- min (0 - 59)
```

Some examples:

```bash
# Run a command at 2:30am every day
30 2 * * * command

# Run a command at 10:30pm on the 1st of every month
30 22 1 * * command

# Run a Python script every 1st of the month at 2:30am
30 2 1 * * /usr/bin/python3 /path/to/script.py
```

The following  schedule: 0,30 * 13 * 5 means that the command will be executed at 0 and 30 minutes past the 13th hour on Friday. If you want to run a command every 30 minutes, you can use the following schedule: */30 * * * *. 

**crontab management**

The **crontab** command is used to create, modify, and delete crontabs. The **-e** option is used to edit the crontab file, the **-l** option is used to list the crontab file, and the **-r** option is used to remove the crontab file.

### Systemd timer

A systemd timer is a unit configuration file whose name ends in **.timer**. systemd timers can be used as an alternative to cron jobs. They are more flexible and more powerful than cron jobs.

A timer unit is activated by a corresponding service unit. The service unit is triggered by the timer unit at the time specified in the timer unit. The timer unit can also be activated by the system boot or by an event.

The **systemctl** command is used to manage systemd units. The **list-timers** option is used to list the active timers.

```bash
$ systemctl list-timers

NEXT                         LEFT          LAST                         PASSED       UNIT                         ACTIVATES
Fri 2021-10-15 00:00:00 UTC  1h 1min left Thu 2021-10-14 00:00:00 UTC  22h ago      logrotate.timer              logrotate.service

1 timers listed.
```

In the example above, the **logrotate.timer** unit is scheduled to activate the **logrotate.service** unit at midnight every day.

Here's what the **logrotate.timer** unit looks like:

```bash
$ cat /usr/lib/systemd/system/logrotate.timer

[Unit]
Description=Daily rotation of log files
Documentation=man:logrotate(8) man:logrotate.conf(5)

[Timer]
OnCalendar=daily
AccuracySec=1h
Persistent=true

[Install]
WantedBy=timers.target

```

The **OnCalendar** option is used to specify when the timer should activate the service. The **AccuracySec** option is used to specify the accuracy of the timer. The **Persistent** option is used to specify whether the timer should catch up on missed runs.


### Common use for scheduled tasks

**Sending mail**

You can automatically email the output of a daily report or the results of a command execution using **cron** or **systemd** timers.

For example:
    
```bash
30 4 25 * * /usr/bin/mail -s "Monthly report"
    abdou@admin.com%Receive the monthly report for the month of July!%%Sincerely,%cron%
```

**Cleaning up a filesystem**

You can use **cron** or **systemd** timers to run a script that cleans up a filesystem. For example, you can use a script to purge the contents of trash directory every day at midnight.

```bash
0 0 * * * /usr/bin/find /home/abdou/.local/share/Trash/files -mtime +30 -exec /bin/rm -f {} \;
```

**Rotating a log file**

To rotate a log file means to divide it into segments by size or by date, keeping several older versions of the log available at all times. Since log rotation is a recurrent and regularly occurring event, it’s an ideal task to be scheduled.

**Running batch jobs**

Some long-running calculations are best run as batch jobs. For example, messages can accumulate in a queue or database. You can use a cron job to process all the queued messages at onces as an ETL (Extract, Transform, Load) to another location, such as a data warehouse.

**Backing up and mirroring**

You can use a scheduled task to automatically back up a directory to a remote system.
Mirrors are byte-for-byte copies of a filesystem or directories that are hosted on another system. They can be used as a form of backup or as a way to distribute files across multiple systems. 
You can use a periodic execution of **rsync** to keep the mirror up to date.
