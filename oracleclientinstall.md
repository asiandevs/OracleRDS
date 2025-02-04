Installing the Oracle 19c Client on the Linux Server

Create a Linux user
adduser oracle

Check your oracle ID's
id oracle

## Instant Client

wget https://download.oracle.com/otn_software/linux/instantclient/2112000/el9/oracle-instantclient-basic-21.12.0.0.0-1.el9.x86_64.rpm
wget https://download.oracle.com/otn_software/linux/instantclient/2112000/el9/oracle-instantclient-sqlplus-21.12.0.0.0-1.el9.x86_64.rpm
wget https://download.oracle.com/otn_software/linux/instantclient/2112000/el9/oracle-instantclient-tools-21.12.0.0.0-1.el9.x86_64.rpm
chmod u+x *.rpm
sudo yum install *.rpm
which sqlplus
sqlplus 'pdbadmin@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=dev-cdb.czkwms6ewv3r.ap-southeast-2.rds.amazonaws.com)(PORT=1529))(CONNECT_DATA=(SID=D1AUS)))'



Download Orclee Client software

https://www.oracle.com/database/technologies/instant-client/linux-x86-64-downloads.html

LINUX.X64_193000_client.zip

root@ ~]# wget "https://www.oracle.com/au/database/technologies/oracle19c-linux-downloads.html" -O LINUX.X64_193000_client.zip
--2025-02-03 03:46:26--  https://www.oracle.com/au/database/technologies/oracle19c-linux-downloads.html
Resolving www.oracle.com (www.oracle.com)... 23.202.169.103, 2600:1415:9c00:190::a15, 2600:1415:9c00:180::a15
Connecting to www.oracle.com (www.oracle.com)|23.202.169.103|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 52812 (52K) [text/html]
Saving to: ‘LINUX.X64_193000_client.zip’

100%[=========================================================================================================================================================>] 52,812      --.-K/s   in 0.001s

2025-02-03 03:46:26 (35.9 MB/s) - ‘LINUX.X64_193000_client.zip’ saved [52812/52812]

Install the Oracle Dependencies

https://docs.oracle.com/en/database/oracle/oracle-database/19/lacli/supported-red-hat-enterprise-linux-7-distributions-for-x86-64.html#GUID-2E11B561-6587-4789-A583-2E33D705E498


yum install -y bc binutils elfutils-libelf elfutils-libelf-devel fontconfig-devel glibc glibc-develksh libaio libaio-devel libXrender libX11 libXau libXi libXtst libgcc libnsl librdmacm libstdc++ libstdc++-devel libxcb libibverbs make smartmontools syssta



Installed:
  elfutils-libelf-devel.x86_64 0:0.176-2.amzn2.0.2 fontconfig-devel.x86_64 0:2.13.0-4.3.amzn2 gcc-c++.x86_64 0:7.3.1-17.amzn2      libX11.x86_64 0:1.6.7-3.amzn2.0.5   libXau.x86_64 0:1.0.8-2.1.amzn2.0.2 libXi.x86_64 0:1.7.9-1.amzn2.0.2   libXrender.x86_64 0:0.9.10-1.amzn2.0.2
  libXtst.x86_64 0:1.2.3-1.amzn2.0.2               libaio-devel.x86_64 0:0.3.109-13.amzn2.0.2 libibverbs.x86_64 0:48.0-1.amzn2.0.3 librdmacm.x86_64 0:48.0-1.amzn2.0.3 libxcb.x86_64 0:1.12-1.amzn2.0.2    smartmontools.x86_64 1:7.0-2.amzn2

Dependency Installed:
  cpp.x86_64 0:7.3.1-17.amzn2                 dejavu-fonts-common.noarch 0:2.33-6.amzn2    dejavu-sans-fonts.noarch 0:2.33-6.amzn2    expat-devel.x86_64 0:2.1.0-15.amzn2.0.4      fontconfig.x86_64 0:2.13.0-4.3.amzn2              fontpackages-filesystem.noarch 0:1.44-8.amzn2
  freetype-devel.x86_64 0:2.8-14.amzn2.1.2    gcc.x86_64 0:7.3.1-17.amzn2                  glibc-devel.x86_64 0:2.26-64.amzn2.0.3     glibc-headers.x86_64 0:2.26-64.amzn2.0.3     kernel-headers.x86_64 0:5.10.233-223.887.amzn2    libX11-common.noarch 0:1.6.7-3.amzn2.0.5
  libXext.x86_64 0:1.3.3-3.amzn2.0.2          libatomic.x86_64 0:7.3.1-17.amzn2            libcilkrts.x86_64 0:7.3.1-17.amzn2         libibverbs-core.x86_64 0:48.0-1.amzn2.0.3    libitm.x86_64 0:7.3.1-17.amzn2                    libmpc.x86_64 0:1.0.1-3.amzn2.0.2
  libmpx.x86_64 0:7.3.1-17.amzn2              libpng-devel.x86_64 2:1.5.13-8.amzn2.0.5     libquadmath.x86_64 0:7.3.1-17.amzn2        libsanitizer.x86_64 0:7.3.1-17.amzn2         libuuid-devel.x86_64 0:2.30.2-2.amzn2.0.11        mailx.x86_64 0:12.5-19.amzn2
  mpfr.x86_64 0:3.1.1-4.amzn2.0.2             zlib-devel.x86_64 0:1.2.7-19.amzn2.0.3

