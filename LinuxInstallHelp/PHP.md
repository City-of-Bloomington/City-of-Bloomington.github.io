---
layout: default
title: PHP Install
---
# PHP
These instructions are the same for PHP version 5.3 >

## Download
Grab the latest source code tarball from:
http://www.php.net/downloads.php

```bash
tar xzvf php-5.X.X.tar.gz
cd php-5.X.X
```

## Configure, Make, and Install
```bash
sudo ./configure --prefix=/usr/local/php \
--with-apxs2=/usr/local/apache/bin/apxs \
--without-pear \
--disable-short-tags \
--disable-cgi \
--enable-mbstring \
--with-curl \
--enable-exif \
--with-ldap \
--with-zlib \
--with-mysql=/usr \
--with-mysqli=/usr/bin/mysql_config \
--with-pdo-mysql=/usr \
--with-openssl \
--enable-soap \
--with-tidy \
--with-xsl \
--with-gd \
--enable-gd-native-ttf \
--with-jpeg-dir=/usr \
--with-freetype-dir=/usr \
--enable-intl \
--with-gettext \
```

### Ubuntu library directorys
Ubuntu has moved the libraries around. For PHP to be able to find shared libraries for many of the extensions, you'll need to also include either

```bash
--with-libdir=/lib/x86_64-linux-gnu
or
--with-libdir=/lib/i386-linux-gnu
```

### Compile
```bash
sudo make
sudo make install
```

## Apache Configuration
PHP will automatically add in a line to httpd.conf to load the php module. To this line, we'll also need to add the application type so Apache will parse PHP files. I like to keep these lines together.
`sudo nano /usr/local/apache/conf/httpd.conf`

```apache
LoadModule php5_module        modules/libphp5.so    # Added automatically by PHP
AddType application/x-httpd-php .php
```

## PHP Configuration
`cp php.ini-development /usr/local/php/lib/php.ini`
or
`cp php.ini-production /usr/local/php/lib/php.ini`

PHP now ships with two example configurations. The production configuration contains settings which hold security, performance, and best practices at its core. The development is very similar to the production one, except it's much more verbose in reporting errors.

We use the development/production configurations as appropriate for each server. There are only a few settings we need to configure in either of them. The settings we change are the same in either configuration.

```ini
; This directive determines whether or not PHP will recognize code between
; <? and ?> tags as PHP source which should be processed as such. It's been
; recommended for several years that you not use the short tag "short cut" and
; instead to use the full <?php and ?> tag combination. With the wide spread use
; of XML and use of these tags by other languages, the server can become easily
; confused and end up parsing the wrong code in the wrong context. But because
; this short cut has been a feature for such a long time, it's currently still
; supported for backwards compatibility, but we recommend you don't use them.
; Default Value: On
; Development Value: Off
; Production Value: Off
; http://php.net/short-open-tag
short_open_tag = Off

; The separator used in PHP generated URLs to separate arguments.
; PHP's default setting is "&".
; http://php.net/arg-separator.output
; Example:
arg_separator.output = ";"

; List of separator(s) used by PHP to parse input URLs into variables.
; PHP's default setting is "&".
; NOTE: Every character in this directive is considered as separator!
; http://php.net/arg-separator.input
; Example:
arg_separator.input = ";&"

; PHP's default character set is set to empty.
; http://php.net/default-charset
default_charset = "utf-8"

[Date]
; Defines the default timezone used by the date functions
; http://php.net/date.timezone
date.timezone = America/Indiana/Indianapolis

; http://php.net/date.default-latitude
date.default_latitude = 39.169927

; http://php.net/date.default-longitude
date.default_longitude = -86.536806

[intl]
intl.default_locale = en_US
```

The lat/long ini settings are used by a few of our applications, like uReport, that include mapping capabilities. Google Maps might be able to help you find coordinates for your location.

## Restart Apache
`sudo service apache restart`
