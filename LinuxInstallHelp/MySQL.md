---
layout: guide
title: MySQL Install
---
# MySQL
## Install from Ubuntu
I've given up on comiling MySQL from source. MySQL and Maria's compiling process has gotten way more complicated. Ubuntu's distro version works just fine for our needs. PHP compiles against it with no problems.

```bash
sudo apt-get install mysql-client mysql-server libmysqlclient-dev
```

## Configuration
I tend to prefer InnoDB tables.  Also, for small linux machines, I prefer having it use a seperate index file for each table.  This way, when/if I need to free up hard drive space, I can delete old databases and MySQL will delete the index space associated with them.

`sudo nano /etc/mysql/my.cnf`

```bash
[mysqld]
default_storage_engine=InnoDB
character_set_server=utf8
innodb_file_per_table=1
```

## Database Installation
Ubuntu has already gone ahead and created initial databases and user accounts.  The databases it created won't have used the settings we just put into the configuration.  I typically, blow away Ubuntu's default databases and run MySQL's vanilla setup.  That way, I know the databases will use the settings I expect, and there won't be any surprises in there.

```bash
sudo service mysql stop
sudo rm -Rf /var/lib/mysql
sudo /usr/bin/mysql_install_db
sudo chown -R mysql /var/lib/mysql
```
## Database user accounts
The vanilla install still has a root account that is wide open.  During any MySQL install, you will always need to lock down your user accounts.  I do this by wiping the user accounts and only installing my own user accounts.

```bash
mysql -u root
```

```mysql
use mysql;
delete from user;
grant all privileges on *.* to myusername@localhost identified by 'somesafepassword' with grant option;
flush privileges;
exit;
```
Now I can log in as myself, and I should have a full root-enabled mysql account.  Make sure you can log in with the username and password you expect.  If it doesn't work, you can just go back to the Database Installation step, wipe the databases and redo it from there.

```bash
mysql -p
```
