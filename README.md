# Network-Traffic-Analysis-with-Wireshark
A write up on my journey to understand how to analyse network traffic with packet sniffers. Mainly wireshark


# Objective
The objective of this project was to help me understand packet analysis and to document my understanding of how to use packet sniffers (mainly wireshark) to detect possible malicious activity on a network. I will first do a write up breaking down the wireshark GUI and packet analysis and then I will then document a analysis of a malicous pcap file sourced from www.malware-traffic-analysis.net.

# Introduction

In this project I have ran wireshark within my Kali Linux VM as wireshark comes preinstalled with this disto and I like to keep my cybersecurity work seperate from my host machine. 

To start we open up wireshark and are greated with this screen below
![Wireshark screen](
wiresharkscreen)

This screen is where we choose the perameters of the packet capture ("pcap") before we start the capture. Here we can choose which network interface that the pcap will capture traffic from and we can also apply filters to the pcap. some filters include
<ul>
  <li>"host" = will capture traffic for a specific ip address, both outgoing and incoming</li>
  <li>"net" = will capture traffic on a specific subnet</li>
  <li>"port" = captures traffic on specitic ports, you can also filter this further by adding "tcp" or "udp" before port to filter through those specific protocols </li>
  <li>"either host" = captures traffic for specific hosts through their mac address instead of their ip</li>
</ul>

you can even combine filters with the terms,"and","or" and "not" to get very specific results on the pcap.
For example.

(host 192.168.1.1 and not port 22) will only capture traffic from the host 192.168.1.1 but ignore any traffic coming through on port 22.

# Breaking down The Wireshark GUI

For this exercise I will first create a pcap file with no filters applied on the eth0 network interface.
![pcap](pcap.png)

To create this traffic on the network I did some google searches on my VM(note that this would not capture any devices connected to the network as this would only apply to my machine's ethernet connection and I don't have wireshark in promiscuous mode).

Breaking this pcap down is easy, first we have the columns of the wireshark gui, they go as follows.
<ul>
  <li>No. = the order of the packets, the newer the packet the higher the number</li>
  <li>Time = The time when the packet was captured on the pcap file, measured in seconds by default but can be changed under the view menu.</li>
  <li>Source = the source address of the packet, where it came from</li>
  <li>Destination = the destination address of the packet, where it's going </li>
  <li>Protocol = the protocol the packet used</li>
  <li>Length = The total size of the packet, measured in bytes</li>
  <li>Info = short summary of the packets data. Can be helpful for filtering or for quick analysis</li>
</ul>

You will also notice that wireshark also colour codes the packets, this is to help us to quickly differentiate between network traffic types and can help us spot errors. The default ones that come preloaded as as follows.
![rules](Rules.png)

Wireshark can also let you add your own custom rules and colour code them, this is important for analysists as it can quickly highlight specific behaviours and allow faster response times.

# Breaking down a packet

With wireshark we can open up packets by double clicking on one to analyse them a little deeper. For this section I made a second pcap file where my VM opened youtube and I've used some filters to narrow down the packets.

![http](http.png)

This has been filtered to show any packets using the http protocol, because youtube is a secured website (HTTPS) we cannot view any data here, the OCSP packets is where the certificate is validated from the website

![tls](tls.png)

This screenshot shows the packets being filtered for the TLS protocol, this data is encrypted and cannot be viewed unless you have a method of decryption which isn't relivent to this project so I won't document this here. But you can get an idea of searches from packets that contain handshake protocols like the highlighted packet.

![dns](dns.png)

This shows the packets being filtered for DNS protocol, here you can get a rough idea of what the target was searching for from the names that appear in the response packets.

For this part I will open a packet from the DNS filters as there it'll be simple to break down compared to TLS.
![frame](frame.png)

Here is the packet broken down, its a bit overwhelming but we can break this down a little
<ul>
  <li>Frame = information relating to the packet </li>
  <li>Ethernet II = information about how the frame traveled through the ethernet connection from the router</li>
  <li>Internet Protocol Version 4 = information about how the packet traveled over IPv4</li>
  <li>User Datagram Protocol = information about how the packet traveled through the UPD protocol</li>
  <li>Domain Name System = information in the packet about the DNS</li>
</ul>
Below I will also break down important information about each of these sections as there is alot here that isn't as important for an analyist in most situations.

![frameopen](frameb.png)

Within this section contains information relating to the packet's frame, here the important information to keep in mind are as follows

