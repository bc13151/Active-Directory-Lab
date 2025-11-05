# Overview
Active Directory is a directory service developed by Microsoft for Windows Server environments. It provides centralized management of users, computers, and other resources within a network domain. Active Directory authenticates users, enforces security policies, and enables administrators to manage permissions, access, and configurations.

In this lab, the following diagram will be used to configure network functionality on a virtual domain using OracleVM. Domain users will be automatically provisioned to the domain using a PowerShell script, and the client device will be configured to join the domain.

<img src=https://i.imgur.com/SJt6aht.png height="700" width="700"> <br>
## 1. Setting up the Virtual Machines
Before configuring this virtual domain, the following downloads are needed:
1. <a href="https://www.virtualbox.org/wiki/Downloads">Install Oracle Virtual Box</a><br>
2. <a href="https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019">Install Windows Server 2019</a><br>
3. <a href="https://www.microsoft.com/en-us/software-download/windows10">Install Windows 10 ISO</a><br>

The Windows Server 2019 and Windows 10 virtual machines can be created using the default configuration settings in Oracle VM VirtualBox. However, because the Windows Server will function as a Domain Controller, it requires two network adapters to support both domain connection and Internet access for domain devices.

The first network adapter should be configured for Network Address Translation (NAT) to provide Internet access. The second adapter should be configured as an Internal Network with the static IP address 172.16.0.1, which will serve as the internal gateway and DNS server for the domain.

The Windows 10 client VM only needs a single Internal Network adapter so that it can communicate with the Domain Controller and access domain resources.

## 2. Configure Domain Network
Boot the VM containing the Windows Server 2019 image and complete the installation. While the server is now created, it must first be configured to allow devices to connect to the network and the domain. 

The servers internal NIC can be assigned with a static IP address of 172.16.0.1 with a subnet mask of 255.255.255.0, which places all addresses in the range 172.16.0.1–172.16.0.254 on the same subnet. Although a larger range would be necessary for the 1,000 devices planned to be provisioned on the domain, I quite frankly completely forgot, and only realized after writing this.

The DNS server is set to 127.0.0.1, the loopback address. This tells the server to use its own DNS service, which is required because the server is acting as the Domain Controller, and Active Directory relies on DNS to locate domain services and authenticate users.

<p align="center"><img src=https://i.imgur.com/vum9juR.png height="400" width="400"></p>

Add Domain Services through Server Manager and configure the domain.

<p align="center">
<img src=https://i.imgur.com/1p1eurD.png height="500" width="500">
<img src=https://i.imgur.com/ZOaTubn.png height="500" width="500">
</p>

Add the Routing and Remote Access (RRAS) role to the server, then enable Network Address Translation (NAT). This allows devices on the internal domain network to access the internet through the server's external adapter.

<p align="center">
<img src=https://i.imgur.com/G6mroj3.png height="500" width="500">
<img src=https://i.imgur.com/HlbMsOf.png height="500" width="500">
</p>

Add DHCP to the server and specify a scope. This allows the server to automatically and dynamically assign IP addresses to devices on the network based on the provided range.

<p align="center">
<img src=https://i.imgur.com/d1tY3gN.png height="500" width="500">
<img src=https://i.imgur.com/94absAz.png height="500" width="500">
</p>

Configure the default gateway, or routing address, as the IP of the server. This allows devices on the internal network to route to the server and obtain a NAT address for internet connection.

<p align="center">
<img src=https://i.imgur.com/9utdoie.png height="500" width="500">
</p>

Now that the network is configured, users can connect to the domain and access shared resources. However, each user requires a domain account to establish their identity and permissions within the network. The next step is to provision user accounts on the server, which can be automated using PowerShell.

## 3. Provisoning User Accounts
User accounts can be added in bulk using a PowerShell Script. The script used in this lab can be found <a href="https://github.com/joshmadakor1/AD_PS/blob/master/1_CREATE_USERS.ps1">here</a>. Download and the script, open it in PowerShell ISE, and run it.

<p align="center">
<img src=https://i.imgur.com/R2E1BX1.png height="1000" width="1000">

This PowerShell script automates the creation of user accounts in Active Directory. Below is a breakdown of how each part functions:

``$PASSWORD_FOR_USERS   = "Password1"`` Sets the default password for all new user accounts to Password1. <br>

``$USER_FIRST_LAST_LIST = Get-Content .\names.txt`` Reads each line from the names.txt file, which contains a list of user first and last names. <br>

``$password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force`` Converts the plaintext password into a secure string that Active Directory can use. <br>

``New-ADOrganizationalUnit -Name _USERS -ProtectedFromAccidentalDeletion $false`` Creates a new Organizational Unit named _USERS to store the new accounts. <br>

``foreach ($n in $USER_FIRST_LAST_LIST) {`` Starts a loop that processes each line (user) from the names list. <br>

``$first = $n.Split(" ")[0].ToLower()`` Extracts the first name from the line and converts it to lowercase. <br>

``$last = $n.Split(" ")[1].ToLower()`` Extracts the last name and converts it to lowercase. <br>

``$username = "$($first.Substring(0,1))$($last)".ToLower()`` Creates a username using the first initial of the first name followed by the full last name. This is common naming convention for domain accounts. <br>

``Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan`` Displays each username being created in the PowerShell console. <br>

Sets the first name, last name, display name, employee id, password expiry, organizational unit, and enables the account.

`New-AdUser -AccountPassword $password ` <br>
          ` -GivenName $first ` <br>
          ` -Surname $last ` <br>
          ` -DisplayName $username ` <br>
          ` -Name $username ` <br>
          ` -EmployeeID $username ` <br>
          ` -PasswordNeverExpires $true ` <br>
          ` -Path "ou=_USERS,$(([ADSI]"").distinguishedName)" ` <br>
          ` -Enabled $true` <br>
`}` Closes the loop.

The output will display each created username in PowerShell. Once complete, Active Directory will show all newly created user accounts under the _USERS organizational unit.

<p align="center">
<img src=https://i.imgur.com/YKbGpv0.png height="350" width="500">
<img src=https://i.imgur.com/iQ50Na8.png height="500" width="500">
</p>

## 4. Setting Up Client Devices
Now that users have been added to the domain, the client devices need to be configured to connect to it. Set the client’s network adapter to use the internal network, complete the installation, and configure the adapter settings to use the server’s IP address as the preferred DNS server. This allows the client device to locate the Domain Controller and successfully join the domain.

<p align="center">
<img src=https://i.imgur.com/mNADrTP.png height="500" width="500">
<img src=https://i.imgur.com/vkAcPgD.png height="300" width="500">
</p>

Join the domain through System configuration. <br>

<p align="center"><img src=https://i.imgur.com/d03zayc.png height="500" width="500"></p>

The client device has now successfully joined the domain, verified through Active Directory. DHCP is functioning correctly, and an IP address has been automatically leased to the client. The device appears under Active Directory Users and Computers as a known computer within the domain. Any user with a domain account can now sign in to this device and access shared domain resources.

<p align="center">
<img src=https://i.imgur.com/Sr3AvYj.png height="500" width="500">
<img src=https://i.imgur.com/X4JwIwQ.png height="500" width="500">
</p>

## Conclusion

The purpose of this lab was to gain hands-on experience in setting up an Active Directory environment and to become familiar with how devices connect to and interact with a domain. Many labs offered through CompTIA CertMaster use Active Directory to manage user accounts for IAM operations, or set up network functionality for general day to day IT tasks. Completing this lab helped me familiarize myself with how these processes work in practice.
