# Sujet Réseau et Infra

On va utiliser GNS3 dans ce TP pour se rapprocher d'un cas réel. On va focus sur l'aspect routing/switching, avec du matériel Cisco. On va aussi mettre en place des VLANs.

![best memes from cisco doc](./pics/the-best-memes-come-from-cisco-documentation.jpg)

# Sommaire

- [Sujet Réseau et Infra](#sujet-réseau-et-infra)
- [Sommaire](#sommaire)
- [I. Dumb switch](#i-dumb-switch)
  - [1. Adressage topologie 1](#1-adressage-topologie-1)
  - [2. Setup topologie 1](#2-setup-topologie-1)
- [II. VLAN](#ii-vlan)
  - [1. Adressage topologie 2](#1-adressage-topologie-2)
  - [2. Setup topologie 2](#2-setup-topologie-2)
- [III. Routing](#iii-routing)
  - [1. Adressage topologie 3](#1-adressage-topologie-3)
  - [2. Setup topologie 3](#2-setup-topologie-3)
- [IV. NAT](#iv-nat)
  - [1. Adressage topologie 4](#1-adressage-topologie-4)
  - [2. Setup topologie 4](#2-setup-topologie-4)
- [V. Add a building](#v-add-a-building)
  - [1. Adressage topologie 5](#1-adressage-topologie-5)
  - [2. Setup topologie 5](#2-setup-topologie-5)
- [6. Un unique serveur DHCP pour distribuer des IPs à tous les clients de tous les VLAN](#6-un-unique-serveur-dhcp-pour-distribuer-des-ips-à-tous-les-clients-de-tous-les-vlan)

# I. Dumb switch

## 1. Adressage topologie 1

| Node  | IP            |
| ----- | ------------- |
| `pc1` | `10.1.1.1/24` |
| `pc2` | `10.1.1.2/24` |

## 2. Setup topologie 1

🌞 **Commençons simple**

```
PC2> show ip all
NAME   IP/MASK              GATEWAY           MAC                DNS
PC2    10.1.1.2/24          255.255.255.0     00:50:79:66:68:01

PC2> ping  10.1.1.1
84 bytes from 10.1.1.1 icmp_seq=1 ttl=64 time=0.131 ms
```

# II. VLAN

## 1. Adressage topologie 2

| Node  | IP            | VLAN |
| ----- | ------------- | ---- |
| `pc1` | `10.1.1.1/24` | 10   |
| `pc2` | `10.1.1.2/24` | 10   |
| `pc3` | `10.1.1.3/24` | 20   |

## 2. Setup topologie 2

🌞 **Adressage**

- définissez les IPs statiques sur tous les VPCS
- vérifiez avec des `ping` que tout le monde se ping
  ```
  PC3> ping 10.1.1.1 -c 1
  84 bytes from 10.1.1.1 icmp_seq=1 ttl=64 time=0.311 ms
  PC3> ping 10.1.1.2 -c 1
  84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=0.640 ms
  ```

🌞 **Configuration des VLANs**

- déclaration des VLANs sur le switch `sw1`
- ajout des ports du switches dans le bon VLAN (voir [le tableau d'adressage de la topo 2 juste au dessus](#1-adressage-topologie-2))
  ```
  sw1#show vlan br
  10   clients                           active    Et0/0, Et0/1
  20   admins                          active    Et0/2
  ```

🌞 **Vérif**

- `pc1` et `pc2` doivent toujours pouvoir se ping
  ```
  PC1> ping 10.1.1.2 -c 1
  84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=0.157 ms
  ```
- `pc3` ne ping plus personne
  ```
  PC3> ping 10.1.1.2 -c 1
  host (10.1.1.2) not reachable
  PC3> ping 10.1.1.1 -c 1
  host (10.1.1.1) not reachable
  ```

# III. Routing

## 1. Adressage topologie 3

Les réseaux et leurs VLANs associés :

| Réseau    | Adresse        | VLAN associé |
| --------- | -------------- | ------------ |
| `clients` | `10.5.10.0/24` | 10           |
| `admins`  | `10.5.20.0/24` | 20           |
| `servers` | `10.5.30.0/24` | 30           |

L'adresse des machines au sein de ces réseaux :

| Node               | `clients`        | `admins`         | `servers`        |
| ------------------ | ---------------- | ---------------- | ---------------- |
| `pc1.clients.tp5`  | `10.5.10.1/24`   | x                | x                |
| `pc2.clients.tp5`  | `10.5.10.2/24`   | x                | x                |
| `adm1.admins.tp5`  | x                | `10.5.20.1/24`   | x                |
| `web1.servers.tp5` | x                | x                | `10.5.30.1/24`   |
| `r1`               | `10.5.10.254/24` | `10.5.20.254/24` | `10.5.30.254/24` |

## 2. Setup topologie 3

🌞 **Adressage**

- définissez les IPs statiques sur toutes les machines **sauf le _routeur_**
  ```
  pc1> show ip all
  NAME   IP/MASK              GATEWAY           MAC                DNS
  pc1    10.5.10.1/24         255.255.255.0     00:50:79:66:68:00
  ```
  ```
  pc2> show ip all
  NAME   IP/MASK              GATEWAY           MAC                DNS
  pc2    10.5.10.2/24         255.255.255.0     00:50:79:66:68:01
  ```
  ```
  adm1> show ip all
  NAME   IP/MASK              GATEWAY           MAC                DNS
  adm1   10.5.20.1/24         255.255.255.0     00:50:79:66:68:02
  ```
  ```
  [leo@web1 ~]$ ip a
  2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
      link/ether 08:00:27:a6:b1:98 brd ff:ff:ff:ff:ff:ff
      inet 10.5.30.1/24 brd 10.5.30.255 scope global noprefixroute enp0s3
        valid_lft forever preferred_lft forever
      inet6 fe80::a00:27ff:fea6:b198/64 scope link
        valid_lft forever preferred_lft forever
  ```

🌞 **Configuration des VLANs**

- déclaration des VLANs sur le switch `sw1`
- ajout des ports du switches dans le bon VLAN (voir [le tableau d'adressage de la topo 2 juste au dessus](#1-adressage-topologie-2))
- il faudra ajouter le port qui pointe vers le _routeur_ comme un _trunk_ : c'est un port entre deux équipements réseau (un _switch_ et un _routeur_)

  ```
  sw1#show vlan

  VLAN Name                             Status    Ports
  ---- -------------------------------- --------- -------------------------------
  1    default                          active    Et1/0, Et1/1, Et1/2, Et1/3
                                                  Et2/0, Et2/1, Et2/2, Et2/3
                                                  Et3/0, Et3/1, Et3/2
  10   clients                          active    Et0/0, Et0/1
  20   admins                           active    Et0/2
  30   servers                          active    Et0/3
  sw1#show interface trunk

  Port        Mode             Encapsulation  Status        Native vlan
  Et3/3       on               802.1q         trunking      1

  Port        Vlans allowed on trunk
  Et3/3       1-4094

  Port        Vlans allowed and active in management domain
  Et3/3       1,10,20,30

  Port        Vlans in spanning tree forwarding state and not pruned
  Et3/3       1,10,20,30
  ```

🌞 **Config du _routeur_**

- attribuez ses IPs au _routeur_
  - 3 sous-interfaces, chacune avec son IP et un VLAN associé
    ```
    R1#show ip int br
    Interface                  IP-Address      OK? Method Status                Protocol
    FastEthernet0/0            unassigned      YES unset  administratively down down
    FastEthernet0/0.10         10.5.10.254     YES manual administratively down down
    FastEthernet0/0.20         10.5.20.254     YES manual administratively down down
    FastEthernet0/0.30         10.5.30.254     YES manual administratively down down
    ```

🌞 **Vérif**

- tout le monde doit pouvoir ping le routeur sur l'IP qui est dans son réseau
  ```
  pc1> ping 10.5.10.254
  84 bytes from 10.5.10.254 icmp_seq=1 ttl=255 time=4.473 ms
  pc2> ping 10.5.10.254
  84 bytes from 10.5.10.254 icmp_seq=2 ttl=255 time=4.510 ms
  ```
  ```
  adm1> ping 10.5.20.254
  84 bytes from 10.5.20.254 icmp_seq=2 ttl=255 time=10.536 ms
  ```
  ```
  [leo@web1 ~]$ ping 10.5.30.254
  PING 10.5.30.254 (10.5.30.254) 56(84) bytes of data.
  64 bytes from 10.5.30.254: icmp_seq=1 ttl=255 time=9.02 ms
  ```
- en ajoutant une route vers les réseaux, ils peuvent se ping entre eux

  - ajoutez une route par défaut sur les VPCS
    ```
    pc1> ip 10.5.10.1 255.255.255.0 10.5.10.254
    pc2> ip 10.5.10.2 255.255.255.0 10.5.10.254
    adm1> 10.5.20.1 255.255.255.0 gateway 10.5.20.254
    ```
  - ajoutez une route par défaut sur la machine virtuelle
    ```
    [leo@web1 ~]$ ip route | grep default
    default via 10.5.30.254 dev enp0s3 proto static metric 102
    ```
  - testez des `ping` entre les réseaux

    ```
    pc1> ping 10.5.20.1
    84 bytes from 10.5.20.1 icmp_seq=1 ttl=63 time=23.065 ms
    pc1> ping 10.5.30.1
    84 bytes from 10.5.30.1 icmp_seq=1 ttl=63 time=19.975 ms

    adm1> ping 10.5.30.1
    84 bytes from 10.5.30.1 icmp_seq=1 ttl=63 time=20.062 ms
    adm1> ping 10.5.10.1
    84 bytes from 10.5.10.1 icmp_seq=1 ttl=63 time=14.471 ms
    ```

# IV. NAT

## 1. Adressage topologie 4

Les réseaux et leurs VLANs associés :

| Réseau    | Adresse        | VLAN associé |
| --------- | -------------- | ------------ |
| `clients` | `10.5.10.0/24` | 10           |
| `admins`  | `10.5.20.0/24` | 20           |
| `servers` | `10.5.30.0/24` | 30           |

L'adresse des machines au sein de ces réseaux :

| Node               | `clients`        | `admins`         | `servers`        |
| ------------------ | ---------------- | ---------------- | ---------------- |
| `pc1.clients.tp5`  | `10.5.10.1/24`   | x                | x                |
| `pc2.clients.tp5`  | `10.5.10.2/24`   | x                | x                |
| `adm1.admins.tp5`  | x                | `10.5.20.1/24`   | x                |
| `web1.servers.tp5` | x                | x                | `10.5.30.1/24`   |
| `r1`               | `10.5.10.254/24` | `10.5.20.254/24` | `10.5.30.254/24` |

## 2. Setup topologie 4

🌞 **Ajoutez le noeud Cloud à la topo**

- branchez à `eth1` côté Cloud
- côté routeur, il faudra récupérer un IP en DHCP
  ```
  R1#show ip int br
  Interface                  IP-Address      OK? Method Status                Protocol
  FastEthernet0/0            unassigned      YES unset  up                    up
  FastEthernet0/0.10         10.5.10.254     YES manual up                    up
  FastEthernet0/0.20         10.5.20.254     YES manual up                    up
  FastEthernet0/0.30         10.5.30.254     YES manual up                    up
  FastEthernet0/1            10.0.3.16       YES DHCP   up                    up
  ```
- vous devriez pouvoir `ping 1.1.1.1`

  ```
  R1#ping 1.1.1.1

  Type escape sequence to abort.
  Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
  .!!!!
  Success rate is 80 percent (4/5), round-trip min/avg/max = 20/62/160 ms
  ```

🌞 **Configurez le NAT**

- référez-vous au mémo NAT

  ```

  R1#conf t
  R1(config)#interface fastEthernet0/1
  R1(config-if)#ip address dhcp
  R1(config-if)#no shut
  R1(config-if)#ip nat outside
  R1(config-if)#exit
  R1(config)#interface fastEthernet0/0
  R1(config-if)#ip nat inside
  R1(config-if)#no shut
  R1(config-if)#exit
  R1(config)#access-list 1 permit any
  R1(config)#ip nat inside source list 1 interface fastEthernet0/1 overload
  R1(config)#exit
  ```

🌞 **Test**

- ajoutez une route par défaut (si c'est pas déjà fait)
  - sur les VPCS
  - sur la machine Linux
- configurez l'utilisation d'un DNS
  - sur les VPCS
    ```
    NAME   IP/MASK              GATEWAY           MAC                DNS
    pc1    10.5.10.1/24         10.5.10.254       00:50:79:66:68:00  8.8.8.8
    pc2    10.5.10.2/24         10.5.10.254       00:50:79:66:68:01  8.8.8.8
    adm1    10.5.20.1/24        10.5.20.254       00:50:79:66:68:02  8.8.8.8
    ```
- vérifiez un `ping` vers un nom de domaine

  ```
  pc1> ping ynov.com
  ynov.com resolved to 104.26.10.233
  84 bytes from 104.26.10.233 icmp_seq=1 ttl=54 time=31.627 ms

  pc2> ping google.com
  google.com resolved to 142.250.179.110
  84 bytes from 142.250.179.110 icmp_seq=1 ttl=112 time=29.360 ms

  adm1> ping github.com
  github.com resolved to 140.82.121.3
  84 bytes from 140.82.121.3 icmp_seq=1 ttl=52 time=40.301 ms
  ```

  ```
  [leo@web1 ~]$ ping -I enp0s3 ynov.com
  PING ynov.com (172.67.74.226) from 10.5.30.1 enp0s3: 56(84) bytes of data.
  64 bytes from 172.67.74.226 (172.67.74.226): icmp_seq=1 ttl=54 time=29.8 ms
  ```

# V. Add a building

## 1. Adressage topologie 5

Les réseaux et leurs VLANs associés :

| Réseau    | Adresse        | VLAN associé |
| --------- | -------------- | ------------ |
| `clients` | `10.5.10.0/24` | 10           |
| `admins`  | `10.5.20.0/24` | 20           |
| `servers` | `10.5.30.0/24` | 30           |

L'adresse des machines au sein de ces réseaux :

| Node                | `clients`        | `admins`         | `servers`        |
| ------------------- | ---------------- | ---------------- | ---------------- |
| `pc1.clients.tp5`   | `10.5.10.1/24`   | x                | x                |
| `pc2.clients.tp5`   | `10.5.10.2/24`   | x                | x                |
| `pc3.clients.tp5`   | DHCP             | x                | x                |
| `pc4.clients.tp5`   | DHCP             | x                | x                |
| `pc5.clients.tp5`   | DHCP             | x                | x                |
| `dhcp1.clients.tp5` | `10.5.10.253/24` | x                | x                |
| `adm1.admins.tp5`   | x                | `10.5.20.1/24`   | x                |
| `web1.servers.tp5`  | x                | x                | `10.5.30.1/24`   |
| `r1`                | `10.5.10.254/24` | `10.5.20.254/24` | `10.5.30.254/24` |

## 2. Setup topologie 5

🌞 **Vous devez me rendre le `show running-config` de tous les équipements**

- de tous les équipements réseau
  - le routeur
    ```
    R1#show running-config
    hostname R1
    interface FastEthernet0/0
    no ip address
    ip nat inside
    interface FastEthernet0/0.10
    encapsulation dot1Q 10
    ip address 10.5.10.254 255.255.255.0
    !
    interface FastEthernet0/0.20
    encapsulation dot1Q 20
    ip address 10.5.20.254 255.255.255.0
    !
    interface FastEthernet0/0.30
    encapsulation dot1Q 30
    ip address 10.5.30.254 255.255.255.0
    !
    interface FastEthernet0/1
    ip address dhcp
    ip nat outside
    !
    ip nat inside source list 1 interface FastEthernet0/1 overload
    !
    access-list 1 permit any
    ```
  - les 3 switches
    - sw1
      ```
      sw1#show running-config
      hostname sw1
      !
      interface Ethernet0/0
      switchport trunk allowed vlan 10,20,30
      switchport trunk encapsulation dot1q
      switchport mode trunk
      !
      interface Ethernet0/1
      switchport trunk allowed vlan 10,20,30
      switchport trunk encapsulation dot1q
      switchport mode trunk
      !
      interface Ethernet3/3
      switchport trunk allowed vlan 10,20,30
      switchport trunk encapsulation dot1q
      switchport mode trunk
      ```
    - sw2
      ```
      sw2#show running-config
      hostname sw2
      !
      interface Ethernet0/0
      switchport access vlan 10
      switchport mode access
      !
      interface Ethernet0/1
      switchport access vlan 10
      switchport mode access
      !
      interface Ethernet0/2
      switchport access vlan 20
      switchport mode access
      !
      interface Ethernet0/3
      switchport access vlan 30
      switchport mode access
      !
      interface Ethernet3/3
      switchport trunk encapsulation dot1q
      switchport mode trunk
      ```
    - sw3
      ```
      sw3#show running-config
      hostname sw3
      !
      interface Ethernet0/0
      switchport access vlan 10
      switchport mode access
      !
      interface Ethernet0/1
      switchport access vlan 10
      switchport mode access
      !
      interface Ethernet0/2
      switchport access vlan 10
      switchport mode access
      !
      interface Ethernet0/3
      switchport access vlan 10
      switchport mode access
      !
      interface Ethernet3/3
      switchport trunk encapsulation dot1q
      switchport mode trunk
      ```

🌞 **Mettre en place un serveur DHCP dans le nouveau bâtiment**

- il doit distribuer des IPs aux clients dans le réseau `clients` qui sont branchés au même switch que lui
- sans aucune action manuelle, les clients doivent...
  - avoir une IP dans le réseau `clients`
  - avoir un accès au réseau `servers`
  - avoir un accès WAN
  - avoir de la résolution DNS

```
[leo@dhcp1 ~]$ sudo cat /etc/dhcp/dhcpd.conf
default-lease-time 900;
max-lease-time 10800;
ddns-update-style none;
authoritative;
subnet 10.5.10.0 netmask 255.255.255.0 {
  range 10.5.10.3 10.5.10.252;
  option routers 10.5.10.254;
  option subnet-mask 255.255.255.0;
  option domain-name-servers 8.8.8.8;

}
```

🌞 **Vérification**

- un client récupère une IP en DHCP
  ```
  pc5> ip dhcp -r
  DORA IP 10.5.10.5/24 GW 10.5.10.254
  ```
- il peut ping le serveur Web
  ```
  pc5> ping 10.5.30.1 -c 3
  84 bytes from 10.5.30.1 icmp_seq=1 ttl=63 time=30.276 ms
  84 bytes from 10.5.30.1 icmp_seq=2 ttl=63 time=19.313 ms
  84 bytes from 10.5.30.1 icmp_seq=3 ttl=63 time=15.419 ms
  ```
- il peut ping `8.8.8.8`

```
 pc5> ping 8.8.8.8 -c 3
 84 bytes from 8.8.8.8 icmp_seq=1 ttl=113 time=32.500 ms
 84 bytes from 8.8.8.8 icmp_seq=2 ttl=113 time=22.881 ms
 84 bytes from 8.8.8.8 icmp_seq=3 ttl=113 time=29.086 ms
```

- il peut ping `google.com`
  ```
  pc5> ping google.com -c 3
  google.com resolved to 216.58.198.206
  84 bytes from 216.58.198.206 icmp_seq=1 ttl=114 time=28.290 ms
  84 bytes from 216.58.198.206 icmp_seq=2 ttl=114 time=30.909 ms
  84 bytes from 216.58.198.206 icmp_seq=3 ttl=114 time=23.949 ms
  ```

# 6. Un unique serveur DHCP pour distribuer des IPs à tous les clients de tous les VLAN

```
R1#conf t
R1(config)#interface fastEthernet0/0.10
R1(config-if)#ip helper-address 10.5.10.253
R1(config-if)#no shut
```

Désormais les clients du vlan 10 dans le réseau de gauche peuvent demander une IP

```
pc1> ip dhcp -r
DORA IP 10.5.10.6/24 GW 10.5.10.254
```

