# Nmap-Wireshark
# Network Topology

The network scan results using Zenmap reveal a Star Topology (see screenshot below) where five hosts are connected to a single switch. Three hosts are running Linux OS, and two are running Windows OS. A Star Topology is a network design where all the hosts are connected to one central hub or a switch, forming a shape resembling a star. Start Topology is known for easy management and troubleshooting, as well as being fault-tolerant, where if one dedicated connection fails, it only affects said device. However, one of the main disadvantages of Star Topology is a single point of failure; in this example, if the switch goes down, so do all the devices connected to it.

<img width="1171" alt="net_topology" src="https://github.com/user-attachments/assets/a90393fa-b764-4696-b12c-ddbc7785ee68" />



# Summary of Vulnerabilities and Implications

# First vulnerability

Host 10.168.27.10 is running NetBIOS on port 139. NetBIOS on port 139 is used by applications and devices to access shared resources on other network devices, transmit large messages, and is used for printer and file services over a network. NetBIOS poses a significant security risk and is prone to cyberattacks due to its lack of encryption and extremely weak authentication protocol. It is susceptible to many types of attacks, such as “Spoofing”, where attackers can redirect traffic to a malicious system by spoofing NetBIOS responses, brute force attacks, and information leakage, where attackers can access and obtain sensitive information.

<img width="1172" alt="NetBIOS" src="https://github.com/user-attachments/assets/a8737e75-521d-41ef-a1e3-3cab36f53dfa" />

# Second vulnerability

