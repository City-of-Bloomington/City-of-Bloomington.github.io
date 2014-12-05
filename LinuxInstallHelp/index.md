---
layout: default
title: Linux Help
---
# Linux Help
If you are new to Linux, it might be useful to see examples of how to set all this stuff up.  This is not the only way to set things up.  Each distribution and any person you ask is going to have varying ways they prefer to set up a linux web server.  This is just how I typically do it.

Here are the latest configuration options that I use for LAMP stuff. See the individual pages for detailed information.  If, like me, you are compiling PHP, you need to have MySQL installed first.

The packages are in the order I typically run through their installation process.  Some of these take some time.  You will have to wait while Apache and PHP compile.  While this is going on, I usually open another ssh terminal and work on other packages.


## /usr/local/src
All of the packages that I download and install, whether they're source packages, or binary tarballs, go into `/usr/local/src`.  It gives me one place to look when I want to know what I've got installed, and what versions I'm running.  When I upgrade something, I delete the old item from `/usr/local/src/` when the new version is up and running.
