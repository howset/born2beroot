# Setup

## User
```
$ su - 					# the - is important!!
$ apt update && apt upgrade -y
$ apt install sudo
$ usermod -aG sudo hsetyamu		# Check with getent group sudo
$ sudo nano /etc/sudoers		# User privilege specification 	
					# add the line hsetyamu	ALL=(ALL) ALL
$ exit 					# exits su
```

## SSH
```
$ sudo apt install openssh-server		# check with sudo systemctl status ssh or dpkg -l | grep ssh
$ sudo nano /etc/ssh/sshd_config		# change port 22 to port 4242 & remove hash sign
						# PermitRootLogin no
$ sudo grep Port /etc/ssh/sshd_config	# check
$ sudo service ssh restart
```
- go to virtual box, ```settings``` >> ```network adapter``` >> ```NAT``` (Network Address Translation) >> ```Advanced``` >> ```port forwarding``` >> ```Add```
- change host to ```4243```, guest to ```4242```
```
$ sudo systemctl restart ssh
$ sudo service sshd status
$ ip addr | grep inet
```
- on host
```
$ ssh hsetyamu@127.0.0.1 -p 4243 	# or ssh hsetyamu@localhost -p 4243
```

## UFW (Uncomplicated Fire Wall)
```
$ sudo apt install ufw
$ sudo ufw enable	# check with sudo ufw status numbered
$ sudo ufw allow ssh
$ sudo ufw allow 4242 	# delete something with sudo ufw delete [number] 

```
## Monitoring script and cron
### Monitoring script

```
$ cd /usr/local/bin/
$ sudo nano monitoring.sh
```
- paste this
```
#!/bin/bash

ARCH=$(uname -srmo)
PCPU=$(cat /proc/cpuinfo | grep 'physical id' | uniq | wc -l)
VCPU=$(cat /proc/cpuinfo | grep 'processor' | uniq | wc -l)
RAM_TOTAL=$(free -h | grep Mem | awk '{print $2}')
RAM_USED=$(free -h | grep Mem | awk '{print $3}')
RAM_PERC=$(free -h | grep Mem | awk '{printf("%.2f%%"), $3 / $2 * 100}')
DISK_TOTAL=$(df -h --total | grep total | awk '{print $2}') #top -bn1 | grep '^MiB Mem' | awk '{print >
DISK_USED=$(df -h --total | grep total | awk '{print $3}') #top -bn1 | grep '^MiB Mem' | awk '{print $>
DISK_PERC=$(df -h --total | grep total | awk '{print $5}')
CPU_LOAD=$(top -bn1 | grep '^%Cpu' | xargs | awk '{printf("%.1f%%"), $2 + $4}') #vmstat
LAST_BOOT=$(who -b | awk '{print($3 " " $4)}')
LVM=$(if [ $(lsblk | grep lvm | wc -l) -eq 0 ]; then echo no; else echo yes; fi)
TCP=$(cat /proc/net/sockstat| grep 'TCP' | awk '{print $3}')
USER_LOG=$(who | wc -l) #users | wc -l
IP_ADDR=$(hostname -I | awk '{print $1}')
MAC_ADDR=$(ip link | grep link/ether | awk '{print $2}')
SUDO_LOG=$(cat /var/log/sudo/sudo.log | grep COMMAND | wc -l)

wall "
------------------------------------------------
Architecture	: $ARCH
Physical CPUs	: $PCPU
Virtual CPUs	: $VCPU
Memory Usage	: $RAM_USED/$RAM_TOTAL ($RAM_PERC)
Disk Usage	: $DISK_USED/$DISK_TOTAL ($DISK_PERC)
CPU Load	: $CPU_LOAD
Last Boot	: $LAST_BOOT
LVM use		: $LVM
TCP Connections	: $TCP established
User(s) logged	: $USER_LOG
Network		: $IP_ADDR ($MAC_ADDR)
Sudo		: $SUDO_LOG command(s) used
------------------------------------------------"
```

```
$ chmod +x monitoring.sh
$ sudo nano /etc/sudoers	# Members to execute any command 	
				# add the line hsetyamu ALL=(ALL) NOPASSWD: /usr/local/bin/monitoring.sh
```

### Cron

- Open crontab ```sudo crontab -u root -e```
- Add this line ```*/10 * * * * /usr/local/bin/monitoring.sh```
-  To make it execute every ten minutes from system startup, create another script (sleep.sh) to calculate the delay between server startup time and the tenth minute of the hour. Add this to cron to apply delay.

```
$ cd /usr/local/bin/
$ sudo nano sleep.sh
```
- paste this
```
#!bin/bash

# Get boot time minutes and seconds
BOOT_MIN=$(uptime -s | cut -d ":" -f 2)
BOOT_SEC=$(uptime -s | cut -d ":" -f 3)

# Calculate number of seconds between the nearest 10th minute of the hour and boot time:
# Ex: if boot time was 11:43:36
# 43 % 10 = 3 minutes since 40th minute of the hour
# 3 * 60 = 180 seconds since 40th minute of the hour
# 180 + 36 = 216 seconds between nearest 10th minute of the hour and boot
DELAY=$(bc <<< $BOOT_MIN%10*60+$BOOT_SEC)

# Wait that number of seconds
sleep $DELAY
```
- Open crontab ```sudo crontab -u root -e```
- Make this line ```*/10 * * * * /usr/local/bin/sleep.sh && /usr/local/bin/monitoring.sh```

## Sudo
### Create sudo log
```
$ cd /var/log
$ sudo mkdir sudo			#if non existant
$ sudo touch sudo/sudo.log
```

### Configure sudoers group
```
$ sudo nano /etc/sudoers
```
- Add
```
Defaults	env_reset
Defaults	mail_badpass
Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/bin:/sbin:/bin"
Defaults	badpass_message="Password is wrong, please try again!"
Defaults	passwd_tries=3
Defaults	logfile="/var/log/sudo/sudo.log"
Defaults	log_input, log_output
Defaults	requiretty
```

## Password policy and group assignment
```
$ sudo nano /etc/login.defs
```
```
PASS_MAX_DAYS	30
PASS_MIN_DAYS	2
PASS_WARN_AGE	7
```
```
$ sudo apt install libpam-pwquality
$ sudo nano /etc/pam.d/common-password
```
- make the line look like this
```
password  requisite     pam_pwquality.so  retry=3 minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root
```
- make user hsetyamu belong to user42 group
```
$ sudo groupadd user42			# create new group 
$ sudo usermod -aG user42 hsetyamu	# check getent group user42
```
- make hsetyamu follows password policy (manually)
```
$ sudo chage hsetyamu -m 2 -M 30 -W 7
```

Notes:
- Set/change password: ```sudo passwd username```
- Create group: ```sudo groupadd groupname```
- Delete group: ```sudo groupdel groupname```
- Create user: ```sudo useradd username```
- Delete user: ```sudo userdel username```
- Sudo file: ```sudo nano /etc/sudoers``` or ```sudo visudo```
- Check all local user: ```cut -d: -f1 /etc/passwd```
- Check jostname: ```hostnamectl```
- Change hostname: ```sudo hostnamectl set-hostname <new_hostname>```	# need reboot
- Stop cron: ```sudo /etc/init.d/cron stop```
- Cron stop: ```sudo /etc/init.d/cron start```



