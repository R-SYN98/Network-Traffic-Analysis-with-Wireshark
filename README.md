# Network-Traffic-Analysis-with-Wireshark
A write up on my journey to understand how to analyse network traffic with packet sniffers.


# Objective
The objective of this project was to help me understand packet analysis and to document my understanding of how to use packet sniffers (mainly wireshark) to detect possible malicious activity on a network.

# Write up

In this project I have ran wireshark within my Kali Linux VM as wireshark comes preinstalled with this disto and I like to keep my cybersecurity work seperate from my host machine. 

To start we open up wireshark and are greated with this screen below
![Wireshark screen](
wiresharkscreen)

This screen is where we choose the perameters of the packet capture ("pcap") before we start the capture. Here we can choose which network device that the pcap will capture traffic from and we can also apply filters to the pcap. some filters include
<ul>
  <li>"host" = will capture traffic for a specific ip address, both outgoing and incoming/li>
  <li>"net" = will capture traffic on a specific subnet</li></li>
  <li>"port" = captures traffic on specitic ports, you can also filter this further by adding "tcp" or "udp" before port to filter through those specific protocols </li>
  <li>either</li>
</ul

you can even combine filters with the terms,"and","or" and "not" to get very specific results on the pcap.
For example.

(host 192.168.1.1 and not port 22) will only capture traffic from the host 192.168.1.1 but ignore any traffic coming through on port 22.
