---
layout: default
title: Ubuntu Installation
---
## Ubuntu Server Minimal install
There are lots of linux distributions out there, and any one will work.  Choose the distribution that you're most confortable with.  These days, I typically use Ubuntu, but compile Apache and PHP from scratch.  I'm not too fond of how Ubuntu configures Apache and PHP.  For me, it's just easiest to work from a vanilla install of Apache and PHP.  However, I have recently started relying on Ubuntu's distribution of MySQL.  The latest versions of MySQL and Maria have become a lot more complicated to compile - and Ubuntu doesn't butcher that configuration too badly.


## Partitioning
Here's my typical partitioning scheme for web application servers. The goal is to have web applications on their own partition, and to maximize the amount of space for web applications. We still want to have a reasonable amount of space for user home directories, but we don't need to create a seperate partition for that.

Linux filesystem standards recommend using `/srv` for data served by this server. This seems the best place for web applications. We used to put them in `/var/www` but there was always the danger of wiping that partition if rebuilding the machine with that directory already mounted. So now, we'll be using `/srv` instead of `/var/www`.

* swap 2G
* `/` 18G
* `/srv` ~all available

## Ubuntu Post-install
After doing a minimal ubuntu installation, here's the base set of stuff that we have the distribution install.

```bash
sudo apt-get install build-essential \
autoconf \
libncurses5-dev \
libssl-dev \
ntp \
emacs \
zip \
unzip \
subversion \
git \
libxml2-dev \
libxslt-dev \
libldap2-dev \
libtidy-dev \
libcurl4-openssl-dev \
imagemagick \
libjpeg-dev \
libpng-dev \
libxpm-dev \
libfreetype6-dev \
libpcre3-dev \
libicu-dev
```

## Firewall
All machines should use their own firewall. Only open ports necessary for each individual machine. Ubuntu uses the UFW program to administer the iptables. Remember to enable UFW!
```bash
sudo ufw default deny
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo ufw enable
```

## NTP
If you're like us, you'll have a few NTP servers that are local for you that you'll want to add to the configuration.
```bash
sudo nano /etc/ntp.conf
```
```bash
# You do need to talk to an NTP server or two (or three).
server ntp.ubuntu.com
```

### Timezone
On a fresh install, sometimes, the timezone is usually not set correctly. You'll need to change this before NTP will pull in a correct date. We're using America/Indiana/Indianapolis.

`sudo dpkg-reconfigure tzdata`

Now, you can stop NTP, update the date, then restart the NTP service.
```bash
sudo service ntp stop
sudo ntpdate ntp.ubuntu.com
sudo service ntp start
```

## Users
The default Ubuntu settings for adding users are not so nice. In particular, we want to set make it so user accounts are created using the bash shell, by default.
`sudo useradd -D -s /bin/bash`

Users can now be added using:
`sudo useradd -m username`

I typically add our sysadmin users to staff and sudo groups:
```bash
sudo gpasswd -a username staff
sudo gpasswd -a username sudo
```

## Good to go
At this point, you have a relatively small linux server all set up and ready to start putting whatever server applications you want to use.

If you're doing this on a virtual machine...at this point, I typically quit out of the VMWare console and ssh in to my new virtual machine.  Doing all this over ssh is much faster than dealing the VMWare's console.

If you're doing this on a physcial machine, this is where I typically log out and rack the machine.  Or just unplug the monitor - at this point it can run headless, and ssh is more convenient than standing in front of some server in the benchroom.

