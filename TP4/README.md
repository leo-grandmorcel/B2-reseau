# TP4 : TCP, UDP et services réseau

# I. First steps

🌞 **Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP**

- UDP :
  - Chrome
  - Spotify
  - Discord
- TCP :
  - VPN
  - Youtube

🌞 **Demandez l'avis à votre OS**

```
C:\Windows\system32> netstat -fbn -p TCP 5
Connexions actives

  Proto  Adresse locale         Adresse distante       État
  TCP    10.33.18.219:56324     216.58.204.106:443     ESTABLISHED
 [chrome.exe]
[...]
  TCP    127.0.0.1:57192        127.0.0.1:57191        ESTABLISHED
 [hydra.exe]
  TCP    127.0.0.1:57193        127.0.0.1:57194        ESTABLISHED
 [hydra.exe]
  TCP    127.0.0.1:57194        127.0.0.1:57193        ESTABLISHED
 [hydra.exe]
TCP    10.33.18.219:56386     51.104.182.209:443     ESTABLISHED
 [Spotify.exe]
 TCP    10.33.18.219:65239     35.186.224.25:443      ESTABLISHED
 [Discord.exe]
```

🦈 **[`Chrome`](./pictures/TCP_chrome.pcapng)**
🦈 **[`Spotify`](./pictures/TCP_spotify.pcapng)**
🦈 **[`Discord`](./pictures/TCP_discord.pcapng)**
🦈 **[`VPN`](./pictures/UDP_vpn.pcapng)**
🦈 **[`Youtube`](./pictures/UDP_youtube.pcapng)**

# II. Mise en place

## 1. SSH

🌞 **Examinez le trafic dans Wireshark**

🦈 **[capture 3-way handshake, un peu de trafic au milieu et une fin de connexion](pictures/SSH.pcapng)**

🌞 **Demandez aux OS**

- repérez, avec un commande adaptée, la connexion SSH depuis votre machine
- ET repérez la connexion SSH depuis votre VM

```
C:\Windows\system32> netstat -fbn -p TCP 5

Connexions actives

  Proto  Adresse locale         Adresse distante       État
  TCP    10.3.1.1:52318         10.3.1.2:22            ESTABLISHED
 [ssh.exe]
```

```
[leo@reseau ~]$ sudo ss -ltpn
State      Recv-Q     Send-Q         Local Address:Port         Peer Address:Port    Process
LISTEN     0          128                  0.0.0.0:22                0.0.0.0:*        users:(("sshd",pid=699,fd=3))
LISTEN     0          128                     [::]:22                   [::]:*        users:(("sshd",pid=699,fd=4))
```

## 2. NFS

🌞 **Mettez en place un petit serveur NFS sur l'une des deux VMs**

```
[leo@Serveur ~]$ sudo dnf -y install nfs-utils
[...]
Complete!
[leo@Serveur ~]$ sudo nano /etc/idmapd.conf
[leo@Serveur ~]$ sudo nano /etc/exports
[leo@Serveur ~]$ cat /etc/exports
/srv/nfsshare 10.3.1.0/24(rw,no_root_squash)
[leo@Serveur ~]$ sudo systemctl enable --now rpcbind nfs-server
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
[leo@Serveur ~]$ sudo firewall-cmd --add-service=nfs
success
[leo@Serveur ~]$ sudo firewall-cmd --runtime-to-permanent
success
[leo@Serveur ~]$ sudo mkdir /srv/nfsshare
[leo@Serveur ~]$ sudo nano /srv/nfsshare/helloworld.txt
[leo@Serveur ~]$ cat /srv/nfsshare/helloworld.txt
helloworld!
```

```
[leo@Client ~]$ sudo dnf -y install nfs-utils
[...]
Complete!
[leo@Client ~]$ sudo nano /etc/idmapd.conf
[leo@Client ~]$ sudo mount -t nfs 10.3.1.3:/srv/nfsshare /mnt
[leo@Client ~]$ df -hT /mnt
Filesystem             Type  Size  Used Avail Use% Mounted on
10.3.1.3:/srv/nfsshare nfs4  6.2G  1.2G  5.1G  18% /mnt
[leo@Client ~]$ cat /mnt/helloworld.txt
helloworld!

```

🌞 **Wireshark it !**

```
[leo@Serveur ~]$ sudo tcpdump -i enp0s8 -c 10 -w nfs.pcapng not port 22
dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), snapshot length 262144 bytes
10 packets captured
25 packets received by filter
0 packets dropped by kernel
```

Le serveur écoute sur le port 2049.

🌞 **Demandez aux OS**

```
[leo@Serveur ~]$ sudo ss -ltpn | grep 2049
LISTEN 0      64           0.0.0.0:2049       0.0.0.0:*
LISTEN 0      64              [::]:2049          [::]:*
```

```
[leo@Client ~]$ sudo ss -tp
State                Recv-Q                Send-Q                                Local Address:Port                                 Peer Address:Port                 Process
ESTAB                0                     0                                          10.3.1.2:937                                      10.3.1.3:nfs
```

🦈 **[trafic NFS](pictures/nfs.pcapng)**

## 3. DNS

🌞 Utilisez une commande pour effectuer une requête DNS depuis une des VMs

- capturez le trafic avec un `tcpdump`
- déterminez le port et l'IP du serveur DNS auquel vous vous connectez
  - IP : 8.8.8.8
  - Port : 53

```
[leo@Serveur ~]$ sudo tcpdump -i enp0s3 -c 10 -w dns.pcapng not port 22 &
[1] 1557
[leo@Serveur ~]$ dig ynov.com

; <<>> DiG 9.16.23-RH <<>> ynov.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37165
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;ynov.com.                      IN      A

;; ANSWER SECTION:
ynov.com.               300     IN      A       172.67.74.226
ynov.com.               300     IN      A       104.26.11.233
ynov.com.               300     IN      A       104.26.10.233

;; Query time: 30 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Sat Oct 15 17:32:33 CEST 2022
;; MSG SIZE  rcvd: 85
```

🦈 **[DNS](pictures/dns.pcapng)**
