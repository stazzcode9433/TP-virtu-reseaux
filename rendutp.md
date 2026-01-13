🌞 Déterminer l'adresse MAC de vos deux machines
🌞 Définir une IP statique sur les deux machines
🌞 Proof !
VM1:

```
VPCS> ip 10.1.1.1
```
```
VPCS> ip 10.1.1.2
```
```
VPCS> show ip

NAME        : VPCS[1]
IP/MASK     : 10.1.1.1/24
GATEWAY     : 0.0.0.0
DNS         : 
MAC         : 00:50:79:66:68:00
LPORT       : 20002
RHOST:PORT  : 127.0.0.1:20003
MTU         : 1500

```
VM2:

```
VPCS> show ip

NAME        : VPCS[1]
IP/MASK     : 10.1.1.2/24
GATEWAY     : 0.0.0.0
DNS         : 
MAC         : 00:50:79:66:68:01
LPORT       : 20004
RHOST:PORT  : 127.0.0.1:20005
MTU         : 1500

```
🌞 Effectuer un ping d'une machine à l'autre
```
VPCS> ping 10.1.1.2

10.1.1.2 icmp_seq=1 ttl=64 time=0.001 ms
10.1.1.2 icmp_seq=2 ttl=64 time=0.001 ms
10.1.1.2 icmp_seq=3 ttl=64 time=0.001 ms
10.1.1.2 icmp_seq=4 ttl=64 time=0.001 ms
10.1.1.2 icmp_seq=5 ttl=64 time=0.001 ms
```
🌞 Protocolz ?

    déterminer quel protocole est utilisé pour envoyer des "pings"
    écrivez votre réponse dans le compte-rendu

```
protocole : ARP
```
🌞 Déterminer l'adresse MAC de vos trois machines

    une commande dans le terminal de chaque machine

🌞 Définir une IP statique sur les trois machines

    prouver que votre changement d'IP est effectif sur chaque machine


machine 1 et 2 voir plus haut 
machine 3 :
```
VPCS> show ip

NAME        : VPCS[1]
IP/MASK     : 10.1.1.3/24
GATEWAY     : 0.0.0.0
DNS         : 
MAC         : 00:50:79:66:68:02
LPORT       : 20010
RHOST:PORT  : 127.0.0.1:20011
MTU         : 1500

```
🌞 Effectuer des ping d'une machine à l'autre

node 1 a 2 :
```
VPCS> ping 10.1.1.2

84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=1.733 ms
84 bytes from 10.1.1.2 icmp_seq=2 ttl=64 time=0.747 ms
84 bytes from 10.1.1.2 icmp_seq=3 ttl=64 time=0.872 ms
84 bytes from 10.1.1.2 icmp_seq=4 ttl=64 time=0.702 ms
84 bytes from 10.1.1.2 icmp_seq=5 ttl=64 time=0.676 ms
```
node 2 a 3 :
```
VPCS> ping 10.1.1.3

84 bytes from 10.1.1.3 icmp_seq=1 ttl=64 time=0.662 ms
84 bytes from 10.1.1.3 icmp_seq=2 ttl=64 time=0.673 ms
84 bytes from 10.1.1.3 icmp_seq=3 ttl=64 time=0.532 ms
84 bytes from 10.1.1.3 icmp_seq=4 ttl=64 time=0.674 ms
84 bytes from 10.1.1.3 icmp_seq=5 ttl=64 time=0.737 ms

```
ping 1 a 3 :
```
VPCS> ping 10.1.1.3

84 bytes from 10.1.1.3 icmp_seq=1 ttl=64 time=0.399 ms
84 bytes from 10.1.1.3 icmp_seq=2 ttl=64 time=0.562 ms
84 bytes from 10.1.1.3 icmp_seq=3 ttl=64 time=0.537 ms
84 bytes from 10.1.1.3 icmp_seq=4 ttl=64 time=0.670 ms
84 bytes from 10.1.1.3 icmp_seq=5 ttl=64 time=0.599 ms


```
🌞 Afficher la table ARP de node1

    on doit y voir les adresses MAC de node2 et node3, associées à leurs adresses IP respectives
```
VPCS> arp

00:50:79:66:68:01  10.1.1.2 expires in 93 seconds 
00:50:79:66:68:02  10.1.1.3 expires in 115 seconds 

```
🌞 installez un serveur DHCP dnsmasq ou kea
```
[crackito@localhost ~]$ sudo dnf install dnsmasq
[sudo] password for crackito: 
Last metadata expiration check: 1:07:45 ago on Tue 06 Jan 2026 11:58:38 AM CET.
Package dnsmasq-2.90-4.el10.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!

```

