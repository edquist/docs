<span class="twiki-macro DOC_STATUS_TABLE"></span>

How to get host and service certificates via Web interface ================================================================================================

<span class="twiki-macro STARTINCLUDE"></span> <span class="twiki-macro EDITTHIS"></span>

About this Document
===================

<span class="twiki-macro ICON">hand</span> This document is for system administrators. After reading this document you should be able to apply for and install a grid certificate using the OIM web interface. If you are looking for the command line tool please click [here](Documentation/Release3/GetHostServiceCertificates). This document does not explain how to apply for a grid user certificate. To apply for a grid user certificate click [here](CertificateUserGet) instead!

Requirements
============

You need a web browser to follow the instructions. These instructions were written for Firefox although the basic steps are similar for other browsers.

We recommend to read background information on grid certificates which can be found [here](Documentation.CertificateWhatIs). In order to proceed you will also need:

-   the **fully qualified domain name** of the host you need a grid certificate for
-   the **purpose** of the certificate that explains your request to the Certificate Authority
-   the full **name of the administrator** responsible for the host
-   the **e-mail address of the administrator**
-   the **telephone number of the administrator**
-   the name of the **Certificate Authority** your project is affiliated with
-   the name of the **Virtual Organization** affiliated with the Certificate Authority

Downloading CA certificate files for new OSG DigiCert-based certificates
========================================================================

[Here](http://www.digicert-grid.com/) you will find the location from which you can download the Digicert root and intermediate CA certs that are needed to add to the OS X Keychain (for example) or to the trusted issuer stores in the various browsers.

The following is a sample of the page and file needed to be downloaded and imported.

<https://twiki.grid.iu.edu/twiki/bin/viewfile/Documentation/CertificateGetWeb/DigiCert_Repository_root_cert_b.png>

Requesting host/service certificate using OIM
=============================================

The OSG PKI Certificate Request & Management System can be found at: <https://oim.grid.iu.edu/oim/certificate>.

For instructions on how to request a host or service certificate using the Web interface please see the [user guide maintained by GOC](https://confluence.grid.iu.edu/pages/viewpage.action?pageId=3244064).

References
==========

-   [OSG PKI Certificate Request & Management System](https://oim.grid.iu.edu/oim/certificate)

<span class="twiki-macro STOPINCLUDE"></span>

Comments
========

<span class="twiki-macro COMMENT" type="tableappend"></span>

