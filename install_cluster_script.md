## node[1-5]
```bash
sudo passwd centos

sudo sh -c "echo '172.31.43.162 node1.sk.com node1'>> /etc/hosts"
sudo sh -c "echo '172.31.34.39 node2.sk.com node2'>> /etc/hosts"
sudo sh -c "echo '172.31.36.116 node3.sk.com node3'>> /etc/hosts"
sudo sh -c "echo '172.31.34.27 node4.sk.com node4'>> /etc/hosts"
sudo sh -c "echo '172.31.33.185 node5.sk.com node5'>> /etc/hosts"

sudo hostnamectl set-hostname node5.sk.com
sudo hostname

sudo sed -i 's/^PermitRootLogin .*/PermitRootLogin yes/' /etc/ssh/sshd_config
sudo sed -i "s/^PasswordAuthentication .*/PasswordAuthentication yes/g" /etc/ssh/sshd_config
sudo sed -i "s/^ChallengeResponseAuthentication .*/ChallengeResponseAuthentication yes/g" /etc/ssh/sshd_config

sudo systemctl restart sshd
ssh node3
```

## node1
```bash
sudo mkdir ~/.ssh
sudo chmod 700 ~/.ssh
ssh-keygen -t rsa -f ~/.ssh/id_rsa
sudo chmod 600 ~/.ssh/id_rsa
sudo chmod 644 ~/.ssh/id_rsa.pub

ssh-copy-id -i ~/.ssh/id_rsa.pub centos@node1.sk.com
ssh-copy-id -i ~/.ssh/id_rsa.pub centos@node2.sk.com
ssh-copy-id -i ~/.ssh/id_rsa.pub centos@node3.sk.com
ssh-copy-id -i ~/.ssh/id_rsa.pub centos@node4.sk.com
ssh-copy-id -i ~/.ssh/id_rsa.pub centos@node5.sk.com

sudo yum -y install wget
sudo wget https://src.fedoraproject.org/repo/pkgs/pssh/pssh-2.1.1.tar.gz/4b355966da91850ac530f035f7404cd5/pssh-2.1.1.tar.gz
tar xvf pssh-2.1.1.tar.gz
cd pssh-2.1.1
sudo python setup.py install

echo 'node2.sk.com'> ~/target_host
echo 'node3.sk.com'>> ~/target_host
echo 'node4.sk.com'>> ~/target_host
echo 'node5.sk.com'>> ~/target_host
```

## node[1-5]
```bash
sudo sysctl vm.swappiness=1
sudo sh -c "echo 'vm.swappiness=1'>> /etc/sysctl.conf"

sudo sh -c "echo never > /sys/kernel/mm/transparent_hugepage/defrag"
sudo sh -c "echo never > /sys/kernel/mm/transparent_hugepage/enabled"
sudo sh -c "echo 'echo never > /sys/kernel/mm/transparent_hugepage/defrag' >> /etc/rc.local"
sudo sh -c "echo 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' >> /etc/rc.local"
sudo cat /etc/rc.local

sudo sh -c "sed -i 's/^\(SELINUX\s*=\s*\).*$/\1disabled/'/etc/selinux/config"
getenforce

sudo yum -y install nscd 
sudo systemctl enable nscd
sudo systemctl start nscd
sudo systemctl status nscd

sudo yum -y install ntp
sudo chkconfig ntpd on
sudo systemctl enable ntpd
sudo systemctl start ntpd
```

## node1
```bash
sudo wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo -P /etc/yum.repos.d/
sudo vi /etc/yum.repos.d/cloudera-manager.repo
/*
name=Cloudera Manager
baseurl=https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.15/
gpgkey =https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera
gpgcheck = 1
*/    
sudo yum -y update
sudo yum repolist

sudo yum -y install oracle-j2sdk1.7
sudo yum -y install cloudera-manager-server cloudera-manager-daemons
sudo sh -c "echo 'export JAVA_HOME=/usr/java/jdk1.7.0_67-cloudera/' >> /etc/default/cloudera-scm-server"
```

## node[1-5]
```bash
sudo yum -y install wget 
sudo wget https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz
sudo tar zxvf mysql-connector-java-5.1.47.tar.gz
sudo mkdir -p /usr/share/java/
cd mysql-connector-java-5.1.47
sudo cp mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar
```

