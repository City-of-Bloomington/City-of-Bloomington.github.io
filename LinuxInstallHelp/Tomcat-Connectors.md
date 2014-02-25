# Tomcat Connectors
This Apache module allows Apache to hand off Tomcat stuff directly to Tomcat via threads, instead of doing a proxy. This makes it easy for Apache to be configured to serve static content and still hand off java stuff to Tomcat.

## Download
You can download this from http://tomcat.apache.org.
Grab the latest release sources tarball.

## Uncompress
```bash
sudo tar xzvf tomcat-connectors-1.2.XX-src.tar.gz
```

## Compile
The source code is actually in the {{native}} subdirectory.  You'll need to go into that directory to do the configure, make, and make install.
```bash
cd tomcat-connectors-1.2.XX-src/native

sudo  ./configure --with-apxs=/usr/local/apache/bin/apxs \
--with-java-home=/usr/local/java
sudo make
sudo make install
```

## Tomcat Configuration
Copy the distributed workers.properties to the Tomcat install, just for convenience. It seems to make more sense there, than in Apache's conf.
```bash
sudo cp /conf/workers.properties.minimal /usr/local/tomcat/conf/workers.properties
```

We're only going to want the ajp13 on port 8009. For some reason, the default workers.properties.minimal did not include the ajp13w worker (which it defined) in the worker.list. We had to add it by hand.

Also, we now change the name of the worker.  Instead of ajp13, we now call it "tomcat".  This is what shows up in the Apache conf when we want to send stuff to tomcat.  Renaming it to "tomcat" here avoids a lot of unnecessary confusion later on.

Here's the final, full workers.properties that we're using
`sudo nano /usr/local/tomcat/conf/workers.properties`
```apache
# workers.properties.minimal -
#
# This file provides minimal jk configuration properties needed to
# connect to Tomcat.
#
# The workers that jk should create and work with
#

worker.list=wlb,jkstatus,tomcat

#
# Defining a worker named tomcat and of type ajp13
# Note that the name and the type do not have to match.
#
worker.tomcat.type=ajp13
worker.tomcat.host=localhost
worker.tomcat.port=8009

#
# Defining a load balancer
#

worker.wlb.type=lb
worker.wlb.balance_workers=tomcat

#
# Define status worker
#

worker.jkstatus.type=status
```

## Apache Configuration
Add this stuff to /usr/local/apache/conf/httpd.conf. It's a LoadModule command, so I usually put it right after the last load module stuff, near the top of httpd.conf.
`sudo nano /usr/local/apache/conf/httpd.conf`
```apache
# mod_JK stuff:
LoadModule jk_module    modules/mod_jk.so
JkWorkersFile /usr/local/tomcat/conf/workers.properties
JkLogFile /var/log/httpd/mod_jk.log
JkShmFile /var/log/httpd/jk-runtime-status
JkLogLevel info
```