```
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:16:29:1a brd ff:ff:ff:ff:ff:ff
    altname enx08002716291a
    inet 10.1.1.253/24 brd 10.1.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::cd6c:8817:3a3d:d194/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
conf dhcp
```
// create new
{
"Dhcp4": {
    "interfaces-config": {
        // specify network interfaces to listen on
        "interfaces": [ "enp0s9" ]
    },
    // settings for expired-leases (follows are default)
    "expired-leases-processing": {
        "reclaim-timer-wait-time": 10,
        "flush-reclaimed-timer-wait-time": 25,
        "hold-reclaimed-time": 3600,
        "max-reclaim-leases": 100,
        "max-reclaim-time": 250,
        "unwarned-reclaim-cycles": 5
    },
    // T1 timer that govern when the client begins the renewal processes (sec)
    "renew-timer": 900,
    // T2 timer that govern when the client begins the rebind processes (sec)
    "rebind-timer": 1800,
    // how long the addresses (leases) given out by the server are valid (sec)
    "valid-lifetime": 3600,

    "subnet4": [
        {
            "id": 1,
            // specify subnet that DHCP is used
            "subnet": "10.1.1.0/24",
            // specify the range of IP addresses to be leased
            "pools": [ { "pool": "10.1.1.10 - 10.1.1.50" } ],
            "option-data": [

            ]
	}
    ],

```

```
[crackito@localhost ~]$ sudo chown root:kea /etc/kea/kea-dhcp4.conf 
[crackito@localhost ~]$ sudo chmod 640 /etc/kea/kea-dhcp4.conf 
[crackito@localhost ~]$ sudo systemctl enable --now kea-dhcp4 
```
votre serveur DHCP doit attribuer des IP entre 10.1.1.10 et 10.1.1.50
🌞 Récupérer une IP automatiquement depuis les 3 nodes

VM 1 :
```
VPCS> ip dhcp 
DORA
VPCS> show ip IP 10.1.1.12/24

NAME        : VPCS[1]
IP/MASK     : 10.1.1.12/24
GATEWAY     : 0.0.0.0
DNS         : 
DHCP SERVER : 10.1.1.253
DHCP LEASE  : 3597, 3600/900/1800
MAC         : 00:50:79:66:68:00
LPORT       : 20009
RHOST:PORT  : 127.0.0.1:20010
MTU         : 1500

```
VM 2:
```
VPCS> ip dhcp
DORA
VPCS> show ip IP 10.1.1.10/24

NAME        : VPCS[1]
IP/MASK     : 10.1.1.10/24
GATEWAY     : 0.0.0.0
DNS         : 
DHCP SERVER : 10.1.1.253
DHCP LEASE  : 3593, 3600/900/1800
MAC         : 00:50:79:66:68:01
LPORT       : 20011
RHOST:PORT  : 127.0.0.1:20012
MTU         : 1500

```
VM 3:
```
VPCS> ip dhcp
DORA
VPCS> show ip IP 10.1.1.11/24

NAME        : VPCS[1]
IP/MASK     : 10.1.1.11/24
GATEWAY     : 0.0.0.0
DNS         : 
DHCP SERVER : 10.1.1.253
DHCP LEASE  : 3596, 3600/900/1800
MAC         : 00:50:79:66:68:02
LPORT       : 20007
RHOST:PORT  : 127.0.0.1:20008
MTU         : 1500

```
ip VM avec rogue dhcp :
VM 1:
```
VPCS> ip dhcp
DORA
VPCS> show ip IP 10.1.1.210/24

NAME        : VPCS[1]
IP/MASK     : 10.1.1.210/24
GATEWAY     : 0.0.0.0
DNS         : 
DHCP SERVER : 10.1.1.254
DHCP LEASE  : 3591, 3600/900/1800
MAC         : 00:50:79:66:68:00
LPORT       : 20010
RHOST:PORT  : 127.0.0.1:20011
MTU         : 1500

```
VM 2:
```
VPCS> ip dhcp
DORA
VPCS> show ip IP 10.1.1.211/24

NAME        : VPCS[1]
IP/MASK     : 10.1.1.211/24
GATEWAY     : 0.0.0.0
DNS         : 
DHCP SERVER : 10.1.1.254
DHCP LEASE  : 3590, 3600/900/1800
MAC         : 00:50:79:66:68:01
LPORT       : 20012
RHOST:PORT  : 127.0.0.1:20013
MTU         : 1500


```
VM 3 :
```
VPCS> ip dhcp
DORA
VPCS> show ip IP 10.1.1.212/24

NAME        : VPCS[1]
IP/MASK     : 10.1.1.212/24
GATEWAY     : 0.0.0.0
DNS         : 
DHCP SERVER : 10.1.1.254
DHCP LEASE  : 3589, 3600/900/1800
MAC         : 00:50:79:66:68:02
LPORT       : 20008
RHOST:PORT  : 127.0.0.1:20009
MTU         : 1500


```
Depuis votre machine attaquante, effectuez un ARP poisoning
🌞 Proof !

montrer l'attaque qui est menée
la preuve que vous pouvez écrire ce que vous voulez dans la table ARP d'un client donné en affichant la table ARP de la victime avant et après empoisonnement

```
VPCS> show arp

arp table is empty

VPCS> show arp

08:00:27:d2:ee:97  10.1.1.254 expires in 109 seconds 
08:00:27:d2:ee:97  10.1.1.200 expires in 113 seconds 


```

```

```



