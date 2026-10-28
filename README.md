# Active Directory lab

## Objective

This Active Directory lab seeks to emulate the management of a domain and domain objects.

### Project Overview/Skills Learned

- Installed Windows Server 2022 desktop edition and Windows 10 Pro as virtual machines (VMs) within VirtualBox.
- Provisioned resources such as CPU cores, RAM capacity, and storage for each VM using VirtualBox.
- Planned and documented a /28 local area network for VMs that will be a part of the domain using subnetting.
- Installed the Active Directory Domain Services (AD DS) and Domain Name System (DNS) server roles on the domain controller using Server Manager.
- Promoted a domain controller using Server Manager.
- Planned, documented, and implemented several Organizational Unit (OU) hierarchies to organize Active Directory Objects using Active Directory Administrative Center (ADAC) and Active Directory Users and Computers (ADUC).
- Created and managed Active Directory Objects (Domain Users, Security Groups, OUs, Computers) within the OU hierarchies using ADAC and ADUC.
- Joined several Windows 10 VMs to the domain.

### Tools Used

- VirtualBox for VM deployment and resource provisioning.
- Server Manager for installing roles on the domain controller and promoting the domain controller.
- Active Directory Administrative Center for managing Active Directory Objects.
- Active Directory Users and Computers for managing Active Directory Objects.
- PowerShell for validating changes, verifying connectivity, and troubleshooting.

## Prerequisites

This section's steps are out of scope for this project. However, this section provides important context behind the underlying infrastructure of the lab and some of the rationale behind my decision making.

Here are some of the steps I took before starting the project:

- I obtained the Operating System (OS) ISO files from Microsoft's website and installed the VirtualBox hypervisor.
- I planned the network that will be used by the domain computers. I decided to go with a /28 network as 14 usable IP addresses will be perfect for the lab environment that I am trying to create.

Network Address: 192.168.40.64/28<br>
Usable Address Range: 192.168.40.65-192.168.40.78<br>
Broadcast Address: 192.168.40.79<br>
Subnet Mask: 255.255.255.240

- Additionally, I planned out the IP addresses that the domain devices will receive:

| Virtual Devices | IPv4 Address |
| --------------- | ------------ |
| Router (Gateway) | 192.168.40.65 255.255.255.240 |
| dc01 (Windows Server 2022 Desktop Domain Controller and DNS server) | 192.168.40.66 255.255.255.240 |
| NC-DHCP-01 (Windows Server 2022 Core DHCP Server) | 192.168.40.67 255.255.255.240 |
| NC-HR-01 (Windows 10 Pro) | DHCP |
| NC-Sales-01 (Windows 10 Pro) | DHCP |
| NC-IT-01 (Windows 10 Pro) | DHCP |

- I planned out the OU hierarchies that would be implemented in ADAC. The objects within the OU hierarchies were organized with simplicity of management and scalability in mind.
  - The naming scheme for domain usernames is: First Initial + First Four Letters Of Last Name + Distinguishing Number.
  - The naming scheme for domain workstations is: Location + Department + Distinguishing Number.
  - The naming scheme for domain servers is: Location + Service + Distinguishing Number.
  - Groups were organized using AGDLP (Accounts, Global, Domain Local, Permissions).
    - **Domain Accounts** are members of **Global groups** that consolidate users who have similar characteristics such as department.
    - **Global groups** are members of **Domain Local groups** that are used to manage access to resources or to assign management rights and responsibilities.
    - **Domain Local groups** are used to control which **Permissions** a department's members have to network resources.

OU hierarchies:<br>
- Arasaka Users (OU)
  - HR (OU)
    - grodr01 (Domain User)
  - Sales (OU)
    - psmit01 (Domain User)
  - IT (OU)
    - mstil01 (Domain User)
- Arasaka Groups (OU)
  - HR-Users (Global Security Group)
    - grodr01 (Group Member)
  - Sales-Users (Global Security Group)
    - psmit01 (Group Member)
  - IT-Users (Global Security Group)
    - mstil01 (Group Member)
  - HR-Share-RW (Domain Local Security Group)
    - HR-Users (Group Member)
  - Sales-Share-RW (Domain Local Security Group)
    - Sales-Users (Group Member)
  - IT-Share-RW (Domain Local Security Group)
    - IT-Users (Group Member)
  - DHCP-Admins* (Universal Security Group)
    - mstil01 (Group Member)
- Arasaka Computers (OU)
  - Servers (OU)
    - NC-DHCP-01* (Domain Computer)
  - Workstations (OU)
    - NC-HR-01 (Domain Computer)
    - NC-Sales-01 (Domain Computer)
    - NC-IT-01 (Domain Computer)

\*Created as a part of the DHCP lab.

- I installed one instance of Windows Server 2022 desktop edition (dc01) to act as the domain controller and DNS server.
- I installed two instances of Windows 10 Pro (NC-HR-01, NC-Sales-01) that will be joined to the domain. 

## Steps

After setting up the operating systems in VirtualBox, the first step will be to sign into the Windows Server 2022 VM and configure the IP address information from the planned network.
These manual configurations include:
- An IPv4 address for the Server.
- An IPv4 subnet mask.
- An IPv4 gateway.
- An IPv4 DNS server pointing to itself (192.168.40.66) or the loopback address (127.0.0.1).
  - This is stated as a best practice by Microsoft. "If the server is the first and only domain controller that you install in the domain, and the server runs DNS, configure the DNS client settings to point to that first server's IP address" (Microsoft, n.d.).
<img width="301" height="178" alt="AD_Lab_IMG_1" src="https://github.com/user-attachments/assets/ed3510d3-f576-45a3-b70b-f8db0d07cb98" />

Next, I changed the hostname to dc01 within Server Manager.
- Changing the server's hostnam post-promotion requires either demoting and re-promoting the server or using netdom in the command prompt. 

Once these pre-promotion tasks were complete, the Active Directory Domain Services and DNS server roles were ready to be installed.<br>
<img width="250" height="119" alt="AD_Lab_IMG_3" src="https://github.com/user-attachments/assets/759afdc0-c37e-4904-bac5-bab893b3312c" /><br><img width="284" height="226" alt="AD_Lab_IMG_7" src="https://github.com/user-attachments/assets/c67c5bf4-b395-49b8-b0a1-c08892749961" />


After installing the roles, the domain controller was promoted.<br>
<img width="169" height="80" alt="AD_Lab_IMG_4" src="https://github.com/user-attachments/assets/bf2963b3-1de7-4976-b5ef-e9931c878baa" /><br><img width="378" height="149" alt="AD_Lab_IMG_6" src="https://github.com/user-attachments/assets/d91c28ba-a633-4566-b922-46a984f53d8a" /><br>
Although the above image shows me using the root domain name "arasaka.example", I ended up having to revert the server VM to a pre-promotion snapshot and change the top-level domain to ".internal". See the <a href="https://github.com/mstill00/Active-Directory-Lab/edit/main/README.md#lessons-learned">Lessons Learned</a> section for more details.



## Lessons Learned

- Do not use .example as the top-level domain (TLD). This TLD caused major issues with the creation of forward lookup zone records upon installing the DNS server role on the domain controller.
  - Solution: Use .internal or .com as the TLD.

## Sources

Microsoft. (n.d.). *Best practices for DNS client settings in Windows Server.* Microsoft Learn. Retrieved October 28, 2026, from https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/best-practices-for-dns-client-settings
