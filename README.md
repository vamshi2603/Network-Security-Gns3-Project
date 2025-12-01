Secure Network Design, Firewall Configuration & Attack Simulation
### **Group 11 – Network Security Project**

This repository contains a complete cybersecurity lab environment built using **GNS3**, with a **FortiGate firewall**, **Cisco R1 router**, **Windows 10 victim**, **Ubuntu server**, and **Kali Linux attacker**.

The project demonstrates:

- Network segmentation  
- Firewall rule configuration  
- Routing between multiple LANs  
- Attack simulation (Nmap, DoS)  
- Packet capture & firewall log analysis  
- Secure offline cyber-range design

1. Network Topology
Kali (Attacker) ---- R1 Router ---- Fortigate Firewall ---- ESW1 Switch ---- Windows + Ubuntu
192.168.20.0/24 10.0.0.0/24 192.168.10.0/24
### **Device Roles**
| Device | Purpose |
|--------|---------|
| **Kali Linux** | Attacker (Nmap, Hping3) |
| **Windows 10** | Victim |
| **Ubuntu** | Internal host |
| **FortiGate** | Firewall + logging |
| **Cisco R1** | Router / gateway |
| **ESW1** | Layer 2 switch |

# 2. IP Addressing Scheme

| Device / Interface | IP Address | Notes |
|--------------------|-----------|-------|
| Windows 10 | 192.168.10.30 | Victim |
| Ubuntu | 192.168.10.x | Internal |
| Fortigate port3 | 192.168.10.1 | LAN gateway |
| Fortigate port2 | 10.0.0.2 | Transit |
| R1 Fa0/0 | 10.0.0.1 | Transit |
| R1 Fa0/1 | 192.168.20.1 | Kali LAN gateway |
| Kali Linux | 192.168.20.x | Attacker | 


# 3. Router R1 Configuration

```bash
interface FastEthernet0/0
 ip address 10.0.0.1 255.255.255.0

interface FastEthernet0/1
 ip address 192.168.20.1 255.255.255.0

ip route 192.168.10.0 255.255.255.0 10.0.0.2
no ip nat inside
no ip nat outside
```

### **Interface Setup**
```bash
interface FastEthernet0/0
 ip address 10.0.0.1 255.255.255.0

interface FastEthernet0/1
 ip address 192.168.20.1 255.255.255.0
```
#  4. Fortigate Firewall Configuration
### **Interfaces**
| **Port** | **IP Address**  | **Purpose**          |
| -------- | --------------- | -------------------- |
| port3    | 192.168.10.1/24 | Windows + Ubuntu LAN |
| port2    | 10.0.0.2/24     | Transit link to R1   |
| port1    | Disabled        | No internet          |

### **LAN → LAN Firewall Policy**

-Source: 192.168.10.0/24, 192.168.20.0/24
-Destination: 192.168.10.0/24, 192.168.20.0/24
- Action: ACCEPT
- NAT: Disabled
- Logging: ALL SESSIONS


### **Purpose of Policy**

- Enables communication between both LANs
- Logs all traffic (useful for Nmap, DoS, and forensic analysis)
- Acts as a central inspection and control point for the entire topology


# 5. Connectivity Testing (Ping Tests)

### **Windows → Firewall**
ping 192.168.10.1

### **Windows → Ubuntu**
ping 192.168.10.40

### **Windows → Router**
ping 10.0.0.1

### **Kali → Router**
ping 192.168.20.1

### **Kali → Firewall**
ping 10.0.0.2

### **Kali → Windows**
ping 192.168.10.30

### **Router → Windows**
ping 192.168.10.30

### **Router → Kali**
ping 192.168.20.x

If all pings succeed → routing and firewall are correct.

# 6. Attack Simulation (Kali Linux)
### **6.1 Nmap Aggressive Scan**
sudo nmap -A 192.168.10.30

Shows:
- OS detection
- Service detection
- Traceroute
- Script output

### **6.2 Full Port Range Scan**
sudo nmap -p- 192.168.10.30

### **6.3 Vulnerability Script Scan**
sudo nmap --script vuln 192.168.10.30

### **6.4 DoS Simulation (SYN Flood)**
sudo hping3 -S --flood -V 192.168.10.30

Attack effects:
- Windows slows slightly
- Firewall logs spike
- Packet capture shows SYN flood

# 7. Firewall Logging & Monitoring
We are unable to show the logs because we have used a firewall which has its licence expired. And We did not have enouch time to replace the Firewall

# 8. Packet Capture Analysis
This packet capture contains traffic from a simulated Denial-of-Service (DoS) attack performed in a GNS3 environment. The analysis shows two main attack phases targeting the victim machine 192.168.10.30 from the attacker 192.168.20.10.

### **Key Observations**

- Total Packets: ~90,000
- Capture Duration: ~8.7 minutes
- Primary Victim: 192.168.10.30
- Primary Attacker: 192.168.20.10

### **Attack Types Identified**
1. TCP SYN Flood

- Time: ~243–290 seconds
- Rate: ~992 packets/sec
- Target Port: 0
- Behavior: Continuous SYN packets with immediate RST responses from the victim.
- Conclusion: Classic SYN flood attack designed to overwhelm the TCP stack.

2. UDP Flood

- Time: ~375–398 seconds
- Rate: ~1,500 packets/sec
- Target Port: 80
- Behavior: High-rate UDP packets with empty payloads.
- Conclusion: UDP flood aimed at the victim’s HTTP port.

### **Additional Notes**

- ICMP Echo traffic shows continuous pings from the attacker to monitor victim responsiveness.
- DNS traffic failures from 192.168.10.40 to 8.8.8.8 indicate a routing issue, not part of the attack.
- Background LAN traffic (ARP, STP, mDNS, LLMNR, CDP) appears normal.



