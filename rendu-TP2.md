
🌞 Tout le monde doit pouvoir se ping
node 1 -> node 2

```
NODE1> ping 10.2.1.12

84 bytes from 10.2.1.12 icmp_seq=1 ttl=64 time=0.417 ms
```
node 3 -> node 4

```
NODE3> ping 10.2.2.12

84 bytes from 10.2.2.12 icmp_seq=1 ttl=64 time=0.571 ms
```

node 1 -> node 3
```
NODE1> ping 10.2.2.11

84 bytes from 10.2.2.11 icmp_seq=1 ttl=63 time=30.241 ms
```

node 4 -> node 2
```
NODE4> ping 10.2.1.12

84 bytes from 10.2.1.12 icmp_seq=1 ttl=63 time=29.855 ms
```
 Configurer une adresse IP sur l'interface qui pointe vers le nuage GNS3
 vérifier que vous avez bien récupéré une adresse IP avec un show ip int br en mode enable
```
R1#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            192.168.122.28  YES DHCP   up                    up      
FastEthernet1/0            10.2.1.254      YES manual up                    up      
FastEthernet1/1            10.2.2.254      YES manual up                    up      
FastEthernet2/0            unassigned      YES NVRAM  administratively down down    
FastEthernet2/1            unassigned      YES NVRAM  administratively down down    
R1#*
```
🌞 Prouver que...

    r1 peut ping internet (une adresse IP publique)
```
R1#ping 8.8.8.8

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/19/24 ms

```
➜ Configurer un NAT simpliste sur votre routeur
🌞 Proooooooooof or lie

    vérifier que toutes les machines peuvent désormais ping internet
```
NODE1> ping 1.1.1.1

84 bytes from 1.1.1.1 icmp_seq=1 ttl=253 time=29.376 ms

NODE2> ping 1.1.1.1

84 bytes from 1.1.1.1 icmp_seq=1 ttl=253 time=30.807 ms

NODE3> ping 1.1.1.1

84 bytes from 1.1.1.1 icmp_seq=1 ttl=253 time=29.946 ms

NODE4> ping 1.1.1.1 

84 bytes from 1.1.1.1 icmp_seq=1 ttl=253 time=30.305 ms

```
Vrai accès internet clients
Configurer 1.1.1.1 comme serveur DNS sur tous les clients
🌞 Prove it depuis node1, effectuez un ping vers efrei.fr

```
NODE1> ip dns 1.1.1.1

NODE1> show ip

NAME        : NODE1[1]
IP/MASK     : 10.2.1.11/24
GATEWAY     : 10.2.1.254
DNS         : 1.1.1.1  
MAC         : 00:50:79:66:68:00
LPORT       : 20026
RHOST:PORT  : 127.0.0.1:20027
MTU         : 1500

NODE1> ping efrei.fr
efrei.fr resolved to 51.210.229.203

84 bytes from 51.210.229.203 icmp_seq=1 ttl=253 time=30.222 ms
84 bytes from 51.210.229.203 icmp_seq=2 ttl=253 time=40.418 ms

```

🌞 Test test test : ajouter un nouveau VPCS au LAN1, le bro node5.tp2.efrei

    demander une adresse IP en DHCP avec lui
    constater que vous avez aussi récupéré l'adresse de la passerelle et l'adresse du serveur DNS
    constater que vous pouvez instantanément ping efrei.fr

```
node5> ip dhcp
DORA IP 10.2.1.100/24 GW 10.2.1.254

node5> show ip

NAME        : node5[1]
IP/MASK     : 10.2.1.100/24
GATEWAY     : 10.2.1.254
DNS         : 1.1.1.1  
DHCP SERVER : 10.2.1.2
DHCP LEASE  : 3597, 3600/900/1800
MAC         : 00:50:79:66:68:04
LPORT       : 20035
RHOST:PORT  : 127.0.0.1:20036
MTU         : 1500

node5> ping efrei.fr
efrei.fr resolved to 51.210.229.203

84 bytes from 51.210.229.203 icmp_seq=1 ttl=253 time=29.635 ms

```

 ARP spoofing
 capturez le trafic qui circule quand node1 envoie des ping vers efrei.fr
 on doit voir un beau spam de ARP (vers la passerelle, et vers le client victime node1)
