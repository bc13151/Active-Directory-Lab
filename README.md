# Overview
Active Directory (AD) is a directory service developed by Microsoft for Windows Server environments. It provides centralized management of users, computers, and other resources within a network domain. Active Directory authenticates users, enforces security policies, and enables administrators to efficiently manage permissions, access, and configurations.

In this lab, the following diagram will be used to configure network functionality on a virtual domain using OracleVM. Domain users will be automatically added to the domain using a PowerShell script.

<img src=https://i.imgur.com/SJt6aht.png height="400" width="400"> <br>
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

Assign the servers internal NIC a static IP address of 172.16.0.1 with a subnet mask of 255.255.255.0, which places all addresses in the range 172.16.0.1–172.16.0.254 on the same subnet. Although a larger range would be necessary for the 1,000 devices planned to be provisioned on the domain, I quite frankly completely forgot, and only realized after writing this.

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

Add DHCP to the server and specify a scope. This allows the server to automatically assign IP addresses to devices on the network based on the provided range. Doing this allows domain devices to seamlessly and dynamically obtain an IP address.

<p align="center">
<img src=https://i.imgur.com/d1tY3gN.png height="500" width="500">
<img src=https://i.imgur.com/94absAz.png height="500" width="500">
</p>

Configure the IP address of the router as the external NIC.

<img src=https://i.imgur.com/9utdoie.png height="500" width="500">
<img src=https://i.imgur.com/59l4REa.png height="500" width="500">
<img src=https://i.imgur.com/R2E1BX1.png height="500" width="500">
<img src=https://i.imgur.com/6qfwa7B.png height="500" width="500">
<img src=https://i.imgur.com/iQ50Na8.png height="500" width="500">
<img src=https://i.imgur.com/mNADrTP.png height="500" width="500">
<img src=https://i.imgur.com/vkAcPgD.png height="500" width="500">
<img src=https://i.imgur.com/d03zayc.png height="500" width="500">
<img src=https://i.imgur.com/Sr3AvYj.png height="500" width="500">
<img src=https://i.imgur.com/X4JwIwQ.png height="500" width="500">
