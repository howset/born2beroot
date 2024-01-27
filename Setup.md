# Setup

## User
```
$ su - 							# the - is important!!
$ apt update && apt upgrade -y
$ apt install sudo
$ usermod -aG sudo hsetyamu		# Check with getent group sudo
$ sudo visudo					# User privilege specification 	
								# add the line hsetyamu	ALL=(ALL) ALL
$ exit 							# exits su
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
$ sudo ufw enable		# check with sudo ufw status numbered
$ sudo ufw allow ssh
$ sudo ufw allow 4242 	# delete something with sudo ufw delete [number] 

```

## Create sudo log
```
$ cd /var/log
$ sudo mkdir sudo			#if non existant
$ sudo touch sudo/sudo.log
```

## Configure sudoers group
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
$ sudo groupadd user42	# create new group 
$ sudo usermod -aG user42 hsetyamu

```
- make hsetyamu follows password policy (manually)
```
$ sudo chage hsetyamu -m 2 -M 30 -W 7
```

Notes:
- Set/change password: ```sudo passwd username```
- Create group: ```sudo groupadd groupname```
- Create user: ```sudo useradd username```
- Delete user: ```sudo deluser username```

## Monitoring script and cron
- todo