on doit voir des ping qui viennent de node1 et qui vont vers l'adresse IP publique qui correspond à efrei.fr
ha et du coup on doit aussi voir la requête DNS du client qui est partie vers 1.1.1.1, avec la réponse
```
[crackito@localhost ~]$ sudo tcpdump -i enp0s9 -n "arp or port 53 or icmp"
[sudo] password for crackito: 
dropped privs to tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on enp0s9, link-type EN10MB (Ethernet), snapshot length 262144 bytes
17:09:14.758746 ARP, Reply 10.2.1.254 is-at 08:00:27:d2:ee:97, length 28
17:09:14.818689 ARP, Reply 10.2.1.11 is-at 08:00:27:d2:ee:97, length 28
17:09:17.344078 IP 10.2.1.11.jdmn-port > 1.1.1.1.domain: 4021+ A? efrei.fr. (26)
17:09:17.344249 IP 10.2.1.11.jdmn-port > 1.1.1.1.domain: 4021+ A? efrei.fr. (26)
17:09:17.347287 ARP, Reply 10.2.1.254 is-at 08:00:27:d2:ee:97, length 28
17:09:17.373764 IP 1.1.1.1.domain > 10.2.1.11.jdmn-port: 4021 1/0/0 A 51.210.229.203 (42)
17:09:17.373845 IP 1.1.1.1.domain > 10.2.1.11.jdmn-port: 4021 1/0/0 A 51.210.229.203 (42)
17:09:17.376911 IP 10.2.1.11 > 51.210.229.203: ICMP echo request, id 10420, seq 1, length 64
17:09:17.376919 IP 10.2.1.11 > 51.210.229.203: ICMP echo request, id 10420, seq 1, length 64
17:09:17.404718 IP 51.210.229.203 > 10.2.1.11: ICMP echo reply, id 10420, seq 1, length 64
17:09:17.404786 IP 51.210.229.203 > 10.2.1.11: ICMP echo reply, id 10420, seq 1, length 64
17:09:17.557011 ARP, Reply 10.2.1.11 is-at 08:00:27:d2:ee:97, length 28

```
Installer votre rogue DHCP server
🌞 Test test test : ajouter un nouveau VPCS au LAN1, le bro node6.tp2.efrei

    demander une adresse IP en DHCP avec lui
    constater que vous avez aussi récupéré l'adresse de l'attaquant en passerelle (et 1.1.1.1 en serveur DNS)
    constater que vous pouvez instantanément ping efrei.fr, malheureusement

