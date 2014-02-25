# Java
## Download
I still use Oracle's java, instead of OpenJDK.  You can download Java from http://www.oracle.com/technetwork/java/index.html.  You will need the full JDK. I usually download the tarball version.  I don't have to worry about any package management problems with that one.

## Install
```bash
sudo tar xzvf jdk-7uXX-linux-i586.tar.gz
sudo mv jdk1.7.0_XX /usr/local/java
cd /usr/local/bin
sudo ln -s /usr/local/java/jre/bin/java
```
