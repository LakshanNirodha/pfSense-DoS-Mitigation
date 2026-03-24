# Network Resilience: Simulating & Mitigating DoS Attacks using pfSense Firewall

## 📌 Project Overview
In this hands-on network security lab, I focused on infrastructure defense by deploying and configuring a **pfSense Firewall**. I simulated an external volumetric Denial of Service (DoS) attack using `hping3` to understand how network firewalls respond to resource exhaustion. The project demonstrates network segmentation, state-table management, real-time traffic analysis, and firewall hardening to maintain service availability.

## 🛠️ Environment & Infrastructure Setup
The lab was built using Oracle VirtualBox, utilizing both Bridged and Internal Network adapters to create a realistic, segmented environment.
* **Firewall/Gateway:** pfSense Community Edition (2 vCPU, 2GB RAM)
* **Attacker Machine (WAN):** Kali Linux
* **Victim Machine (LAN):** Ubuntu Desktop
* **Analysis Tools:** hping3, Wireshark

---

## ⚙️ Phase 1: Network Architecture & Firewall Configuration
Before simulating the attack, I built the network topology and configured the firewall routing.

1. **pfSense Interface Assignment:** Configured two interfaces:
   * **WAN (vtnet0):** Bridged to the physical network for external access.
   * **LAN (vtnet1):** Internal Network ('LabNet') set to `192.168.1.1/24`.
2. **DHCP Configuration:** Enabled the DHCP server on the pfSense LAN interface to automatically assign IPs to internal clients (e.g., Ubuntu receiving `192.168.1.100`).
3. **Routing Setup:** Configured a static route on the Kali Linux attacker machine to ensure reachability to the isolated internal LAN subnet.
   `sudo ip route add 192.168.1.0/24 via <PFSENSE_WAN_IP>`

<img width="1919" height="1010" alt="Screenshot 2026-03-17 105046" src="https://github.com/user-attachments/assets/9056703d-7f05-45f6-b4ee-6d48391a5561" />

<img width="1538" height="912" alt="Screenshot 2026-03-17 235512" src="https://github.com/user-attachments/assets/ca4852e3-68cc-4edb-8306-c3ab64031416" />

---

## 🛑 Phase 2: Attack Simulation (TCP SYN Flood)
To test the firewall's resilience, I launched a Denial of Service (DoS) attack from the external WAN.

1. **Initiating the Flood:** Used `hping3` on Kali Linux to send a high volume of TCP SYN packets, simulating a state-exhaustion attack against the Ubuntu machine on the LAN.
   `sudo hping3 --flood -S -p 80 192.168.1.100`
2. **Resource Exhaustion Monitoring:** Observed the pfSense Dashboard during the attack, noting a significant CPU spike (up to 91%), demonstrating the impact of unmitigated volumetric traffic on network appliances.
3. **Traffic Analysis:** Deployed Wireshark on the Ubuntu victim machine to capture and inspect the influx of malicious TCP handshakes.
   `sudo wireshark &`

<img width="1918" height="1005" alt="Screenshot 2026-03-17 235746" src="https://github.com/user-attachments/assets/a2a419e2-c8c7-496d-ae6e-6543f30001a6" />

<img width="1919" height="1001" alt="Screenshot 2026-03-17 231502" src="https://github.com/user-attachments/assets/fe66544a-7ee2-49e8-940f-1346b69b530d" />

<img width="1919" height="1008" alt="Screenshot 2026-03-17 232949" src="https://github.com/user-attachments/assets/c02c364a-bc3b-4d9c-91a2-74c2cf1b5ba9" />

---

## 🛡️ Phase 3: Mitigation & Firewall Hardening
To neutralize the threat and restore network stability, I implemented granular firewall rules.

1. **Rule Creation:** Navigated to the pfSense WebGUI and created a specific 'Block' rule on the WAN interface.
   * **Action:** Block
   * **Source:** Kali-IP
   * **Destination:** `192.168.1.100/24`
   * **Logging:** Enabled
2. **Verification:** Applied the rule and verified the mitigation. The Wireshark packet capture on the Ubuntu machine showed an immediate halt in the flood traffic, and the pfSense CPU usage returned to a stable state (~10%).
3. **Log Analysis:** Reviewed the pfSense firewall logs to confirm that the malicious flood traffic matching the defined signature was successfully being dropped.

<img width="1919" height="1010" alt="Screenshot 2026-03-17 231041" src="https://github.com/user-attachments/assets/f384a6c2-6518-4f55-810d-b44001f29d03" />

<img width="1919" height="1007" alt="Screenshot 2026-03-17 235339" src="https://github.com/user-attachments/assets/48769736-d632-4faf-8c71-aa8b80c9a141" />

<img width="1919" height="1008" alt="Screenshot 2026-03-17 235431" src="https://github.com/user-attachments/assets/d510d886-1e46-4113-abe8-63e3aefdffeb" />

---

## 💡 Troubleshooting & Challenges Faced
* **Static Routing Configuration:** Initially, the Kali machine on the WAN could not reach the Ubuntu machine on the internal LAN. I troubleshot this by manually adding a static route (`ip route add`) on Kali, pointing traffic destined for the `192.168.1.0/24` subnet to the pfSense WAN IP as the gateway.
* **Traffic Isolation in Wireshark:** During the high-volume flood, isolating the specific malicious SYN packets from standard background noise in Wireshark was challenging. I utilized specific Wireshark display filters (e.g., `tcp.flags.syn == 1 and tcp.flags.ack == 0`) to accurately identify and analyze the attack patterns.

## 🏁 Conclusion
This lab provided practical experience in network perimeter defense. It highlighted the devastating effects a DoS attack can have on device resources and underscored the necessity of robust firewall configurations, continuous monitoring, and prompt incident response to ensure network resilience and availability.
