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
    - Admins (OU)
      - mstil01-admin
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
    - mstil01-admin (Group Member)
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

### Domain Controller Setup

After setting up the operating systems in VirtualBox, the first step will be to sign into the Windows Server 2022 VM and configure the IP address information from the planned network.
These manual configurations include:
- An IPv4 address for the Server.
- An IPv4 subnet mask.
- An IPv4 gateway.
- An IPv4 DNS server pointing to itself (192.168.40.66) or the loopback address (127.0.0.1).
  - This is stated as a best practice by Microsoft. "If the server is the first and only domain controller that you install in the domain, and the server runs DNS, configure the DNS client settings to point to that first server's IP address" (Microsoft, n.d.).
<img width="301" height="178" alt="AD_Lab_IMG_1" src="https://github.com/user-attachments/assets/ed3510d3-f576-45a3-b70b-f8db0d07cb98" />

Next, I changed the hostname to dc01 within Server Manager.
- Changing the server's hostname post-promotion requires either demoting and re-promoting the server or using netdom in the command prompt. 

Once these pre-promotion tasks were complete, the Active Directory Domain Services and DNS server roles were ready to be installed.<br><img width="250" height="119" alt="AD_Lab_IMG_3" src="https://github.com/user-attachments/assets/759afdc0-c37e-4904-bac5-bab893b3312c" /><br><img width="284" height="226" alt="AD_Lab_IMG_7" src="https://github.com/user-attachments/assets/c67c5bf4-b395-49b8-b0a1-c08892749961" />

After installing the roles, the domain controller was promoted.<br><img width="169" height="80" alt="AD_Lab_IMG_4" src="https://github.com/user-attachments/assets/bf2963b3-1de7-4976-b5ef-e9931c878baa" /><br><img width="378" height="149" alt="AD_Lab_IMG_6" src="https://github.com/user-attachments/assets/d91c28ba-a633-4566-b922-46a984f53d8a" /><br>
Although the above image shows me using the root domain name "arasaka.example", I ended up having to revert the server VM to a pre-promotion snapshot and change the top-level domain to ".internal". See the Lessons Learned section for more details.

Post-promotion, I configured OU hierarchies for domain users, groups, and computers and placed the appropriate Active Directory Objects within those hierarchies using ADAC on the domain controller.<br><img width="290" height="106" alt="AD_Lab_IMG_25" src="https://github.com/user-attachments/assets/d7108eb9-286c-47b7-8c3e-e106e8d0a158" /><br>See the Prerequisites section for more details regarding the OU hierarchies and Active Directory objects used in this lab.

I then configured a separate Windows Server 2022 VM as the DHCP server for the domain. See the DHCP lab for more details.

### Joining the Human Resources workstation to the domain

Starting with the Windows 10 Pro workstation for the Human Resources department (NC-HR-01), I attached the VM's network adapter to the Lab network I created using VirtualBox.<br><img width="584" height="383" alt="Screenshot 2026-10-28 134413" src="https://github.com/user-attachments/assets/c707f638-6cab-4882-919d-5f71b9fe39ee" /><br><img width="164" height="84" alt="Screenshot 2026-10-28 134526" src="https://github.com/user-attachments/assets/d84faf4f-c9af-4a7d-9cc5-de3a599951fe" /><br>The adapters for the domain controller and the DHCP server are also attached to this Lab network.<br><img width="580" height="381" alt="AD_Lab_IMG_22" src="https://github.com/user-attachments/assets/9ec77353-640c-4832-a8e3-1a155112870a" /><br><img width="578" height="382" alt="AD_Lab_IMG_23" src="https://github.com/user-attachments/assets/436b6346-97ab-41d2-b228-f25dcdc35ac5" /><br>Placing all of these VMs in the Lab network allows the Windows 10 VMs to receive an automatic IP configuration from the DHCP server.

To verify that the IP address, subnet mask, gateway address, and DNS server were properly configured by DHCP, I signed in to the Windows 10 VM and used the "ipconfig /all" command in PowerShell.<br><img width="287" height="203" alt="AD_Lab_IMG_8" src="https://github.com/user-attachments/assets/4a5339a7-302a-4338-aba4-7b31b38e88a6" /><br>As we can see from the image, the IP information configured on the VM lines up with the networking information and forward lookup table in the Prerequisites section.

Now that we've confirmed that the DHCP configuration is correct, the connection between the domain controller and the VM should be verified before attempting to join the VM to the domain.<br><img width="224" height="167" alt="AD_Lab_IMG_9" src="https://github.com/user-attachments/assets/881e2555-ab4d-4f1d-9ff8-55c25c7deef8" /><br>This image shows us that the VM (192.168.40.70) was able to reach the domain controller (192.168.40.66) and the DHCP server (192.168.40.67).

Since both the IP configuration and reachability have been verified, it is time to domain-join the VM to the "arasaka.internal" domain. On this VM, I decided to use the Graphical User Interface (GUI) to domain join the device using these steps:
1. Right-click the Start menu and select "Settings"
2. Type and select "View advanced system settings" in the search box
3. In the System Properties pop-up:
   1. Navigate to the Computer Name tab
   2. Select "Change…" next to the "To rename this computer or change its domain…" text
