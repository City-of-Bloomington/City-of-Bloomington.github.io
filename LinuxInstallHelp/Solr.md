---
layout: default
title: Solr Install
---
# SOLR
## Installing
These instructions work for Solr 4.4+.  Previous Solr web apps have slightly more involved ways of configuring multiple cores.  The overall process is the same, just the contents of the actual configuration xml files are going to be different.

These are the abridged instructions.  They only cover how to install Solr as a war file in Tomcat.  If your Java environment is different, you should refer to the full instructions on the Solr Wiki http://wiki.apache.org/solr/.  Again, all of these directories are the ones I typically use for web applications.  (Your server and directory decisions are your own.)

## Download and Extract
Download the latest version of Solr from http://lucene.apache.org/solr.  I put the tarball into /usr/local/src and extract it there.
```bash
sudo /usr/local/src
sudo tar xzvf solr-4.5.1.tgz
```

## Copy war file into place
```bash
sudo cp /usr/local/src/solr-4.5.1/dist/solr-4.5.1.war /srv/webapps/solr.war
sudo service tomcat start
```
Tomcat will expand the war file and install it as a web application.  Once it's expanded, you'll need to stop Tomcat again, and finish the rest of the configuration.  We have to have the solr application expanded so we can get at the configuration files inside.

```bash
sudo service tomcat stop
```

## Create a SOLR_HOME
This is the directory where all the actual index data will be stored.  You want to keep this outside of the web application directory.  (The web application directory is the directory Tomcat created when it expanded the War file.)  This way, you can easily upgrade Solr over time, without losing your data or configurations.

```bash
sudo mkdir /srv/solr_home
```

## Configure the SOLR_HOME environment variable
Solr is written to look for this variable.  That's how it knows where the directory you created is.  I prefer to set this variable in the Tomcat enviroment only for the Solr application.  You do this with a Tomcat XML file.

```bash
sudo nano /usr/local/tomcat/conf/Catalina/localhost/solr.xml
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<Context docBase="/srv/webapps/solr.war" debug="0" crossContext="true">
  <Environment name="solr/home" type="java.lang.String" value="/srv/solr_home" override="true"/>
</Context>
```

## Copy the example solr_home configuration
The source code you downloaded includes a directory of example configurations.  Copy the basic solr configuration to your SOLR_HOME directory.

```bash
cd -R /usr/local/src/solr-4.5.1/example/solr /srv/solr_home
```

This example includes a single core already created, called, "collection1".  In our case, we want to rename this to be called "crm".

### Configure the core
Seperate search indexes for Solr are called, "cores".  A single Solr web application can host search indexes for many web applications.  In our case, the CRM is just one application.  However, I typically have a production, dev, and test installation of the CRM that I need to set up Solr cores for.

In the latest (4.4+) version of Solr, cores are just directories in your SOLR_HOME.  The simplest solution can be to just create a directory and let Solr use it's defaults for everything.  In our case, we want things a bit more precise.

First, we want to rename the core to be called, "crm".

```bash
cd /srv/solr_home
sudo mv collection1 crm
sudo nano crm/core.properties
```

```
name=crm
```

We also need to update the core name in the solrconfig.xml.
There are two lines to look for and change:  the data directory and the transaction log.  These settings are separated by a bit of distance in the file, so you'll need to page through the file, until you find them.

Replace the existing values with the new, CRM named variables.  (Yes, that's right we're going to keep the log in the data directory.)

```bash
sudo nano conf/solrconfig.xml
```

```xml
  <!-- Data Directory

       Used to specify an alternate directory to hold all index data
       other than the default ./data under the Solr home.  If
       replication is in use, this should match the replication
       configuration.
    -->
  <dataDir>${solr.crm.data.dir:}</dataDir>

```

```xml
    <!-- Enables a transaction log, used for real-time get, durability, and
         and solr cloud replica recovery.  The log can grow as big as
         uncommitted changes to the index, so use of a hard autoCommit
         is recommended (see below).
         "dir" - the target directory for transaction logs, defaults to the
                solr data directory.  -->
    <updateLog>
      <str name="dir">${solr.crm.data.dir:}</str>
    </updateLog>
```
#### Comment out the extra, external dependencies
The example configuration also includes some lines loading external dependencies.  However, the example solrconfig.xml cannot really know ahead of time wher you've put your source code.  So, in order for these to load correctly, you would need to edit these lines, and put in the full path to the Solr source code your downloaded.

Fortunately, these are only needed for some more advanced things you can do with Solr.  None of our applications use any of this fancy stuff, and these are not needed.  So, you can safely just comment out these external dependencies.

```bash
sudo nano conf/solrconfig.xml
```

```xml
<!--
  <lib dir="../../../contrib/extraction/lib" regex=".*\.jar" />
  <lib dir="../../../dist/" regex="solr-cell-\d.*\.jar" />

  <lib dir="../../../contrib/clustering/lib/" regex=".*\.jar" />
  <lib dir="../../../dist/" regex="solr-clustering-\d.*\.jar" />

  <lib dir="../../../contrib/langid/lib/" regex=".*\.jar" />
  <lib dir="../../../dist/" regex="solr-langid-\d.*\.jar" />

  <lib dir="../../../contrib/velocity/lib" regex=".*\.jar" />
  <lib dir="../../../dist/" regex="solr-velocity-\d.*\.jar" />
-->
```
### Configure Logging
Solr is set up to use Log4j, and if you do not configure it, Solr will not start up.  If you see this error in your catalina.out, your logging is not configured correctly.

```
INFO: Deploying web application archive solr.war
May 29, 2013 3:59:40 PM org.apache.catalina.core.StandardContext start
SEVERE: Error filterStart
May 29, 2013 3:59:40 PM org.apache.catalina.core.StandardContext start
SEVERE: Context [/solr] startup failed due to previous errors
```

You first have to add the jar files for log4j into the solr web app.

```bash
cd /usr/local/src/solr-4.5.1/example/lib/ext
sudo cp *.jar /srv/webapps/solr/WEB-INF/lib
```

Then, you'll need to add the configuration file

```bash
cd /srv/webapps/solr/WEB-INF
sudo mkdir classes
cd /usr/local/src/solr-4.5.1/example/resources
sudo cp log4j.properties /srv/webapps/solr/WEB-INF/classes
```
## Put the Schema into place
You have to tell Solr what fields each of your records will be attempting to store.  For instance, in uReport, we've included the Solr schema.xml that defines all the fields the CRM will be indexing in Solr.  You need to put the CRM schema.xml into place in your Solr core.

```bash
sudo cp /srv/sites/crm/scripts/solr/schema.xml /srv/solr_home/crm/conf
```
## Restart Tomcat
Now you can restart Tomcat and everything should come up cleanly.  Watch your /usr/local/tomcat/logs/catalina.out for any errors that happen.  You can usually decipher the problems from that and deal with them.  Or, if you're not sure, you can do a web search for any error you are getting.

### Remember to switch Log4j to WARN, instead of INFO
The example log4j configuration is set to log everything at the INFO level.  This is reasonable when you're first installing, to make sure things are set up correctly.  However, once you're up and running, this will fill up your hard drive with useless log entries.

You will want to change the logging level from INFO to WARN, once you've got everything set up correctly.  This will still report any errors that might occur, but won't bother writing out all the details of normal operations.

After you make a change to your file, you need to restart Tomcat for it to take effect.

```bash
sudo service tomcat stop
sudo service tomcat start
```