Complete!

## Create user
adduser oracle
id oracle
3. Check your oracle ID's
id oracle
** Note the UID, GID, and GROUPS are all showing "oracle" as their identifier
** this will be used later for the "UNIX_GROUP_NAME"

## Create directories
[root@ data]# cd /u01
sudo mkdir oracle oraInventory
sudo chown oracle oracle
sudo chgrp oracle oracle
sudo chown oracle oraInventory
sudo chgrp oracle oraInventory

## Change the owner of these new directories to "oracle"

mkdir -p /u01/app/oracle/client19c
chown -R oracle. /u01


$ sudo su - oracle
Last login: Tue Feb  4 00:45:29 UTC 2025 on pts/1
[oracle@ ~]$ cd OracleDump/
[oracle@ OracleDump]$ ls
19cclient.zip  software
[oracle@ OracleDump]$

## Extract the Oracle Client files to your Installer location
unzip 19cclient.zip -d /u01/app/oracle/client19c

oracle@ OracleDump]$ cd /u01/app/oracle/client19c/client
[oracle@ client]$ ls
install  response  runInstaller  stage  welcome.html
[oracle@ client]$ cd response/

## Modify the "client_install.rsp" file

[oracle@ response]$ cp client_install.rsp client_install.rsp.main

[oracle@ u01]$ cat /u01/app/oracle/client19c/client/response/client_install.rsp
###############################################################################
## Copyright(c) Oracle Corporation 1998,2019. All rights reserved.           ##
##                                                                           ##
## Specify values for the variables listed below to customize                ##
## your installation.                                                        ##
##                                                                           ##
## Each variable is associated with a comment. The comment                   ##
## can help to populate the variables with the appropriate                   ##
## values.                                                                   ##
##                                                                           ##
###############################################################################


#-------------------------------------------------------------------------------
# Do not change the following system generated value.
#-------------------------------------------------------------------------------
oracle.install.responseFileVersion=/oracle/install/rspfmt_clientinstall_response_schema_v19.0.0

#-------------------------------------------------------------------------------
# Unix group to be set for the inventory directory.
#-------------------------------------------------------------------------------
UNIX_GROUP_NAME=oracle
#-------------------------------------------------------------------------------
# Inventory location.
#-------------------------------------------------------------------------------
INVENTORY_LOCATION=/data/u01/oraInventory
#-------------------------------------------------------------------------------
# Complete path of the Oracle Home
#-------------------------------------------------------------------------------
ORACLE_HOME=/data/u01/app/oracle/product/client19c
#-------------------------------------------------------------------------------
# Complete path of the Oracle Base.
#-------------------------------------------------------------------------------
ORACLE_BASE=/data/u01/app/oracle
#------------------------------------------------------------------------------
#Name       : INSTALL_TYPE
#Datatype   : String
#Description: Installation type of the component.
#
#             The following choices are available. The value should contain
#             only one of these choices.
#               - Administrator
#               - Runtime
#               - InstantClient
#               - Custom
#
#Example    : INSTALL_TYPE = Administrator
#------------------------------------------------------------------------------
oracle.install.client.installType=Administrator

#-------------------------------------------------------------------------------
# Name       : oracle.install.client.customComponents
# Datatype   : StringList
#
# This property is considered only if INSTALL_TYPE is set to "Custom"
#
# Description: List of Client Components you would like to install
#
#   The following choices are available. You may specify any
#   combination of these choices.  The components you choose should
#   be specified in the form "internal-component-name:version"
#   Below is a list of components you may specify to install.
#
# oracle.sqlj:19.0.0.0.0 -- "Oracle SQLJ"
# oracle.rdbms.util:19.0.0.0.0 -- "Oracle Database Utilities"
# oracle.javavm.client:19.0.0.0.0 -- "Oracle Java Client"
# oracle.sqlplus:19.0.0.0.0 -- "SQL*Plus"
# oracle.dbjava.jdbc:19.0.0.0.0 -- "Oracle JDBC/THIN Interfaces"
# oracle.ldap.client:19.0.0.0.0 -- "Oracle Internet Directory Client"
# oracle.rdbms.oci:19.0.0.0.0 -- "Oracle Call Interface (OCI)"
# oracle.precomp:19.0.0.0.0 -- "Oracle Programmer"
# oracle.xdk:19.0.0.0.0 -- "Oracle XML Development Kit"
# oracle.network.aso:19.0.0.0.0 -- "Oracle Advanced Security"
# oracle.oraolap.mgmt:19.0.0.0.0 -- "OLAP Analytic Workspace Manager and Worksheet"
# oracle.network.client:19.0.0.0.0 -- "Oracle Net"
# oracle.network.cman:19.0.0.0.0 -- "Oracle Connection Manager"
# oracle.network.listener:19.0.0.0.0 -- "Oracle Net Listener"
# oracle.ordim.client:19.0.0.0.0 -- "Oracle Multimedia Client Option"
# oracle.odbc:19.0.0.0.0 -- "Oracle ODBC Driver"
# oracle.has.client:19.0.0.0.0 -- "Oracle Clusterware High Availability API"
# oracle.dbdev:19.0.0.0.0 -- "Oracle SQL Developer"
# oracle.rdbms.scheduler:19.0.0.0.0 -- "Oracle Scheduler Agent"
#
# Example    : oracle.install.client.customComponents="oracle.precomp:19.0.0.0.0","oracle.oraolap.mgmt:19.0.0.0.0","oracle.rdbms.scheduler:19.0.0.0.0"
#-------------------------------------------------------------------------------
oracle.install.client.customComponents=

