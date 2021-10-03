# Week-5-Submission-File-Archiving-and-Logging-Data

### Step 1: Create, Extract, Compress, and Manage tar Backup Archives

Command to extract the TarDocs.tar archive to the current directory: tar xvvf TarDocs.tar

Command to create the Javaless_Doc.tar archive from the TarDocs/ directory, while excluding the TarDocs/Documents/Java directory: tar cvvf Javaless_Docs.tar --exclude=TarDocs/Documents/Java/TarDocs

Command to ensure Java/ is not in the new Javaless_Docs.tar archive: tar tvvf Javaless_Docs.tar | grep "Java"
Bonus

Command to create an incremental archive called logs_backup_tar.gz with only changed files to snapshot.file for the /var/log directory:  sudo tar czvvf logs_backup.tar.gz --listed-incremental = snapshot.snar /var/log 

### Critical Analysis Question

Why wouldn't you use the options -x and -c at the same time with tar? --create (-c) - would create a new tar archive and --extract (-x) would extract the entire archive or one or more files from an archive. Linux would only allow you to create an archive and then extract it’s contents afterwards but not at the same time.

### Step 2: Create, Manage, and Automate Cron Jobs

Cron job for backing up the /var/log/auth.log file: 0 6 * * 3 tar czf /auth_backup.tgz /var/log/auth.log

### Step 3: Write Basic Bash Scripts

Brace expansion command to create the four subdirectories: mkdir -p /home/sysadmin/backups/{freemem,diskuse,openlist,freedisk}

Paste your system.sh script edits below:

 #!/bin/bash
Solution script contents: free -h > ~/backups/freemem/free_mem.txt 
du -h > ~/backups/diskuse/disk_usage.txt
lsof > ~/backups/openlist/open_list.txt 
df -h > ~/backups/freedisk/free_disk.txt

Command to make the system.sh script executable: sudo chmod +x system.sh

### Optional

Commands to test the script and confirm its execution: sudo ./system.sh
cd backups/diskuse, ls, cat disk_usage.txt
cd backups/freemem, ls, cat free_mem.txt
cd backups/freedisk, ls, cat free_disk.txt
cd backups/openlist, ls, cat open_list.txt

### Bonus

Command to copy system to system-wide cron directory: sudo mv system.sh /etc/cron.weekly/

### Step 4. Manage Log File Sizes

1. Run sudo nano /etc/logrotate.conf to edit the logrotate configuration file.

Add your config file edits below: sudo nano /etc/logrotate.conf
# see "man logrotate" for details
# rotate log files weekly
# weekly

# use the syslog group by default, since this is the owning group
# of /var/log/syslog.
# su root syslog

# keep 7 weeks worth of backlogs
# rotate 7


# create new (empty) log files after rotating old ones
# create


# uncomment this if you want your log files compressed
# delaycompress


#Do not rotate empty logs
notifempty


# packages drop log rotation information into this directory
# include /etc/logrotate.d

2. Configure a log rotation scheme that backs up authentication messages to the /var/log/auth.log [Your logrotate scheme edits here]

cd /etc/logrotate.d
cd nano auth
#!/bin/bash
/var/log/auth.log {
    rotate 7
    weekly
    notifempty
    compress
    delaycompress
    missingok
}

### Bonus: Check for Policy and File Violations

1. Command to verify auditd is active: systemctl status auditd (had to install program via sudo apt install auditd -y command)

2. Command to set number of retained logs and maximum log file size: sudo nano /etc/audit/auditd.conf

Add the edits made to the configuration file below:
- max_log_file = 35
- num_logs = 7

3. Command using auditd to set rules for /etc/shadow, /etc/passwd and /var/log/auth.log:
Enter the command sudo nano /etc/audit/rules.d/audit.rules to set rules that watch the following paths:
Add the edits made to the rules file below: -w /etc/shadow -p wra -k hashpass_audit, -w /etc/passwd -p wra -k userpass_audit, - w /var/log/auth.log -p wra -k authlog_audit

4. Command to restart auditd: /sbin/service auditd

5. Command to list all auditd rules: command auditctl -l

6. Command to produce an audit report: aureport

7. Create a user with sudo useradd attacker and produce an audit report that lists account modifications: sudo useradd attacker and then aureport -m

8. Command to use auditd to watch /var/log/cron: auditctl -w /var/log/cron -p warx -k cron_file_change

9. Command to verify auditd rules: auditctl -s

### Bonus (Research Activity): Perform Various Log Filtering Techniques

1. Command that performs a log search that returns all messages, with priorities from emergency to error, since the current system boot.
journalctl -b -1  -p "emerg".."crit"

2. Command to check the disk usage of the system journal unit since the most recent boot: journalctl --disk-usage

3. Command to remove all archived journal files except the most recent two: sudo journalctl --vacuum-files=2

4. Command to filter all log messages with priority levels between zero and two, and save output to /home/sysadmin/Priority_High.txt: ? journalctl -p 0..2 > ~/home/sysadmin/Priority_High.txt

5. Command to automate the last command in a daily cronjob. Add the edits made to the crontab file below: crontab -e followed by 30 2 * * * journalctl -p 0..2 > ~/home/sysadmin/Priority_High.txt
