**Banning Users at Site**
=========================

<span class="twiki-macro DOC_STATUS_TABLE"></span> 
About This Document
-------------------

This document describes the different methods of how to ban users in OSG whether sites are using GUMS or gridmap files for authorization.

Sites using GUMS 1.3
--------------------

-   Go to the GUMS web interface
    -   <https://gums-host:8443/gums/>
-   Add new manual group called banned
    -   Configuration -> User Groups -> Add

<img src="%ATTACHURLPATH%/UserGroupAdd.png" alt="UserGroupAdd.png" width='700' height='750' />

-   Select type = manual and provide name, description. Then save

<img src="%ATTACHURLPATH%/BannedGroupAdd.png" alt="BannedGroupAdd.png" width='733' height='304' />

-   Add this group to a “banned user group”
    -   Click on Configuration
    -   Select the group from drop down menu and save

<img src="%ATTACHURLPATH%/ConfigBannedGroup.png" alt="ConfigBannedGroup.png" width='700' height='300' />

-   Add user FQAN to the banned group
    -   Click on “Manual User Group Members” in “User Management” section
    -   Click Add, select the appropriate “user group”
    -   Add the user FQAN and save <span class="twiki-macro IMPORTANT"></span> We have noticed that just banning DNs instead of FQANs does not work. So please remember to ban FQAN.

<img src="%ATTACHURLPATH%/AddDNBannedGroup.png" alt="AddDNBannedGroup.png" width='700' height='300' />

### Test if Banning is successfully applied

-   Test the mapping from your CE (as su)

``` console
[root@client ~] $ gums-host mapUser %RED%"DN"%ENDCOLOR% 
null
```

-   Only if the mapping returns null, the user is banned

Sites using GUMS 1.2
--------------------

-   No banning option is available directly in GUMS 1.2.x.
-   To ban users an upgrade to 1.3.x is required

Sites using edg-mkgridmap
-------------------------

-   Update /etc/edg-mkgridmap.conf \[NOTE: Old Location is edg/etc/edg-mkgridmap.conf (in $VDT\_LOCATION)\]
    -   Add a line 'deny “DN”'
    -   Wild cards are also accepted
    -   Regenerate grid-mapfile executing /usr/sbin/edg-mkgridmap \[Old: edg/sbin/edg-mkgridmap\]
    -   Log file can be generally found at /var/log/edg-mkgridmap.log \[Old: edg/log/edg-mkgridmap.log\]
-   Check your grid-mapfile and confirm that the DN has indeed been removed

Comments
--------

<span class="twiki-macro COMMENT" type="tableappend"></span>

