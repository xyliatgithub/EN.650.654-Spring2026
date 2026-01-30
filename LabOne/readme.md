# Lab 1 - Local DNS Attack and Detection

In this lab assignment, you will need to setup multiple SEED lab VMs and perform an DNS attack. In addition to those tasks required by the SEED lab documentation, you also need to finish the additional tasks described below.

### Setup  

## Instructions for Intel/AMD Machines x86-64

1. **Download the Pre-built VM Image**  
   - Visit [SEED Labs Setup](https://seedsecuritylabs.org/labsetup.html) to download the pre-built VM image (Ubuntu 16.04, 32-bit).
   - <img width="600" alt="image" src="https://github.com/user-attachments/assets/f603b6bf-57bd-4042-8798-1cfa72f5336b" />



2. **VM Setup Instructions (in VirtualBox)**  
   - Download and install [Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads)
   - Refer to the [SEED VM VirtualBox Manual](https://seedsecuritylabs.org/Labs_16.04/Documents/SEEDVM_VirtualBoxManual.pdf) for detailed instructions on setting up the VM in VirtualBox.
   - This manual also contains account information, such as usernames and passwords.
  
> [!NOTE]
> If you encounter the following error while launching VM in VirtualBox:
>
> <img width="400" alt="image" src="https://github.com/user-attachments/assets/872a9ced-87bd-4b3f-bfc3-91d65a090acd" />
>
> Try "Headless Start"
>
> <img width="400" alt="image" src="https://github.com/user-attachments/assets/b68749c3-c46d-4928-9388-525ca1043f48" />
>
> If issue persists convert VirtualBox VM to VMware VM. For details on converting, refer to to this [manual](https://knowledge.broadcom.com/external/article/341189/importing-virtual-machine-from-oracle-vi.html). 


  
## Instructions for Apple Computer with ARM M Chips

1. **Download and Install the VMWare Software**
   - Create [Broadcom account](https://profile.broadcom.com/web/registration) to download VMWare software.
   - Download and install [VMWare Fusion Pro 25H2](https://support.broadcom.com/group/ecx/productfiles?subFamily=VMware%20Fusion&displayGroup=VMware%20Fusion%2025H2&release=25H2&os=&servicePk=&language=EN&freeDownloads=true).
     > Refer to to SEED Lab [Setup instructions](https://github.com/seed-labs/seed-labs/blob/master/lab-setup/apple-arm/seedvm-v2/SeedVM-Fusion_Installation.md) for more details.

2. **Download the Ubuntu 22.04 Image**  
   - Download the [Ubuntu 22.04.5 LTS](https://cdimage.ubuntu.com/ubuntu/releases/22.04/release/). Make sure you download the 64-bit ARM (ARMv8/AArch64) server install image.
   - <img width="600" alt="image" src="https://github.com/user-attachments/assets/366896d5-56e0-4109-bd17-14bbe3bd39de" />

   - Follow the instructions in the [SEED Labs VM Setup](https://github.com/seed-labs/seed-labs/blob/master/lab-setup/apple-arm/seedvm-v2/SeedVM-Ubuntu_Installation.md).  
     - **Please note that there may be some differences on Ubuntu 22.04 compared to Ubuntu 16.04 using VirtualBox**.
       
## Instructions to setup labs at MSSI Lab Computers
   - If you encounter any difficulties, consider using the computers available in the MSSI lab (Malone 316).
   - Current MSSI student are automatically enrolled to access lab computers.
   - Workstations are accesible both in-person and remotely with Remote Desktop Protocol by [VPN](https://wiki.isi.jhu.edu/index.php?title=VPN). For more information please refer to this ISI Wiki [article](https://wiki.isi.jhu.edu/index.php?title=MSSI_General-Use_Workstations#Connecting_Remotely).
   - In case of inability to access lab equipment or any help with lab computers arise refer to this [manual](https://livejohnshopkins-my.sharepoint.com/:b:/g/personal/abissar1_jh_edu/IQCAfvchEfJPRpwhTuJaiNHlAWayD5oXtq4uTbgDvGXMSdw?e=qlS5ga), [ISI Wiki](https://wiki.isi.jhu.edu/index.php?title=Category:MSSI_Lab) or contact [support](mailto:isisupport@jh.edu).


## Instructions

1. You will need to set up **three VMs** connected to the same local network.  
   - Ensure these VMs are configured in **promiscuous mode** to listen to network traffic from other VMs.  
   - After configuring one VM, you can simply clone it **two additional times** to complete the setup.  

2. Please follow the instructions in the DNS_Local PDF file to complete all the 9 tasks. The first three tasks are basically the environment and DNS setup.
> [!IMPORTANT]
> You will not be able to complete all the tasks without proper setup. If you encounter any problem, please reach out to the CA for help ASAP. 

3. Please complete the following **additional tasks**:  
-	In **Task 5 - Directly Spoofing Response to User**, use tcpdump on the "User" machine to capture all the DNS packets.  
-	Are you able to use tcpdump to specifically capture only those spoofed DNS packets?  Explain why or why not clearly. If not, how do you detect such attacks that may include additional processing steps? Please specifically describe your attempts.  

## Notes

**Something needs to be noticed in order to successfully run this lab:**
- Task 3: If step 1 (create zones) did not work with you, you may add the zone entries to /etc/bind/**named.conf.local** file insted of /etc/bind/**named.conf** file.
- Task 5: If the attack is not successful at first, it is probably that the request you sent using netwox does not arrive at the user's machine before the local DNS server's packet. You can try to use dig to send more requests on the user machine while running netwox.
- Task 6: We use Netwox 105 to spoof the response to DNS server. The filter field setting in the instruction is incorrect. It should be set to "src host [your local DNS server IP]". 
- Task 7: To improve the attack success rate, you can modify the final line of the program to only respond to packets from the server: pkt = sniff(filter='udp and dst port 53 and src <your DNS server's address>', prn=spoof_dns).
- Task 8 & 9: If you don't attack successfully, maybe you need to flush the cache and retry the DNS request multiple times.

## Required Files for DNS Setup

**Zone Files for DNS Setup:**
- Zone file for domain example.com: https://seedsecuritylabs.org/Labs_16.04/Networking/DNS_Local/example.com.db
- Zone file for DNS reverse lookup: https://seedsecuritylabs.org/Labs_16.04/Networking/DNS_Local/192.168.0
- Note: If you choose different IP addresses or domain names, you need to modify the above configuration and zone files accordingly.

## Grading (50 points)
Please take screenshots periodically and regularly and include them in your report. They not only serve as evidence of completion but also help the grader understand what you try to achieve. Add adeuqate explaination as needed. See the lab submission example for what it should look like.
* Completeness (35 pts): All the steps as instructed in the lab manual must be included in the report with adequate evidence.
* Presentation (15 pts): The report must be clear and correct in organization and writing with adequate explanation.