4. In the Computer Name/Domain Changes pop-up:
   1. Enter the computer name in the "Computer name:" text box<br><img width="161" height="194" alt="AD_Lab_IMG_10" src="https://github.com/user-attachments/assets/114cb115-05bf-478e-b54f-2943b7612296" />
   2. Select the "Domain:" radio in the "Member of" box and type the domain in the text box
   3. Select "OK"
5. In the Windows Security pop-up:
   1. Type the username and password for the domain administrator.<br><img width="226" height="148" alt="AD_Lab_IMG_10 1" src="https://github.com/user-attachments/assets/abba96a9-edf7-4e17-8755-b8598033aa91" />
   2. Select "OK"
6. Select "OK" or "Close" on all pop-up boxes<br><img width="151" height="73" alt="AD_Lab_IMG_11" src="https://github.com/user-attachments/assets/fe0ea46a-0c9d-4445-b61f-1874dd23cf1b" />
7. In the Microsoft Windows pop-up: Select "Restart Now"
8. In the Sign In window: Select Other user
9. Sign in using a domain account<br><img width="159" height="110" alt="AD_Lab_IMG_12" src="https://github.com/user-attachments/assets/75d0bd7a-6ace-4d7c-9317-4d9b3076332e" />

Since I pre-made the NC-HR-01 computer object before domain-joining the VM to "arasaka.internal", I was able to verify that the device correctly inherited the object by viewing the computer object's properties using ADAC on the domain controller.<br><img width="892" height="91" alt="AD_Lab_IMG_13" src="https://github.com/user-attachments/assets/286e3e14-7e7e-4ac7-b0c0-97ab469bfc18" /><br>Properties that were not populated before such as the OS name and OS version are now populated, showing us that the VM inherited the pre-made object.

### Joining the Sales workstation to the domain

I performed similar steps with the Windows 10 Pro workstation for the Sales department (NC-Sales-01). First, I placed the VM on the Lab network in VirtualBox.<br><img width="580" height="383" alt="AD_Lab_IMG_24" src="https://github.com/user-attachments/assets/e0be5fc4-9b2b-4984-8bea-bf2c45704943" /><br>Next, I confirmed that the DHCP configuration on the VM was correct.<br><img width="302" height="203" alt="AD_Lab_IMG_14" src="https://github.com/user-attachments/assets/12236b18-ebc0-44a6-81ca-15fc12f2295d" /><br>Then, I verified reachability to the domain controller.<br><img width="266" height="167" alt="AD_Lab_IMG_17" src="https://github.com/user-attachments/assets/ac7a181e-7c89-409d-8fb5-629997f2e177" />

After completing the network connectivity validation on the Sales department workstation, I decided to domain-join the device to "arasaka.internal" using PowerShell instead of the GUI by following these steps:
1. Right-click the Start menu and select "Windows PowerShell (Admin)"
2. In PowerShell:
   1. Rename-Computer -NewName "[new hostname]" -Restart<br><img width="263" height="38" alt="AD_Lab_IMG_16" src="https://github.com/user-attachments/assets/50be1072-d50c-4f7b-8520-2ab3d1b62fea" />
      1. This cmdlet gives the computer a new hostname and restarts the device.
3. Log back in using the local account's credentials
4. Right-click the Start menu and select "Windows PowerShell (Admin)"
5. In Powershell:
   1. Add-Computer -DomainName "[domain name]" -Credential "[domain administrator username]"<br><img width="362" height="104" alt="AD_Lab_IMG_15" src="https://github.com/user-attachments/assets/57b97e18-6267-4dfb-be4e-473e20cb299d" />
      1. This cmdlet adds the device to the domain.
      2. Domain administrator username format: [NetBIOS domain name]\\[username]
6. In the Windows PowerShell credential request pop-up:
   1. Type the password in the text box
   2. Select "OK"
7. In PowerShell:
   1. shutdown /r
8. In the Sign In window: Select Other user
9. Sign in using a domain account

Once these steps were complete, I used the "ipconfig /all" command to verify that the changes to the hostname and domain membership were implemented correctly.<br><img width="309" height="203" alt="AD_Lab_IMG_18" src="https://github.com/user-attachments/assets/5c0a0494-0eb2-4781-8db8-66fca18a98fa" /><br>From the output, we can verify that the hostname was changed in the Host Name row. We can also verify that the device successfully joined the domain since the Primary DNS Suffix row is populated with the domain name.

Back on the domain controller, we can see that the VM inherited the correct computer object.<br><img width="894" height="97" alt="AD_Lab_IMG_19" src="https://github.com/user-attachments/assets/37804939-bd1e-45e4-939a-d690f3844137" />


## Lessons Learned

- Do not use .example as the top-level domain (TLD). This TLD caused major issues with the creation of forward lookup zone records upon installing the DNS server role on the domain controller.
  - Solution: Use .internal or .com as the TLD.

## Sources

Microsoft. (n.d.). *Best practices for DNS client settings in Windows Server.* Microsoft Learn. Retrieved October 28, 2026, from https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/best-practices-for-dns-client-settings
