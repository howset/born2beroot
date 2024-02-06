# Evaluation Guide

- Compare signature.txt with the ```sha1sum *.vdi```

## Mandatory Part
### Project Overview
- How a VM works & - Purpose of VM
	- Like a computer that uses software instead of a physical computer to run programs
	- Virtualization uses software to simulate virtual hardware
	- runs its own operating system and functions separately from the other VMs
	- to run software that requires a different operating system, or to test applications in an isolated environment
- Why Debian
	- seems easier, already quite familiar to ubuntu and some other debian family
- Differences between Rocky and Debian
	- Rocky is a relatively new downstream release from RHEL, to respond the discontinuation of CentOS by Red Hat. So, maybe more fedora-like.
	- Debian follows a stable philosophy. Rocky since very close to fedora (also from redhat) maybe more bleeding-edge. Debian uses apt, rocky uses dnf.
	- Both fedora and debian are very popular due to strong support by the respective communities, which makes rocky also a very good candidate of choice.
- Difference between apt and aptitude
	- Both are package managers that use dpkg behind the scenes
	- Aptitude is a high-level package manager while APT is lower-level package manager which can be used by other higher-level package managers.
	- https://www.tecmint.com/difference-between-apt-and-aptitude/
	- https://www.fosslinux.com/43884/apt-vs-aptitude.htm
- What is AppArmor
	- a Linux kernel security module that allows the system administrator to restrict programs' capabilities with per-program profiles. (wiki)
	- supplements the traditional Unix discretionary access control (DAC) model by providing mandatory access control (MAC)
	- Before any syscall the kernel check with AppArmor (or SELinux) if the process is allowed to execute that command or access the file. By configuring them we can restrict the actions that subjects (processes) can perform on objects (files, IO, memory, Network ports, etc.)
	- AppArmor allows access by default; policies then restrict access to objects. SELinux restricts access by default; policies then allow access to objects.
- Script must work

### Simple setup
- No GUI
- login (must not be root)
- Check UFW (Uncomplicated Firewall) ```sudo ufw status```
- Check SSH (Secure Shell/Secure Socket Shell) ```sudo systemctl status ssh```
- Check OS ```cat /etc/os-release```

### User
- Check student login (hsetyamu) 
```bash
$ getent group sudo
$ getent group user42
```

- Create new user (evaluator's name): ```sudo adduser <username>```
- Explain password policy, refer to Setup.md
- Check password policy: ```sudo chage -l <username>```
- Create new group ("evaluating"): ```sudo groupadd <groupname>```
- Assign evaluator to evaluating: ```sudo usermod -aG <groupname> <username>```
- Check group: ```groups <username>```
- Password policy (/etc/login.defs) 
	- Password change & expiry
	- Adv: maintain security integrity of the system
	- Dis: frequent change, easy to forget.  
- Password policy (/etc/pam.d/common-password)
	- Password complexity
	- Adv: maintain security integrity of the system
	- Dis: complex password, easy to forget.
- PAM (Pluggable Authentication Module)
	- A suite of libraries that allows a Linux system administrator to configure methods to authenticate users. 

### Hostname and Partitions
- Check hostname (hsetyamu42)
- Modify hostname (reboot afterwards)
```bash
$ sudo hostnamectl set-hostname <new_hostname>	
or
$ sudo nano /etc/hostname
```
- Restore & reboot ```sudo hostnamectl set-hostname hsetyamu42```
- View partitions: ```lsblk```
- LVM (Logical Volume Manager) --> modern & flexible
	- Able to create logical volumes out of physical volumes, allowing dynamic resizing.
	- Can be moved between physical volumes.

### Sudo
- Check sudo is installed: ```dpkg -l | grep sudo```
- Assign evaluator to sudo: ```sudo usermod -aG sudo <username>```
- Sudo 
	- Manage the priviledge of users, what is allowed, what is accessible, and what is forbidden.
	- Sudoers file (/etc/sudoers) contains the configurations
	- "If requiretty is set, sudo will only run when the user is logged in to a real tty. When this flag is set, sudo can only be run from a login session and not via other means such as cron(8) or cgi-bin scripts"
	- Ex. ```nano /etc/sudoers``` would fail without sudo
- Check sudo log existence: ```ls /var/log/sudo```
- ```cat /var/log/sudo/sudo.log | wc -l```

### UFW
- Check installation: ```dpkg -l | grep ufw```
- Check function
```bash
$ sudo ufw status
$ sudo ufw allow 8080 # open port 8080
$ sudo ufw status numbered # get number
$ sudo ufw delete [number] # delete
```
- UFW
	- manage firewall
	- configure connections, which ports to open/close

### SSH
- Check installation: ```dpkg -l | grep ssh```
- Check function: ```sudo systemctl status ssh```
- SSH
	- a network protocol that gives users a secure way to access a system over an unsecured network.
- ```ssh <username>@localhost -p 4243```
- https://www.tecmint.com/disable-or-enable-ssh-root-login-and-limit-ssh-access-in-linux/

### Script
- Show code ```sudo cat /usr/local/bin/monitoring.sh```
- basically like neofetch
- Cron
	- program that runs commands or scripts automatically at a certain schedule.
- crontab
	- the configuration file
	- ```sudo crontab -u root -e```
- Stop cron: ```sudo /etc/init.d/cron stop``` or ```sudo cron stop```
- Start cron: ```sudo /etc/init.d/cron start``` or ```sudo cron start```

## Bonus
### Partitions
- ```$ lsblk```
- Refer to Bonus.md

### Wordpress setup

- Wordpress is a **web content management system**, written in **PHP** and paired with a **data base** (MySQL or MariaDB). To function, WordPress has to be installed on a **web server** (http, either apache or nginx or lighttpd)

- Requirements:
	- **PHP** version 7.4 or greater.
	- MySQL version 5.7 or greater OR **MariaDB** version 10.4 or greater.
	- HTTPS support (**Lighttpd**)

- Neither Apache2 nor NGINX are used. Check services by:
```bash
$ systemctl list-units --type=service # || sudo service --status-all || ls -l /etc/init.d/*
```
- Refer to Bonus.md
- Check on host browser http://127.0.0.1:8080/info.php
- Check on host browser http://127.0.0.1:8080
- Check on host browser http://127.0.0.1:8080/wp-admin/

### Free Choice
- vsftpd: ftp service
```bash
$ ftp 127.0.0.1 				# via terminal in guest, exit by ctrl + d or "bye"
						# user other than hsetyamu should fail
						# enter passwd
ftp> ls						# look around
ftp> cd files
ftp> get testfile.txt 				# transfer testfile.txt to local machine
ftp> put testfile.txt uploadtest.txt		# upload with a new name to test write permissions
ftp> ls
```
