---
layout: post
title:  "Quick PCAP Analysis with Wireshark"
date:   2021-11-16 13:08:53 -0500
categories: jekyll update
---
In this post, I am going to conduct a quick PCAP analysis using Brad Duncan's [Malware Traffic Analysis](https://www.malware-traffic-analysis.net/) exercises. This time, I will be analyzing the PCAP in Wireshark. My objective is to discover the IP addresss, MAC address, host name, and user account name of the infected host, followed by potential indicators of compromise.

One way to find some details on the host in Wireshark is to filter by DHCP (Dynamic Host Configuration Protocol). DHCP is used to assign IP addresses to devices within a network. The leases on these IP addresses must be renwed periodically, and the devices will automatically request an IP address from the DHCP server. So if you happen to have DHCP traffic in your PCAP,  this can be very useful. In the filter bar in Wireshark, I am simply going to type **DHCP**.

Now, all we see is DHCP traffic between the client and the DHCP server:

![dhcp arrow](https://user-images.githubusercontent.com/84248865/142335722-0ef2fc44-4511-467a-9cc4-d9870f662d87.png)

If you expand the arrow next to Dynamic Host Configuration Protocol, you'll find the client's host name, MAC address, and assigned IP address:

![dhcp host name mac address](https://user-images.githubusercontent.com/84248865/142335831-3554ce7e-7cbb-4483-a0d7-0f34c9b1a47a.png)

Now we know that the client's MAC address is 00: 4f: 49: b1: e8: c3(a Realtek product), the IP address is 10.9.10.102, and the host name is DESKTOP-KKITB6Q. You could also type **ip contains DESKTOP-** in your filter bar to find this information. If the user has not renamed their PC, the name will most likely begin with DESKTOP-. The typical default device name on a Windows machine is DESKTOP- followed by a string of letters and numbers.

Another detail to search for is the client's user account name. A Windows machine that is part of a domain uses an authentication protocol within Active Directory called Kerberos, which is used to verify the identity of a user or host. We can use the **kerberos.CNameString** filter to only show Kerberos authetication service requests from the client. 

![kerberos arrow](https://user-images.githubusercontent.com/84248865/142336002-2cd41a7b-2007-4f4c-92e5-c4b77045301b.png)
If we dig deeper into the Kerberos packet, you'll find the **CnameString**, which holds the cleartext username the client is trying to autheticate.

![Kerberos name string](https://user-images.githubusercontent.com/84248865/142336246-3165a21e-37ed-40e9-8250-7aac1c081ab1.png)

We have discovered the user account name is **hobart.gunnarsson**.

Now that we have found a few details about the infected host, let's look at some web traffic to find indiciators of compromise. We can use the filter value **http.request** to find various HTTP requests that have occured. To further pare down the results, we can use **http.request and!(ssdp)**. The **and!(SSDP)** portion will filter out SSDP (Simple Service Discovery Protocol) traffic, which is HTTP over UDP that is used to discover Plug & Play devices in home networks or small office environments. It is considered normal activity not associated with regular web traffic, so that is why we filter it out.

![microsoft connection](https://user-images.githubusercontent.com/84248865/142336476-86cc5777-000a-4769-936c-f6dd8a3b16f2.png)

With this filter in place, first we see a connection to **www.msftncsi.com**. This URL is used by Windows machines to verify network connectivity and can be consdiered harmless. MSFTNCSI stands for Microsoft Network Connectivity Status Indicator. 

There are some other connections to **microsoft.com** and **windowsupdate.com**, which we can consider normal traffic. Automatic updates are common, and if you search Microsoft's documentation, you'll find that the URL **store-images.s-microsoft.com** is used to get images that are used for Microsoft Store suggestions.

Something that sticks out among these URLs is a connection to **simpsonsavingss.com**. When we follow the TCP stream, we see a GET request that appears to be for an executable file:

![follow tcp stream](https://user-images.githubusercontent.com/84248865/142339067-a4e6126a-009b-4c07-a353-8a2ee53c776a.png)
![date6 file](https://user-images.githubusercontent.com/84248865/142339097-23b36099-df68-4554-ba7f-9dcc3ae4df53.png)

We may want to anaylyze this file later in a sandboxed environment or create a hash for it. To quickly save the file off, you can go to **File -> Export Objects -> HTTP** and then select the packet, as shown here:

![export object hightlighted](https://user-images.githubusercontent.com/84248865/142339274-aaf60d75-c188-4725-bc1f-97aa9c45c843.png)

![simpsonsavings save object1](https://user-images.githubusercontent.com/84248865/142339283-50bb65c8-35d1-4766-8bc4-69cef597b8c3.png)

Before doing anything with the file, it would also be a good idea to run the domain name through a few threat detection sites such as VirusTotal, Cisco Talos Reputation Center, AbuseIPDB, etc. and see if it has been reported as malicious. Now that we have the file saved for later use, I am going back to the PCAP to use the filter **(http.request or tls.handshake.type eq 1) and!(ssdp)**. Adding the **tls.handshake.type eq 1** value allows us to see domain names used in HTTPS or SSL/TLS traffic. Upon applying this filter, I noticed that immediately following the GET request from **simpsonsavingss.com**, numerous "Client Hello" messages are being sent from the host to IP addresses with no associated domain names over HTTPS port 443. 

![C2 traffic](https://user-images.githubusercontent.com/84248865/142339591-a6042a78-351b-447d-8fb4-21a718737bbe.png) 

It appears there are two IP addresses associated with this traffic - 167.172.37.9 and 94.158.245.52. Encrypted traffic to IP addresses with no associated domain name is unusual and could be an indicator of C2 traffic, so these IPs should be investigated. If you search for these IP addresses and nothing comes up, you may need to pivot to other sources of information to invesigate.

Thank you for reading this quick Wireshark PCAP analysis. A big thanks to Brad Duncan for offering these real-world PCAPs and exercises. Brad also recently published a [Wireshark video tutorial workshop](https://unit42.paloaltonetworks.com/wireshark-workshop-videos/), which I found very informative and engaging. This was a brief post, but I hope you learned something about Wireshark and the different filters you can use to pivot through data and find relevant information.



References:

Duncan, Brad. “Malware-Traffic-Analysis.Net - 2021-09-10 - Traffic Analysis Exercise - AngryPoutine.” Malware-Traffic-Analysis.Net, 10 Sept. 2021, [https://www.malware-traffic-analysis.net/2021/09/10/index.html](https://www.malware-traffic-analysis.net/2021/09/10/index.html).

“FrontPage - The Wireshark Wiki.” FrontPage - The Wireshark Wiki, [https://wiki.wireshark.org/](https://wiki.wireshark.org/)].

“Wireshark Tutorial: Wireshark Workshop Videos Now Available.” Unit42, 1 Oct. 2021, [https://unit42.paloaltonetworks.com/wireshark-workshop-videos/](https://unit42.paloaltonetworks.com/wireshark-workshop-videos/).
