I recently read two articles about:
1) What happens when you type an url in the browser and press enter, and...
2) beginners guide for network troubleshooting. 

So I decided to combine them and write down what I learned about both making sense in one article. 


combinar estos 2 post para crear un post que resuma que aprendí

Usando como ejemplo que pasa cuando haces hit en una url, agregar los comandos de networking para hacer troubleshooting
Y agregar developers tools

https://medium.com/@maneesha.wijesinghe1/what-happens-when-you-type-an-url-in-the-browser-and-press-enter-bb0aa2449c1a
https://www.redhat.com/sysadmin/beginners-guide-network-troubleshooting-linux




Layer 1: The physical layer, or, are the cables connected?

What's a physical layer problem? A pysical interface being down.

How do we check if our physical interface is up? 
ip link show

And if it turns out to be down then just run:
ip link set ethX up

Layer 2: The data link layer, or, ok the cables are connected yet I can't speak to my neighbours house.

What's a data lnk layer problem? 
The data link layer is responsible for local network connectivity, so a problem here would be two hosts on the same LAN not being able to communicate.

It could be due to one host not having the correct gateway IP address to check for the ARP table, so, 

Check entries in the ARP table:
ip neighbor show

* If there was a problem with ARP we would see a resolution failure.

Force a new ARP discovery process by deleting an ARP entry:
ip neighbor delete 192.168.122.170 dev eth0

Layer 3: The network/internet layer (When you can't reach a host outside of the local network)

Ok,
the cables are connected
the local network configuration is correct 
but now you can't reach a host outside of the local network. 

1) Check machine local IP address
ip -br address show

* The lack of an IP address can be caused by a local misconfiguration, such as an incorrect network interface config file, or it can be caused by problems with DHCP.

ok, I do have an IP address configured, now let's ping the remote host:
ping www.google.com

* this is not definitive since many network admins block ICMP packets as a security precaution

Check the route your localhost uses to go out:
ip route show

Review the path the traffic takes in order to identify where the problem resides. If you are able to reach the remote host then it might means this is a layer 3 issue. Just remember that paths are not always the same

traceroute www.google.com


References:
https://networklessons.com/cisco/ccna-routing-switching-icnd1-100-105/traceroute