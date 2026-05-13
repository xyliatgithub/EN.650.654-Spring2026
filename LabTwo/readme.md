# Lab 2 - Network Traffic & DoS Attack

In this lab, we will perform an experimental Denial-of-Service attack and collect network packets. In addition to those tasks required by the SEED lab documentation, you need to finish several additional tasks described below.

## Setup
> [!NOTE]
> Please reuse the VM/environment from Lab 1. If you already finished Lab 1 setup, you can skip this setup section. Otherwise, follow: [Lab 1 – Environment Setup](../LabOne/readme.md#setup).
- Download and install [Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- Refer to the [SEED VM VirtualBox Manual](https://seedsecuritylabs.org/Labs_16.04/Documents/SEEDVM_VirtualBoxManual.pdf) for detailed instructions on setting up the VM in VirtualBox.
- This manual also contains account information, such as usernames and passwords.

In this lab, you need to have three VMs under the same local network. Once you have configured a VM, you can simply clone that VM for two more times to complete the VM setup. Please refer to Appendix A and B of the VM setup instruction.
> [!IMPORTANT]
> Note that these three VMs should be in the promiscuous mode in order to listen to traffics from other VMs.


## Lab Instructions 

1. Please follow the instructions on this [lab manual](TCP_Attacks.pdf) to complete **Task 1: SYN Flooding Attack**.
2. Make sure to set SYN backlog size to default value if you changed SYN backlog size
3. Then you need to complete some additional tasks. See the sections below for specific instructions.

## Additional Tasks

> [!NOTE]
> The I/O graph in Wireshark measures raw packet volume at the network interface. SYN cookies do **not** reduce the number of incoming attack packets — both cookie states will show an identical flood spike if only total traffic is measured. The real effect of SYN cookies is at the **service availability layer**: whether legitimate connections survive during the attack. The steps below are designed to make that difference clearly visible.

---

### 1. Environment Configuration and Baseline Tuning

Check the current SYN cookie state on the **Server** machine:
  ```bash
  sysctl net.ipv4.tcp_syncookies
  ```

**Reduce the SYN backlog queue** to a small value so it exhausts quickly and the cookie effect is dramatic. On the **Server** machine run:
  ```bash
  sudo sysctl -w net.ipv4.tcp_max_syn_backlog=64
  ```
  Verify the change took effect:
  ```bash
  sysctl net.ipv4.tcp_max_syn_backlog
  ```

Also reduce the SYN-ACK retry count so half-open entries expire faster and queue exhaustion is visible sooner:
  ```bash
  sudo sysctl -w net.ipv4.tcp_synack_retries=1
  ```

Install iperf on the **Server** and **Client** machines:
  ```bash
  sudo apt-get install iperf
  ```

---

### 2. Capture A — Baseline: SYN Cookies OFF

This capture will record what happens to legitimate traffic when the server has no protection against SYN flooding.

**Step 1 — Disable SYN cookies on the Server:**
```bash
sudo sysctl -w net.ipv4.tcp_syncookies=0
```
Confirm:
```bash
sysctl net.ipv4.tcp_syncookies
# Expected output: net.ipv4.tcp_syncookies = 0
```

**Step 2 — Start packet capture on the Server** (bidirectional — both incoming and outgoing traffic):
```bash
sudo tcpdump -i eth1 -s0 -w capture_cookies_off.pcap
```
Replace *eth1* with the correct network interface. To find it, run `ifconfig` and look for the interface associated with your server IP address (e.g. `10.0.2.6`).

**Step 3 — Start the iperf server on the Server machine** in a second terminal:
```bash
iperf -s -p 5001
```

**Step 4 — Start the iperf client loop on the Client machine.** This loop will keep attempting new connections throughout the entire experiment, so that service availability is continuously measured:
```bash
while true; do iperf -c 10.0.2.6 -p 5001 -t 2; sleep 1; done
```
Replace `10.0.2.6` with the real IP address of the Server machine. Wait approximately **8–10 seconds** and confirm that iperf connections are completing successfully before moving to the next step.

**Step 5 — Open a third terminal on the Server machine** and start monitoring the half-open connection queue in real time. Leave this running for the duration of the experiment:
```bash
watch -n 1 'netstat -ant | grep SYN_RECV | wc -l'
```

**Step 6 — Launch the SYN flood from the Attacker machine** without stopping any of the above:
```bash
sudo netwox 76 -i 10.0.2.6 -p 5001 -s
```

**Step 7** — Let the attack run for **20 seconds**, observing the `watch` output on the Server.

**Step 8** — Press Ctrl+C to stop the attack on the Attacker machine. Wait another **10 seconds** while all other terminals keep running, so recovery is captured.

**Step 9** — Press Ctrl+C to stop: the iperf loop on the Client, the iperf server on the Server, the `watch` monitor, and finally the tcpdump capture. The file `capture_cookies_off.pcap` is now complete.

---

### 3. Capture B — Protected: SYN Cookies ON

Repeat the exact same sequence as Section 2 with one change only.

**Step 1 — Enable SYN cookies on the Server:**
```bash
sudo sysctl -w net.ipv4.tcp_syncookies=1
```
Confirm:
```bash
sysctl net.ipv4.tcp_syncookies
# Expected output: net.ipv4.tcp_syncookies = 1
```

**Steps 2–9** — Follow all remaining steps from Section 2 exactly, but save the capture to a different file:
```bash
sudo tcpdump -i eth1 -s0 -w capture_cookies_on.pcap
```

Keep all timing consistent with Capture A (same wait periods, same attack duration) so the two captures are directly comparable.

---

### 4. Traffic Analysis Using Wireshark

Open each `.pcap` file separately in Wireshark. To access the I/O Graph, go to `Statistics` in the top menu and select `I/O Graph`.

The default view shows all packets. To reveal the SYN cookie effect, you must add multiple filter rows to the graph. Click the `+` button at the bottom of the I/O Graph window to add each of the following rows:

| Row name | Display filter | What it measures |
|---|---|---|
| All packets | *(leave empty)* | Total traffic volume including flood |
| Legitimate traffic | `tcp.port == 5001` | iperf connections — service survival indicator |
| SYN flood packets | `tcp.flags.syn == 1 && tcp.flags.ack == 0` | Incoming SYN requests only |
| Server SYN-ACK responses | `tcp.flags.syn == 1 && tcp.flags.ack == 1` | Whether server is still responding |
| Retransmissions | `tcp.analysis.retransmission` | Legitimate client retry attempts due to dropped connections |

Set the interval to `1 sec` for fine-grained resolution. Apply the same filter set to both capture files so the graphs are directly comparable.

- In your report, include the I/O graphs from **both** captures with all filter rows visible. Describe what each filter line does during the baseline period, during the attack, and after the attack ends.

**Questions:** Do you see at which time you started the flooding attack? Why is it very distinctive? Did the attack end at some point? What do you think happened at this point?

- Compare the `tcp.port == 5001` (legitimate traffic) line between `capture_cookies_off.pcap` and `capture_cookies_on.pcap`. This line is the primary indicator of the SYN cookie effect.

**Questions:** Do you see difference in graphs while attack was run with SYN cookies on? Is it different from the graph when attack was run with SYN cookies off? Why is there difference or why not? Describe your graphs, compare, explain the results.


- Repeat the above steps at least four more times each for the task below. You need to change the name of the data file everytime, e.g., *capture_cookies_off_1.pcap*, *capture_cookies_on_1.pcap*, ....
> You do not need to show the I/O graph for the repeated experiements.
- Use Wireshark to calculate the average packet size and the average traffic rate (measured in packets per second) for both normal and attack traffic with SYN cookies turned on and off. Record these values in an Excel spreadsheet.
- Your spreadsheet should include three separate tables: one showing the average packet size and average traffic rate for normal traffic, the other showing the same measurements for traffic captured during an attack with cookies on and another showing the measurements for traffic captured during an attack with SYN cookies off.
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