```
node6> ip dhcp 
DORA IP 10.2.1.202/24 GW 10.2.1.2

node6> show ip

NAME        : node6[1]
IP/MASK     : 10.2.1.202/24
GATEWAY     : 10.2.1.2
DNS         : 1.1.1.1  
DHCP SERVER : 10.2.1.2
DHCP LEASE  : 3399, 3401/900/1800
MAC         : 00:50:79:66:68:05
LPORT       : 20038
RHOST:PORT  : 127.0.0.1:20039
MTU         : 1500

node6> ping efrei.fr
efrei.fr resolved to 51.210.229.203

84 bytes from 51.210.229.203 icmp_seq=1 ttl=253 time=30.179 ms

```
Wireshark this
on doit voir des ping qui viennent de node6 et qui vont vers l'adresse IP publique qui correspond à efrei.fr
ha et du coup on doit aussi voir la requête DNS du client qui est partie vers 1.1.1.1, avec la réponse
```
[crackito@localhost ~]$ tcpdump -r p3_dhcp_mitm.pcap -n "port 53 or icmp"
reading from file p3_dhcp_mitm.pcap, link-type EN10MB (Ethernet), snapshot length 262144
17:48:57.331602 IP 10.2.1.202.54765 > 1.1.1.1.domain: 43320+ A? efrei.fr. (26)
17:48:57.331649 IP 10.2.1.2 > 10.2.1.202: ICMP redirect 1.1.1.1 to host 10.2.1.254, length 62
17:48:57.331672 IP 10.2.1.202.54765 > 1.1.1.1.domain: 43320+ A? efrei.fr. (26)
17:48:57.378794 IP 10.2.1.202 > 51.210.229.203: ICMP echo request, id 31165, seq 1, length 64
17:48:57.378847 IP 10.2.1.2 > 10.2.1.202: ICMP redirect 51.210.229.203 to host 10.2.1.254, length 92
17:48:57.378876 IP 10.2.1.202 > 51.210.229.203: ICMP echo request, id 31165, seq 1, length 64
17:48:58.437877 IP 10.2.1.202 > 51.210.229.203: ICMP echo request, id 31421, seq 2, length 64
17:48:58.437978 IP 10.2.1.2 > 10.2.1.202: ICMP redirect 51.210.229.203 to host 10.2.1.254, length 92
17:48:58.438016 IP 10.2.1.202 > 51.210.229.203: ICMP echo request, id 31421, seq 2, length 64
17:48:59.497184 IP 10.2.1.202 > 51.210.229.203: ICMP echo request, id 31677, seq 3, length 64
17:48:59.497280 IP 10.2.1.2 > 10.2.1.202: ICMP redirect 51.210.229.203 to host 10.2.1.254, length 92
17:48:59.497338 IP 10.2.1.202 > 51.210.229.203: ICMP echo request, id 31677, seq 3, length 64
17:49:00.556889 IP 10.2.1.202 > 51.210.229.203: ICMP echo request, id 31933, seq 4, length 64
17:49:00.556979 IP 10.2.1.2 > 10.2.1.202: ICMP redirect 51.210.229.203 to host 10.2.1.254, length 92
17:49:00.557012 IP 10.2.1.202 > 51.210.229.203: ICMP echo request, id 31933, seq 4, length 64
17:49:01.612045 IP 10.2.1.202 > 51.210.229.203: ICMP echo request, id 32189, seq 5, length 64
17:49:01.612148 IP 10.2.1.2 > 10.2.1.202: ICMP redirect 51.210.229.203 to host 10.2.1.254, length 92
17:49:01.612197 IP 10.2.1.202 > 51.210.229.203: ICMP echo request, id 32189, seq 5, length 64

```
Vérification 
```
[crackito@localhost ~]$ ps -ef | grep dnsmasq
root        1816    1546  0 17:59 pts/1    00:00:00 sudo dnsmasq -C /tmp/dnsmasq.conf -q -d
root        1818    1816  0 17:59 pts/2    00:00:00 sudo dnsmasq -C /tmp/dnsmasq.conf -q -d
root        1819    1818  0 17:59 pts/2    00:00:00 dnsmasq -C /tmp/dnsmasq.conf -q -d
crackito    1824    1499  0 18:00 pts/0    00:00:00 grep --color=auto dnsmasq
[crackito@localhost ~]$ sudo ss -ulnp | grep dnsmasq
[sudo] password for crackito: 
UNCONN 0      0            0.0.0.0:53        0.0.0.0:*    users:(("dnsmasq",pid=1819,fd=4))   
UNCONN 0      0               [::]:53           [::]:*    users:(("dnsmasq",pid=1819,fd=6))  
```
```
[crackito@localhost ~]$ sudo ss -ulnp | grep dnsmasq
[sudo] password for crackito: 
UNCONN 0      0            0.0.0.0:53        0.0.0.0:*    users:(("dnsmasq",pid=1819,fd=4))   
UNCONN 0      0               [::]:53           [::]:*    users:(("dnsmasq",pid=1819,fd=6))   

```

```
[crackito@localhost ~]$ dig @1.1.1.1 efrei.fr

; <<>> DiG 9.18.33 <<>> @1.1.1.1 efrei.fr
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51923
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;efrei.fr.			IN	A

;; ANSWER SECTION:
efrei.fr.		3600	IN	A	51.210.229.203

;; Query time: 40 msec
;; SERVER: 1.1.1.1#53(1.1.1.1) (UDP)
;; WHEN: Sat Jan 17 18:02:08 CET 2026
;; MSG SIZE  rcvd: 53

```