<ul>
  <li>Arrival Time = The time when the packet arrived to its destination, goes as Month, Day, Year, Hour, Minute, Second.</li>
  <li>Frame Length = The size of the frame</li>
  <li>Protocols in frame = Contains the protocols used in the packet.</li>
</ul>

![eth](Eth2.png)

Within the Ethernet section this will display information pertaining to the path used by the frame on the eth0 network interface. On ethernet the addresses will be the MAC addresses but if this was wlan0 it would be 

![ip](IP.png)

This section describes the information relating to how the packet traveled over the internet. Important parts in this section are as follows.
<ul>
  <li>Fragmentation Offset = shows if the packet has been broken up into multiple packets, the flags section helps with refragmenting the frames</li>
  <li>Time to live = The amount of time that a packet lives in a network before being dropped. Measured in how many times it passes through a router (hops) </li>
  <li>Protocol = the protocol used in the frame, it shows in this case that the packet used UDP instead of TCP. UDP</li>
  <li>Source/Desintation Address = the path taken by the packet</li>
</ul>

![udp](udp.png)

This section contains information about the User Datagram Protocol, the main parts that help us are the source port and the destination port to follow the flow of traffic. If the packet was sent through the TCP protocol then it would show here instead.

![dnspacket](dnsp.png)

This part of the frame contains the information of the packet, referred to as the body. as you can see within this frame is the response to the request for the "youtube.com" domain back in packet 6941.

Now I will showcase some packet analysis of some malicous activity on a network. This pcap was sourced from <a href="https://www.malware-traffic-analysis.net/2024/07/30/index.html">This exercise </a>found on Malware-Traffic-Analysis.net

# Packet Analysis Write Up

For this section I will showcase some packet analysis of some malicous activity on a network. This pcap was sourced from <a href="https://www.malware-traffic-analysis.net/2024/07/30/index.html">This exercise </a>found on Malware-Traffic-Analysis.net

as per this exercise the objectives are as follows

<ol>
  <li>Summarise what happened during the incident. Breaking down the when, the who and the what hapened.</li>
  <li>Document the victims details, including their hostname, ip address, Mac Address and their user account name.</li>
  <li>Document any indicators of compromise present in the pcap, ip addresses, domains and url's accosicated with the malicious activity as well as any extra information like SHA256 hashes if any malware binaries can be extracted.</li>
</ol> 

first I will add some custom rules to wireshark that might be able to help me narrow the search down a little. <img width="999" height="112" alt="customrules" src="https://github.com/user-attachments/assets/00afc2ed-897d-4081-b173-2bea1928cdaf" />
<ol>
  <li>http.file_data matches "^MZ" will highlight and signatures of a windows exec file being ran</li>
  <li>http.user_agent contains "curl" will show any curl requests on the network</li>
  <li>tcp.flags.syn == 1 && tcp.flags.ack == 0 makes any potential port scanning traffic more visible as when portscanning the packets sent as a probe will drop once they receieve the syn/ack response from the destination but then will not send back the ack packet to inntiate the connection.</li>
  <li>http.request.method == "POST" will highlight post requests made in the pcap file</li>
  <li>http.request will just make any http requests more visible</li>
</ol>

Now that we have set up some custom rules, we can start with analysing the pcap file.

Our first objective is to identify the source of the attack and the victim. Good filters to check this are the DHCP and the NBNS filters. When using the DHCP filter on this pcap we have no displayed packets so we can try NBNS to get some information.
<img width="1878" height="1105" alt="nbns" src="https://github.com/user-attachments/assets/0f22848b-0453-42de-b232-3284bdad86e9" />
From this search we gained some insight into the Who section
<ol>
  <li>The victim's ip address is 172.16.1.66 </li>
  <li>Their mac address is 00:1e:64:ec:f3:08</li>
  <li>Their host name is DESKTOP-SKBR25F</li>
</ol>

The only thing we are missing now is the victim's windows account username. we can do this by filtering for kerberos traffic as kerberos is an authentication protocol that is used to verify the identity of a user or host on a windows machine. To filter for this we type kerberos.CNameString and then open a packet. From there we follow the steps taken in the image below and we right click the highlighted section and "apply as a column".

<img width="620" height="393" alt="image" src="https://github.com/user-attachments/assets/aaee8f53-ccb3-4415-9a53-bd1593091f28" />

From doing this we will be greated by this on wireshark.

<img width="1878" height="1105" alt="image" src="https://github.com/user-attachments/assets/2d398a31-621a-4130-b4f0-7ab860bfd9a2" />

