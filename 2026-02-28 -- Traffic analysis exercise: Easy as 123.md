
### 2026-02-28 -- Traffic analysis exercise: Easy as 123 - Lab From MALWARE-TRAFFIC-ANALYSIS.NET
## Pre-Investigation Context:
# The Threat: 
NetSupport Manager RAT
The alert says "NetSupport Manager RAT."
NetSupport Manager is actually a legitimate, commercial remote support software (like TeamViewer or AnyDesk). Admin teams use it to help users.
However, threat actors love to steal it, modify it, and use it as a Remote Access Trojan (RAT).
Once it infects a machine, it phones home to a command-and-control (C2) server.

# The Danger: 
It gives the attacker full GUI control over the victim's computer. They can see the screen, move the mouse, steal passwords, and drop more malware.

## Vulnerability Type: 
NetSupport Manager RAT

## Target component (IP, ports, MAC address, information of the target):

#### Note: Defanging is the practice of intentionally breaking the format of a malicious link or IP address (like changing google.com to google[.]com) so that it becomes unclickable text. 

* Malicious IP: 45.131.214[.]85  (This is the C2 server sitting somewhere out on the internet).

Protocol/Port: TCP port 443. Port 443 is normally used for secure web traffic (HTTPS). Attackers use this port deliberately because firewalls usually let 443 traffic pass right through without blocking it.

* LAN segment range: 10.2.28[.]0/24   (10.2.28[.]0 through 10.2.28[.]255)

* Domain: easyas123[.]tech

* AD environment name: EASYAS123

* Active Directory (AD) domain controller: 10.2.28[.]2 - EASYAS123-DC

* LAN segment gateway: 10.2.28[.]1

* LAN segment broadcast address: 10.2.28[.]255

An unknown Windows computer inside your building (10.2.28.X) got infected. At 19:55 UTC, it reached out through your router (10.2.28.1) to talk to an attacker's server (45.131.214.85) over port 443.
Your job with the pcap is to look at that exact conversation and rip out the identities of the machine and the user who messed up.


## TASKS
 For this exercise, answer the following questions for your incident report:

####  What is the IP address of the infected Windows client?
 The IP of the infected client is 10.2.28.88

#### What is the MAC address of the infected Windows client?
00:19:d1:b2:4d :ad

#### What is the host name of the infected Windows client?
DESKTOP-TEYQ2NR

#### What is the user account name from the infected Windows client?

#### What is the full name of the user from the user account?

# Execution:
1. We looked at the pre-investigation notes to see the full scenario and before starting the investigation

2. We download the associated file that is a PCAP file from the internal IP that has been triggering the alerts ” 45.131.214[.]8 “ to open it we need the password The password for zip archives from blogs on a specific day is the term infected followed by an underscore followed by the date. For example, if I had posted a zip archive from June 14th 1972, the password would be “ infected_19720614 “ , so for this zip file the password will be infected_20260228.
<img width="1779" height="550" alt="Image" src="https://github.com/user-attachments/assets/1f219882-8df1-4ff8-a05f-cd8e4498caf7" />
3. Since its a PCAP file we cant use the regular commands to see inside its contents like vim or head -n 15 2026-02-28-traffic-analysis-exercise.pcap Instead we will use the wireshark tool to inspect further using the command wireshark 2026-02-28-traffic-analysis-exercise.pcap & we will open the wireshark tool. To investigate the file for the host name that is causing trouble.

4. We open the Statistics/conversations tab then we look at the ipv4 tab to see all the conversations recorded in the PCAP. Especially looking for the malicious IP : 45.131.214[.]85
<img width="1392" height="1126" alt="Image" src="https://github.com/user-attachments/assets/20d61f85-8ae6-41b8-9ba1-f8f65e7c9540" />
<img width="1386" height="1134" alt="Image" src="https://github.com/user-attachments/assets/4f886c11-a340-4b9c-8fbf-01747c4adb19" />
The first question is answered from the Task we had : What is the IP address of the infected Windows client?
The IP of the infected client is 10.2.28.88

5. Found the malicious IP and looked at the packers between the infected client and the malicious IP a total of 550 packers : 276 from the client , and 274 back from the server. The 96kb total of data transferred is too small for a regular data exfiltration but looking at the duration of the conversation showing that it lasted 15,637 seconds that is like 4 hours, its indicator of a Beacon or Command and Control(C2). The infected machine is checking in with the attacker every second , saying “ I am here, still infected. What are the commands for me to do next?
6. To answer the second question: What is the MAC address of the infected Windows client? We will need to look exclusively at the infected client using the Wireshark filter. Ip.addr == 10.2.28.88 
We get :
<img width="1876" height="1990" alt="Image" src="https://github.com/user-attachments/assets/3447c93a-30e7-4418-88db-9b7f2d66cd26" />
7. Now to find the infected host MAC address we will need to look at the “Ethernet II” and look at the area of Source type since the source of this specific packet is our infected host: and we see that our MAC address is : 00:19:d1:b2:4d :ad
8. To answer the next question: What is the host name of the infected Windows client? To find the hostname of the infected Windows client, we filter the network capture for “dhcp” (Dynamic Host Configuration Protocol) traffic, which is the protocol computers use to automatically request an IP address when they join a network. This request process follows a four-step conversation known as DORA (Discover, Offer, Request, Acknowledge).

We specifically look at Packet No. 103 (timestamp 4.294204), which is the DHCP Request phase. This packet is crucial because it represents the exact moment the computer tells the network, "Yes, I want that IP address, and here is my identity." By selecting this packet and expanding the Dynamic Host Configuration Protocol (Request) section in the packet details pane, we can scroll down to Option 12 (Host Name). Inside this option, the computer officially broadcasts its local name to the server, which Wireshark decodes to reveal the infected client's hostname: DESKTOP-TEYQ2NR.
<img width="1892" height="1999" alt="Image" src="https://github.com/user-attachments/assets/ba6adf59-4b53-4459-86e8-76786659b44a" />
9. Moving to the next question: What is the user account name from the infected Windows client? To identify who successfully logged into the compromised machine within a Windows Active Directory environment, we must analyze Kerberos authentication traffic. Because users must authenticate to the Domain Controller to prove their identity, inspecting Kerberos packets will reveal both the protocol exchange and the specific account names used

