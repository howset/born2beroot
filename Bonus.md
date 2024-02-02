# Bonus 1 Partition

## Virtual box prep
- Get [debian](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/) netinst
- Creating a Virtual Machine in Virtualbox
	1. Launch VirtualBox & click New.
	2. Name the VM whatever, put in ```/sgoinfre/```. Leave type & version alone.
	3. Memory ```1024MB``` .
	4. Create a virtual hard disk now >> ```VDI``` >>```Dynamically allocated```.
	5. Size ```12 GB```  (30 G to exactly imitate bonus -> 33079636992 B).
	6. Go to ```Settings``` >> ```Storage``` >> change empty optical drive to debian image.
	8. Start virtual machine.

## Start installation
- Hostname: hsetyamu42 
- Domain name: -leave empty-
- Root password: -secure password X-
- Username: hsetyamu
- Full name: hsetyamu
- User password: -secure password X-
- Timezone: Germany
- Partition disks: 
	1. ```Guided - use entire disk and setup encrypted LVM``` --> no bonus
	2. ```Manual``` --> bonus
- ```Manual``` >> ```SCSI2 (0,0,0)(sda)....```
- Yes to create partition table
	1. ```pri/log xxGB FREE SPACE``` >> ```Create a new partition``` >> ```500M (524488000 B)``` >> ```Primary``` >> ```Beginning``` >> ```Mount point``` >> ```/boot``` >> ```Done```.
	2. ```pri/log xxGB FREE SPACE``` >> ```Create a new partition``` >> ```max``` >> ```Logical``` >> ```Mount point``` >> ```Do not mount it``` >> ```Done```.
- Encrypting disks
	1. ```Configure encrypted volumes``` >> ```Yes```.
	2. ```Create encrypted volumes```
	3. Choose ```sda5``` to encrypt. 
	4. ```Done``` >> ```Finish``` >> ```Yes```.
	5. Choose password (-secure password X-)
- **Logical Volume Manager (LVM)**
Create a volume group:
	1. ```Configure the Logical Volume Manager``` >> ```Yes```.
	2. ```Create Volume Group``` >> [LVMGroup] >> ```/dev/mapper/sda5_crypt```.

- Create Logical Volumes:
	- ```Create Logical Volume``` >> ```LVMGroup``` >> [root] >> ```10G (10737418240 B)```
	- ```Create Logical Volume``` >> ```LVMGroup``` >> [home] >> ```5G (5368709120 B)```
	- ```Create Logical Volume``` >> ```LVMGroup``` >> [swap] >> ```2.3G (2469606195.2 B)```
	- ```Create Logical Volume``` >> ```LVMGroup``` >> [tmp] >> ```3 G (3221225472 B)```
	- ```Create Logical Volume``` >> ```LVMGroup``` >> [srv] >> ```3 G (3221225472 B)```
	- ```Create Logical Volume``` >> ```LVMGroup``` >> [var] >> ```3 G (3221225472 B)```
	- ```Create Logical Volume``` >> ```LVMGroup``` >> [var-log] >> ```4G (4294967296 B/the rest)```
- Set filesystems and mount points for each logical volume:
	- [home] >> ```Use as``` >> ```Ext4``` >> ```Mount point``` >> ```/home``` >> ```Done```
	- [root] >> ```Use as``` >> ```Ext4``` >> ```Mount point``` >> ```/``` >> ```Done```
	- [swap] >> ```Use as``` >> ```swap area``` >> ```Done```
	- [srv] >> ```Use as``` >> ```Ext4``` >> ```Mount point``` >> ```/srv``` >> ```Done```
	- [tmp] >> ```Use as``` >> ```Ext4``` >> ```Mount point``` >> ```/tmp``` >> ```Done```
	- [var] >> ```Use as``` >> ```Ext4``` >> ```Mount point``` >> ```/var``` >> ```Done```
	- [var-log] >> ```Use as``` >> ```Ext4``` >> ```Mount point``` >> ```Enter manually``` >> ```/var/log``` >> ```Done```
- ```No``` to scan.
- Choose ```country``` & ```mirror```.
- Leave proxy field ```blank```.
- ```No``` to participation.
- No GUI.
- ```Yes```to install GRUB >> ```/dev/sda```
- ```Continue```.


# Bonus 2 Wordpress

## PHP
- Install php
```bash
$ sudo apt install php php-common php-cgi php-cli php-mysql
$ php -v				# check installation & version
```

## Lighttpd
- If Apache is installed as dependency by php, remove it
```bash
$ systemctl status apache2		# Check
$ sudo apt purge apache2		# Uninstall
```
- Install lighttpd
```bash
$ sudo apt install lighttpd		# Install
$ sudo lighttpd -v			# Check
$ sudo systemctl start lighttpd		# Start
$ sudo systemctl enable lighttpd	# Enable
$ sudo systemctl status lighttpd	# Check status
```
- Allow port 80 (http) through UFW:
```bash
$ sudo ufw allow http
$ sudo ufw status
```
- Forward host port 8080 to guest port 80 in VirtualBox:
	- Go to VM >> ```Settings``` >> ```Network``` >> ```Adapter 1``` >> ```Port Forwarding```
	- Add rule for host port 8080 to forward to guest port 80