## node1
```bash
sudo yum -y install mariadb-server
sudo systemctl stop mariadb
sudo vi /etc/my.cnf
/*
	[mysqld]
	datadir=/var/lib/mysql
	socket=/var/lib/mysql/mysql.sock
	transaction-isolation = READ-COMMITTED
	# Disabling symbolic-links is recommended to prevent assorted security risks;
	# to do so, uncomment this line:
	symbolic-links = 0
	# Settings user and group are ignored when systemd is used.
	# If you need to run mysqld under a different user or group,
	# customize your systemd unit file for mariadb according to the
	# instructions in http://fedoraproject.org/wiki/Systemd

	key_buffer = 16M
	key_buffer_size = 32M
	max_allowed_packet = 32M
	thread_stack = 256K
	thread_cache_size = 64
	query_cache_limit = 8M
	query_cache_size = 64M
	query_cache_type = 1

	max_connections = 550
	#expire_logs_days = 10
	#max_binlog_size = 100M

	#log_bin should be on a disk with enough free space.
	#Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your
	#system and chown the specified folder to the mysql user.
	log_bin=/var/lib/mysql/mysql_binary_log

	#In later versions of MariaDB, if you enable the binary log and do not set
	#a server_id, MariaDB will not start. The server_id must be unique within
	#the replicating group.
	server_id=1

	binlog_format = mixed

	read_buffer_size = 2M
	read_rnd_buffer_size = 16M
	sort_buffer_size = 8M
	join_buffer_size = 8M

	# InnoDB settings
	innodb_file_per_table = 1
	innodb_flush_log_at_trx_commit  = 2
	innodb_log_buffer_size = 64M
	innodb_buffer_pool_size = 4G
	innodb_thread_concurrency = 8
	innodb_flush_method = O_DIRECT
	innodb_log_file_size = 512M

	[mysqld_safe]
	log-error=/var/log/mariadb/mariadb.log
	pid-file=/var/run/mariadb/mariadb.pid

	#
	# include all files from the config directory
	#
	!includedir /etc/my.cnf.d
*/
sudo systemctl enable mariadb
sudo systemctl start mariadb
sudo systemctl status mariadb

sudo /usr/bin/mysql_secure_installation
/*
	enter
	y
	y
	n
	y
	y
*/

sudo vi /etc/my.cnf.d/server.cnf
/*
	[mysqld]
	server-id           = 1
	log_bin             = /var/log/mariadb/mariadb-bin
	log_bin_index       = /var/log/mariadb/mariadb-bin.index
	expire_logs_days    = 10
	max_binlog_size     = 100M
*/
sudo systemctl restart mariadb
mysql -u root -p
/*
	GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'replication_user'@'%' IDENTIFIED BY 'replication_user123!';
	FLUSH TABLES WITH READ LOCK;
*/
```

## node2
```bash
sudo yum -y install mariadb-server
sudo systemctl stop mariadb
sudo vi /etc/my.cnf
/*
	[mysqld]
	datadir=/var/lib/mysql
	socket=/var/lib/mysql/mysql.sock
	transaction-isolation = READ-COMMITTED
	# Disabling symbolic-links is recommended to prevent assorted security risks;
	# to do so, uncomment this line:
	symbolic-links = 0
	# Settings user and group are ignored when systemd is used.
	# If you need to run mysqld under a different user or group,
	# customize your systemd unit file for mariadb according to the
	# instructions in http://fedoraproject.org/wiki/Systemd

	key_buffer = 16M
	key_buffer_size = 32M
	max_allowed_packet = 32M
	thread_stack = 256K
	thread_cache_size = 64
	query_cache_limit = 8M
	query_cache_size = 64M
	query_cache_type = 1

	max_connections = 550
	#expire_logs_days = 10
	#max_binlog_size = 100M

	#log_bin should be on a disk with enough free space.
	#Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your
	#system and chown the specified folder to the mysql user.
	log_bin=/var/lib/mysql/mysql_binary_log

	#In later versions of MariaDB, if you enable the binary log and do not set
	#a server_id, MariaDB will not start. The server_id must be unique within
	#the replicating group.
	server_id=1

	binlog_format = mixed

	read_buffer_size = 2M
	read_rnd_buffer_size = 16M
	sort_buffer_size = 8M
	join_buffer_size = 8M

	# InnoDB settings
	innodb_file_per_table = 1
	innodb_flush_log_at_trx_commit  = 2
	innodb_log_buffer_size = 64M
	innodb_buffer_pool_size = 4G
	innodb_thread_concurrency = 8
	innodb_flush_method = O_DIRECT
	innodb_log_file_size = 512M

	[mysqld_safe]
	log-error=/var/log/mariadb/mariadb.log
	pid-file=/var/run/mariadb/mariadb.pid

	#
	# include all files from the config directory
	#
	!includedir /etc/my.cnf.d
*/
sudo systemctl enable mariadb
sudo systemctl start mariadb
sudo systemctl status mariadb

sudo /usr/bin/mysql_secure_installation
/*
	enter
	y
	y
	n
	y
	y
*/

sudo vi /etc/my.cnf.d/server.cnf
/*
	[mysqld]
	server-id           = 2
	log_bin             = /var/log/mariadb/mariadb-bin
	log_bin_index       = /var/log/mariadb/mariadb-bin.index
	expire_logs_days    = 10
	max_binlog_size     = 100M
	relay_log           = /var/log/mariadb/relay-bin
	relay_log_index     = /var/log/mariadb/relay-bin.index
	relay_log_info_file = /var/log/mariadb/relay-bin.info
	log_slave_updates
*/
sudo systemctl restart mariadb
mysql -u root -p
/*
	CHANGE MASTER TO
	MASTER_HOST='node1번 ip',
	MASTER_USER='replication_user',
	MASTER_PASSWORD='replication_user123!',
	MASTER_PORT=3306,
	MASTER_LOG_FILE='확인 후 입력',
	MASTER_LOG_POS=확인 후 입력,
	MASTER_CONNECT_RETRY=10;
	FLUSH PRIVILEGES;
	START SLAVE;
	SHOW SLAVE STATUS\G;
*/
```

## node1
```bash
mysql -u root -p
/*
	unlock tables;
	create database scm;
	grant all on *.* to 'scm'@'%' identified by 'scm123!' with grant option;
	create database rman;
	grant all on rman.* to 'rman'@'%' identified by 'rman123!' with grant option;
	create database hue;
	grant all on hue.* to 'hue'@'%' identified by 'hue123!' with grant option;
	create database metastore;
	grant all on metastore.* to 'hive'@'%' identified by 'hive123!' with grant option;
	create database oozie;
	grant all on oozie.* to 'oozie'@'%' identified by 'oozie123!' with grant option;
	show databases;
*/
sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm scm123!

sudo systemctl start cloudera-scm-server
sudo systemctl status cloudera-scm-server
```

