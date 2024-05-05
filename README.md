<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>Task</h2>

- Task 1. Creating a Resource Group and Deploying Windows 10 and Windows Server VMs.
- Task 2. How to ensure connectivity between the client and Domain Controller.
- Task 3. How to install Active Directory.
- Task 4. How to create an Admin and Normal User Account in AD.
- Task 5. How to join Client-1 to your domain (mydomain.com)
- Task 6. How to setup Remote Desktop for non-administrative users on Client-1
- Task 7. How to create a bunch of additional users and attempt to log into client-1 with one of the users

<h2>Deployment and Configuration Steps</h2>

Task 1: Creating a Resource Group and Deploying Windows 10 and Windows Server VMs.

Step 1: Create the Domain Controller VM (DC-1)

    Sign in to Azure Portal:
        Go to portal.azure.com and sign in with your Azure account.

    Navigate to Virtual Machines:
        Click on "Virtual machines" in the left-hand menu.

    Create New Virtual Machine for Domain Controller:
        Click "+ Add" to create a new virtual machine.
        Configure the following settings:
            Subscription: Choose your Azure subscription.
            Resource Group: Create a new resource group (e.g., MyResourceGroup).
            Virtual Machine Name: Enter DC-1.
            Region: Choose an Azure region.
            Image: Select "Windows Server 2022".
            Size: Choose an appropriate VM size.
            Administrator Account: Set username and password for the VM.
            Inbound port rules: Ensure that "RDP (3389)" is allowed.
            Networking: Create a new Virtual Network (VNet) or select an existing VNet.
            Subnet: Choose a subnet within the selected VNet.
            Public IP: Select "None" (since it's a Domain Controller).

    Review and Create:
        Review the configuration settings and click "Review + create".
        Click "Create" to deploy the Domain Controller VM.

Step 2: Set Domain Controller's NIC Private IP Address to Static

    Configure Network Interface:
        Once the VM deployment is complete, navigate to the "Networking" section of the VM.
        Click on the network interface (NIC) associated with DC-1.
        Under "Settings", click on "IP configurations".
        Select the IP configuration and change the "Assignment" to "Static".
        Enter a desired private IP address and subnet mask.
        Save the changes.

Step 3: Create the Client VM (Client-1)

    Navigate to Virtual Machines:
        Click on "Virtual machines" in the left-hand menu.

    Add New Virtual Machine for Client:
        Click "+ Add" to create a new virtual machine.
        Use the same settings as for the Domain Controller VM, but choose the same Resource Group (MyResourceGroup) and Virtual Network (VNet) that was created earlier.
        Name the VM as Client-1.
        Follow the same steps to configure size, administrator account, networking, and other settings.

![image](https://github.com/John-Duria/Config-On-Prem---ADDS---Azure/assets/168502429/db720e59-486a-426f-8dd0-ab9df32971c4)

How to ensure connectivity between the client and Domain Controller.
Step 5: Login to Client-1 and Initiate Perpetual Ping to DC-1

    Retrieve DC-1's Private IP Address:
        Navigate to the Azure Portal.
        Go to "Virtual machines" and select DC-1.
        Note down the private IP address of DC-1.

    Login to Client-1 using Remote Desktop (RDP):
        Open Remote Desktop Connection on your local machine.
        Enter Client-1's public IP address or DNS name.
        Click "Connect" and enter the credentials to log in to Client-1.

    Initiate Perpetual Ping to DC-1:
        Open Command Prompt on Client-1.
        Execute the following command to perform perpetual ping to DC-1's private IP address:

        bash

        ping -t <DC-1_Private_IP>

        Replace <DC-1_Private_IP> with the private IP address noted earlier.

Step 6: Login to Domain Controller (DC-1) and Enable ICMPv4 in Windows Firewall

    Enable ICMPv4 in Windows Firewall:
        Login to the Domain Controller (DC-1) using Remote Desktop (RDP).
        Open Windows Firewall with Advanced Security:
            Press Win + R, type wf.msc, and press Enter.
        In the Windows Firewall management console:
            Click on "Inbound Rules".
            Scroll down and locate the rule named "File and Printer Sharing (Echo Request - ICMPv4-In)".
            Right-click on the rule and select "Enable Rule" if it's currently disabled.
            This allows inbound ICMPv4 (ping) traffic to be processed by the Windows Firewall on DC-1.

Step 7: Verify Ping Success from Client-1 to DC-1

    Check Perpetual Ping on Client-1:
        Switch back to the Command Prompt on Client-1 where the perpetual ping to DC-1 was initiated.
        Observe the output of the ping command. You should start seeing successful ping replies from DC-1's private IP address.

    Verification of ICMPv4 Traffic:
        Return to the Remote Desktop session on DC-1.
        Open Command Prompt or PowerShell.
        Execute the following command to verify incoming ping requests:

        bash

ping localhost

If ICMPv4 is enabled successfully, you should see replies indicating that the ping traffic is now allowed through the Windows Firewall.

![image](https://github.com/John-Duria/Config-On-Prem---ADDS---Azure/assets/168502429/8c00b055-47cd-466e-87e7-dd739c6ca36e)

How to install Active Directory.

Step 8: Install Active Directory Domain Services on DC-1

    Login to DC-1 using Remote Desktop (RDP):
        Open Remote Desktop Connection on your local machine.
        Enter DC-1's public IP address or DNS name.
        Click "Connect" and enter the credentials to log in to DC-1.

    Install Active Directory Domain Services:
        Open Server Manager:
            Click on "Manage" and select "Add Roles and Features".
        Click "Next" until you reach the "Server Roles" section.
        Select "Active Directory Domain Services" and proceed with the installation by accepting the default settings.
        Click "Install" to start the installation process.
        Once installation completes, click "Close".

![image](https://github.com/John-Duria/Config-On-Prem---ADDS---Azure/assets/168502429/38114f57-84c7-488d-8748-700e11521a77)

Step 9: Promote DC-1 as a Domain Controller

    Promote DC-1 to a Domain Controller:
        In Server Manager, click on "Promote this server to a domain controller" (or launch dcpromo if prompted).
        Choose "Add a new forest" and enter the root domain name:
            Domain name: mydomain.com (or your desired domain name).
        Set the Directory Services Restore Mode (DSRM) password.
        Configure additional options as needed and proceed with the promotion.
        Review the configuration and click "Install" to promote DC-1 as a Domain Controller for the new forest.

    Complete Promotion and Restart:
        After the promotion completes, DC-1 will restart automatically.
        Wait for the server to restart and be ready for login.

![image](https://github.com/John-Duria/Config-On-Prem---ADDS---Azure/assets/168502429/06252808-43a9-4999-bdf4-d31f072506aa)


Step 10: Log in to DC-1 as User labuser from mydomain.com

    Login to DC-1 using Domain Credentials:
        After DC-1 has restarted, login again using Remote Desktop (RDP).
        In the login credentials, use the following format:
            Username: mydomain.com\labuser
            Password: Enter the password for the labuser account.
        Click "OK" to log in.

    Verify Domain User Login:
        Once logged in, verify that you are logged in as labuser from the domain mydomain.com.
        Open Server Manager or Active Directory Users and Computers to confirm the domain configuration and user account presence.

Task 4. How to create an Admin and Normal User Account in AD.

Step 11: Create an Organizational Unit (OU) called "_EMPLOYEES"

    Open Active Directory Users and Computers (ADUC):
        Login to DC-1 using Remote Desktop (RDP).
        Click on "Start", search for "Active Directory Users and Computers", and open the application.

    Create "_EMPLOYEES" Organizational Unit (OU):
        In the ADUC console:
            Right-click on the domain name (mydomain.com) and select "New" > "Organizational Unit".
            Name the new OU as _EMPLOYEES and click "OK" to create it.

Step 12: Create a new OU named "_ADMINS"

    Create "_ADMINS" Organizational Unit (OU):
        Similarly, create another new OU under the domain:
            Right-click on the domain name (mydomain.com) again.
            Select "New" > "Organizational Unit".
            Name the new OU as _ADMINS and click "OK" to create it.

Step 13: Create a New User Account "Jane Doe" (jane_admin)

    Create New User Account "jane_admin":
        Right-click on the _EMPLOYEES OU.
        Select "New" > "User".
        Enter the following details for the new user:
            First name: Jane
            Last name: Doe
            User logon name: jane_admin
            Set a password for the user account (same password as specified).

Step 14: Add "jane_admin" to the "Domain Admins" Security Group

    Add User to "Domain Admins" Group:
        Open the properties of the user account jane_admin.
        Go to the "Member Of" tab.
        Click "Add" to add a group membership.
        Type Domain Admins and click "Check Names" to verify.
        Click "OK" to add jane_admin to the "Domain Admins" group.

Step 15: Log out/close Remote Desktop and Log back in as "mydomain.com\jane_admin"

    Log Out from DC-1 and Log In as "jane_admin":
        Disconnect or log out from the Remote Desktop session to DC-1.
        Reconnect to DC-1 using Remote Desktop.
        In the login credentials, use:
            Username: mydomain.com\jane_admin
            Password: Enter the password for the jane_admin account.

Step 16: Use "jane_admin" as the Admin Account from now on

    Administer as "jane_admin":
        After logging in as jane_admin:
            Open administrative tools, such as Server Manager or Active Directory Users and Computers, to perform administrative tasks using the jane_admin account within the mydomain.com domain.

![image](https://github.com/John-Duria/Config-On-Prem---ADDS---Azure/assets/168502429/0a77fb15-1567-4425-9e0c-3e6e9c207925)

Task 5. How to join Client-1 to your domain (mydomain.com)

Step 17: Set Client-1's DNS Settings to DC's Private IP Address

    Navigate to Client-1's Networking Settings in Azure Portal:
        Login to the Azure Portal (portal.azure.com).
        Go to "Virtual machines" and select Client-1.

    Update DNS Settings:
        Under "Settings", click on "Networking".
        Click on the network interface (NIC) associated with Client-1.
        Under "Settings", click on "DNS servers".
        Set the DNS server to the private IP address of the Domain Controller (DC-1).
        Click "Save" to apply the DNS settings.

Step 18: Restart Client-1 from Azure Portal

    Restart Client-1:
        While still in the Azure Portal on the Client-1 page:
            Click on "Overview" or "Start" (depending on the Azure portal layout).
            Click "Restart" to restart Client-1 with the updated DNS settings.

Step 19: Join Client-1 to the Domain (mydomain.com)

    Login to Client-1 as Local Admin (labuser) and Join Domain:
        After Client-1 has restarted:
            Open Remote Desktop Connection and log in as the original local admin (labuser) using the local credentials.
            Press Win + R, type sysdm.cpl, and press Enter to open System Properties.
            Go to the "Computer Name" tab and click "Change".
            Select "Domain", enter mydomain.com in the domain field, and click "OK".
            Enter domain credentials when prompted (e.g., mydomain.com\jane_admin and password).
            Follow the prompts to join Client-1 to the domain. The computer will restart automatically.

Step 20: Verify Client-1's Presence in Active Directory

    Verify in Active Directory Users and Computers (ADUC):
        Login to the Domain Controller (DC-1) using Remote Desktop.
        Open Active Directory Users and Computers (ADUC):
            Click on "Start", search for "Active Directory Users and Computers", and open the application.
        Navigate to the "Computers" container under the domain (mydomain.com):
            Expand mydomain.com > "Computers".
        Verify that Client-1 appears in the list of domain-joined computers within the Computers container.

![image](https://github.com/John-Duria/Config-On-Prem---ADDS---Azure/assets/168502429/e8463d56-1698-4f89-b89e-acfebccd859d)
    
Task 6. How to setup Remote Desktop for non-administrative users on Client-1

Step 21: Log into Client-1 as mydomain.com\jane_admin and Open System Properties

    Login to Client-1 as jane_admin:
        Open Remote Desktop Connection and log in to Client-1 as mydomain.com\jane_admin using RDP.

    Open System Properties:
        Press Win + R, type sysdm.cpl, and press Enter to open System Properties on Client-1.

Step 22: Click "Remote Desktop" Tab in System Properties

    Navigate to Remote Desktop Settings:
        In the System Properties window:
            Click on the "Remote" tab.

Step 23: Allow "Domain Users" Access to Remote Desktop

    Enable Remote Desktop for Domain Users:
        Under the "Remote Desktop" section in System Properties:
            Click "Select Users..." to open the Remote Desktop Users dialog.
            Click "Add" to add users/groups that can access this computer remotely.
            Type Domain Users and click "Check Names" to verify.
            Click "OK" to add Domain Users to the list of users allowed for Remote Desktop access.
            Click "OK" again to close the Remote Desktop Users dialog.

Step 24: Login to Client-1 as a Normal, Non-Administrative User

    Log into Client-1 as a Non-Admin User:
        Logout of the current session (jane_admin).
        Login to Client-1 using Remote Desktop (RDP) as a normal, non-administrative user from the mydomain.com domain (e.g., mydomain.com\username).

Step 25: Consider Using Group Policy for Bulk Changes

    Implement Group Policy for Widespread Changes:
        While this tutorial demonstrates manual configuration for Client-1, for larger environments, use Group Policy to manage Remote Desktop settings across multiple systems:
            Open Group Policy Management Console (gpmc.msc) on the Domain Controller (DC-1).
            Create or edit a Group Policy Object (GPO) linked to the appropriate Organizational Unit (OU) containing client computers (Client-1 in this case).
            Configure Remote Desktop settings (e.g., allowing Domain Users to access Remote Desktop) within the GPO under Computer Configuration > Policies > Administrative Templates > Windows Components > Remote Desktop Services > Remote Desktop Session Host.
            Apply the GPO to the relevant OU to ensure consistent policy enforcement across multiple systems.

![image](https://github.com/John-Duria/Config-On-Prem---ADDS---Azure/assets/168502429/aea03024-7aad-4d2d-9402-52b7dea1cf6e)

Task 7. How to create a bunch of additional users and attempt to log into client-1 with one of the users

Step 26: Log into DC-1 as jane_admin

    Login to DC-1 as jane_admin:
        Open Remote Desktop Connection and log in to DC-1 as mydomain.com\jane_admin using RDP.

Step 27: Open PowerShell ISE as Administrator

    Open PowerShell ISE as Administrator:
        Click on Start or use the Search function to find PowerShell ISE.
        Right-click on PowerShell ISE and select "Run as administrator".

Step 28: Create a New File and Paste Script Contents

    Copy and Paste Script Contents:
        Download the script from the GitHub repository: Generate-Names-Create-Users.ps1.
        Open PowerShell ISE.
        Create a new script file:

        powershell

New-Item -Path "C:\Scripts\CreateUsers.ps1" -ItemType File

Open the newly created script file:

powershell

        ise C:\Scripts\CreateUsers.ps1

        Paste the contents of the downloaded script (Generate-Names-Create-Users.ps1) into the PowerShell ISE editor.

Step 29: Run the Script and Observe Account Creation

    Run the Script:
        Save the script file (CreateUsers.ps1) in PowerShell ISE.
        Run the script by clicking "Run Script" or pressing F5.
        Observe the script as it generates and creates new user accounts in Active Directory.

Step 30: Open ADUC and Observe Created Accounts

    Open Active Directory Users and Computers (ADUC):
        Click on Start and search for Active Directory Users and Computers.
        Open the ADUC console.

    Navigate to the Appropriate OU:
        Expand mydomain.com > "Users" or the specific Organizational Unit (OU) where the script creates new accounts.
        Verify that the newly created user accounts appear in the appropriate OU.

Step 31: Attempt to Log into Client-1 with New Account

    Attempt Login with New Account on Client-1:
        Logout of the current session on Client-1 if necessary.
        Use Remote Desktop Connection to log into Client-1.
        Enter the username and password of one of the newly created accounts (refer to the script for passwords).
        Attempt to log in and verify access using one of the newly created user accounts.

![image](https://github.com/John-Duria/Config-On-Prem---ADDS---Azure/assets/168502429/acd6ff20-93b6-47e3-995e-9639dc0b35c4)