- Test Lighttpd
	- Go to host machine browser and type in address http://127.0.0.1:8080 or http://localhost:8080. 
	- Should see a Lighttpd "placeholder page".
- Activate FastCGI module in VM (a protocol that interfaces applications (like PHP) to web servers)
```bash
$ sudo lighty-enable-mod fastcgi
$ sudo lighty-enable-mod fastcgi-php
$ sudo service lighttpd force-reload
```
- Test php is working with lighttpd
	- create a file in /var/www/html named info.php
	- write 
```php
<?php
phpinfo();
?>
```
- Save and go to host browser and type in the address http://127.0.0.1:8080/info.php
- Should see a page with php info

## MariaDB
- Install mariadb
```bash
$ sudo apt install mariadb-server	# Install
$ sudo systemctl start mariadb		# Start
$ sudo systemctl enable mariadb		# Enable
$ systemctl status mariadb		# Check status
```
- Config mysql
```bash
$ sudo mysql_secure_installation

Enter current password for root (enter for none): <Enter>	# Root user of db, not vm, but same pass anyway
Switch to unix_socket authentication [Y/n]: Y
Set root password? [Y/n]: Y
New password: NumberCharsWhatever				# Make a secure one, same as before
Re-enter new password: NumberCharsWhatever				
Remove anonymous users? [Y/n]: Y
Disallow root login remotely? [Y/n]: Y
Remove test database and access to it? [Y/n]:  Y
Reload privilege tables now? [Y/n]:  Y

$ sudo systemctl restart mariadb				# Restart service
$ mysql -u root -p						# Enter interface
```
 - Create db for wordpress
```sql
MariaDB [(none)]> CREATE DATABASE wordpress_db_bonus;
MariaDB [(none)]> CREATE USER 'hsetyamu'@'localhost' IDENTIFIED BY 'NumberCharsWhatever';
MariaDB [(none)]> GRANT ALL ON wordpress_db_bonus.* TO 'hsetyamu'@'localhost' IDENTIFIED BY 'NumberCharsWhatever' WITH GRANT OPTION;
MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]> EXIT;
```
 - Check that the database was created
```bash
$ mysql -u root -p
MariaDB [(none)]> show databases; # Show db
```

## Wordpress itself
```bash
$ sudo apt install wget tar
$ wget http://wordpress.org/latest.tar.gz
$ tar -xzvf latest.tar.gz
$ sudo mv wordpress/* /var/www/html/
$ rm -rf latest.tar.gz wordpress/
$ sudo mv /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
```
- Edit the config file
```php
<?php
/* blablabla whatever */
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress_db_bonus' );

/** Database username */
define( 'DB_USER', 'hsetyamu' );

/** Database password */
define( 'DB_PASSWORD', 'NumberCharsWhatever' );

/** Database host */
define( 'DB_HOST', 'localhost' );
```
- Change permissions of WordPress directory to grant rights to web server and restart lighttpd:
```bash
$ sudo chown -R www-data:www-data /var/www/html/
$ sudo chmod -R 755 /var/www/html/
$ sudo systemctl restart lighttpd
```
- http://127.0.0.1:8080 in host browser to see
- http://127.0.0.1:8080/wp-admin/

# Bonus 3 Service (FTP)

## FTP
```
$ sudo apt install vsftpd 	# ftp
$ dpkg -l | grep vsftpd		# Verify install
$ sudo ufw allow 21		# Allow port 21 (ftp) 
$ sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.bak # Make backup
$ sudo nano /etc/vsftpd.conf	# Config vsftpd
```
-  enable FTP write command & prevent user from accessing files or using commands outside the directory tree
```bash
write_enable=YES 		# allow changes to the filesystem(uploading)
chroot_local_user=YES 		# make local users jailed by default
```
- add these lines to /etc/vsftpd.conf
```bash
# Insert user and path
user_sub_token=$USER
local_root=/home/$USER/ftp

# Access are given only when explicitly added.
userlist_enable=YES
userlist_file=/etc/vsftpd.userlist 	# specifies the file which lists users that are not able to login
userlist_deny=NO 			#  to allow only certain users to login
```
- set root folder for FTP-connected user to /home/hsetyamu/ftp
```bash
$ sudo mkdir /home/hsetyamu/ftp
$ sudo mkdir /home/hsetyamu/ftp/files
$ sudo chown nobody:nogroup /home/hsetyamu/ftp
$ sudo chmod a-w /home/hsetyamu/ftp		# remove write permission, conflict with chroot config on ftp
```
-  whitelist FTP
```bash
$ echo hsetyamu | sudo tee -a /etc/vsftpd.userlist 	# Make the file and add username
$ cat /etc/vsftpd.userlist 				# Test
$ sudo systemctl restart vsftpd 			# Restart to load
```

### Connecting to Server via FTP
- Prepare test file and restart
```bash
$ echo "vsftpd borntoberoot test file content" | sudo tee /home/hsetyamu/ftp/files/testfile.txt
$ sudo service vsftpd restart
```

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
or 
```sftp -P <port> <username>@<ip-address>```

Notes:
- Check user in mariadb: ```SELECT User,Host FROM mysql.user;```
- Delete db in mariadb: ```DROP DATABASE <db_name>;```
- Delete user in maria db: ```DROP USER 'hsetyamu'@'localhost';```
