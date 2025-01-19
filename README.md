# 🧠 Technologies Used
- Active Directory Domain Services
- Microsoft Azure (Cloud computing)
- Microsoft Remote Desktop
- Powershell

#  ⚙️ Deployment & Configuration
### 🖥️ [Create both Azure Virtual Machines](https://github.com/drewmarsh/azure-creating-VM)
- **Client-1** running Windows 10 Pro, version 22H2 - x64 Gen2
- **DC-1** running Windows Server 2022 Datacenter Azure Edition - x64 Gen2

*Note:* Put both of these Azure Virtual Machines into a Resource Group called DC-1_group

### Configure static IP address on Domain Controller 1 (DC-1)
Open the DC-1 virtual machine and navigate to Networking > Network settings > The name of the network interface (ex. dc-1410) > IP configurations > Select 'ipconfig1' > Change ```Dynamic``` to ```Static``` > Click `Save`

<img src="/images/static-ip.png" alt="Static IP">

### RDP into DC-1 and enable ICMP inbound/outbound

After the domain controller fulling initiliazes, minimize the Server Manager window for now.

Next, use windows search to open 'Windows Defender Firewall'. Select `🛡️ Advanced settings`

Navigate to the 'Inbound Rules' tab and find the two rules labled 'Core Networking Diagnostics - ICMP Echo Request (ICMPv4-In)'. Right-click it and select ```Enable``` on both of these rules.

<img src="/images/inbound-rules.png" alt="Inbound Rules">

### Install Active Directory on DC-1
Open Serving Manager back up and click 'Manage' > 'Add Roles and Features' > Click 'Next' 3 times to get to the Server Roles tab > Enable ☑️ Active Directory Domain Services > Click ```Add Features``` > Click 'Next' 3 more times until clicking the ```Install``` button > When it finishes installing, click ```Close```

<img src="/images/tick-ad-domain-services.png" alt="Tick AD Domain Services">

<img src="/images/install-ad.png" alt="Install AD">

In the top-right of the Server Manager window, click the flag icon with a warning notification and select 'Promote this server to a domain controller'

<img src="/images/promote-server.png" alt="Promote Server to a DC">

Then, the Active Directory Domain Services Configuration Wizard window will appear. On the 'Deployment Configuration' tab, under the 'Select deployment operation' section, tick the ☑️ Add a new forest options 

Right below that, enter "mydomain.com" in the 'Root domain name:' text field.

<img src="/images/deployment-config.png" alt="Deployment Config">

Click, ```Next``` and it will bring you to the 'Domain Controller Options' tab. Here, set a secure password of your choice in the 'Password:' and 'Confirm password:' text fields.

<img src="/images/set-password.png" alt="Set Password">

On the 'DNS Options' tab, ensure the following option is unchecked: ◻️Create DNS delegation

Continue clicking ```Next``` until the 'Prerequisites Check' tab is selected (it is normal for each tab to take a while to load). Then, click ```Install```.

<img src="/images/click-install.png" alt="Click Install">

<img src="/images/installing.png" alt="Installing">

When this process completes, your connection to the DC-1 virtual machine will be lossed and you will need to RDP back into it.

<img src="/images/computer-is-restarting.png" alt="Computer Is Restarting">

