# TP2 : Ethernet, IP, et ARP


# Sommaire
- [TP2 : Ethernet, IP, et ARP](#tp2--ethernet-ip-et-arp)
- [Sommaire](#sommaire)
- [I. Setup IP](#i-setup-ip)
- [II. ARP my bro](#ii-arp-my-bro)
- [III. DHCP you too my brooo](#iii-dhcp-you-too-my-brooo)
- [IV. Avant-goût TCP et UDP](#iv-avant-goût-tcp-et-udp)

# I. Setup IP

🌞 **Mettez en place une configuration réseau fonctionnelle entre les deux machines**

PC 1
```
lgran@Zenbook MINGW64 ~/Desktop/Cours
$ netsh interface ipv4 set address name="VirtualBox Host-Only Network" static 10.33.16.1 255.255.255.192 10.33.16.63
lgran@Zenbook MINGW64 ~/Desktop/Cours
$ ipconfig

Configuration IP de Windows

Carte Ethernet VirtualBox Host-Only Network :

   Suffixe DNS propre à la connexion. . . :
   Adresse IPv6 de liaison locale. . . . .: fe80::da0:41cd:46cf:1eff%11
   Adresse IPv4. . . . . . . . . . . . . .: 10.33.16.1
   Masque de sous-réseau. . . . . . . . . : 255.255.255.192
   Passerelle par défaut. . . . . . . . . : 10.33.16.63
```
PC 2
```
[leo@reseau ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
NAME=enp0s8
DEVICE=enp0s8
BOOTPROTO=static
ONBOOT=yes
IPADDR=10.33.16.2
NETMASK=255.255.255.192
[leo@reseau ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:ed:ca:c4 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 85293sec preferred_lft 85293sec
    inet6 fe80::a00:27ff:feed:cac4/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:23:be:d1 brd ff:ff:ff:ff:ff:ff
    inet 10.33.16.2/26 brd 10.33.16.63 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe23:bed1/64 scope link
       valid_lft forever preferred_lft forever
```
Adresse réseau : 10.33.16.0/26
Adresse broadcast : 10.33.16.63/26
Adresse PC 1 : 10.33.16.1/26
Adresse PC 2 : 10.33.16.2/26


🌞 **Prouvez que la connexion est fonctionnelle entre les deux machines**

```
[leo@reseau ~]$ ping -c 1 10.33.16.1
PING 10.33.16.1 (10.33.16.1) 56(84) bytes of data.
64 bytes from 10.33.16.1: icmp_seq=1 ttl=128 time=0.449 ms

--- 10.33.16.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.449/0.449/0.449/0.000 ms
```

🌞 **Wireshark it**

- **déterminez, grâce à Wireshark, quel type de paquet ICMP est envoyé par `ping`**
  - Le ping est un `0 echo (ping) request`
  - Le pong est un `8 echo (ping) reply`

🦈 **[PCAP qui contient les paquets ICMP qui vous ont permis d'identifier les types ICMP](./pictures/ping.pcapng)**

# II. ARP my bro

🌞 **Check the ARP table**

- utilisez une commande pour afficher votre table ARP
  ```
  lgran@Zenbook MINGW64 ~/Desktop/Cours
  $ arp -a
  [...]
  Interface : 10.33.16.1 --- 0xb
    Adresse Internet      Adresse physique      Type      
    10.33.16.2            08-00-27-23-be-d1     dynamique 
    10.33.16.63           ff-ff-ff-ff-ff-ff     statique
    224.0.0.2             01-00-5e-00-00-02     statique
    224.0.0.22            01-00-5e-00-00-16     statique
    224.0.0.251           01-00-5e-00-00-fb     statique
    224.0.0.252           01-00-5e-00-00-fc     statique
    239.255.255.250       01-00-5e-7f-ff-fa     statique
  ```
- déterminez la MAC de votre binome depuis votre table ARP
  
  L'adresse Mac de PC 2 est 08-00-27-23-be-d1
- déterminez la MAC de la *gateway* de votre réseau 
  - celle de votre réseau physique, WiFi, genre YNOV, car il n'y en a pas dans votre ptit LAN
  - c'est juste pour vous faire manipuler un peu encore :)

🌞 **Manipuler la table ARP**

- utilisez une commande pour vider votre table ARP et prouvez que ça fonctionne en l'affichant et en constatant les changements
  ```
  lgran@Zenbook MINGW64 ~/Desktop/Cours
  $ arp -d

  lgran@Zenbook MINGW64 ~/Desktop/Cours
  $ arp -a

  Interface : 192.168.1.88 --- 0x4
    Adresse Internet      Adresse physique      Type
    224.0.0.2             01-00-5e-00-00-02     statique  
    224.0.0.22            01-00-5e-00-00-16     statique
    239.255.255.250       01-00-5e-7f-ff-fa     statique

  Interface : 10.33.16.1 --- 0xb
    Adresse Internet      Adresse physique      Type
    10.33.16.63           ff-ff-ff-ff-ff-ff     statique
    224.0.0.22            01-00-5e-00-00-16     statique
  ```
- ré-effectuez des pings, et constatez la ré-apparition des données dans la table ARP
  ```
  lgran@Zenbook MINGW64 ~/Desktop/Cours
  $ ping -c 1 10.33.16.2

  Envoi d’une requête 'Ping'  10.33.16.2 avec 32 octets de données :
  Réponse de 10.33.16.2 : octets=32 temps<1ms TTL=64

  Statistiques Ping pour 10.33.16.2:
      Paquets : envoyés = 1, reçus = 1, perdus = 0 (perte 0%),
  Durée approximative des boucles en millisecondes :
      Minimum = 0ms, Maximum = 0ms, Moyenne = 0ms

  lgran@Zenbook MINGW64 ~/Desktop/Cours
  $ arp -a

  Interface : 192.168.1.88 --- 0x4
    Adresse Internet      Adresse physique      Type
    192.168.1.254         f4-05-95-07-c7-2c     dynamique 
    224.0.0.2             01-00-5e-00-00-02     statique
    224.0.0.22            01-00-5e-00-00-16     statique
    224.0.0.251           01-00-5e-00-00-fb     statique
    224.0.0.252           01-00-5e-00-00-fc     statique
    239.255.255.250       01-00-5e-7f-ff-fa     statique

  Interface : 10.33.16.1 --- 0xb
    Adresse Internet      Adresse physique      Type
    10.33.16.2            08-00-27-23-be-d1     dynamique
    10.33.16.63           ff-ff-ff-ff-ff-ff     statique
    224.0.0.22            01-00-5e-00-00-16     statique
    239.255.255.250       01-00-5e-7f-ff-fa     statique
  ```

🌞 **Wireshark it**

  - déterminez, pour les deux trames, les adresses source et destination
    - Trame 1 :
      - Source : 0a:00:27:00:00:0b (PC1)
      - Destination : ff:ff:ff:ff:ff:ff (BroadCast)
    - Trame 2 :
      - Source : 08:00:27:23:be:d1 (PC2)
      - Destination : 0a:00:27:00:00:0b (PC1)

🦈 **[PCAP qui contient les trames ARP](./pictures/ARP.pcapng)**

# III. DHCP you too my brooo

🌞 **Wireshark it**

- identifiez les 4 trames DHCP lors d'un échange DHCP et mettez en évidence les adresses source et destination de chaque trame
  - Trame 1 : 
    - Source : 0.0.0.0
    - Destination : 255.255.255.255
  - Trame 2 :
    - Source : 192.168.1.254
    - Destination : 192.168.1.88
  - Trame 3 :
    - Source : 0.0.0.0
    - Destination : 255.255.255.255
  - Trame 4 :
    - Source : 192.168.1.254
    - Destination : 192.168.1.88
- identifiez dans ces 4 trames les informations **1**, **2** et **3** dont on a parlé juste au dessus
  - 1 : 192.168.1.88
  - 2 : 192.168.1.254
  - 3 :

🦈 **[PCAP qui contient l'échange DORA](./pictures/DHCP.pcapng)**


# IV. Avant-goût TCP et UDP

TCP et UDP ce sont les deux protocoles qui utilisent des ports. Si on veut accéder à un service, sur un serveur, comme un site web :

- il faut pouvoir joindre en terme d'IP le correspondant
  - on teste que ça fonctionne avec un `ping` généralement
- il faut que le serveur fasse tourner un programme qu'on appelle "service" ou "serveur"
  - le service "écoute" sur un port TCP ou UDP : il attend la connexion d'un client
- le client **connaît par avance** le port TCP ou UDP sur lequel le service écoute
- en utilisant l'IP et le port, il peut se connecter au service en utilisant un moyen adapté :
  - un navigateur web pour un site web
  - un `ncat` pour se connecter à un autre `ncat`
  - et plein d'autres, **de façon générale on parle d'un client, et d'un serveur**

---

🌞 **Wireshark it**

- déterminez à quelle IP et quel port votre PC se connecte quand vous regardez une vidéo Youtube
  - IP Destination : 2a00:1450:4007:807::200e
  - Port : 443

🦈 **[PCAP qui contient un extrait de l'échange qui vous a permis d'identifier les infos](./pictures/TCP.pcapng)**