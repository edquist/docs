<span class="twiki-macro DOC_STATUS_TABLE"></span>

GlideinWMS Factory Installation
===============================


About This Document
===================

This document describes how to install the Glidein Workflow Managment System (GlideinWMS) Factory instance.

This document assumes expertise with Condor and familiarity with the glideinWMS software. It **does not** cover anything but the simplest possible install. Please consult the [Glidein WMS reference documentation](http://www.uscms.org/SoftwareComputing/Grid/WMS/glideinWMS/doc.prd/install.html) for advanced topics, including non-root, non-RPM-based installation.

This document covers three primary components of the Glidein WMS system:

-   **WMS Collector / Schedd**: A set of `condor_collector` and `condor_schedd` processes that allow the submission of pilots to Grid entries.
-   **Glidein Factory**: The process submitting the pilots when needed

This document follows the general OSG documentation conventions: <span class="twiki-macro TWISTY">%TWISTY\_OPTS\_OUTPUT% showlink="Click to expand document conventions..."</span> <span class="twiki-macro INCLUDE" section="Header">Trash/DocumentationTeam/DocConventions</span> <span class="twiki-macro INCLUDE" section="CommandLine">Trash/DocumentationTeam/DocConventions</span> <span class="twiki-macro ENDTWISTY"></span>

<span class="twiki-macro WARNING"></span> We really recommend you to use the OSG provided Factory and not to install your own. A [VO Frontend](InstallGlideinWMSFrontend) is sufficient to submit your jobs and to decide scheduling policies. And this will avoid for you the complexity to avoid directly with grid sites. If you really need you own factory be aware that it is a complex component and may require a non trivial maintenance effort.

Requirements
============

Host and OS
-----------

1 A host to install the GlideinWMS Factory (pristine node). 1 OS is <span class="twiki-macro SUPPORTED_OS"></span>. Currently most of our testing has been done on Scientific Linux 6. 1 Root access

The Glidein WMS VO Factory has the following requirements:

-   **CPU**: 4-8 for a large installation (1 should suffice on a small install)
-   **RAM**: 4-8GB on a large installation (1GB should suffice for small installs)
-   **Disk**: 10GB will be plenty sufficient for all the binaries, config and log files related to glideinWMS. If you are a large site with need to keep significant history and logs, you may want to allocate 100GB+ to store long histories.

Users
-----

The Glidein WMS Factory installation will create the following users unless they are already created.

<span class="twiki-macro STARTSECTION">Users</span>

| User       | Default uid | Comment                                   |
|:-----------|:------------|:------------------------------------------|
| `condor`   | none        | Condor user (installed via dependencies). |
| `gfactory` | none        | This user runs the Glidein VO factory.    |

Note that, if you use a different user (or `gfactory` has a different group than `gfactory`), then you will have to modify `/etc/condor/privsep_config` to allow condor to use the condor root switchboard. To verify that the user `gfactory` has `gfactory` as primary group check the output of **`getent passwd gfactory | cut -d: -f4 | xargs getent group`**. It should be the `gfactory` group.

<span class="twiki-macro ENDSECTION">Users</span>

Certificates
------------

| Certificate      | User that owns certificate | Path to certificate                                                           |
|:-----------------|:---------------------------|:------------------------------------------------------------------------------|
| Host certificate | `root`                     | `/etc/grid-security/hostcert.pem` <br> `/etc/grid-security/hostkey.pem` |

[Here](Trash.ReleaseDocumentationGetHostServiceCertificates) are instructions to request a host certificate.

The host certificate/key is used for authorization, however, authorization between the factory and the wms collector is done by file system authentication.

Networking
----------

<span class="twiki-macro STARTSECTION">Firewalls</span> <span class="twiki-macro INCLUDE" section="FirewallTable" lines="http,condorcollector">FirewallInformation</span>

It must be on the public internet, with at least one port open to the world; all worker nodes will load data from this node trough HTTP. Note that worker nodes will also need outbound access in order to access this HTTP port.

<span class="twiki-macro ENDSECTION">Firewalls</span>

Installation Procedure
======================

<span class="twiki-macro INCLUDE" section="OSGRepoBrief" TOC_SHIFT="+">YumRepositories</span>

<span class="twiki-macro INCLUDE" section="OSGBriefCaCerts" TOC_SHIFT="+">InstallCertAuth</span>

Install HTCondor
----------------

Most required software is installed from the Factory RPM installation. HTCondor is the only exception since there are [many different ways to install it](CondorInformation), using the RPM system or not. You need to have HTCondor installed before installing the Glidein WMS Factory. If yum cannot find a HTCondor RPM, it will install the dummy `empty-condor` RPM, assuming that you installed HTCondor using a tarball distribution.

If you don't have HTCondor already installed, you can install the HTCondor RPM in the OSG repository:

``` console
[root@client ~] $ yum install condor.x86_64
# If you have a 32 bit host use instead:
[root@client ~] $ yum install condor.i386
```

See [this HTCondor document](CondorInformation) for more information on the different options.

### Install HTCondor-BOSCO

If you plan to send jobs using [direct batch submission](http://www.uscms.org/SoftwareComputing/Grid/WMS/glideinWMS/doc.prd/recipes/batch.html) (aka BOSCO), then you need also the `condor-bosco` package. You'll have to install the package and remove one of its file (`/etc/condor/config.d/60-campus_factory.config`) because it interferes with the factory configuration.

``` console
[root@client ~] $ yum install condor-bosco
[root@client ~] $ rm /etc/condor/config.d/60-campus_factory.config
[root@client ~] $ touch /etc/condor/config.d/60-campus_factory.config
```

There is an [Advanced configuration section](Trash/Trash/CampusGrids.BoscoInstall#8_Advanced_use) in the BOSCO documentation that can be useful to know, specially how to deal with [multi homed BOSCO resources](Trash/Trash/CampusGrids.BoscoInstall#8_2_Multi_homed_hosts) and the [specification of custom submit properties](Trash/Trash/CampusGrids.BoscoInstall#8_4_Custom_submit_properties).

<span class="twiki-macro STARTSECTION">InstallGWMSFactory</span>

Download and install the Factory RPM
------------------------------------

The RPM is available in the OSG repository:

Install the RPM and dependencies (be prepared for a lot of dependencies).

<pre class="rootscreen">[root@client ~] $ yum install glideinwms-factory</pre>

This will install the current production release verified and tested by OSG with default condor configuration. This command will install the glideinwms factory, condor, the OSG client, and all the required dependencies. If you wish to install a different version of GlideinWMS, add the "--enablerepo" argument to the command as follows:

-   `yum install --enablerepo=osg-testing glideinwms-factory`: The most recent production release, still in testing phase. This will usually match the current tarball version on the GlideinWMS home page. (The osg-release production version may lag behind the tarball release by a few weeks as it is verified and packaged by OSG). Note that this will also take the osg-testing versions of all dependencies as well.
-   `yum install --enablerepo=osg-contrib glideinwms-factory`: The most recent development series release, ie version 3 release. This has newer features such as cloud submission support, but is less tested.

<span class="twiki-macro ENDSECTION">InstallGWMSFactory</span>

Configuration Procedure
=======================

After installing the RPM you need to configure the components of the Glidein WMS Factory:

1.  Edit Factory configuration options
2.  Edit Condor configuration options
3.  Create a Condor grid map file
4.  Reconfigure and Start factory

Download condor tarballs
------------------------

You will need to download Condor tarballs for each architecture that you want to deploy pilots on. At this point, glideinWMS factory does not support pulling condor binaries from your system area. Suggested is that you put these binaries in `/var/lib/gwms-factory/condor` but any `gfactory` accessible location should suffice.

Configuring the Factory
-----------------------

The configuration file is `/etc/gwms-factory/glideinWMS.xml`. The next steps will describe each line that you will need to edit for most cases, but you may want to review the whole file to be sure that it is configured correctly.

### Frontend security configuration

In the security section, you will need to provide each frontend that is allowed to communicate with the factory: <pre class="file"> <security allow\_proxy="frontend" key\_length="2048" pub\_key="RSA" reuse\_oldkey\_onstartup\_gracetime="900"> <frontends> <frontend name="%ORANGE%vofrontend\_sec\_name%ENDCOLOR%" [identity="%GREEN%vofrontend\_service%ENDCOLOR%@FACTORY\_COLLECTOR\_HOSTNAME](mailto:identity=%22%GREEN%vofrontend_service%ENDCOLOR%@FACTORY_COLLECTOR_HOSTNAME)"> <security\_classes> <security\_class name="%RED%frontend\_sec\_class%ENDCOLOR%" username="frontend"/> </security\_classes> </frontend> </frontends> </security> </pre>

These attributes are very important to get exactly right or the frontend will not be trusted. This should match one of the *factory* and *security* sections of the [frontend configuration](InstallGlideinWMSFrontend#Configuring_the_Frontend) in the following way:

``` file
%RED%This is a snippet from the Frontend configuration (for reference), not the Factory that you are configuring now!%ENDCOLOR%
<factory query_expr='((stringListMember("%PINK%VO%ENDCOLOR%", GLIDEIN_Supported_VOs)))'>
  ...
  <collectors>
     <collector DN="/DC=com/DC=DigiCert-Grid/O=Open Science Grid/OU=Services/CN=FACTORY_COLLECTOR_HOSTNAME" 
          comment="Define factory collector globally for simplicity" factory_identity="gfactory@FACTORY_COLLECTOR_HOSTNAME" 
          my_identity="%GREEN%vofrontend_service%ENDCOLOR%@FACTORY_COLLECTOR_HOSTNAME" 
          node="FACTORY_COLLECTOR_HOSTNAME"/>
     </collectors>
</factory>

   < security classad_proxy="/tmp/vo_proxy" proxy_DN="%BLUE%/DC=org/DC=doegrids/OU=People/CN=Some Name 123456%ENDCOLOR%" 
      proxy_selection_plugin="ProxyAll" security_name="%ORANGE%vofrontend_sec_name%ENDCOLOR%" sym_key="aes_256_cbc">
      <proxies>
         <proxy absfname="/tmp/pilot_proxy" security_class="%RED%frontend_sec_class%ENDCOLOR%"/>
      </proxies>
   </security >
```

Note that the identity of the frontend must match what **condor** authenticates the DN of the frontend to. In `/etc/condor/certs/condor_mapfile` (See next section for more details), there must be an entry:

``` file
GSI "%BLUE%^\/DC\=org\/DC\=doegrids\/OU\=People\/CN\=Some\ Name\ 834323%ENDCOLOR%$" %GREEN%vofrontend_service%ENDCOLOR%
```

### Web server configuration

Verify web area: <pre class="file"> <stage base\_dir="/var/lib/gwms-factory/web-area/stage" use\_symlink="True" web\_base\_url="[http://HOSTNAME:PORT/factory/stage"/>](http://HOSTNAME:PORT/factory/stage%22/%3E) </pre> This will determine the location of your web server. It is advisable (but not necessarily) to change the port here and in apache by modifying the "Listen" directive in `/etc/httpd/conf/httpd.conf`. Note that web servers are an often attacked piece of infrastruture, so you may want to go through the Apache configuration in `/etc/httpd/conf/httpd.conf` and disable unneeded modules.

### Entry configuration

Entries are grid endpoints (Compute Elements) that can accept job requests and run pilots (which will run user jobs). Each needs to be configured to go to a specific gatekeeper.

An example test entry is provided in the configuration file. At the very least, you will need to modify the entry line:

``` file
%RED%# These lines are form the configuration of v 3.x%ENDCOLOR%
       <entry name="%RED%ENTRY_NAME%ENDCOLOR%" enabled="True" auth_method="grid_proxy" trust_domain="OSG" 
       gatekeeper="%RED%gatekeeper.domain.tld/jobmanager-type%ENDCOLOR%" gridtype="gt2" rsl="(queue=default)(jobtype=single)"
       schedd_name="%RED%schedd_glideins2@FACTORY_HOSTNAME%ENDCOLOR%" verbosity="std" work_dir="OSG">
%RED%# These lines are the same section form the configuration of v 2.x%ENDCOLOR%
       <entry name="%RED%ENTRY_NAME%ENDCOLOR%" enabled="True" gatekeeper="%RED%gatekeeper.domain.tld/jobmanager-type%ENDCOLOR%" 
       gridtype="gt2" rsl="(queue=default)(jobtype=single)" schedd_name="%RED%schedd_glideins2@FACTORY_HOSTNAME%ENDCOLOR%" 
       verbosity="std" work_dir="OSG" >
```

You will need to modify the entry name and gatekeeper. This will determine the gatekeeper that you access. Specific gatekeepers often require specific "rsl" attributes that determine the job queue that you are in or other attributes. Add them in the *rsl* attribute. Also, be sure to distribute your entries across the various condor schedd work managers to balance load. To see the available schedd use `condor_status -schedd -l | grep Name`. Several schedd options are configured by default for you: schedd\_glideins2, schedd\_glideins3, schedd\_glideins4, schedd\_glideins5, as well as the default schedd. This can be modified in the condor configuration. Add any specific options, such as limitations on jobs/pilots or glexec/voms requirements in the entry section below the above line.

If there is no match between auth\_metod and trust\_domain of the entry and the type and trust domain listed in one of the [credentials of one of the frontends](InstallGlideinWMSFrontend#Configuring_the_Frontend) using this factory, then no job can run on that entry.

The factory must advertise the correct Resource Name of each entry for accounting purposes. Then the factory must also advertise in the entry all the attributes that will allow to match the query expression used in the frontends connecting to this factory (e.g. **`<factory query_expr='((stringListMember("%PINK%VO%ENDCOLOR%", GLIDEIN_Supported_VOs)))'>`** in the [VO frontend configuration document](InstallGlideinWMSFrontend#Configuring_the_Frontend)). Then you must advertise correctly if the site supports gLExec. If it does not set `GLEXEC_BIN` to `NONE`, if gLExec is installed via OSG set it to `OSG`, otherwise set it to the path of gLExec. For example this snippet advertises `GLIDEIN_Supported_VOs` attribute with the supported VO so that can be used with the query above in the VO frontend and says that the resource does not support gLExec:

``` file
       <entry name="RESOURCE_NAME" ...
          <config>
         ...
          <attrs>
            ...
             <attr name="GLIDEIN_Supported_VOs" const="True" glidein_publish="True" job_publish="True" 
             parameter="True" publish="True" type="string" value="%PINK%VO%ENDCOLOR%"/>
             <attr name="GLEXEC_BIN" const="True" glidein_publish="False" job_publish="False" parameter="True" 
             publish="True" type="string" value="%RED%NONE%ENDCOLOR%"/>
             <attr name="GLIDEIN_Resource_Name" const="True" glidein_publish="True" job_publish="True" 
             parameter="True" publish="True" type="string" value="%RED%SiteNameFromOIM%ENDCOLOR%"/>
          </attrs>
```

<span class="twiki-macro NOTE"></span> Specially if jobs are sent to OSG resources, it is very important to set the GLIDEIN\_Resource\_Name and to be consistent with the Resource Name reported in OIM because that name will be used for job accounting in Gratia. It should be the name of the Resource in OIM or the name of the Resource Group (specially if there are many gatekeepers submitting to the same cluster).

More information on options can be found here: <http://www.uscms.org/SoftwareComputing/Grid/WMS/glideinWMS/doc.prd/factory/configuration.html>

Configuring Tarballs
--------------------

Each pilot will download condor binaries from the staging area. Often, multiple binaries are needed to support various architectures and platforms. Currently, you will need to provide at least one tarball for glideinWMS to use. (Using the system binaries is currently not supported).

Download a condor tarball from: <http://research.cs.wisc.edu/condor/downloads-v2/download.pl>. Suggested is to put the binaries in `/var/lib/gwms-factory/condor`, but any factory-accessible location will do just fine.

Once you have downloaded the tarball, configure it in `/etc/gwms-factory/glideinWMS.xml` like in the following: <pre class="file"> < condor\_tarball arch="default" base\_dir="/var/lib/gwms-factory/condor/condor-7.6.6-x86\_64\_rhap\_5-stripped" os="default" version="default"/> </pre>

Remember also to modify the `condor_os` and `condor_arch` attributes in the entries (the configured Compute Elements) to pick the correct HTCondor binary. \[\[<http://www.uscms.org/SoftwareComputing/Grid/WMS/glideinWMS/doc.prd/factory/configuration.html#tarballs>\]\[Here\] are more details on using multiple condor binaries. Note that is sufficient to set the =base\_dir=; the reconfigure command will prepare the tarball and add it to the XML config file.

Configuring Condor
------------------

The condor configuration for the frontend is placed in `/etc/condor/config.d`.

-   00\_gwms\_factory\_general.config
-   01\_gwms\_factory\_collectors.config
-   02\_gwms\_factory\_schedds.config
-   03\_gwms\_factory\_local.config

Get rid of the pre-loaded condor default.

``` console
  rm /etc/condor/config.d/00personal_condor.config
```

For most installations, the items you need to modify are in `03_gwms_factory_local.config`. The lines you will have to edit are:

1.  Credentials of the machine. You can either run using a proxy, or a service certificate. It is recommended to use a host certificate and specify it's location in the

variables `GSI_DAEMON_CERT` and `GSI_DAEMON_KEY`. The host certificate should be owned by `root` and have the correct permissions, 600.

1.  Condor ids in the form UID.GID (both are integers)
2.  Condor admin email it will email when services fail. <pre class="file">

\#-- Condor user: condor CONDOR\_IDS = \#-- Contact (via email) when problems occur CONDOR\_ADMIN =

\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\# \# GSI Security config \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\# \#-- Grid Certificate directory GSI\_DAEMON\_TRUSTED\_CA\_DIR= /etc/grid-security/certificates

\#-- Credentials GSI\_DAEMON\_CERT = /etc/grid-security/hostcert.pem GSI\_DAEMON\_KEY = /etc/grid-security/hostkey.pem

\#-- Condor mapfile CERTIFICATE\_MAPFILE= /etc/condor/certs/condor\_mapfile

\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\# \# Whitelist of condor daemon DNs \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\# \#DAEMON\_LIST = COLLECTOR, MASTER, NEGOTIATOR, SCHEDD, STARTD </pre>

After configuring condor, be sure to restart condor:

``` console
service condor restart
```

### Configuring Condor Privilege Separation

Lastly, verify the settings in `/etc/condor/privsep_config`. By default, the values in this file should not need to be modified. However, if your user/group differs from `gfactory` / `gfactory`, or if you are operating a factory with multiple frontends, you will have to modify this file.

This file controls the condor root switchboard, which allows the factory to change permissions of files, specifically the proxies and files passed to it by the frontends.

``` file
valid-caller-uids = gfactory
valid-caller-gids = gfactory
valid-target-uids = fecmsucsd : fehcc : fecmscern
valid-target-gids = fecmsucsd : fehcc : fecmscern
valid-dirs = /var/lib/gwms-factory/client-proxies
valid-dirs = /var/lib/gwms-factory/client-logs
valid-dirs = /var/log/gwms-factory/client
valid-dirs = /var/lib/gwms-factory
valid-dirs = /var/log/gwms-factory
procd-executable = /usr/sbin/condor_procd
```

Thus, the `valid-called-uids` and `valid-caller-gids` should match the user/group of your factory user. The `valid-target-uids` and `valid-target-gids` should be a colon-separated list of all the frontend users and groups. This should match the security\_classes section in `/etc/gwms-factory/glideinWMS.xml`.

### Using UW Madison Condor RPM

The above procedure will work if you are using the OSG condor RPMS. If you are using the UW Madison Condor RPMS, be aware of the following changes:

-   This Condor RPM uses a file `/etc/condor/condor_config.local` to add your local machine slot to the user pool.
-   If you want to disable this behavior (recommended), you should blank out that file or comment out the line in `/etc/condor/condor_config` for LOCAL\_CONFIG\_FILE. (Make sure that LOCAL\_CONFIG\_DIR is set to `/etc/condor/config.d`)
-   Note that the variable LOCAL\_DIR is set differently in UW Madison and OSG RPMs. This should not cause any more problems in the glideinwms RPMs, but please take note if you use this variable in your job submissions or other customizations.

Create a Condor grid mapfile.
-----------------------------

The Condor grid mapfile (`/etc/condor/certs/condor_mapfile`) is used for authentication between the glidein running on a remote worker node, and the local collector. Condor uses the mapfile to map certificates to pseudo-users on the local machine. It is important that you map the DN's of each frontend you are talking to.

Below is an example mapfile, by default found in `/etc/condor/certs/condor_mapfile`: <pre class="file"> GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=People\\/CN\\=Some\\ Name\\ 123456$" frontend GSI (.**) anonymous FS (.**) \\1 </pre> Each frontend needs a line that maps to the user specified in the identity argument in the frontend security section of the factory configuration.

After configuring the mapfile, be sure to restart condor:

``` console
service condor restart
```

Reconfigure and verify installation
-----------------------------------

In order to use the factory, first you must reconfigure it. Each time you change the configuration you must reconfigure it using the "upgrade" or "reconfigure" option in order to populate scripts and information correctly.

<span class="twiki-macro WARNING"></span> Before you start the factory service first time, you should always use the "upgrade" option. <pre class="rootscreen"> %RED%\# For RHEL 6, CentOS 6, and SL6<span class="twiki-macro ENDCOLOR"></span> [root@client ~] $ service gwms-factory upgrade

%RED%\# For RHEL 7, CentOS 7, and SL7<span class="twiki-macro ENDCOLOR"></span> [root@client ~] $ /usr/sbin/gwms-factory upgrade</pre>

After reconfiguring, you can start the factory:

``` console
%RED%# For RHEL 6, CentOS 6, and SL6%ENDCOLOR%
[root@client ~] $ service gwms-factory start 

%RED%# For RHEL 7, CentOS 7, and SL7%ENDCOLOR%
[root@client ~] $ systemctl start gwms-factory
```

Service Activation and Deactivation
===================================

To **start the Factory** you must start also HTCondor and the Web server beside the Factory itself:

-   HTCondor <pre class="rootscreen">

%RED%\# For RHEL 6, CentOS 6, and SL6<span class="twiki-macro ENDCOLOR"></span> [root@client ~] $ service condor start [root@client ~] $ service httpd start [root@client ~] $ service gwms-factory start

%RED%\# For RHEL 7, CentOS 7, and SL7<span class="twiki-macro ENDCOLOR"></span> [root@client ~] $ systemctl start condor [root@client ~] $ systemctl start httpd [root@client ~] $ systemctl start gwms-factory</pre>

<span class="twiki-macro NOTE"></span> (Old)Anytime you change the `/etc/gwms-factory/glideinWMS.xml` file and/or some glidein script, you will need to reconfigure the factory with: <pre class="rootscreen"> %RED%\# For RHEL 6, CentOS 6, and SL6<span class="twiki-macro ENDCOLOR"></span> [root@client ~] $ service gwms-factory reconfig %RED%\# And if you change also some code<span class="twiki-macro ENDCOLOR"></span> [root@client ~] $ service gwms-factory upgrade

%RED%\# For RHEL 7, CentOS 7, and SL7<span class="twiki-macro ENDCOLOR"></span> [root@client ~] $ systemctl reload gwms-factory %RED%\# And if you change also some code (note that on SL7 the factory must be stopped to upgrade)<span class="twiki-macro ENDCOLOR"></span> [root@client ~] $ /usr/sbin/gwms-factory upgrade </pre>

<span class="twiki-macro NOTE"></span> Once you successfully start using the factory service, anytime you change the `/etc/gwms-factory/glideinWMS.xml` file and/or some glidein script, you will need to run the following command<pre class="rootscreen"> %RED%\# For RHEL 6, CentOS 6, and SL6<span class="twiki-macro ENDCOLOR"></span> [root@client ~] $ service gwms-factory reconfig %RED%\# And if you change also some code<span class="twiki-macro ENDCOLOR"></span> [root@client ~] $ service gwms-factory upgrade

%RED%\# But the situation is a bit more complicated in RHEL 7, CentOS 7, and SL7 due to systemd restrictions<span class="twiki-macro ENDCOLOR"></span> %BLUE%\# For reconfig:<span class="twiki-macro ENDCOLOR"></span> %RED%A. when the factory is running <span class="twiki-macro ENDCOLOR"></span> %RED%A.1 without any additional options <span class="twiki-macro ENDCOLOR"></span> /usr/sbin/gwms-factory reconfig or systemctl reload gwms-factory %RED%A.2 if you want to give additional options <span class="twiki-macro ENDCOLOR"></span> systemctl stop gwms-factory /usr/sbin/gwms-factory reconfig "and your options" systemctl start gwms-factory

%RED%B. when the factory is NOT running <span class="twiki-macro ENDCOLOR"></span> /usr/sbin/gwms-factory reconfig ("and your options if any")

%BLUE%\# For upgrade:<span class="twiki-macro ENDCOLOR"></span> %RED%A. when the factory is running <span class="twiki-macro ENDCOLOR"></span> systemctl stop gwms-factory /usr/sbin/gwms-factory upgrade ("and your options if any") systemctl start gwms-factory

%RED%B. when the factory is NOT running <span class="twiki-macro ENDCOLOR"></span> /usr/sbin/gwms-factory upgrade ("and your options if any") </pre>

To enable the services so that they restart after a reboot:

``` console
%RED%# For RHEL 6, CentOS 6, and SL6%ENDCOLOR%
[root@client ~] $ /sbin/chkconfig fetch-crl-cron on 
[root@client ~] $ /sbin/chkconfig fetch-crl-boot on 
[root@client ~] $ /sbin/chkconfig condor on 
[root@client ~] $ /sbin/chkconfig httpd on 
[root@client ~] $ /sbin/chkconfig gwms-factory on

%RED%# For RHEL 7, CentOS 7, and SL7%ENDCOLOR%
[root@client ~] $ systemctl enable fetch-crl-cron 
[root@client ~] $ systemctl enable fetch-crl-boot
[root@client ~] $ systemctl enable condor 
[root@client ~] $ systemctl enable httpd 
[root@client ~] $ systemctl enable gwms-factory
```

<br/> To **stop the Factory**:<pre class="rootscreen"> %RED%\# For RHEL 6, CentOS 6, and SL6<span class="twiki-macro ENDCOLOR"></span> [root@client ~] $ service gwms-factory stop

%RED%\# For RHEL 7, CentOS 7, and SL7<span class="twiki-macro ENDCOLOR"></span> [root@client ~] $ systemctl stop gwms-factory</pre>

You can also stop these other services if you don't use them for other programs sunning on your host:

-   E.g. stop HTCondor and httpd<pre class="rootscreen">

%RED%\# For RHEL 6, CentOS 6, and SL6<span class="twiki-macro ENDCOLOR"></span> [root@client ~] $ service condor stop [root@client ~] $ service httpd stop

%RED%\# For RHEL 7, CentOS 7, and SL7<span class="twiki-macro ENDCOLOR"></span> [root@client ~] $ systemctl stop condor [root@client ~] $ systemctl stop httpd</pre>

Validation of Service Operation
===============================

The validation of the factroy is the submission of actual jobs.

You can also check that the services are up and running:

``` console
[user@client ~] $ condor_status -any

MyType               TargetType           Name

glidefactoryclient         None                 12345_TEST_ENTRY@gfactory_instance@
glideclient          None                 12345_TEST_ENTRY@gfactory_instance@
glidefactory         None                 TEST_ENTRY@gfactory_instance@
glidefactoryglobal   None                 gfactory_instance@gfactory_ser
glideclientglobal    None                 gfactory_instance@gfactory_ser
Scheduler            None                 hostname.fnal.gov
DaemonMaster         None                 hostname.fnal.gov
Negotiator           None                 hostname.fnal.gov
Scheduler            None                 schedd_glideins2@hostname
Scheduler            None                 schedd_glideins3@hostname
Scheduler            None                 schedd_glideins4@hostname
Scheduler            None                 schedd_glideins5@hostname
Collector            None                 wmscollector_service@hostname
```

You should have one "glidefactory" classad for each entry that you have enabled. If you have already configured the frontends, you will also have one glidefactoryclient and one glideclient classad for each frontend / entry.

You can check also the monitoring Web page: **`http://YOUR_HOST_FQDN/factory/monitor/`**

You can also test the local submission of a job to a resource using the test script `local_start.sh` but you must first install [OSG client](InstallOSGClient) and generate a proxy. After that you can run the test (replace ENTRY\_NAME with the name of one of the entries in `/etc/gwms-factory/glideinWMS.xml`):

``` console
[user@client ~] $ cd /var/lib/gwms-factory/work-dir; ./local_start.sh ENTRY_NAME fast -- GLIDEIN_Collector `hostname`
```

Troubleshooting
===============

File Locations
--------------

|  File Description  |               File Location              |                                          Comment|
|:------------------:|:----------------------------------------:|------------------------------------------------:|
| Configuration file |     /etc/gwms-factory/glideinWMS.xml     |                          Main configuration file|
|        Logs        |   /var/log/gwms-factory/server/factory   |                              Overall server logs|
|                    | /var/log/gwms-factory/server/entry\_NAME |      Specific entry logs (generally more useful)|
|                    |       /var/log/gwms-factory/client       |  Glidein Pilot logs seperated by user and entry.|
|   Startup script   |         /etc/init.d/gwms-factory         |                                                 |
|    Web Directory   |      /var/lib/gwms-factory/web-area      |                                                 |
|      Web Base      |      /var/lib/gwms-factory/web-base      |                                                 |
|  Working Directory |      /var/lib/gwms-factory/work-dir/     |                                                 |

Increase the log level and change rotation policy
-------------------------------------------------

You can increase the log level of the frontend. To add a log file with all the log information add the following line with all the message types in the `process_log` section of `/etc/gwms-factory/glideinWMS.xml`:

``` file
<log_retention>
   <process_logs>
       <process_log extension="all" max_days="7.0" max_mbytes="100.0" min_days="3.0" msg_types="DEBUG,EXCEPTION,INFO,ERROR,ERR"/>
```

You can also change the rotation policy and choose whether compress the rotated files, all in the same section of the config files:

-   max\_bytes is the max size of the log files
-   max\_days it will be rotated.
-   compression specifies if rotated files are compressed
-   backup\_count is the number of rotated log files kept

Further details are in the [reference documentation](http://www.uscms.org/SoftwareComputing/Grid/WMS/glideinWMS/doc.prd/factory/configuration.html#process_logs).

Failed authentication errors
----------------------------

If you get messages such as these in the logs, the factory does not trust the frontend and will not submit glideins.

``` console
WARNING: Client fermicloud128-fnal-gov_OSG_gWMSFrontend.main (secid: frontend_name) not in white list. Skipping request
```

\* This error means that the frontend name in the security section of the factory does not match the security\_name in the frontend.

``` console
Client fermicloud128-fnal-gov_OSG_gWMSFrontend.main (secid: frontend_name) is not coming from a trusted source;
 AuthenticatedIdentity vofrontend_condor@fermicloud130.fnal.gov!=vofrontend_factory@fermicloud130.fnal.gov. 
Skipping for security reasons.
```

\* This error means that the identity in the security section of the factory does not match what the `/etc/condor/certs/condor_mapfile` authenticates the frontend to in condor (AuthenticatedIdentity in the classad).

Make sure the attributes are correctly lined up as in the Frontend security configuration section above.

Glideins start but do not connect to User pool / VO Frontend
------------------------------------------------------------

Check the appropriate job err and out logs in `/var/log/gwms-factory/client` to see if any errors were reported. Often, this will be a pilot unable to access a web server or with an invalid proxy. Also, verify that the condor\_mapfile is correct on the VO Frontend's user pool collector and configuration.

Glideins start but fail before running job with error "Proxy not long lived enough"
-----------------------------------------------------------------------------------

If the glideins are running on a resource (entry) but the jobs are not running and the log files in `/var/log/gwms-factory/client/user_frontend/glidein_gfactory_instance/ENTRY_NAME` report an error like "Proxy not long lived enough (86096 s left), shortened retire time ...", then probably the HTCondor RLM on the Compute Element is delegating the proxy and shortening its lifespan.

This can be fixed by setting **`DELEGATE_JOB_GSI_CREDENTIALS = FALSE`** as suggested in the [CE install document](InstallComputeElement#5_1_If_you_are_using_Condor).

Condor\_root\_switchboard errors
--------------------------------

Make sure that you run `service gwms-factory upgrade` instead of the more light-weight `service gwms-factory reconfig` to ensure that all scripts are created correctly.

Next verify `/etc/condor/privsep_config` to make sure the users and groups are listed correctly.

Lastly, verify that permissions are correct. The parent directory of all valid-dirs must be owned by root.

References
==========

-   <http://www.uscms.org/SoftwareComputing/Grid/WMS/glideinWMS/doc.prd>
-   <http://twiki.grid.iu.edu/bin/view/Documentation/Release/GlideinWMSVOFrontendInstall>

Comments
========

<span class="twiki-macro COMMENT" type="tableappend"></span>

