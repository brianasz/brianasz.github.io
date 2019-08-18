---
layout: post
title: The confusing haproxy and keepalived configuration I found.
---

Recently I had a very confusing question in my job about a configuration I knew nothing about. 
It was a configuration made before I joined the company and it was about an entry in /etc/hosts on every VPS of a specific project, let's call it project Panda!

The entry was the following:
```bash
192.168.3.200	prod-db-internal.panda.com
```

The question was, who is prod-db-internal.panda.com? and why is it accesible from the internet? (the customer pinged the IP and it returned a public IP).
It sounded like an easy question, but it turned out to be more than I expected. Before digging into the investigation, first let me show you a diagram of the infrastructure:

![Architecture Diagram](/images/architecture-haproxyblog.png){: .center-image }

As you can see, the short answer must be that it is one of the db servers but it wouldn't make sense, so the next short answer is that it must be one of the haproxy servers and most likely the master, right?, but if so, then why the hell is it public? (according to the customer), it shouldn't.

Well, let's find out and for once and for all, the purpose of it as well. 

Since the customer could ping it and returned an IP address, I did the same. But, locally from my machine, ping was not resolving the hostname, neither nslookup nor dig were returning any answer about it. How on earth were they able to ping it? I don't know, but they probably had an entry in their local /etc/hosts for it. 

So, back to the investigation. 
I knew the entry was being added by an internal chef cookbook to the VPS's (it is based on the community hostfile cookbook) but I didn't know how it was being added by the cookbook. After a quick look at it, it turned out it was being directly added as an static IP:

```ruby
default['hostsfile']['statics'] = [
  { ip_address: '192.168.3.200', host_name: 'prod-db-internal.panda.com' }
```

But, what was the purpose of it?... From my knowledge about the project I knew the following:
1. The db cluster was composed of 3 nodes.
2. The cluster was being load balanced by an haproxy cluster of two nodes... and...
3. ... after reviewing a little bit more, the domain name (prod-db-internal.panda.com) was actually being used by the java application of the project, as such:

```
db.url=jdbc:mysql://prod-db-internal.panda.com/
```

It started to make more sense, that domain name was the entry from the java application to the database. But, still, who was that IP address?

Next step I checked for the private IP addresses of the haproxy and db servers. At first I used ```ifconfig``` which was misleading and it made the whole research a mess because it didn't show all the information I needed, specifically that one network interface (eth2) had secondary IP addresses. ```ifconfig``` only showed me one IP address for eth2 interface, omitting the secondary where in fact was the one I was looking for, making the investigation process a mess for a moment. Rooky mistake from my part, I admit it!

After checking the IP addresses with ```ip addr show```, I got the answer I was looking for. 
None of the db servers were ```192.168.3.200``` IP, but one of the haproxy servers was. 

**prod-haproxy-000.panda.com "ip addr show" command output:**
```bash
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether bc:76:4e:05:6c:1c brd ff:ff:ff:ff:ff:ff
    inet 192.168.3.4/24 brd 192.168.3.255 scope global eth2
    inet 192.168.3.200/32 scope global eth2
    inet6 fe80::be76:4eff:fe05:6c1c/64 scope link
       valid_lft forever preferred_lft forever
```

Bingo!!! 
But, what happens then with the second haproxy server? (It had a different secondary IP address)

**prod-haproxy-001.panda.com "ip addr show" command output:**
```bash
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether bc:76:4e:05:eb:70 brd ff:ff:ff:ff:ff:ff
    inet 192.168.3.5/24 brd 192.168.3.255 scope global eth2
    inet 192.168.3.201/32 scope global eth2
    inet6 fe80::be76:4eff:fe05:eb70/64 scope link
       valid_lft forever preferred_lft forever
```

I'm not very knowledgeable about haproxy configuration so I remained confused. 

So, what I didn't thought about before was that a single haproxy server can act as a load balancer for the db cluster but having such configuration would have made the LB the single point of failure for the system. So for true high availability 2 haproxy servers (load balancers) would be needed with some sort of failover. The thing is, haproxy doesn't provide a way to do that by itself, however, the failover capability hence the true high available system can be accomplished by using VRRP and keepalived, which are the technologies being used in this particular project. 

Accoding to wikipedia: 

VRRP is:
> Virtual Router Redundancy Protocol (VRRP) is a computer networking protocol that provides for automatic assignment of available 
> Internet Protocol (IP) routers to participating hosts. This increases the availability and reliability of routing paths via automatic
> default gateway selections on an IP subnetwork.
> The protocol achieves this by creation of virtual routers, which are an abstract representation of multiple routers, i.e. master and
> backup routers, acting as a group. The default gateway of a participating host is assigned to the virtual router instead of a 
> physical router. If the physical router that is routing packets on behalf of the virtual router fails, another physical router is 
> selected to automatically replace it. The physical router that is forwarding packets at any given time is called the master router.

Keepalived is:
> Keepalived is a routing software written in C. The main goal of this project is to provide simple and robust facilities for 
> loadbalancing and high-availability to Linux system and Linux based infrastructures. 

