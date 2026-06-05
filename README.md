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

STEP 2: Creating an OU and installing RADIUS server

1. Active directory users and computers -> right click on domain name (domain1.lab) -> new -> OU -> Name of OU: Enterprise_Staff -> Firstname/logon name: secadmin and set password -> Finish
    -Inside Enterprise_Staff -> right click -> New group -> fill in details (group name:Radius_Users, Group scope: Global, Group type: Security) ->click OK
    - Then right click on secadmin -> properties -> click on memberof tab-> add Radius_users -> OK  (these 2 steps are recommended to avoid login issues)

3.   Setting up Ubuntu to communicate with RADIUS Server:

   a. sudo apt update
   b. sudo apt install openssh-server libpam-radius-auth -y (To install PAM module and OpenSSH server)
   c. sudo nano /etc/pam_radius_auth.conf
       -and add the line ``` 10.0.0.24   {sharedsecret}  3``` in the file
   d. sudo nano /etc/pam.d/sshd
      -add the line ```auth sufficient pam_radius_auth.so``` at the top of the file (This locks down Ubuntu SSH deamon and forces the system to look to our infrastructure configuration for authentication)

3. sudo systemctl restart ssh    {To restart SSH service to update changes}

---------------------------------------------------------------------------------------------------------

STEP 3: Creating GPO

1. Server Manager -> Group Policy Management -> right click on domain name-> create new GPO ->name: Enterprise_Hardening_Policy ->right click and click on edit to open "Group Policy Management Editor"
   <img width="746" height="537" alt="IAM-gPO1" src="https://github.com/user-attachments/assets/16a5de1a-8d2d-4bff-84bf-b603e509d0ae" />

3. I set some password complexity rules in this GPO
   
   <img width="780" height="567" alt="IAM-Passwd complexity" src="https://github.com/user-attachments/assets/58889219-964b-42d8-814a-5a58b0e46a12" />

4. 	gpupdate /force   (Run this on the server's terminal to update rules)

-------------------------------------------------------------------------------------------------------

STEP 4: Installing Network Policy Server (NPS) on the Windows Server 2025 machine to act as a RADIUS engine.

1. Server Manager -> Add Roles and features -> -> click next until server roles page -> Select Network Policy and Access Services -> click next & Install.
   To veryfy passwords against AD account, I did the following:
2. Now go to Network Policy server in the server manager -> In the NPS Management Window click on NPS(local) ->Select Regester Server in Active Directory ->Click OK
   <img width="782" height="557" alt="IAM - RADIUS setup" src="https://github.com/user-attachments/assets/a6ef11e7-779e-4ef5-bea6-f7bb9b7dee1e" />

3. Now on Network Policy Server Window click on RADIUS Clients and Servers folder -> Radius clients and select new -> details to fill (Friendly name: Ubuntu_Client, Address: {ubuntu IP address}, Shared Secret: {sharedsecret}[same one used above in ubuntu configuration] -> click OK  (This tells the server which devices are allowed to send it authentication requests (based on shared secret) )

   Creating Policy so server can log the Ubuntu Machine
4. Open "Network Server Policy" console -> expand policies folder ->right click on 'network policies' -> select new -> Name policy "Allow_Ubuntu_Client" -> next
5. On the Conditions screen, click Ad -> Scroll down, select Windows Groups, and click Add -> Click Add Groups..., type 'Radius_Users' -> OK-> Click Add again to add a second condition-> select Client Friendly Name, and click Add-> Type Ubuntu_Client (this must match the exact friendly name you gave your Ubuntu VM in the previous step) and click OK -> Next
6. On the "Specify Access Permission" screen, ensure Access granted is selected -> Next
Authentication Method
7. Uncheck any default boxes that are selected (like Microsoft-CHAPv2) -> Check the box for Unencrypted authentication (PAP, SPAP) -> Next
<img width="682" height="601" alt="IAM - auth allow" src="https://github.com/user-attachments/assets/e03724aa-e38a-4228-893a-3d09aa54c893" />

Yay everything is set up!
   Now go to Ubuntu VM and run ```sudo secadmin@localhost``` to login into the secadmin account.


TroubleSHooting before next step:<img width="782" height="557" alt="IAM - RADIUS setup" src="https://github.com/user-attachments/assets/f76b1f77-8365-42cc-a15a-fb35509bf86b" />

1) If you can't login after all this, on Ubuntu run ```sudo nano /etc/pam_radius_auth.conf``` and comment out other IP addresses you see
2) Then restart SSH : ```sudo systemctl restart ssh```
3) ssh secadmin@localhost   [this should work]

-------------------------------------------------------------------------------------------------------

STEP 5: MFA

On ubuntu VM
1.	```sudo apt install libpam-google-authenticator -y```
2.	```google-authenticator ``` (generates token/QR code - scan the qr code usning google authenticator app on smartphone) 
3.	run ```sudo nano /etc/pam.d/sshd```  (Forconfiguring Ubuntu to stack this token upon login)
     - add this a the bottom of the file: ```auth required pam_google_authenticator.so``` and save file
4. now run ```sudo nano /etc/ssh/sshd_config``` (used to enable MFA tokens during SSH)
   - add ``` KbdInteractiveAuthentication yes 
             ChallengeResponseAuthentication yes
             PasswordAuthentication yes
             ```
     - also add ``` AuthenticationMethods password,keyboard-interactive``` at the bottom of the file
    
5. run	```sudo sshd -t``` this command to verify the ssh configuration file is good with no errors.
6. sudo systemctl restart ssh
7. ssh secadmin@<ubuntu_ip>
   -This asks for password first, then asks for OTP. Input these and you'll get in.

   

   




 

      

      

   
