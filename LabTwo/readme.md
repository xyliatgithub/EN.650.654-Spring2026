# Lab 2 - Network Traffic & DoS Attack

In this lab, we will perform an experimental Denial-of-Service attack and collect network packets. In addition to those tasks required by the SEED lab documentation, you need to finish several additional tasks described below.

## Setup
> [!NOTE]
   > Please reuse the VM/environment from Lab 1. If you already finished Lab 1 setup, you can skip this setup section. Otherwise, follow: [Lab 1 – Environment Setup](https://github.com/xyliatgithub/EN.650.654-Spring2026/edit/main/LabOne/readme.md#setup).
- Visit [SEED Labs Setup](https://seedsecuritylabs.org/labsetup.html) to download the pre-built VM image (Ubuntu 16.04, 32-bit).
- VM setup instruction (in Virtualbox): this manual also contains account information (usernames and passwords) https://seedsecuritylabs.org/Labs_16.04/Documents/SEEDVM_VirtualBoxManual.pdf

In this lab, you need to have three VMs under the same local network. Once you have configured a VM, you can simply clone that VM for two more times to complete the VM setup. Please refer to Appendix A and B of the VM setup instruction.
> [!IMPORTANT]
> Note that these three VMs should be in the promiscuous mode in order to listen to traffics from other VMs.


## Lab Instructions 

1. Please follow the instructions on this [lab manual](TCP_Attacks.pdf) to complete **Task 1: SYN Flooding Attack**. 
2. Then you need to complete some additional tasks. See the sections below for specific instructions.

## Additional Tasks

### 1. Environment Configuration and Traffic Generation
- Install iperf on the "Server" machine and the "Client" machine using:
  ```bash
  sudo apt-get install iperf
  ```
- We will now collect data of normal traffic and DoS traffic and compare them. First, go to a terminal that corresponds to the "Server" machine type:
  ```bash
  sudo tcpdump -i eth1 -s0 -w capture1.pcap
  ```
  replace *eth1* with the correct network interface on your machine. To find the right one, run `ifconfig` and look for the IP address, the interface name will be listed before it. Use that interface name in the command and hit Enter on your keyboard. This command captures the packet traffic and writes it in a file called *capture1.pcap*.
- Open another terminal on the "Server" machine and press Ctrl+C key. Now in this terminal, Type:
  ```bash
  iperf -s
  ```
  and hit Enter on your keyboard. When you use `iperf -s` you are running an iperf server. 
- Next you will run an iperf client on the “Client” machine and generate some regular traffic. Go to a terminal that corresponds to the "Client" machine. Type:
  ```bash
  iperf -c Server_IP
  ```
  and hit Enter on your keyboard. You need to use the real IP address of the "Server" machine in this command. Wait for at least 5 seconds and do the next step.

### 2. SYN Flooding DoS Attack Traffic Generation
- Open the terminal on the "Attacker" machine. Conduct the same SYN flood attack from the [task 1](https://seedsecuritylabs.org/Labs_16.04/PDF/TCP_Attacks.pdf) of this SEED lab. After 10 seconds, press Ctrl+C and stop all the tasks running on terminal windows of three VMs.

### 3. Traffic Analysis Using Wireshark
- Use Wireshark to view the *capture1.pcap* file. This file contains both normal and attack traffic data.
> To access the I/O Graph in Wireshark, go to `Statistics` in the top menu and select `I/O Graph`.
- This is the graph of the traffic you ran in the previous tasks. 
- In your report, explain and describe generated I/O graph. Include the graph in your report.

**Questions:** Do you see at which time you started the flooding attack? Why is it very distinctive? Did the attack end at some point? What do you think happened at this point?

- Repeat the above steps at least four more times for the task below. You need to change the name of the data file everytime, e.g., *capture1.pcap*, *capture2.pcap*, ....
> You do not need to show the I/O graph for the repeated experiements.
- Use Wireshark to calculate the average packet size and the average traffic rate (in pakets per second) for both normal and attack traffic. Record these values in an Excel spreadsheet. Your spreadsheet should include two separate tables: one showing the average packet size and average traffic rate for normal traffic, and the other showing the same measurements for traffic captured during an attack.
> Look at the `Statistics > I/O` graph and locate a point where the attack started, and then use `Statistics > Packet Lengths` with filters to display the packets received before or after that point. Check the statistics of the displayed packets.
 
### 4. Statistical Analysis of Normal and Attack Traffic
- Calculate the average and standard deviation of these two sets of data. Describe your observations of the results. Include the spreadsheet, average, and standard deviation in your report.
> [!NOTE]
> <details>
> <summary> What is standart deviation?</summary>
> <p align="center">
> <img width="350" alt="image" src="https://github.com/user-attachments/assets/3909dac3-730d-4c9b-861e-726a5ba503a7" />
> </p>  
>
> The **standard deviation** is a measure of the amount of variation of the values of a variable about its average. A low standard deviation indicates that the values tend to be close to their average of the set, while a high standard deviation indicates that the values are spread out over a wider range.
>   
> Use the formula below to calculate the standart deviation for your data sets.
> 
> $$
> s = \sqrt{\frac{1}{n - 1}\sum_{i=1}^{n}(x_i - \bar{x})^2}
> $$
>
> Where:
>
> - **s** = sample standard deviation  
> - **n** = number of values  
> - **xᵢ** = each value  
> - **x̄** = sample mean  
>
> 
> </details>


## Grading (50 points)
Please take screenshots periodically and regularly and include them in your report. They not only serve as evidence of completion but also help the grader understand what you try to achieve. Add adeuqate explaination as needed. See the lab submission example for what it should look like.
* Completeness (35 pts): All the steps as instructed in the lab manual must be included in the report with adequate evidence.
* Presentation (15 pts): The report must be clear and correct in organization and writing with adequate explanation.