The most common configuration in these cases would be that if the load balancer went down (node prod-haproxy-000.panda.com), the website would become unavailable. The easiest way to avoid having the load balancer being a single point of failure is to set up an active / passive HAProxy pair. This would require two HAProxy servers and a virtual IP that can float between the two servers. The active HAProxy server would handle all of the requests unless it went down, at which point the passive HAProxy server would take over the requests, and VRRP and Keepalived can help with it.

Keepalived acts as a daemon running on both haproxy servers and checks for the haproxy process status. On its configuration there is a priority flag that defines which node of the two haproxies (LB nodes) is the master or active node. Master has a higher priority. If the haproxy process fails on the master node, keepalived will lower the priority of the failed node and failover to the other node. So, basically, the two load balancer nodes monitor each other using keepalived, and if the master fails, the slave becomes the master, which means the users will not notice any disruption of the service. Above information is under the assumption that a shared IP is being used, which is nothing more than a shared IP between the two load-balancers that is active only on one of them at any moment (on whichever is the master).

But, that is not how the configuration on this project was set up. This particular project was not using a shared IP and I confirmed that because,
1. IP ```192.168.3.200``` belonged specifically to node ```prod-haproxy-000.panda.com```, so it wasn't interchangeable, and...
2. Because ```keepalived``` for this project was being configured by the keepalive chef cookbook version 1.2.0 and it had the ```default['keepalived']['shared_address']``` attribute set to false, meaning it is not working under that shared IP configuration.

In fact, this particular project, by default (well, by the static entry added in /etc/hosts) is identifying node ```192.168.3.200 - prod-haproxy-000.panda.com``` as the load balancer for the db cluster and the application uses it for that, that's the one being identified as the master as long as the rest of the architecture can tell. Always!...

But, now let's take a look at the keepalived configuration for each LB node.

**Node prod-haproxy-000.panda.com:**
```bash
$ cat /etc/keepalived/keepalived.conf
# ------------------------------------------------
# Chef controlled file - Edits will be overwritten
# Environment - prod
# Node - prod-haproxy-000.panda.com
# ------------------------------------------------

vrrp_script chk_haproxy {
  script "killall -0 haproxy"
  interval 2
  weight -2
}


vrrp_instance 192.168.3.200 {
  interface eth2
  state MASTER
  virtual_router_id 200
  priority 100
  virtual_ipaddress {
    192.168.3.200
  }
  track_script {
    chk_haproxy
  }
}
vrrp_instance 192.168.3.201 {
  interface eth2
  state MASTER
  virtual_router_id 201
  priority 99
  virtual_ipaddress {
    192.168.3.201
  }
  track_script {
    chk_haproxy
  }
}
```
**Node prod-haproxy-001.panda.com:**
```bash
$ cat /etc/keepalived/keepalived.conf
# ------------------------------------------------
# Chef controlled file - Edits will be overwritten
# Environment - prod
# Node - prod-haproxy-001.panda.com
# ------------------------------------------------

vrrp_script chk_haproxy {
  script "killall -0 haproxy"
  interval 2
  weight -2
}


vrrp_instance 192.168.3.201 {
  interface eth2
  state MASTER
  virtual_router_id 201
  priority 100
  virtual_ipaddress {
    192.168.3.201
  }
  track_script {
    chk_haproxy
  }
}
vrrp_instance 192.168.3.200 {
  interface eth2
  state MASTER
  virtual_router_id 200
  priority 99
  virtual_ipaddress {
    192.168.3.200
  }
  track_script {
    chk_haproxy
  }
}
```

From above keepalived configuration both nodes are always the master but only one is being used at a time. And from the ```/etc/hosts``` entry, it results obvious that the application servers are always reaching out to ```prod-haproxy-000.panda.com``` as the load balancer for the db cluster. Only when the haproxy process goes down on it, then the keepalived configuration on ```prod-haproxy-000.panda.com```, turns to the slave node (```prod-haproxy-001.panda.com```), because it is the next in the priority line, and with an out of the box forwarding process keepalived then routes all the traffic coming to ```prod-haproxy-000.panda.com``` to ```prod-haproxy-001.panda.com```, which ultimately acts as the new master of the high available load balancer set up for the db cluster. Finally, when the haproxy process is restored on ```prod-haproxy-000.panda.com```, then it automatically becomes again the master load balancer node.

## Conclusion ##
It was a little bit confusing to map all the different components of this setup mainly because I had no knowledge of VRRP and keepalived. Secondly because the settings were on different cookbooks and files so I had to map everything together, and finally because the current configuration is a little bit different of what I commonly found on the internet (because no shared IP is being used). Actually, if someone knows how this specific configuration is named I would be glad to know or be corrected if any asumptinos made here are wrong. 

Anyway, I'm glad I was asked about this set up, otherwise I wouln't know how it worked and wouldn't have improved my skills. Now if someone else ask, I can speak about it with more confidence. 

## References ##
https://www.digitalocean.com/community/tutorials/how-to-set-up-highly-available-web-servers-with-keepalived-and-floating-ips-on-ubuntu-14-04
http://www.haproxy.org/download/1.2/doc/architecture.txt
https://www.howtoforge.com/setting-up-a-high-availability-load-balancer-with-haproxy-keepalived-on-debian-lenny