```
[crackito@localhost ~]$ dig @127.0.0.1 efrei.fr

; <<>> DiG 9.18.33 <<>> @127.0.0.1 efrei.fr
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8099
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;efrei.fr.			IN	A

;; ANSWER SECTION:
efrei.fr.		0	IN	A	10.2.1.2

;; Query time: 1 msec
;; SERVER: 127.0.0.1#53(127.0.0.1) (UDP)
;; WHEN: Sat Jan 17 18:02:33 CET 2026
;; MSG SIZE  rcvd: 53

```
tout est ok

🌞 Relance ton attaque DHCP spoof depuis la machine attaquante

    il faut modifier ta conf : ton rogue DHCP doit indiquer au client l'adresse IP de la machine attaquante comme serveur DNS
    
```
{
    "name": "domain-name-servers",
    "data": "10.2.1.2"
}
```
modifié avec l'adresse de ma machine attaquante

🌞 Test test test : ajouter un nouveau VPCS au LAN1, le bro node7.tp2.efrei

    récupère une adresse IP en DHCP
    constate que l'adresse IP du DNS que t'as récupéré c'est la machine attaquante
    fais un ping efrei.fr et constate que tu ping la machine attaquante
```
node7> ip dhcp
DORA IP 10.2.1.204/24 GW 10.2.1.2

node7> ping efrei.fr
efrei.fr resolved to 10.2.1.2

84 bytes from 10.2.1.2 icmp_seq=1 ttl=64 time=1.149 ms
84 bytes from 10.2.1.2 icmp_seq=2 ttl=64 time=1.928 ms
84 bytes from 10.2.1.2 icmp_seq=3 ttl=64 time=2.185 ms
84 bytes from 10.2.1.2 icmp_seq=4 ttl=64 time=2.185 ms

```
le vpcs ping bine ma machine 10.2.1.2 au lieu de efrei.fr

WIRESHARK

la requête DNS du client
la réponse de votre DNS malveillant
```
[crackito@localhost ~]$ tcpdump -r part4_dns_spoof.pcap -n "udp port 53"
reading from file part4_dns_spoof.pcap, link-type EN10MB (Ethernet), snapshot length 262144
18:08:51.629257 IP 10.2.1.204.orion-rmi-reg > 10.2.1.2.domain: 50340+ A? efrei.fr. (26)
18:08:51.630528 IP 10.2.1.2.domain > 10.2.1.204.orion-rmi-reg: 50340* 1/0/0 A 10.2.1.2 (42)

```
le ping qui part vers votre machine attaquante (genre l'IP de destination c'est la machine attaquante).
```
[crackito@localhost ~]$ tcpdump -r part4_dns_spoof.pcap -n "icmp"
reading from file part4_dns_spoof.pcap, link-type EN10MB (Ethernet), snapshot length 262144
18:08:51.632762 IP 10.2.1.204 > 10.2.1.2: ICMP echo request, id 7618, seq 1, length 64
18:08:51.632792 IP 10.2.1.2 > 10.2.1.204: ICMP echo reply, id 7618, seq 1, length 64
18:08:52.659827 IP 10.2.1.204 > 10.2.1.2: ICMP echo request, id 7874, seq 2, length 64
18:08:52.659900 IP 10.2.1.2 > 10.2.1.204: ICMP echo reply, id 7874, seq 2, length 64
18:08:53.688268 IP 10.2.1.204 > 10.2.1.2: ICMP echo request, id 8130, seq 3, length 64
18:08:53.688349 IP 10.2.1.2 > 10.2.1.204: ICMP echo reply, id 8130, seq 3, length 64
18:08:54.716375 IP 10.2.1.204 > 10.2.1.2: ICMP echo request, id 8386, seq 4, length 64
18:08:54.716455 IP 10.2.1.2 > 10.2.1.204: ICMP echo reply, id 8386, seq 4, length 64

```