Now we have the username of the victim "ccollier", even though it wasnt specified in the challenge I will also find the name of the user as this would be added to a real incident report within a professional setting. Using the Lightweight Directory Access Protocol (ldap) filter we can use the rule (ldap contains "CN=Users") to find the credentials of the user "ccollier". Which displays this on wireshark

<img width="1878" height="1105" alt="image" src="https://github.com/user-attachments/assets/d50cd1eb-72c0-4110-9cbb-f48ee770aed1" />

Now we know that "ccollier" is the username for Clark Collier. And with that we have completed one of the three tasks of this challenge.

Now lets move on to the Executive Summary and Indicators of compromise as we can fill in both at the same time while investigating. 

First off we are aware that the incident was about malware being executed on the victims machine, this helps alot in narrowing down our search. Two key things to look out for are.
<ol>
  <li> Weird website traffic from the host, Malware needs to be installed onto a computer and executed to work. We'd look for any traffic that deviates from the baseline of this user</li>
  <li> Ip checks. Most malware will check the external ip address of the host as well as query other information to send back to the host or to adjust its behaviour within the infected host.</li>
</ol>

With this information a good place to check is the DNS filters.

<img width="1870" height="508" alt="image" src="https://github.com/user-attachments/assets/e983a946-288b-4202-8bc3-47c442a04b90" />

Immedietly you can see there are multiple suspicious queries in this section, The major one being the Github request. especially the object/githubuser/com. Checking the ip address on a website like AbuseIPDB give us confirmation of malicious activity. 

<img width="1365" height="893" alt="image" src="https://github.com/user-attachments/assets/bd3bce13-a28c-4cec-ac09-9a1309c577ee" />

We can also see that once the malware was installed onto the computer it called out to ip-api.com to geolocate the victims computer. You can see this backed up by filtering through HTTP traffic, you'll see that a JSON file was downloaded from ip-api.

<img width="1425" height="148" alt="image" src="https://github.com/user-attachments/assets/8da4566a-afea-46c3-bd7d-4207df332aa9" />

Now we can start looking for some more indicators of compromise, ideally we want to find the attacker's ip address and the port they used following the download of the malware. I used the conversations tab to find anything that looks out of place.

To find connversations you would go to Analyze > Conversations. From there I filtered through the TCP conversations as there isn't anything more I can gather from looking at HTTP. I then order by Bytes. 

<img width="1878" height="837" alt="image" src="https://github.com/user-attachments/assets/6619910c-747a-45a9-8611-ccd247ec9ef8" />

What stands out alot about this selection is that most communication happening on the network is done through port 443 or HTTPS. This string of conversations happened on port 12132 which is abnormal. Researching into this port reveals that this port is commonly used as a command and control (C2) communication channel by the STRRAT malware for Data Exfiltration. 

To confirm this we will first look for some more signs of the STRRAT malware, on researching into this malware its common for remote access trojans to transmit data as cleartext, so by pushing ctrl+f and searching for the string STRRAT in the packet bytes we get these results.

<img width="1749" height="1105" alt="image" src="https://github.com/user-attachments/assets/f2426a93-0dc5-4c53-a50b-48a4f7da4121" />

As you can see that the destination ip is the suspected attackers and the port again is 12132. If we then follow the tcp stream we get this string of plaintext

<img width="1255" height="1105" alt="image" src="https://github.com/user-attachments/assets/56de365b-c335-4b3e-b988-d0f4dcdc8ad1" />

So with this we have found out three things
<ol>
  <li> The attackers IP is 141.98.10.79</li>
  <li> The attack is using port 12132 </li>
  <li> The Malware is STRRAT and is using C2 for Data exfiltration</li>
</ol>

Sadly I am unable to extract any hashes as it seems the file was downloaded over TLS rather than HTTP. Without the encrpytion keys I cannot access this.

Now finally we have to prove that data exfiltration happened during this pcap. 
We start this by filtering for traffic relating to the attackers ip address. We'd use the filter "ip.addr == 141.98.10.79" to achieve this. Then we click on any of the packets and follow the tcp stream to get the plaintext we had in the previous image.

<img width="1255" height="1105" alt="image" src="https://github.com/user-attachments/assets/5194fb3e-cf9d-4f4d-9a07-746cc39964be" />

By default red text means data being sent by source to the destination and blue is vice versa. This text also reveals that the victim sent over their username, the OS running on their pc and the antivirus in use too.

