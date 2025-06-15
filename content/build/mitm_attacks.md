---
title: "mitm attacks"
date: "2025-06-10"
description: "building on network security, understanding and executing 'man in the middle attacks'"
tags: ["cysec","network security"]
mermaid: true
---

having gained some knowhow of network security, i thought it was time to do some cool shit — let the hacking begin!

i came across a video by network chuck on sniffing packets, by executing what is known as a man in the middle attack, and decided to do it myself.

mitm allows us to sniff all network packets being sent from a device to the router and vice-versa.

essentially, how this attack works is by bluffing your router into believing that your (the hacker’s) device is the target device and the target device into believing that your (the hacker’s) device is the router. this happens through arp (address resolution protocol) poisoning. 

<div style='text-align:center;'>

{{< mermaid >}}
flowchart LR
A["router"]--->|"packet"|B["your device"]--->|"packet"|C["target device"]
C["target device"]--->|"packet"|B["your device"]--->|"packet"|A["router"]
{{< /mermaid >}}

</div>

so now, every packet that was supposed to be sent to the router to be further forwarded to the internet now firstly goes through your device which then forwards it to the router to be sent further, and everything coming from the internet through the router firstly enters your device and then is forwarded to the target device.

---

executing it out:

1. i need three major tools for this - 
    1. nmap: for fixing my crosshair on the target — analysing the network and identifying the connected devices
    2. ettercap: for executing arp poisioning
    3. wireshark: for analysing the packets being exchanged
2. getting my private ip -
    
    ran ifconfig, got a huge response, out of which the ‘en0’ part is important:
    
    ![image.png](/img/build/mitm_attacks/image.png)
    
    why en0? because that is used to refer to your wifi interface (en1 is used for lan)(generally).
    
    what can we gauge out of this? -
    
    - ether
        
        ![image.png](/img/build/mitm_attacks/image%201.png)
        
        this is my device’s mac address
        
    - inet6
        
        ![image.png](/img/build/mitm_attacks/image%202.png)
        
        this is my local ipv6 address
        
    - inet
        
        ![image.png](/img/build/mitm_attacks/image%203.png)
        
        this gives a bunch of information. 192.168.1.6 is my local ipv4 address.
        
        0xffffff00 is the netmask, which is in hex format. we’ll convert this into the 8bit format. since one hex character is 4 bit (0 to 2^4-1), we divide the hex into groups of 2 to get - ff ff ff 00, which when converted gives 255.255.255.0 - this implies that the first **24** bits are for the network and the last 8 for devices - .0 to .255
        
        further .255 is the broadcast address, which means anything directed to .255 is broadcasted to all devices connected to the network. // can some attack happen via this?
        
        since .0 is reserved for the network id and .255 for broadcast, .1-.254 can be occupied by devices.
        
        now, how do we get the network subnet?
        
        the whole ip network - 192.168.0.0/16 
        
        a subnet - 192.168.1.0/24
        
        a device - 192.168.1.1 
        
        // the part after ‘/’ implies the number of bits saved for the network.
        
        so our subnet is 192.168.1.0/24
        