Host 10.168.27.15 is running FTP on port 21. FTP, or File Transfer Protocol, is a client-server protocol used as a standard way to move files between devices on a network. FTP is used to upload files to a server, download files from a server, and transfer data between devices. FTP has many security issues. The absence of encryption fails to provide confidentiality to the data that is being shared.  FTP also allows for Anonymous access (as shown in the screenshot below, which is a gateway for malicious actors and fails to provide authentication and accounting. File Transfer Protocol is also susceptible to session hijacking and spoofing, where a hacker can impersonate a server or a client and gain access. 


<img width="1277" alt="FTP_NMAP" src="https://github.com/user-attachments/assets/57ef878f-53dc-4966-b014-cedcffab737e" />

# Third vulnerability

Host 10.168.27.10 is running Windows Server 2008 R2, which poses a significant risk as it reached its end-of-support on January 14, 2020. End of support means the service no longer receives updates, security patches, and fixes. Windows Server 2008 R2 is vulnerable to Zero-Day attacks without security patches and updates since newly discovered vulnerabilities are unaddressed, leaving the system open to unknown exploits. Potential consequences of running Windows Server 2008 are Data Breaches, which can lead to potential legal and financial repercussions if sensitive data is breached or accessed without authorization. Without security updates, systems carry an increased risk of being prime targets of malware and ransomware attacks as well as denial of service attacks, which can bring the system down and disrupt services.



<img width="679" alt="NetBIOS" src="https://github.com/user-attachments/assets/06e04d61-8582-4c51-9b49-432690114fbc" />

# Wireshark Anomalies

# First Anomaly

While filtering pcap1.pcapng for “ftp” traffic I have discovered that host 49.12.121.47 was able to successfully login to host 10.168.27.10 using anonymous login. Packet 213816 shows host 49.12.121.47 requesting login to host 10.168.27.10 “USER  FileZilla”, to which ftp host responds with “331 Give any password” as show in packet 213818. Finally packet 213823 shows evidence of successful anonymous login “230 logged on”.

<img width="1169" alt="FTP WireShark" src="https://github.com/user-attachments/assets/8a440f76-8643-4fa0-9da2-6a8ecd1f4e02" />


# Second Anomaly

When filtering for “tcp.flags.syn == 1”  I have discovered an unusual amount of TCP SYN packets from host 10.16.80.243 going out to multiple hosts on network 10.168.27.0/24, packets going out to multiple ports (80, 443, 110, 8080, 113, etc.). Over 45,000 packets were sent. Packets 1677,1680-1681,1683-1684,1686,1689-1690,1693 show evidence of host 10.16.80.243 sending a large amount of SYN packets to ports 111 and 110 to multiple hosts on 10.168.27.0/24 network block. Further, packets 1694,1698,1699,1700,1703,1705,1706,1710,1712,1714-1715,1717,1719 and 1722 show more evidence of a large number of SYN packets sent to ports 8080,113,110 and 5900 on multiple hosts on network block 10.168.27.0/24.

<img width="1171" alt="TCP SYN Packets" src="https://github.com/user-attachments/assets/d53c0a59-cbfa-498c-bbec-09a94392e871" />


# Third Anomaly

While examining for HTTP traffic in pcap2.pcapng file. I have discovered an “HTTP GET” request with “/login.php” in the info section. After taking a look, I have identified that the session Cookie is transmitted in cleartext, including what seems to be a authentication token “security” and session ID “PHPSESSID”. Packet: 2007.



<img width="1174" alt="Cookies Screen" src="https://github.com/user-attachments/assets/ce7e615b-b3e8-4672-8834-3abbd0e22ebf" />

# Implications of each Wireshark Anomaly

# Implications of taking no action 1
Anonymous login through FTP is a serious security risk. If no action is taken the implications could be quite severe, starting with unauthorized access with no way of accounting.  Theft of sensitive data, which could lead to serious financial and legal repercussions as well as damage company’s/entity’s reputation. And potentially lead to malware and ransomware attacks.

# Implications of taking no action 2
Such activity is associated with information gathering and vulnerability discovery, techniques such as network mapping, identifying open ports and services and operating systems and versions. This could lead to a number of attacks ranging from unauthorized access, denial of service, disruption of services, data breaches and more.

# Implications of taking no action 3
Transmitting session cookies over unencrypted HTTP connections can be a serious vulnerability. An attacker intercepting network traffic can capture the “PHPSESSID”, and effectively hijack user’s session. Session hijacking allows an attacker to impersonate the user and gain unauthorized access, leading to potential data breaches. 

# Recommended Solutions
# First Vulnerability: NetBIOS on Port 139
To mitigate security risks associated with NetBIOS being open on port 139, Disable NetBIOS on host 10.168.27.10. If NetBIOS is required, restrict access to port 139 at the firewall to only allow communication within the necessary internal network segments to prevent unauthorized access and transition from SMBv1 to more secure protocols like SMBv2 or SMBv3. Additionally, implementing strong access controls, such as limiting port access to authorized users and systems, and regularly monitoring network traffic can further enhance security.

# Second Vulnerability: FTP on Port 21
One of the solutions to FTP Anonymous Login is to disable anonymous FTP access. If FTP is necessary, it could be a good idea to configure FTPS, which encrypts data channels using TLS/SSL. Alternatively, we should monitor FTP server logs for any suspicious activity and disable uploads through anonymous login if it is absolutely required.

# Third Vulnerability: Windows Server 2008 R2
Upgrade to a supported version of Windows Server, for example, Windows Server 2019 or 2022, or migration to Microsoft Azure. If an upgrade is not immediately available, implement controls such as network segmentation, IDS/IPS, and stringent access controls.

# First Anomaly: Anonymous FTP Login
Immediate investigation of the unauthorized login. As stated previously, disable anonymous FTP login on host 10.168.27.10. Reviewing FTP logs for any other suspicious activity, implementing strong access controls, as well as blocking the attacker’s IP, and implementing MAC filtering.

# Second Anomaly: Excessive TCP SYN Packets
The first step to mitigate these risks would be blocking the suspicious host 10.16.80.243 at the firewall and investigating the source of the traffic to determine whether it was a malicious scan. Also, solutions such as IDS/IPS should be implemented to monitor and/or block similar activity in the future.

# Third Anomaly: Cleartext Cookie Transmission
Implement HTTPS to encrypt all traffic, including cookies. Set the Secure attribute on the session cookies. Regenerate session IDs after successful login to prevent session fixation attacks.
