**GlideinWMS in a Campus Grid**
===============================

<span class="twiki-macro DOC_STATUS_TABLE"></span> 
About this Document
===================

To submit jobs from a Campus Grid to Open Science Grid using the GlideinWMS system you have two options:

1.  Use an existing VO Frontend submitting to OSG (e.g. HCC, OSG-XSEDE) and flock to it 2. Use your own VO Frontend and configure your submit hosts to use it.

Both these options use HTCondor *flocking*, a mechanism that allows jobs to overflow from a HTCondor pool to a second one. Jobs migrate from a *schedd* of a pool to a *collector* and *negotiator* of a different pool. Option 1 is covered in Trash/Trash/CampusGrids.InstallCondorFlockSubmit. This document describe option 2, how to configure a GlideinWMS VO Frontend and submit hosts in a Campus Grid. This will show how jobs submitted to the submit host can flock to campus resources as well as going out to the OSG using GlideinWMS.

Introduction
============

Being familiar with the [HTCondor security model](http://research.cs.wisc.edu/htcondor/manual/v8.0/3_6Security.html) and its configuration will help configuring everything correctly. Anyway here are few notes that will help understand the configuration below and avoid pitfalls.

HTCondor user id  
it is how HTCondor manages identities, it may vary depending on the type of authentication and in its short form may be an identifier `condor_name` or in its extended form will look like `condor_name@condor_UID_DOMAIN/host_name.domain` , where the last part may be also an IP address. These ID are used in the condor\_mapfile and configuration file and are independent from the OS: the condor\_name does not have to be a Unix user.

Authentication  
the authentication process accomplishes the proper identification of a user.

Authorization  
specifies who is allowed to do what.

Security negotiation  
is the process used by HTCondor to decide which type of security is needed and wether is met. The **Access level** is used to specify which actions are possible. HTCondor has many access levels [described in the manual](http://research.cs.wisc.edu/htcondor/manual/v8.0/3_6Security.html#SECTION00461100000000000000): READ, WRITE, ADMINISTRATOR, SOAP, CONFIG, OWNER, DAEMON, NEGOTIATOR, ADVERTISE\_MASTER, ADVERTISE\_STARTD, ADVERTISE\_SCHEDD, CLIENT. A **Policy** specifies which aspect of the interaction is covered: AUTHENTICATION, ENCRYPTION, INTEGRITY, NEGOTIATION. The **Value** specifies how to negotiate a certain security requirement. Valid values in HTCondor are: REQUIRED, PREFERRED, OPTIONAL, NEVER.

The [Security section](http://research.cs.wisc.edu/htcondor/manual/v8.0/3_6Security.html) of the HTCondor manual specifies in detail how all interactions are regulated.

Requirements
============

Required Hardware and Software
------------------------------

1.  [GlideinWMS Frontend](InstallGlideinWMSFrontend) (`gwms-vofe.mydomain.edu` in the examples below). You must have installed and configured a VO Frontend as described in [the document](InstallGlideinWMSFrontend). Refer to that document for further requirements (hardware, users, network, ...) on the frontend host.
2.  HTCondor submit node (`campus-submit.mydomain.edu` in the examples below). You must have installed and configured a HTCondor submit node. InstallCondor provides instructions to install HTCondor off the OSG Yum repository, follow the instructions for the Submit node. This document works also if you choose a [different option to install HTCondor](CondorInformation), including a personal HTCondor not installed as `root`, even [BOSCO](http://bosco.opensciencegrid.org/).

Users
-----

Refer to the frontend installation or the HTCondor installation for the required users

Certificates
------------

All hosts using GSI authentication, e.g. `campus-submit.mydomain.edu` in the example below, must have a valid host certificate. [Check this document](Trash.ReleaseDocumentationGetHostServiceCertificates) to see what host certificates are and how to request them. You may need also a user certificate to operate the frontend. There is no necessity of a user certificate to submit jobs on the HTCondor submit host.

| Certificate      | User that owns certificate | Path to certificate                                                           |
|:-----------------|:---------------------------|:------------------------------------------------------------------------------|
| Host certificate | `root`                     | `/etc/grid-security/hostcert.pem` <br> `/etc/grid-security/hostkey.pem` |

Networking
----------

From the submit host you must have outgoing connectivity to the frontend host and the worker nodes.

Check the [frontend installation](InstallGlideinWMSFrontend) for the frontend network requirements.

GlideinWMS VO Frontend - Flocking Configuration The VO Frontend hosts a HTCondor pool with the job slots of the glideins sent by the GlideinWMS Factory.
========================================================================================================================================================

To receive jobs form other pools via flocking the configuration of VO Frontend, e.g. `gwms-vofe.mydomain.edu`, must contain a FLOCK\_FROM line including the submit host, like: <pre class="file"> FLOCK\_FROM = $(FLOCK\_FROM) %RED%campus-submit.mydomain.edu<span class="twiki-macro ENDCOLOR"></span> </pre>

GlideinWMS VO Frontend - Security Configuration
===============================================

The configuration of the GlideinWMS VO Frontend must allow other hosts to communicate with the frontend. By default, the GlideinWMS Frontend has very strong security, requiring GSI certs to communicate. In this example, a campus submit host is running at `campus-submit.mydomain.edu`. This section will describe the configuration of the GlideinWMS Frontend.

Choose between GSI or alternate authentication and follow the corresponding instructions, no need to do both (unless you use one with some hosts and the other with others)

Using GSI authentication (recommended) on the Frontend
------------------------------------------------------

If you can use GSI authentication to connect all your submit hosts (schedd) to the frontend we recommend to use it. In this case you can skip the section about alternate authentication methods. If one of the submit hosts cannot use GSI authentication, then you'll have to enable some of the other methods.

### Modify the `condor_mapfile` for GSI authenticated hosts The [`condor_mapfile`](http://research.cs.wisc.edu/htcondor/manual/v8.1/3_6Security.html#SECTION00464000000000000000) (normally `/etc/condor/certs/condor_mapfile`, use **`condor_config_val certificate_mapfile`** to find the exact path) is used by Condor to map an authenticated user to a human readable identity to be used in authentication rules. <span class="twiki-macro NOTE"></span> The `condor_mapfile` uses regular expressions to map from authentication methods to users. This is why there are many escapes (`\`) in the below example.

For each GSI authenticated submit host that you want to submit (flock) to the the VO Frontend find out the DN of its certificate (**`openssl x509 -noout -subject -in /etc/grid-security/hostcert.pem`**) and add the following line to the `condor_mapfile`:<pre class="file"> GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=campus\\-submit\\.mydomain\\.edu$" schedd </pre>

The file, including the part already there from the GlideinWMS Frontend install, should look similar to the following example:<pre class="file"> GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=campus\\-submit\\.mydomain\\.edu$" schedd GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=gwms\\-vofe\\.mydomain\\.edu$" frontend GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=Name Familyname 12345$" frontendpilot GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=gfactory\\-1\\.t2\\.ucsd\\.edu$" factory GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=submit\\-host1\\.mydomain\\.edu$" schedd2 GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=personal\\-submit\\-host2\\.mydomain\\.edu$" myuser2 GSI (.**) anonymous FS (.**) \\1 </pre>

If all your hosts use GSI now you can jump to the Submit host configuration, configure the flocking and remember to follow the instructions for GSI authentication.

Enabling alternative authentication methods on the Frontend
-----------------------------------------------------------

It is necessary to enable authentication methods that are acceptable to the campus. If you are on a safe network and no GSI authentication is possible with the submit hosts, then you can use CLAIMTOBE . It will be used to implement host (ip address) based security in this document.

First enable the CLAIMTOBE method. Copy and paste the below into your local condor configuration file. It will append the existing authentication methods to include CLAIMTOBE.

``` file
SEC_DEFAULT_AUTHENTICATION_METHODS = $(SEC_DEFAULT_AUTHENTICATION_METHODS), CLAIMTOBE
```

### Modify the `condor_mapfile` for CLAIMTOBE

Also here you have o modify the `condor_mapfile` like in the GSI authentication. This time add the following line, that most hosts attempting to authenticate will use:<pre class="file"> CLAIMTOBE .\* <anonymous@claimtobe></pre> This means that any machine using `CLAIMTOBE` authentication method will be mapped to user <anonymous@claimtobe> (The user does not have to exist on the machine, it is use for HTCondor identification and accounting). Therefore all authorization rules will use `anonymous@claimtobe` to refer to this user and can be more restrictive than the rules for GSI authenticated users.

The file, including the part already there from the GlideinWMS Frontend install, should look similar to the following example:<pre class="file"> GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=Name Familyname 12345$" frontendpilot GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=gwms\\-vofe\\.mydomain\\.edu" frontend GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=gfactory\\-1\\.t2\\.ucsd\\.edu$" factory GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=submit\\-host1\\.mydomain\\.edu$" schedd2 GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=personal\\-submit\\-host2\\.mydomain\\.edu$" myuser2 GSI (.**) anonymous FS (.**) \\1 CLAIMTOBE .\* <anonymous@claimtobe> </pre>

### Authorizing hosts

Below is an example security configuration file from the HCC GlideinWMS Frontend.

``` file
DENY_WRITE = anonymous@* 
ALLOW_WRITE = *@mydomain.edu *@glidein.mydomain.edu execute-side@matchsession *@daemon.mydomain.edu *@daemon.guestdomain.edu <strong>*/campus-submit.mydomain.edu</strong>
DENY_ADVERTISE_SCHEDD = anonymous@*
DENY_ADMINISTRATOR = anonymous@*
DENY_DAEMON = anonymous@glidein.mydomain.edu 
ALLOW_DAEMON = *@mydomain.edu *@glidein.mydomain.edu  execute-side@matchsession *@daemon.mydomain.edu *@daemon.guestdomain.edu 
DENY_NEGOTIATOR = anonymous@glidein.mydomain.edu 
ALLOW_NEGOTIATOR = */*.mydomain.edu *@daemon.guestdomain.edu <strong>*/campus-submit.mydomain.edu</strong>
```

Some quick definitions:

`ALLOW_WRITE`  
Hosts that are able to write to the local HTCondor instance. This should include all hosts running HTCondor clusters you wish to flock to or from.

`ALLOW_NEGOTIATOR`  
Hosts that run Negotiators that we should trust. This should include all hosts running the Campus Factory and HTCondor clusters you wish to flock to.

Breaking down the file some:

``` file
DENY_WRITE = anonymous@* 
ALLOW_WRITE = *@mydomain.edu *@glidein.mydomain.edu execute-side@matchsession *@daemon.mydomain.edu *@daemon.guestdomain.edu <strong>*/campus-submit.mydomain.edu</strong>
```

The `DENY_WRITE` statement causes Condor to disallow write access to anyone not explicitly listed in the `condor_mapfile`. The `ALLOW_WRITE` line allows explicitly listed people access to write, along with anyone and everyone from `campus-submit.mydomain.edu`.

------------------------------------------------------------------------

``` file
ALLOW_NEGOTIATOR = */*.mydomain.edu *@daemon.guestdomain.edu <strong>*/campus-submit.mydomain.edu</strong> 
```

Allow `campus-submit.mydomain.edu` to negotiate jobs submitted at this host.

Submit host - Flocking Configuration This configuration refer to the Submit host, `campus-submit.mydomain.edu`, and any other host that should flock jobs to the VO Frontend and the OSG
========================================================================================================================================================================================

The HTCondor configuration must contain a FLOCK\_TO line with a space separated line with the VO Frontend and all the HTCondor pools the submit host should flock to.

``` file
FLOCK_TO = $(FLOCK_TO) %RED%gwms-vofe.mydomain.edu%ENDCOLOR%
```

Replacing %RED%gwms-vofe.mydomain.edu<span class="twiki-macro ENDCOLOR"></span> with the full hostname or IP of the VO Frontend.

To list all the pools the jobs will flock to type `condor_config_val FLOCK_TO`. If multiple hosts are listed, flocking happens in the order the hosts appear in the line, from left to right

Submit host - Security Configuration
====================================

The configuration of the Submit host must allow its communication with the frontend. By default, the GlideinWMS Frontend has very strong security, requiring GSI certs to communicate also for the submit host. Submit hosts instead normally do not use GSI authentication, so you'll have to do some extra work to use it. This section will describe the configuration of the Submit host. The authentication methods should match: if you chose GSI authentication above for the VO Frontend you must choose GSI authentication also for the Submit host and vice-versa.

Using GSI authentication on the Submit host
-------------------------------------------

GSI authentication is based on x509 certificates and requires some infrastructure to work correctly. Normally HTCondor submit hosts are not configured to use GSI authentication and some of the required software, like the CA certificates, may not be installed.

### Install the CA Certificates

If you have no CA certificates in `/etc/grid-security/certificates` (e.g. if that directory does not exist or is empty) then you must setup the OSG Yum repositories (you may have them already if you installed other OSG software on this host) and install the CA certificates.

<span class="twiki-macro TWISTY">%TWISTY\_OPTS\_OUTPUT% showlink="Click to expand how to install the CA certificates..."</span> <span class="twiki-macro INCLUDE" section="OSGRepoBrief" TOC_SHIFT="+++">YumRepositories</span> <span class="twiki-macro INCLUDE" section="OSGBriefCaCerts" TOC_SHIFT="+++">InstallCertAuth</span>

End of the section about CA certificates installation. <span class="twiki-macro ENDTWISTY"></span>

### Configure HTCondor to use GSI authentication

Now you can configure HTCondor to use GSI authentication. The hostname and the DN of the VO Frontend, in %RED%red<span class="twiki-macro ENDCOLOR"></span>, must be changed to match yours. The path of the host certificate and key, also in %RED%red<span class="twiki-macro ENDCOLOR"></span>, may need to be changed if you used a different file name. The example uses the default location (in `/etc/grid-security`). <span class="twiki-macro NOTE"></span> A host certificate is required. If you don't have one, before proceeding [check this document](Trash.ReleaseDocumentationGetHostServiceCertificates) to see what host certificates are and how to request them.

``` file
# VO Frontend hostname (FQDN) and Distinguished Name (subject of the certificate)
MY_VOFE = %RED%gwms-vofe.mydomain.edu%ENDCOLOR%
MY_VOFE_DN = %RED%/DC=org/DC=doegrids/OU=Services/CN=host\gwms-vofe.mydomain.edu%ENDCOLOR%

# If you have other pools using GSI authentication and that you'd like to flock to 
# you can add the hostname, DN and add them  to the flock_to and gsi_daemon_name lists

# Who to trust?  Include all the DN that the submit host should trust (comma separated)
GSI_DAEMON_NAME = $(GSI_DAEMON_NAME), $(MY_VOFE_DN)


# Host certificates
# Change the path of the host certificate and key if you put them in a different location
GSI_DAEMON_CERT = %RED%/etc/grid-security/hostcert.pem%ENDCOLOR%
GSI_DAEMON_KEY = %RED%/etc/grid-security/hostkey.pem%ENDCOLOR%
# Change if you installed the CA certificates in a non standard location
GSI_DAEMON_TRUSTED_CA_DIR = /etc/grid-security/certificates


# Enable authentication from tne Negotiator (This is required to run on glidein jobs)
SEC_ENABLE_MATCH_PASSWORD_AUTHENTICATION = TRUE

# Enable GSI authentication (It is already in the default: FS, KERBEROS, GSI
SEC_DEFAULT_AUTHENTICATION_METHODS = GSI, $(SEC_DEFAULT_AUTHENTICATION_METHODS)

# Mapping of users
CERTIFICATE_MAPFILE = /etc/condor/certs/condor_mapfile
```

Write a certificate mapfile with the DNs mapped to nicknames. In the case of a Submit node or Execute node running as root, the user nicknames must represent actual Unix account. In all other cases, the nicknames have no special meaning, and are there just for use in configuration and log files.

Make sure that the frontend DN is there: <pre class="file">GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=gwms\\-vofe\\.mydomain\\.edu$" schedd </pre> Here is an example:<pre class="file"> GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=campus\\-submit\\.mydomain\\.edu$" schedd GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=gwms\\-vofe\\.mydomain\\.edu$" frontend GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=Name Familyname 12345$" frontendpilot GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=gfactory\\-1\\.t2\\.ucsd\\.edu$" factory GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=submit\\-host1\\.mydomain\\.edu$" schedd2 GSI "^\\/DC\\=org\\/DC\\=doegrids\\/OU\\=Services\\/CN\\=personal\\-submit\\-host2\\.mydomain\\.edu$" myuser2 GSI (.**) anonymous FS (.**) \\1 </pre>

Enabling alternative authentication methods on the Submit host
--------------------------------------------------------------

If you decided to authenticate your hosts with CLAIMTOBE you must enable it also on the Submit host. Copy and paste the line below into your local condor configuration file. It will append the existing authentication methods to include CLAIMTOBE.

``` file
SEC_DEFAULT_AUTHENTICATION_METHODS = $(SEC_DEFAULT_AUTHENTICATION_METHODS), CLAIMTOBE
```

\#GratiaAccounting

Adding OSG Gratia Accounting You must report to Gratia if you are running on OSG more than a few test jobs.
===========================================================================================================

Accounting.ProbeConfigGlideinWMS explains how to instal and configure the HTCondor Gratia probe. If you are on a Campus Grid without x509 certificates pay attention to the [Users without Certificates part](Accounting.ProbeConfigGlideinWMS#Users_without_Certificates) in the Unusual Use Cases section.

Validation of Success
=====================

The job submission is an excellent test if the submission is working correctly.

Troubleshooting
===============

Where to look for information
-----------------------------

If any of these steps have failed, errors will be written by HTCondor to log files. Log files are by default in `/var/log/condor` and can be found by querying HTCondor:

``` console
[user@client ~] $ condor_config_val LOG 
```

On the submit host, the relevant log files are the `SchedLog`. On the frontend, the `CollectorLog` and `NegotiatorLog` are relevant.

On the frontend, HTCondor should have a record of the submission host when there are idle jobs on the submission host (with a few minute delay). To query Condor for this information, run the command: <pre class="screen"> [user@client ~] $ condor\_status -schedd </pre> You should see the hostname of the submission host.

What to look for in the Log Files
---------------------------------

If there is a problem, it is likely a security issue. Look in the logs for `DENIED`. For example, on the frontend:

``` console
[user@client ~] $ grep DENIED /path/to/logs/CollectorLog 
```

Or on the submission host.

``` console
[user@client ~] $ grep DENIED /path/to/logs/SchedLog 
```

References
==========

Other possible components of a Campus Grid

-   [GlideinWMS VO Frontend](InstallGlideinWMSFrontend)
-   [A HTCondor Submit host using OGS run Front-ends](Trash/Trash/CampusGrids.InstallCondorFlockSubmit)
-   [Another document about a HTCondor remote submit host](Trash/Trash/CampusGrids.ConfiguringRemoteSubmissionHost)
-   [HTCondor Gratia probe](Accounting.ProbeConfigGlideinWMS), specially check the [Users without Certificates part](Accounting.ProbeConfigGlideinWMS#Users_without_Certificates)

OSG Software

-   [How to setup OSG Yum repositories](YumRepositories)
-   [How to install and use the CA certificates and other components of the x509 PKI infrastructure](InstallCertAuth)

HTCondor documents:

-   [HTCondor security](http://research.cs.wisc.edu/htcondor/manual/v8.1/3_6Security.html)
-   [GSI Authentication in HTCondor](http://research.cs.wisc.edu/htcondor/manual/v8.1/3_6Security.html#sec:GSI-Authentication)
-   [GSI authentication in Glidein WMS systems](http://www.uscms.org/SoftwareComputing/Grid/WMS/glideinWMS/doc.prd/components/gsi.html)

Comments
========

<span class="twiki-macro COMMENT" type="tableappend"></span>

