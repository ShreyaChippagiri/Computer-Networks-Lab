LAB-05 
Name: Shreya Chippagiri 
USN : 1PI10IS134

TEAM MEMBERS: 
Neha Tj (USN: 1PI10IS133)-Hb  
Surabhi R(USN: 1PI10IS109)-R1
Vivek Agarwal (USN: 1PI10IS112)-Ha 
Shreya Chippagiri-R2
_______________________________________________________________________________________________________________________________________________

SET UP:

Ha            R1                       R2           Hb
  <---Ipv6-->   <----Tunnel-Ipv4------>  <--IPv6--->
  <---N1---->   <----N2--------------->  <--N3----->
 
N1 = Network1 - fd00:<usn1>::/64
N2 = Network2 - fd00:<usn2>
N3 = Network3 - fd00:<usn3>::/64
_______________________________________________________________________________________________________________________________________________

IPv6 and IPv4 ADDRESSES AND THE INTERFACES: 

The IP addresses were added via the following command: 
sudo ip -6 addr add <ipv6 address>/64 dev eth1 

IP addresses of R1: fd00:1001::52e5:49ff:fe1b:f172/64 dev eth1
                  172.30.109.1/24 eth2

IP Address of Ha: fd00:1001::52e5:49ff:fe1b:f0b4/64 dev eth1


IP Address of Hb: fd00:134::52e5:49ff:fe1b:f14e/64 dev eth1

IP addresses of R2: fd00:134::52e5:49ff:fe1b:f182/64 dev eth2
                  172.30.109.2/24 eth1 

student@pesit-To-be-filled-by-O-E-M:~$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 5c:d9:98:f6:03:04 brd ff:ff:ff:ff:ff:ff
    inet 192.168.13.41/21 brd 192.168.15.255 scope global eth1
    inet 172.30.109.2/24 scope global eth1
    inet6 fe80::5ed9:98ff:fef6:304/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 50:e5:49:1b:f1:72 brd ff:ff:ff:ff:ff:ff
    inet 192.168.13.41/21 brd 192.168.15.255 scope global eth2
    inet 172.16.10.2/24 scope global eth2
    inet6 fd00:134::52e5:49ff:fe1b:f182/64 scope global 
       valid_lft forever preferred_lft forever
    inet6 fe80::52e5:49ff:fe1b:f182/64 scope link 
       valid_lft forever preferred_lft forever
_______________________________________________________________________________________________________________________________________________

CONVERSION OF R1 and R2 TO ROUTERS: 

In order to convert the systems into routers, the following command was executed: 
sysctl –w net.ipv6.conf.all.forwarding=1

The above command was executed on R1 and R2. 

student@pesit-To-be-filled-by-O-E-M:~$ sudo sysctl -w net.ipv6.conf.all.forwarding=1
net.ipv6.conf.all.forwarding = 1
student@pesit-To-be-filled-by-O-E-M:~$ sudo sysctl -w net.ipv4.ip_forward=1
net.ipv4.ip_forward = 1
_______________________________________________________________________________________________________________________________________________

ROUTING TABLE ENTRIES: 

We had to add a routing table entry for Network1 to Communicate to Network3. 

The routing table entries were added via the following commands: 

sudo ip -6 route add <N3> via <e1-R1>

sudo ip -6 route add <N1> via <e2-R2>

The Routing table entries were added in Ha and Hb. 
_____________________________________________________________________________________________________________________________________________

CREATION OF THE TUNNEL: 

The Ipv4 tunnel was created between the Routers. R1 and R2. 

The following command was executed on R1 and R2. 

	1. sudo ip tunnel add mytun mode sit remote 172.30.109.1 local 172.30.109.2 dev eth1	(executed for R2)
	2. sudo ip link set mytun up 
	3. sudo ip -6 addr add 2002:<hexa dec equiv of the ipv4 address>::1/48 dev mytun
        4. sudo ip -6 route add fd00:<usn>::/64 dev mytun

remote refers to the IPv4 address of R1 which is the other end of the tunnel and local refers to the current router which is R2. 

Similar commands were executed on R1. The mode sit implies IPv6 tunnel over IPv4.


The 3rd and 4th commands are executed because we need to assign a IPv6 address to this tunnel derived from its IPv4 address and creating a routing entry for the other Ipv6 address derived from the IPv4 address at the other end of the tunnel. 

student@pesit-To-be-filled-by-O-E-M:~$ sudo ip tunnel add mytun mode sit remote 172.30.109.1 local 172.30.109.2 dev eth1
student@pesit-To-be-filled-by-O-E-M:~$ sudo ip link set mytun up
student@pesit-To-be-filled-by-O-E-M:~$ sudo ip -6 addr add 2002:ac1e:6d02::1/48 dev mytun
student@pesit-To-be-filled-by-O-E-M:~$ sudo ip -6 route add fd00:1001::/64 dev mytun
_____________________________________________________________________________________________________________________________________________

The last step was to ping Hb from Ha. 
The wireshark captures are attached. The file names are self explanatory.  

OBSERVATIONS: 
As we can see from the wireshark captures, the Ipv6 datagram is contained in the data part of Ipv4 datagram. 

CHALLENGES: 
	1. Network problems. The connection kept getting reset. 
	2. Taking the wireshark captures for the two interfaces on R1 and R2.Learnt to give the right capture filter for the hosts.Also learnt why wireshark keeps getting hung when typing the capture filter.
_______________________________________________________________________________________________________________________________________________

UDP CHAT: 

From host Ha, the following command was issued. 
nc -6 –u -l 12335

From host Hb, the following command was issued. 
nc -6 -u fd00:1001::52e5:49ff:fe1b:f0b4 12345

Whatever we typed in Hb was displayed on Ha.
_______________________________________________________________________________________________________________________________________________

SSH: 

The following command was issued on Hb. 

ssh student@fd00:1001::52e5:49ff:fe1b:f0b4

and we got the following output: 

student@pesit-To-be-filled-by-O-E-M:~/Desktop$ ssh student@fd00:1001::52e5:49ff:fe1b:f0b4
student@fd00:1001::52e5:49ff:fe1b:f0b4's password: 
Welcome to Ubuntu quantal (development branch) (GNU/Linux 3.2.0-26-generic-pae i686)

 * Documentation:  https://help.ubuntu.com/

529 packages can be updated.
0 updates are security updates.

Last login: Thu Mar 14 15:55:16 2013 from fd00:134::52e5:49ff:fe1b:f14e

We were able to access Ha from Hb. 
_______________________________________________________________________________________________________________________________________________

WEB SERVER:

We also had to type the ipv6 address of the destination in the browser and get the default page. The catch is that, this wont work until the Apache webserver is restarted. 

____________________________________________________________________________________________________________________________
