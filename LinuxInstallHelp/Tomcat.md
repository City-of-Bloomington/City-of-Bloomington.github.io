# Tomcat
## Download
http://tomcat.apache.org/

I use the binary version for this.  Download the latest Binary Distribution of the Core.  I usually grab the tarball version.

## Install
```bash
sudo tar -xzvf apache-tomcat-x.x.xx.tar.gz
sudo mv apache-tomcat-x.x.xx /usr/local/tomcat
```

## Configuration
For the most part, Tomcat is ready to run. We're just going to point it to our custom web directory, instead of the webApps directory that ships with Tomcat.  Look for the Host declaration tag.  It's down towards the bottom of the file.
`sudo nano /usr/local/tomcat/conf/server.xml`
```xml
	<Host name="localhost" appBase="/srv/webapps"
	unpackWARs="true" autoDeploy="true"
	xmlValidation="true" xmlNamespaceAware="true">
```

## Create the webApps directory
If you haven't set up the /srv partition, you'll need to create the directory we just referenced.
```bash
sudo mkdir -p /srv/webapps
```

## Starting Tomcat
Tomcat comes with it's own startup script catalina.sh. By default, the catalina script does not know where Java is installed. Either set an environment variable for root, or edit the catalina script, adding in the JAVA_HOME and JRE_HOME paths. I usually just edit the catalina script and add the variables to the top of the catalina.sh
`sudo nano /usr/local/tomcat/bin/catalina.sh`
```bash
JAVA_HOME=/usr/local/java
JRE_HOME=/usr/local/java/jre
```

## Starting Tomcat at Boot
Tomcat does not ship with a boot startup script. However, on Ubuntu, you can use the catalina.sh script as an init.d startup script. I just create a symbolic link, then use Ubuntu's sysv-rc-conf to turn Tomcat on in runlevels 2,3,4,5.
```bash
sudo ln -s /usr/local/tomcat/bin/catalina.sh /etc/init.d/tomcat
sudo sysv-rc-conf
```