However, now that DC-1 has been promoted to Domain Controller, the username used in Remote Desktop Connection will be different than what was used before. ("mydomain\" must prescede the previously used username)

<img src="/images/new-credentials.png" alt="New Credentials">

When the credentials are accepted, it will take a moment to load back to the desktop while the system waits for the Group Policy Client

<img src="/images/wait-for-group-policy.png" alt="Wait for Group Policy">

### Add Organizational Units for Employees & Admins

In the top-right of the Server Manager window, navigate to 'Tools' > 'Active Directory Users and Computers'

<img src="/images/users-and-computers.png" alt="Users and Computers">

When the Active Directory Users and Computers window appears, right-click ```mydomain.com```  and then navigate to New > Organizational Unit

In the New Object - Organizational Unit window, enter "_EMPLOYEES" into the 'Name:` field and then click ```OK```

Repeat this same process except for this time, enter "_ADMINS" into the 'Name:` field and then click ```OK```

<img src="/images/employees.png" alt="_EMPLOYEES">

<img src="/images/admins.png" alt="_ADMINS">

### Add a new admin named Jane Doe

Next, right-click ```_ADMINS```  and then navigate to New > User

In the New Object - User window that will appear, enter the following information to the corresponding fields and then click ```Next```:
- First name: Jane
- Last name: Doe
- User logon name: Jane_admin

<img src="/images/new-user.png" alt="New User">

On the next page, enter a secure password for the Jane Doe admin.

Uncheck ◻️ User must change password at next logon

Check ☑️ Password never expires

Finally, click ```Next``` and then click ```Finish```

<img src="/images/jane-doe-password.png" alt="Jane Doe Password">

Back in the Active Direcotry Users and Computers window, right-click the 👤Jane Doe user and click 'Properties'. In the jane Doe Properties window, navigate to the 'Member of' tab and then click 'Add…'

In the 'Enter the object names to select (examples):' text field, enter "domain admins" and then click the ```Check Names``` button and then click ```OK``` to close this window.

In the Jane Doe Properties window, click ```Apply``` and then click ```OK```.

<img src="/images/domain-admins.png" alt="Domain Admins">

Now, you can log out of DC-1 and log back on as the new administrator account (Jane_admin).

<img src="/images/jane-logon.png" alt="Jane Logon">

<img src="/images/jane-welcome.png" alt="Jane Welcome">

### Add the private IP address of DC-1 as the custom DNS server for Client-1
Open the Client-1 virtual machine and navigate to Networking > Network settings > The name of the network interface (ex. client-1797) > DNS servers

In the 'DNS servers' tab, Change ```Inherit from virtual network``` to ```Custom```. Then, in the 'Add DNS server' text field, enter the private IP address of the DC-1 virtual machine. Click ```Save```.

Next, restart the Client-1 virtual machine.

<img src="/images/add-dns.png" alt="Add DNS">

### Join Client-1 to the domain

RDP into the Client-1 virtual machine and then set the preferred privacy settings before booting into the desktop.

Open a Command Prompt window and test if DC-1 can pinged from the client machine (ex. ping 10.0.0.4) and verify that there is a response (this works because of the ICMP rules that were enabled earlier)

<img src="/images/ping-DC.png" alt="Ping DC">

Open a Settings window and navigate to System > About > 'Rename this PC (advanced)' > ```Change…```

When the Computer Name/Domain Changes window appears, enter "mydomain.com" into the 'Domain:' text field and then click ```OK```. When prompted, enter the "mydomain.com\Jane_admin" username with the corresponding password and observe the message welcoming Client-1 to the mydomain.com domain. Next, restart the Client-1 virtual machine.

<img src="/images/change-pc-name.png" alt="Change PC Name">

<img src="/images/join-domain-with-admin-credentials.png" alt="Join Domain with Admin Credentials">

While Client-1 is restarting, RDP back into the DC-1 virtual machine with "Jane_admin" account and open a new Server Manager window. 

In the top-right of the Server Manager window, navigate to 'Tools' > 'Active Directory Users and Computers'

Under mydomain.com > Computers, the Client-1 virtual machine should be listed. 

<img src="/images/client1-computer.png" alt="Client-1 Computer">

### Set-up Remote Desktop for Non-administrative Users on Client-1

In Client-1, on the "mydomain.com\Jane_admin" account, open a Settings window.

Navigate to System > Remote Desktop > ```Select users that can remotely access this PC``` > ```Add…```

In the 'Enter the object names to select (examples):' text field, enter "domain users" and then click the ```Check Names``` button and then click ```OK``` to close this window.

<img src="/images/domain-users.png" alt="Domain Users">

Now, all the users in that security group have the permission to RDP into the Client-1 machine.

### Active Directory Bulk User Creation Script
RDP back into the DC-1 virtual machine with "Jane_admin" account and run Windows PowerShell ISE. When the window opens, click 'File' > 'New'. A file called "Untitled1.ps1" should appear within the PowerShell ISE window. In this window, write a script to bulk create Active Directory users, or use this one. Then, run the script using the green run Run Script button on the toolbar. After a desired amount of users have been created, click the red Stop Operation button on the toolbar.

<img src="/images/bulk-create-ad-users.png" alt="Bulk Create AD Users">

Now, if you naviagte to the Active Directory Users and Computers window, the newly created users can be observed under the '_EMPLOYEES' folder.

<img src="/images/added-users-confirmed.png" alt="Added Users Confirmed">

### Testing a Random Newly Created Users — Can Their Credentials RDP Into Client-1?

See if a one of the new users in the '_EMPLOYEES' folder in the Active Directory Users and Computers window can use RDP into the Client-1 machine. The script used to bulk create active directory users gave all of these users the password of "Password1".

<img src="/images/test-new-user.png" alt="Test New User">

<img src="/images/user-test-success.png" alt="User Test Success">