3. enumeration -
    1. ran the follwoing nmap command to view a list of devices connected to my network and list the open/closed ports associated with that device.
        
        ```bash
        sudo nmap -sN <subnet_for_my_home_network>
        ```
        
        the flag -sN implies a stealth TCP null scan, ie, it does not use standard TCP flags like SYN, ACK. if the response is none that implies that the port is open. in case the response is RST(reset), it implies that the port is closed. why? because tcp’s rfc says that if a packet does not match the existing connection and does not have a header like SYN or FIN then the system should respond with an RST if the port is closed. 
        
        i got the following response:
        
        ![Screenshot 2025-05-26 at 15.07.10.png](/img/build/mitm_attacks/Screenshot_2025-05-26_at_15.07.10.png)
        
        this gives me the ip addresses and mac ids of the devices connected on my network
        
        // on further reading, i understood that a no response doesn’t always necessarily imply that the port is open, it might as well be being blocked by a firewall. 
        
        // i also got to know that i can get the list of devices through the ‘-sn’ flag than the ‘-sN’ flag. the key differences b/w -sn and -sN are:
        
        1. -sn does a device scan through icmp echo requests (ping), does not scan ports
        2. -sN scans ports by sending tcp null requests, we get the device info along with port info
    2. so then i ran this command:
        
        ```bash
        sudo nmap -sn <subnet_for_my_home_network>
        ```
        
        got the following response:
        
        ![image.png](/img/build/mitm_attacks/image%204.png)
        
        this gives all the devices connected to my network. .9 is my phone (iphone also specified in the previous response), .7 my alexa (says amazon technologies) and .6 is my laptop. 
        
        what’s weird is that other devices connected to my network are not being displayed. 
        
        ^^ need to look into this
        
        let’s attack my phone — ipv4: 192.168.1.9
        
    3. executing the attack
        
        well executing it is pretty chill, all i have to do is run the following command:
        
        ```bash
        sudo ettercap -T -S -i en0 -M arp:remote /192.168.1.1// /192.168.1.9//
        ```
        
        well that’s a very long one, here’s what it all means:
        
        - **-T**: for text only interface
        - **-S**: for sm shi i didnt understand
        - **-i**: for specifying the interface
        - **-M**: for man in the middle attacks
        - **arp:remote /<ip1>// /<ip2>//**: arp signifies that it must be done via arp spoofing, ip1 is the router’s ip and ip2 the target device’s
        
        here is what happened after that:
        
        ![image.png](/img/build/mitm_attacks/image%205.png)
        
        ![image.png](/img/build/mitm_attacks/image%206.png)
        
        // this is definitely something i couldnt make sense of so i opened it on wireshark.
        
    4. sniffing the packets via wireshark
        
        opened wireshark, opened the ‘eth0’ interface, and applied the following filter to see traffic coming in/going out from my phone
        
        ![image.png](/img/build/mitm_attacks/image%207.png)
        
        i opened [www.youtube.com](http://www.youtube.com) on my phone and saw packets being exchanged with 172.217.166.14 which on lookup turned out to be google’s server. the application data was ofc encrypted and i couldnt figure out what it was.
        
        ![image.png](/img/build/mitm_attacks/image%208.png)
        
        ![image.png](/img/build/mitm_attacks/image%209.png)
        
        then i tried a few different websites like [amazon.in](http://amazon.in) which interacted with a no. of ips all of which belonged to amazon web services. 
        
        but this isnt going to let me find out what exact websites the target is going to.
        
        provided, they are secured. in case they arent, ie, in the case of http websites (i visited [http://www.testingmcafeesites.com](http://www.testingmcafeesites.com/)), you can directly see the application data being transferred in plain text.
        
        i added the following filter to see only http protocol packets:
        
        ![image.png](/img/build/mitm_attacks/image%2010.png)
        
        then followed the http stream of the latest one
        
        ![image.png](/img/build/mitm_attacks/e938b04f-1de0-4544-af88-44fa6fba3905.png)
        
        and got this:
        
        ![image.png](/img/build/mitm_attacks/image%2011.png)
        
        ^^ this essentially gives away the entire response being returned from the site in plain text. this is why http isnt secure.
        
        another way to get to know about the websites being visited is by looking at the dns requests. i updated the filter to only view dns requests and visited instagram, my personal webpage, and - uhm - a little questionable site.
        
        ![image.png](/img/build/mitm_attacks/image%2012.png)
        
        then i visited patrick collisions [advice](https://patrickcollison.com/advice) and that too is visible.
        
        ![image.png](/img/build/mitm_attacks/image%2013.png)
        

well yea, i guess that’s it for mitm attacks. very fun, but would’ve been better if i’d’ve executed it on a different device than my own phone. will figure out why nmap’s not showing other devices and see what could be done to fix that.

---

update-

i found a better way to do this and that is by using bettercap. just the commands used change, the basic principles remain the same. 

there’s no need for nmap now as bettercap itself can show me the devices connected to my router.

sudo bettercap —> net.probe on —> net.show

![image.png](/img/build/mitm_attacks/image%2014.png)

![image.png](/img/build/mitm_attacks/image%2015.png)

then go over to wireshark to analyse the packets.

lol i just intercepted my dad’s computer