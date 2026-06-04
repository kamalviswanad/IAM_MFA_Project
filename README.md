# IAM_MFA_Project

I have deployed a Identidy management system on Windows Server 2025 and created Users,a Organizational Unit (OU) and a GPO. I used this configuration and used the created mock credentials to login into a Ubuntu VM. I have also implemented Multi-Factor Authentication using Google Authenticator TOTP locally on Ubuntu to increase security. 

This project is divided into ___ Steps:

1. Initial Setup and enable Active Directory Server
2. s
3. s

---------------------------------------------------------------------------------------------------------

STEP 1: Initial Setup and Enabling Active Directory Server

1. DNS configuration needed for Active Directory Domain Service (AD DS) server
    - IP Address: a static IP address (10.0.0.24 for mime)
    - Preferred DNS Server: 127.0.0.1 (localhost)
    - DNS Suffix: Domain.lab (name od my AD DS server)
    - SUbnet mash: 255.0.0.0
  
2. AD DS server can be enabled by : Server Manager -> Manage -> Add Roles and Features -> click next until server roles page -> Active Directory Domain Services -> Install.

3. To verify connection between both VMs and AD DS, I pinged Domain1.lab from Ubuntu VM, but it failed
    -Troubleshooting: a. ran this command ```sudo nano /etc/resolv.conf``` on Ubuntu
                       b. added ```nameserver 10.0.0.24 
                nameserver 8.8.8.8 ```  lines to the file
                       c. ```ping Domain1.lab``` worked.
---------------------------------------------------------------------------------------------------------

STEP 2: Creating User,OU, and GPO and installing RADIUS server

1. 
   
