**<noop><span class="twiki-macro SPACEOUT">%TOPIC%</span>**
=================================================================

<span class="twiki-macro DOC_STATUS_TABLE"></span> 
Introduction
------------

This page documents how to create a plugin for Bestman-Gateway (or Bestman2) that selects the best gridftp server to use. This particular plugin uses the storage path of the file to be accessed to select the server that has that storage path locally mounted.

Checklist
---------

This document assumes the following has been installed:

-   A working Bestman-Gateway installed from one of the RPM installs from [Storage Installations](Documentation/Release3.NavAdminStorage#Installation_Documents)
-   A POSIX-compliant file system mounted in the above Bestman system.
-   Java SDK with javac compiler and jar utility.

You will need two files for this configuration, which are included below.

-   **`MountPoint.java`**
-   **`servers.txt`**

Installation and Configuration
------------------------------

Edit **`MountPoint.java`**: change the following variables to match your local setup:

-   **`log_filename`**: Location where the output of this plugin will go to. This will be useful if you want to know which servers are handling particular requests or for debugging. This directory should be created manually before Bestman is started. The Bestman daemon user will need access to write to this directory.
-   **`server_filename`**: The location of the `servers.txt` file given below.
-   **`defaultServer`**: If no patterns match, this server will be given as a default server.
-   **`displayContents`**: Replace the line below in **`MountPoint.java`** with the list of the gridftp servers that are specified in `/etc/bestman2/conf/bestman2.rc` (this info overwrites bestman2.rc). These servers will show up in srm-ping output as the valid gridftp servers.

``` file
String as[]={"gw014k1.fnal.gov:2811"};
```

Multiple servers can added to this list such as in the following example (each will be listed in the results from a srm-ping):

``` file
String as[]={"server1.fnal.gov:2811","server2.fnal.gov:2811","server3.fnal.gov:2811"};
```

Next, customize **`servers.txt`** for your site and file system structure. Each line in this file will consist of two parts separated by an equals sign. The first will be a directory path that the system will match incoming file requests against. The second part will be the server that requests that contain that pattern will go to. Note that the java uses "contains" for pattern matching, so that the matching will occur even if this path occurs in the middle of the file. (i.e. "/usr/" will still match "/mnt/hadoop/usr/engage"). If multiple patterns match the file, it will use the last one in the list.

Issue the following commands:

``` console
cp -p servers.txt MountPoint.java /usr/share/java/bestman2/plugin
cd /usr/share/java/bestman2/plugin
javac -classpath "/usr/share/java/bestman2/bestman2.jar" MountPoint.java
jar cvf mount.jar MountPoint.class
cp -p /etc/bestman2/conf/bestman.rc /etc/bestman2/conf/bestman.rc.orig
```

Add the following to **`/etc/bestman2/conf/bestman.rc`**:

``` file
###########################################################
# gridftp server selection plugin 
###########################################################
protocolSelectionPolicy=class=MountPoint&jarFile=mount.jar&name=gsiftp
pluginLib=/usr/share/java/bestman2/plugin
```

Then, execute the following commands

``` console
cp -p /etc/bestman2/conf/bestman.rc /etc/bestman2/conf/bestman.rc.save

service bestman stop
service bestman start
```

Note that the plugin will log its output to `/var/log/bestman2/servers.log` by default.

Attached Files
--------------

-   [MountPoint.java](%ATTACHURL%/MountPoint.java)
-   [servers.txt](%ATTACHURL%/servers.txt)

Also provided is a user-contributed round-robin version of this plugin. It is not OSG supported, but can be used as an additional plugin example:

\* [SelectServerFilePath.java](%ATTACHURL%/SelectServerFilePath.java)

-- Main.HorstSeverini - 29 Oct 2010

