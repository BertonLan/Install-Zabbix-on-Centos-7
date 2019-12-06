# Install-Zabbix-on-Centos-7
## Step1 Disable SELinux
```
# vim /etc/sysconfig/selinux
```
Change “SELINUX=enforcing” to  “SELINUX=disabled”
Save and exit the file. Then reboot the system.
## Step2 Install and Configure Apache
```
# yum -y install httpd
```
Check Service status.
```
# systemctl status httpd.service
```
If Apache service is not running, start it manually.
```
# systemctl start httpd.service
```
Enable httpd service on system boot.
```
# systemctl enable httpd
```
## Step3 Configure Needed Repositories
Install epel and remi repos.
```
# yum -y install epel-release
# yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
```
Disable PHP 5 repositories and enable PHP 7.2 repo.
```
# yum-config-manager --disable remi-php54
# yum-config-manager --enable remi-php72
```
## Step4 Install PHP
```
# yum -y install php php-pear php-cgi php-common php-mbstring php-snmp php-gd php-pecl-mysql php-xml php-mysql php-gettext php-bcmath
```
Edit the php.ini file.
```
# vim /etc/php.ini
# php_value max_execution_time 300
# php_value memory_limit 128M
# php_value post_max_size 16M
# php_value upload_max_filesize 2M
# php_value max_input_time 300
# php_value max_input_vars 10000
# php_value always_populate_raw_post_data -1
# php_value date.timezone Asia/Shanghai
```
Then save and exit the file. 

Restart httpd service.
```
# systemctl restart httpd.service
```
## Step5 Install MariaDB
```
# yum --enablerepo=remi install mariadb-server
```
Start the MariaDB service.
```
# systemctl start mariadb.service
```
Enable MariaDB on system boot.
```
# systemctl enable mariadb.service
```
Run the following command to secure MariaDB.
```
# mysql_secure_installation
```
Add a new root password and continue. Then it will ask a few questions. Type “Y” to agree to that.
Login to DB server and verify.
## Step6  Create initial database
```
# mysql -uroot -p
password
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> grant all privileges on zabbix.* to zabbix@localhost identified by 'zabbix'；
mysql> quit；
```
Import initial schema and data. You will be prompted to enter your newly created password.
```
# zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix 
```
Edit file /etc/zabbix/zabbix_server.conf
```
DBPassword=zabbix
```
Then save and exit the file. 
## Step7 Install Zabbix and needed dependencies
 Install Zabbix repository 
```
# rpm -Uvh https://repo.zabbix.com/zabbix/4.4/rhel/7/x86_64/zabbix-release-4.4-1.el7.noarch.rpm
```
Modify zabbix.repo
```
# vim /etc/yum.repos.d/zabbix.repo
# baseurl=http://repo.zabbix.com/zabbix/4.4/rhel/7/$basearch/
# baseurl=http://repo.zabbix.com/zabbix/4.4/rhel/7/$basearch/
```
Then save and exit the file. 
```
# yum clean all
# rpm --rebuilddb
# yum -y update
```
Install Zabbix server，frontend，agent
```
# yum -y install zabbix-server-mysql zabbix-web-mysql zabbix-agent 
```
Now modify the Zabbix configuration file with Database details.
```
# vim /etc/zabbix/zabbix_server.conf
```
Modify the following parameters
```
# DBHost=localhost
# DBName=zabbix
# DBUser=zabbix
# DBPassword=zabbix
```
Then save and exit the file. 
## Step8 Start Zabbix server and agent processes and make it start at system boot
```
# systemctl restart zabbix-server zabbix-agent httpd
# systemctl enable zabbix-server zabbix-agent httpd 
# systemctl status zabbix-server.service
```
Modify firewall rules.
```
# firewall-cmd --add-service={http,https} --permanent
# firewall-cmd --add-port={10051/tcp,10050/tcp} --permanent
# firewall-cmd --reload
```
Then save and exit the file. 

Now restart httpd service.
```
# systemctl restart httpd
```
Now your Zabbix server is up and running!

Connect to your newly installed Zabbix frontend: http://server_ip_or_name/zabbix 