#-------------------------------------------------------------------------------
# Host name to be used for by the Oracle Scheduler Agent.
# This needs to be entered in case oracle.rdbms.scheduler is selected in the
# list of custom components during custom install
#
# Example    : oracle.install.client.schedulerAgentHostName = acme.domain.com
#------------------------------------------------------------------------------
oracle.install.client.schedulerAgentHostName=

#------------------------------------------------------------------------------
# Port number to be used for by the Oracle Scheduler Agent.
# This needs to be entered in case oracle.rdbms.scheduler is selected in the
# list of custom components during custom install
#
# Example: oracle.install.client.schedulerAgentPortNumber = 1500
#------------------------------------------------------------------------------
oracle.install.client.schedulerAgentPortNumber=
[oracle@ u01]$

oracle@ response]$ pwd
/u01/app/oracle/client19c/client/response
[oracle@ response]$ cd ..
[oracle@ client]$ ls
install  response  runInstaller  stage  welcome.html

## Run the Oracle Client Installer

./runInstaller -silent -responseFile /u01/app/oracle/client19c/client/response/client_install.rsp

oracle@ oraInventory]$ cd /u01/app/oracle/client19c/client
[oracle@ client]$ ./runInstaller -silent -responseFile /u01/app/oracle/client19c/client/response/client_install.rsp
Starting Oracle Universal Installer...

Checking Temp space: must be greater than 415 MB.   Actual 3698 MB    Passed
Checking swap space: must be greater than 150 MB.   Actual 494 MB    Passed
Preparing to launch Oracle Universal Installer from /tmp/OraInstall2025-02-04_01-19-09AM. Please wait ...[oracle@ client]$ [WARNING] [INS-32016] The selected Oracle home contains directories or files.
   ACTION: To start with an empty Oracle home, either remove its contents or choose another location.
[WARNING] [INS-13014] Target environment does not meet some optional requirements.
   CAUSE: Some of the optional prerequisites are not met. See logs for details. installActions2025-02-04_01-19-09AM.log
   ACTION: Identify the list of failed prerequisite checks from the log: installActions2025-02-04_01-19-09AM.log. Then either from the log file or from installation manual find the appropriate configuration to meet the prerequisites and fix it manually.
The response file for this session can be found at:
 /u01/app/oracle/product/client19c/install/response/client_2025-02-04_01-19-09AM.rsp

The installation of Oracle Client 19c was successful.
Please check '/u01/oraInventory/logs/silentInstall2025-02-04_01-19-09AM.log' for more details.

As a root user, execute the following script(s):
        1. /u01/oraInventory/orainstRoot.sh



Successfully Setup Software with warning(s).
The log of this install session can be found at:
 /u01/oraInventory/logs/installActions2025-02-04_01-19-09AM.log

Last login: Tue Feb  4 01:07:00 UTC 2025 on pts/0
[root@ ~]# /u01/oraInventory/orainstRoot.sh
Changing permissions of /u01/oraInventory.
Adding read,write permissions for group.
Removing read,write,execute permissions for world.

Changing groupname of /u01/oraInventory to oracle.
The execution of the script is complete.

Set environment
export ORACLE_HOME=/u01/app/oracle/product/client19c
export PATH=$ORACLE_HOME/bin:$PATH

oracle@ ~]$ which impdp
/u01/app/oracle/product/client19c/bin/impdp


[oracle@ ~]$ sqlplus 'pdbadmin@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=dev-cdb.czkwms6ewv3r.ap-southeast-2.rds.amazonaws.com)(PORT=1529))(CONNECT_DATA=(SID=D1AUS)))'

SQL*Plus: Release 19.0.0.0.0 - Production on Tue Feb 4 01:27:33 2025
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.

Enter password:
Last Successful login time: Tue Feb 04 2025 00:51:39 +00:00

Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.25.0.0.0

SQL>
