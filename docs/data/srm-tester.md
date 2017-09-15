%TOPIC%
=======

<span class="twiki-macro DOC_STATUS_TABLE"></span>


1.0 About this Document
=======================

This document explains how to install SrmTester using the Open Science Grid (OSG) provided RPMs. This procedure will guide one through the installation and usage of SrmTester software that can be used as a validation tool for SRM based Storage Sites. It validates SRM v2.2 interface for compatibility and interoperability. It also verifies the operation of file transfer protocols such as gsiftp, http or ftp. The testing results may be resulted in a file so that web publishing is possible.

2.0 Requirements
================

2.1 Host and OS
---------------

You need atleast one node in order to install SrmTester

1.  OS must be <span class="twiki-macro SUPPORTED_OS"></span>.
2.  [EPEL](http://fedoraproject.org/wiki/EPEL) repos enabled.
3.  All procedures in this document require *root* privileges

2.2 Certificates
----------------

<span class="twiki-macro STARTSECTION">Certificates</span>

| Certificate      | User that owns certificate | Path to certificate                                                           |
|:-----------------|:---------------------------|:------------------------------------------------------------------------------|
| Host certificate | `root`                     | `/etc/grid-security/hostcert.pem` <br> `/etc/grid-security/hostkey.pem` |

You will also need a copy of CA certificates (see below).

2.3 Networking
--------------

<span class="twiki-macro STARTSECTION">Firewalls</span> <span class="twiki-macro INCLUDE" section="FirewallTable" lines="gridftp,srm2,portrange,portsource">Documentation/Release3.FirewallInformation</span> \\ <span class="twiki-macro ENDSECTION">Firewalls</span>

3.0 Install Instructions
========================

<span class="twiki-macro INCLUDE" section="OSGRepoBrief" headingoffset="1">YumRepositories</span> <span class="twiki-macro INCLUDE" section="OSGBriefCaCerts" TOC_SHIFT="+">InstallCertAuth</span>

3.1 Installing SrmTester
------------------------

Next, you will need to install the SrmTester

<span class="twiki-macro STARTSECTION">Install</span>

``` console
[root@client ~] $ yum install bestman2-tester
```

4.0 Running SrmTester
=====================

After installation of the above package, 'srm-tester' command will be available in the path. To run it, you will need a valid user proxy (typically found in `/tmp/x509up_uUID`).

The tester can be run to test various configurations of the SRM host. Each of these will be listed under the "op" command line parameter. You will also need to specify the full SRM URL in the "serviceurl" argument. You may need to specify additional arguments based on the operations requested.

For instance, the easiest test to perform is a simple srm-ping test:

``` console
srm-tester -op ping -serviceurl srm://%RED%hostname.fqdn.domain%ENDCOLOR%:8443/srm/v2/server
```

Usage
-----

A full list of command-line arguments can be found from the usage statement.

``` console
        -serviceurl     wsdl server host (Required)
        example :       srm://hostname:portname/wsdlServiceName
        -proxyfile      proxyfile location
        -localtargetdir         <targetUrl>
        -drive        (drives the tester using the param file)
        -concurrency    (runs tests on multiple SRMs)default:1
        -spacetoken     <valid space token>
        -authid         <valid authorization id>
        -cleanupall     default:true
        -abortonfail   (abort is called when file transfer fails)
        -releasefile    default:true
        -remotesfn      <remote sfn>
        -copysource     <copy_sourceurl> (Required for -all option)
        -filestoragetype        <permanent|durable|volatile>    -gsiftpcopysource       <source_url>
        -localsource            <sourceUrl> (used for put option)
        -remotefilelifetime     <integer> (used for remote file life time for put operation)
        -desiredlifetime        <integer> (used for SRMBringonline operation)
        -gucscriptpath          location to the guc script path
        -localpublish   location to the publish the test results
        -browseafterput         browses the file after putting the file into SRM
        -nooverwrite    default:false
        -nocopyoverwrite        default:false
        -direct         <do direct gsiftp transfer>
        -nodcau
        -buffersize     <integer>default:1048576 bytes
        -retrytimeallowed       <integer>default:5 minutes
        -statuswaittime         <integer>default:30 seconds
        -conf           properties file path
        -op             (ping,put,get,bringonline,push,pull,gsiftp,ls,mv,srmrm,mkdir,rmdir,reserve,getspacemeta,getspacetokens,release,gettransferprotocols)
        -pushmode       (uses push mode to perform the copy operation)
```

A full explanation of the arguments for the srm-tester command can also be found at the [srm-tester manual](https://sdm.lbl.gov/twiki/bin/view/Software/SRMTester/SRMTesterGuide/WebHome) in the [usage section](https://sdm.lbl.gov/twiki/bin/view/Software/SRMTester/SRMTesterGuide/Usage).

Sample usage
------------

-   A series of testing be done with srm-tester. Sample command for srm-tester is below.

<!-- -->

    srm-tester \ 
    -serviceurl srm://osp4.lbl.gov:62443/srm/v2/server \ 
    -op ping,put,get,bringonline,ls,mv,srmrm,gsiftp,mkdir,rmdir,reserve,getspacemeta,getspacetokens,release,gettransferprotocols \ 
    -deleteafterput false \ 
    -putsource file:////tmp/sourcetest.data \ 
    -gettarget file:////tmp/targettest.data \ 
    -gsiftpcopysource gsiftp://datagrid.lbl.gov//testdata/S/test.data \ 
    -browseafterput \ 
    -remotetarget "srm://osp4.lbl.gov:62443/srm/v2/server?SFN=/srmcache/star/srmtest.data" \ 
    -browseurl "srm://osp4.lbl.gov:62443/srm/v2/server?SFN=/srmcache/star/srmtest.data" \ 
    -getsource "srm://osp4.lbl.gov:62443/srm/v2/server?SFN=/srmcache/star/srmtest.data" \ 
    -s "srm://osp4.lbl.gov:62443/srm/v2/server?SFN=/srmcache/star/srmtest.data" \ 
    -copytarget "srm://osp4.lbl.gov:62443/srm/v2/server?SFN=/srmcache/star/srmtest2.data" \ 
    -mvtarget "srm://osp4.lbl.gov:62443/srm/v2/server?SFN=/srmcache/star/srmtest3.data" \ 
    -removeurl "srm://osp4.lbl.gov:62443/srm/v2/server?SFN=/srmcache/star/srmtest3.data" \ 
    -dirurl "srm://osp4.lbl.gov:62443/srm/v2/server?SFN=/srmcache/star/srmtest4" \ 
    | & tee bestman.test.log

-   Sample command for srm-tester with a parameter input file is below.

<!-- -->

    srm-tester -drive -conf osp4.properties 

    % more osp4.properties 
    testsites=NERSC_ITB 
    proxyfile=/tmp/x509up_u9999 
    target=file:////tmp/targettest.data 
    putsource=file:////tmp/sourcetest.data 
    op=put,get,bringonline,ls,mv,srmrm,gsiftp,mkdir,rmdir,reserve,getspacemeta,getspacetokens,release,gettransferprotocols,getspacetokens 
    retrytimeallowed=5 
    gsiftpcopysource=gsiftp://datagrid.lbl.gov//testdata/S/test.data 
    pullcopysrm=NERSC_ITB@srm://osp4.lbl.gov:62443/srm/v2/server?SFN=/srmcache/star/test.data 
    browseafterput=true 
    advisorydelete=false 
    putoverwrite=NERSC_ITB@true 

5.0 Upgrading SrmTester
=======================

Upgrade can be done with the following command:

``` console
[root@client ~] $ yum upgrade bestman2-tester
```

6.0 How to get Help?
====================

If you cannot resolve the problem, there are several ways to receive help:

-   For bug support and issues, submit a ticket to the [Grid Operations Center](https://ticket.grid.iu.edu/goc).
-   For community support and best-effort software team support contact <osg-software@opensciencegrid.org>.

For a full set of help options, see [Help Procedure](HelpProcedure). <span class="twiki-macro STOPINCLUDE"></span>

