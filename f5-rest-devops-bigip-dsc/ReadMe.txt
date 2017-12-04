Problem this package solves:
===============================
This F5 Contributed DevOps package automatically configures an Active-Standby cluster of BIG-IPs.

v2.0
Release notes:
===============
•	Initial release.
•	Supports clustering of two BIG-IPs.
•	Must be deployed to a BIG-IP. That BIG-IP will be included in the cluster.
•	Supports BIG-IP version 12.1 and above.

Known issues:
================
Occasionally the package can become unresponsive while in BINDING or UNBINDING state, especially when used in combination with resetting BIG-IPs to factory defaults. In order to recover from this condition, any of below workarounds can be used :
 
- Restart BIG-IP services (execute 'bigstart restart’ from command line on BIG-IP where package instance resides)
- Reboot BIG-IP (execute ‘reboot’ from command line on BIG-IP where package instance resides)
- In rare cases when none of previously recommended workarounds help, please use below sequence of commands which will delete all existing package instances on BIG-IP in question:
 
 bigstart stop restjavad restnoded
 rm -rf /var/config/rest/storage
 rm -rf /var/config/rest/index
 bigstart start restjavad restnoded
Please also note that it is necessary to re-install all package rpm(s) after execution of above sequence.

