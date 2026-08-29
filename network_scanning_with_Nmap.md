# Lab 02: Network Reconnaissance & Port Scanning with Nmap

## Objective
Identify active hosts, open ports, running services, and potential vulnerabilities on a target network using Nmap CLI to establish baseline network visibility.

## Setup Details
* **Attacker System:** Kali Linux (64-bit)
* **Target Environment:** Local Subnet / VirtualBox Host-Only Network
* **Tools Used:** `nmap`

---

## Command Execution & Results

### 1. Host Discovery (Ping Sweep)
Discovered live host on the local network without performing full port scans:
```bash
sudo nmap -sn my IP
```
Key findings: Identified target host activity and MAC address

###2. Service & Version Detection
  Scanned the top 1000 standard TCP ports to enumerate running service versions:
```bash
sudo nmap -sV -sC -T4 my IP
```
Result: All 1000 ports in ignored state.
I suspect this is due to a firewall stopping the pings.
  I will skip initial ping, scan all ports rather than the 1000, and avoid OS fingerprinting

```bash
sudo nmap -Pn -p- -sV my IP
```
Result: Ports 7680,21462,27035 are open which indicates it is a gaming computer with Steam open. 
  Follow up; I'm going to double check what services are listening behind these ports.
  
```bash
sudo nmap -sV -sC -p 7680,21462,27035 my IP
```

Results: 
  Port 7680/tcp (open)
    Listed as pando-pub? by default because 7680 is historically registered for Pando Media Booster in Nmap's service file, but on modern Windows machines, this is used by Windows Update Delivery Optimization (DoSvc).
  Port 21462/tcp (open)
    When Nmap probed port 21462 with web HTTP requests the local service responded with a JSON payload explicitly identifying itself: {"status":102,"statusString":"ERROR-BAD-REQUEST","spotifyError":0,"responseSource":"Spotify"}
  Port 27035/tcp (filtered)
    Status: filtered means packets were dropped (likely by Windows Firewall or Steam not currently running host streaming). This confirms Steam In-Home Streaming is installed on the machine, but active traffic is blocked on that port.
  MAC Address (mine)
  
UDP Scan

```bash
sudo nmap -sU -F -T4 my IP
```
Result:
  All 100 ports are in ignored states.

Even if TCP port 445 (SMB) showed up as closed or filtered earlier, running NetBIOS script checks over UDP port 137 can force the desktop to reveal its identity

```bash
sudo nmap -sU -p 137 --script nbstat my IP
```

Result:
  port 137 filtered service netbios-ns. This is now more than enough evidence to the OS being windows.

I'm going to try mDNS (port 5353) for local network discovery.

```bash
sudo nmap -sU -p 5353 --script dns-service-discovery my IP
```
Result: 
  port 5353/udp filtered service zeroconf this being filtered means the same as port 137 being filtered which means the firewall is preventing it from getting through.

Executive Summary of Reconnaissance Findings
During this lab module, non-intrusive network reconnaissance was conducted against a local target system using Nmap, service script dissectors (NSE), and banner grabbing techniques.

By analyzing the network signatures, open ports, and dropped probe behaviors, the target was fingerprinted as a physical Windows PC (Windows 10/11) active on a home/local network:

Target Services Identified:

Port 7680/tcp (open): Identified as Windows Update Delivery Optimization (DoSvc).

Port 21462/tcp (open): Identified via HTTP JSON response headers as the local Spotify Desktop App API daemon.

Port 27035/tcp (filtered): Identified as the Steam Client In-Home Remote Play service (blocked/dropped by host firewall).

Ports 137/udp & 5353/udp (open|filtered): NetBIOS Name Service and Multicast DNS (mDNS) ports silently dropping unicast probes.

Defense Fingerprint: The combination of silently dropped UDP probes on 137/5353 alongside allowed inbound TCP traffic on 7680/21462 confirms active host-level filtering enforced by Windows Defender Firewall.

How an Attacker Could Generate Attacks
While an open port is not inherently a vulnerability, exposing local application APIs and network management ports provides actionable intelligence for a malicious actor:

1. Targeted OS & Application Exploit Selection (CVE Mapping)
Attack Method: Attackers use passive enumeration to narrow down target profiles. Knowing the machine is a Windows host running Spotify and Steam allows an attacker to bypass generic Linux/Unix exploits and focus exclusively on Windows post-exploitation techniques, local privilege escalations (e.g., historical DoSvc memory handling or file permission vulnerabilities), or client-side application exploits targeting media players.

2. Local Web API Exploitation & CORS Bypasses (Port 21462)
Attack Method: The Spotify daemon listens on a high TCP port and responds to standard HTTP GET and POST requests without requiring initial network authentication. An attacker could craft a malicious webpage (or a phishing link). If the target user visits the page, client-side JavaScript could perform a Cross-Origin Request / Server-Side Request Forgery (SSRF) to http://localhost:21462 to leak local application configuration, manipulate playback, or attempt local service abuse.

3. LAN Man-in-the-Middle & Name Poisoning (LLMNR / mDNS)
Attack Method: Because the machine attempts local name resolution over mDNS/LLMNR when looking for local peers, an attacker positioned on the same local Wi-Fi network could run tools like Responder. When the target PC broadcasts a request for a local network share or printer, Responder can spoof the response, forcing the Windows host to send its local user NTLMv2 password hashes to the attacker.

4. Stealth Port Covert Channels & Malware Masquerading
Attack Method: Attackers often use common default ports (like 7680 or high dynamic ports) to blend malicious command-and-control (C2) traffic in with legitimate system traffic. An attacker who gains low-privilege access could bind a backdoor listener to port 7680 or 21462 to bypass basic perimeter checks, as firewall rules may already trust traffic on those ports.
