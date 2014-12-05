---
layout: default
title: Apache Install
---
## Download
I grab the tarball of the latest unix source from Apache:
http://httpd.apache.org/

Also, version 2.4 no longer includes the necessary APR source code.  You'll need to download the latest version of that from
http://apr.apache.org/download.cgi
You will need to download latest Unix Source tarballs for
apr and apr-util.


## Extract
The apr and apr-util will need to be uncompressed and moved to the Apache source code's srclib directory.

```bash
sudo tar xzvf httpd-2.4.x.tar.gz
sudo tar xzvf apr-1.4.x.tar.gz
sudo tar xzvf apr-util-1.5.x.tar.gz
sudo mv apr-1.4.x httpd-2.4.x/srclib/apr
sudo mv apr-util-1.5.x httpd-2.4.x/srclib/apr-util
```

## Compile
`cd /usr/local/src/httpd-2.4.x`

```bash
sudo ./configure --prefix=/usr/local/apache \
--with-included-apr \
--enable-proxy \
--enable-so \
--enable-ssl \
--enable-expires \
--enable-headers \
--enable-rewrite \
--enable-cgi \
--disable-status \
--disable-userdir \
--disable-autoindex \
--with-mpm=prefork

sudo make
sudo make install
```

## Apache User
You want to run Apache as it's own user, so it has a limited permissions.  Make sure there's an apache user in the system.  You can create it, if it's not already there.
`sudo useradd -r -d /usr/local/apache -s /dev/null apache`

## Configuration
I use the Apache conf files mostly as is.  The vanilla layout works very well.  I do, however, create a separate `websites.conf` (and `ssl-websites.conf`).  These are where I put the configuration for all the applications and sites this machine will host.  I add include commands to load these files.

It's worth scanning over `/usr/local/apache/conf/httpd.conf` to get a feel for where stuff is.  The latest versions of Apache have done a great job of leaving this file lean, mean, and easy to understand.

### Load modules
Apache 2.4 uses dynamic modules for everything. The default httpd.conf, however, does not have all our needed modules uncommented out of the box. You'll need to make sure to uncomment these modules.

```apache
LoadModule socache_shmcb_module modules/mod_socache_shmcb.so
LoadModule slotmem_shm_module modules/mod_slotmem_shm.so
LoadModule ssl_module modules/mod_ssl.so
LoadModule rewrite_module modules/mod_rewrite.so
```

### Global site setup
Here's some more snippits to look for and change in `httpd.conf`.  They are in the order they appear in the conf file.

```apache
# User/Group: The name (or #number) of the user/group to run httpd as.
# It is usually good practice to create a dedicated user and group for
# running httpd, as with most system services.
#
User apache
Group apache


#
# DocumentRoot: The directory out of which you will serve your
# documents. By default, all requests are taken from this directory, but
# symbolic links and aliases may be used to point to other locations.
#
DocumentRoot "/srv/sites/master"
<Directory "/srv/sites/master">


#
# DirectoryIndex: sets the file that Apache will serve if a directory
# is requested.
#
<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>

#
# ErrorLog: The location of the error log file.
# If you do not specify an ErrorLog directive within a <VirtualHost>
# container, error messages relating to that virtual host will be
# logged here.  If you *do* define an error logfile for a <VirtualHost>
# container, that host's errors will be logged there and not here.
#
ErrorLog "/var/log/httpd/error.log"

    #
    # The location and format of the access logfile (Common Logfile Format).
    # If you do not define any access logfiles within a <VirtualHost>
    # container, they will be logged here.  Contrariwise, if you *do*
    # define per-<VirtualHost> access logfiles, transactions will be
    # logged therein and *not* in this file.
    #
    #CustomLog "logs/access_log" common

    #
    # If you prefer a logfile with access, agent, and referer information
    # (Combined Logfile Format) you can use the following directive.
    #
    CustomLog "/var/log/httpd/access.log" combined
</IfModule>
```

### Include our websites conf file
If you have a bunch of websites to host, it helps to keep httpd.conf clean and put all your website entries in a separate file.

Page down to the very end of httpd.conf.  You'll notice a bunch of other Includes there, ready to be uncommented as you need them in the future.  For now, just add our own:

```apache
Include conf/websites.conf
```

## Create the files and directories we referred to in the configuration
We go ahead and give ownership of the web directories to Apache, and set our users up as the group.  That way, the handful of us web developers can all edit the stuff in websites.  (To each his own, this is just how we do it)

```bash
sudo mkdir -p /srv/sites
sudo cp -R /usr/local/apache/htdocs /srv/sites/master
sudo chown -R apache:staff /srv/sites
sudo chmod -R g+w /srv/sites

sudo touch /usr/local/apache/conf/websites.conf
```

## Starting at boot
We'll use the apachectl script as the startup script during boot. Since we're using Ubuntu now, which uses sysv-rc-conf, instead of chkconfig, we can just symbolically link it.

```bash
sudo ln -s /usr/local/apache/bin/apachectl /etc/init.d/apache
```

Ubuntu has started requiring LSB information in the startup scripts, which Apache does not ship with.  You might need to paste the LSB header into apachectl
```bash
### BEGIN INIT INFO
# Provides:          apache
# Required-Start:    $local_fs $remote_fs $network $syslog $named
# Required-Stop:     $local_fs $remote_fs $network $syslog $named
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# X-Interactive:     true
# Short-Description: Apache web server
# Description:       Start the web server
### END INIT INFO
```

Go into Ubuntu's SysV configuration tool and turn apache on for run levels 2,3,4,5.

```bash
sudo sysv-rc-conf
```

Apache should start up okay now.  If there's any errors, check the error log in `/var/log/httpd`.

```bash
sudo service apache start
```
## Log Rotation
We want to use the system's logrotate script to manage our logs. It needs to know where we keep our apache logs, and what to do with them. Create or edit the `/etc/logrotate.d/apache` file
`sudo nano /etc/logrotate.d/apache`

```bash
/var/log/httpd/*.log {
        monthly
        missingok
        rotate 12
        compress
        delaycompress
        notifempty
        sharedscripts
        postrotate
        if [ -f /usr/local/apache/logs/httpd.pid ]; then
                /etc/init.d/apache restart > /dev/null
        fi
        endscript
}
```
