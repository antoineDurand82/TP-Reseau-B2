# TP1 : Back to basics

# Sommaire

* [Intro](#intro)
* [0. Etapes préliminaires](#0-etapes-préliminaires)
* [I. Gather informations](#i-gather-informations)
* [II. Edit configurtion](#ii-edit-configuration)
  * [1. Configuration cartes réseau](#1-configuration-cartes-réseau)
  * [2. Serveur SSH](#2-serveur-ssh)
* [III. Routage simple](#iii-routage-simple)
* [IV. Autres applications et métrologie](#iv-autres-applications-et-métrologie)
  * [1. Commandes](#1-commandes)
  * [2. Cockpit](#2-cockpit)
  * [3. Netdata](#3-netdata)

# Intro

First TP smooth, remise dans le bain tranquillement (pour ceux qui l'ont quitté). Pour ce premier TP, on va rester du côté du réseau client et de l'administration simplifiée de services utilisant le réseau.

Au menu :  
* installation/configuration d'une VM CentOS8 (si c'est pas déjà fait :angry:)
* exploration de la pile TCP/IP d'une machine Linux
* configuration de services réseau
* configuration firewall
* analyse de trames
* métrologie

Notions abordées : 
* IP
* ARP
* Ethernet
* Ports (TCP/UDP)
* Firewalling (filtrage de paquets)
* DNS
* DHCP
* SSH
* Service réseau

**Référez-vous [au README des TPs](../README.md) pour des infos sur le déroulement et le rendu des TPs.**

# 0. Etapes préliminaires

* assurez-vous d'avoir au moins une carte réseau qui permet de joindre Internet
* assurez-vous d'avoir au moins une AUTRE carte réseau dans un réseau UNIQUEMENT privé que vous pouvez joindre localement
* effectuez une connexion SSH à la machine
* assurez-vous d'avoir les droits `sudo`

# I. Gather informations

**Première étape : récupération d'infos sur le système.**  
Vous arrivez sur une nouvelle machine, et vous avez besoin de connaître sa configuration réseau. Ce sera essentiel en cas de soucis. 

*Des trucs vous paraissent un peu cons ? C'est pour manipuler et se remettre dans le bain, n'hésitez pas à me faire rendu concis pour les parties qui vous paraissent izi.*

Commandes à utiliser (ou pas) : `ip`, `ss`, `nmcli`, `cat`, `dig`, `firewall-cmd`

* 🌞 récupérer une **liste des cartes réseau** avec leur nom, leur IP et leur adresse MAC

```
[root@localhost ~]# ip a
1: lo:                                                      Nom
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00   Mac
    inet 127.0.0.1/8 scope host lo                          IP
2: enp0s3:                                                  Nom
    link/ether 08:00:27:e7:9f:00 brd ff:ff:ff:ff:ff:ff      Mac
    inet 10.0.2.15/24 brd 10.0.2.255  IP
3: enp0s8:                                                  Nom
    link/ether 08:00:27:90:ea:90 brd ff:ff:ff:ff:ff:ff      MAC
    inet 192.168.56.101/24 brd 192.168.56.255               IP 
```


* 🌞 déterminer si les cartes réseaux ont récupéré une **IP en DHCP** ou non
  ```powershell
  [root@localhost network-scripts]# sudo nmcli -f DHCP4 con show enp0s3
    DHCP4.OPTION[1]:                        domain_name = auvence.co
    DHCP4.OPTION[2]:                        domain_name_servers = 10.33.10.20 10.33.10.2 8.8.8.8 8.8.4.4
    DHCP4.OPTION[3]:                        expiry = 1569590969
    DHCP4.OPTION[4]:                        ip_address = 10.0.2.15
    DHCP4.OPTION[22]:                       routers = 10.0.2.2
    DHCP4.OPTION[23]:                       subnet_mask = 255.255.255.0
    [root@localhost network-scripts]# sudo nmcli -f DHCP4 con show enp0s8
    DHCP4.OPTION[1]:                        expiry = 1569505753
    DHCP4.OPTION[2]:                        ip_address = 192.168.56.101
    DHCP4.OPTION[20]:                       subnet_mask = 255.255.255.0
  ```
    Ils ont des temps d'expiration donc ils utilisent un DHCP.

    
* 🌞 afficher la **table de routage** de la machine et sa **table ARP**

```powershell
[root@localhost NetworkManager]# arp -a
? (192.168.56.100) at 08:00:27:1c:5d:48 [ether] on enp0s8
_gateway (10.0.2.2) at 52:54:00:12:35:02 [ether] on enp0s3
? (192.168.56.103) at 0a:00:27:00:00:06 [ether] on enp0s8
[root@localhost NetworkManager]# ip route
default via 10.0.2.2 dev enp0s3 proto dhcp metric 102
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 102
192.168.56.0/24 dev enp0s8 proto kernel scope link src 192.168.56.101 metric 101
```
  * cette route est vers le réseau nat 10.0.2.0 , elle est utilisée pour une connexion externe, la passerelle de cette route est à l'IP 10.0.2.2 et cette IP est portée par enp0s3

  * cette route est vers le réseau privé hote 192.168.56.0 , elle est utilisée pour une connexion locale, la passerelle de cette route est à l'IP 192.168.56.100 et cette IP est portée par enp0s8


* 🌞 récupérer **la liste des ports en écoute** (*listening*) sur la machine (TCP et UDP)
  * ```powershell
    [root@localhost NetworkManager]# ss -lput
        Netid          State            Recv-Q           Send-Q                                Local Address:Port                         Peer Address:Port
        udp            UNCONN           0                0                             192.168.56.101%enp0s8:bootpc                            0.0.0.0:*               users:(("NetworkManager",pid=806,fd=22))
        udp            UNCONN           0                0                                  10.0.2.15%enp0s3:bootpc                            0.0.0.0:*               users:(("NetworkManager",pid=806,fd=19))
        udp            UNCONN           0                0                                         127.0.0.1:323                               0.0.0.0:*               users:(("chronyd",pid=770,fd=6))
        udp            UNCONN           0                0                                             [::1]:323                                  [::]:*               users:(("chronyd",pid=770,fd=7))
        tcp            LISTEN           0                128                                         0.0.0.0:ssh                               0.0.0.0:*               users:(("sshd",pid=824,fd=6))
        tcp            LISTEN           0                128                                            [::]:ssh                                  [::]:*               users:(("sshd",pid=824,fd=8))
    ```
* 🌞 récupérer **la liste des DNS utilisés par la machine**

  * ```
    [root@localhost etc]# cat resolv.conf
    # Generated by NetworkManager
    search auvence.co
    nameserver 10.33.10.20
    nameserver 10.33.10.2
    nameserver 8.8.8.8
    ```
  * ```
    [root@localhost etc]# dig www.reddit.com

    ; <<>> DiG 9.11.4-P2-RedHat-9.11.4-17.P2.el8_0.1 <<>> www.reddit.com
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 57724
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 5, AUTHORITY: 13, ADDITIONAL: 27
    ```

* 🌞 afficher **l'état actuel du firewall**
  * ```
    [root@localhost etc]# firewall-cmd --list-all
        public (active)
        target: default
        icmp-block-inversion: no
        interfaces: enp0s3 enp0s8
        sources:
        services: cockpit dhcpv6-client ssh
        ports:
        protocols:
        masquerade: no
        forward-ports:
        source-ports:
        icmp-blocks:
        rich rules:
    ```
Les interfaces filtrées sont enp0s3 et la s8 et dans la mesure où nous avons fais un petit set enforce 0 nous n'avons plus aucun TCP ou UDP de filtrées.

## II. Edit configuration

**Deuxièmement : Modifier la configuration existante**

Commandes à utiliser (ou pas) : `vim`, `cat`, `nmcli`, `systemctl`, `firewall-cmd`

---

### 1. Configuration cartes réseau

**NB** : sur CentOS8, la gestion des cartes réseau a légèrement changé. Il existe un démon qui gère désormais tout ce qui est relatif au réseau : NetworkManager.  

Marche à suivre pour modifier la configuration d'une carte réseau :
* édition du fichier de configuration
  * `sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s8`
* refresh de NetworkManager ("Hey prend mes modifications en compte stp !")
  * `sudo nmcli connection reload` 
  * `sudo nmcli con reload` même chose, on peut abréger les commandes `nmcli`
  * `sudo nmcli c reload` même chose aussi
* restart de l'interface
  * `sudo ifdown enp0s8` puis `sudo ifup enp0s8`
  * **OU** `sudo nmcli con up enp0s8`

> Pour les hipsters, y'a moyen de ne plus passer du tout par les fichiers dans `/etc/sysconfig` et tout gérer directement avec NetworkManager, cf [la doc officielle](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/configuring_and_managing_networking/index#Selecting-Network-Configuration-methods_overview-of-Network-configuration-methods). 

---

* 🌞 modifier la configuration de la carte réseau privée
  * ```
    [root@localhost network-scripts]# cat ifcfg-enp0s8
    BOOTPROTO=static
    DEVICE=enp0s8
    NAME=enp0s8
    ONBOOT=yes
    IPADDR=192.168.56.105
    ```
* ajouter une nouvelle carte réseau dans un DEUXIEME réseau privé UNIQUEMENT privé
  * ```
        [root@localhost network-scripts]# ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
        valid_lft forever preferred_lft forever
    2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether 08:00:27:e7:9f:00 brd ff:ff:ff:ff:ff:ff
        inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
        valid_lft 86098sec preferred_lft 86098sec
        inet6 fe80::4ad6:a897:e477:132f/64 scope link noprefixroute
        valid_lft forever preferred_lft forever
    3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether 08:00:27:90:ea:90 brd ff:ff:ff:ff:ff:ff
        inet 192.168.56.105/24 brd 192.168.56.255 scope global noprefixroute enp0s8
        valid_lft forever preferred_lft forever
        inet6 fe80::a00:27ff:fe90:ea90/64 scope link
        valid_lft forever preferred_lft forever
    4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether 08:00:27:9d:c4:27 brd ff:ff:ff:ff:ff:ff
        inet 192.168.252.3/24 brd 192.168.252.255 scope global noprefixroute enp0s9
        valid_lft forever preferred_lft forever
        inet6 fe80::a00:27ff:fe9d:c427/64 scope link tentative
        valid_lft forever preferred_lft forever
    ```
* vérifier vos changements
  * ```
    [root@localhost network-scripts]# route
        Kernel IP routing table
        Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
        default         _gateway        0.0.0.0         UG    100    0        0 enp0s3
        10.0.2.0        0.0.0.0         255.255.255.0   U     100    0        0 enp0s3
        192.168.56.0    0.0.0.0         255.255.255.0   U     101    0        0 enp0s8
        192.168.252.0   0.0.0.0         255.255.255.0   U     102    0        0 enp0s9
        [root@localhost network-scripts]# arp
        Address                  HWtype  HWaddress           Flags Mask            Iface
        _gateway                 ether   52:54:00:12:35:02   C                     enp0s3
        192.168.56.103           ether   0a:00:27:00:00:06   C                     enp0s8
    ```
* 🐙 mettre en place un NIC *teaming* (ou *bonding*)
  * il vous faut deux cartes dans le même réseau puisque vous allez les agréger (vous pouvez en créer de nouvelles)
  * le *teaming* ou *bonding* consiste à agréger deux cartes réseau pour augmenter les performances/la bande passante
  * je vous laisse free sur la configuration (active/passive, loadbalancing, round-robin, autres)
  * prouver que le NIC *teaming* est en place

---

### 2. Serveur SSH

* 🌞 modifier la configuration du système pour que le serveur SSH tourne sur le port 2222
  * adapter la configuration du firewall (fermer l'ancien port, ouvrir le nouveau)
* pour l'étape suivante, il faudra un hôte qui ne s'est jamais connecté à la VM afin d'observer les échanges ARP (vous pouvez aussi juste vider la table ARP du client). Je vous conseille de faire une deuxième VM dans le même réseau, mais vous pouvez utiliser votre PC hôte.
* 🌞 analyser les trames de connexion au serveur SSH
  * intercepter avec Wireshark et/ou `tcpdump` le trafic entre le client SSH et le serveur SSH
  * détailler l'établissement de la connexion
    * doivent figurer au moins : échanges ARP, 3-way handshake TCP
    * 🐙 configurer une connexion par échange de clés, analyser les échanges réseau réalisés par le protocole SSH au moment de la connexion
  * une fois la connexion établie, choisir une trame du trafic SSH et détailler son contenu

# III. Routage simple

Dans cette partie, vous allez remettre en place un routage statique simple. Vous êtes libres du choix de la techno (CentOS8, Cisco, autres. Vous pouvez utiliser GNS3). 

Vous devez reproduire la mini-archi suivante : 
```
                   +-------+
                   |Outside|
                   | world |
                   +---+---+
                       |
                       |
+-------+         +----+---+         +-------+
|       |   net1  |        |   net2  |       |
|  VM1  +---------+ Router +---------+  VM2  |
|       |         |        |         |       |
+-------+         +--------+         +-------+
```

* **Description**
  * Le routeur a trois interfaces, dont une qui permet de joindre l'extérieur (internet)
  * La `VM1` a une interface dans le réseau `net1`
  * La `VM2` a une interface dans le réseau `net2`
  * Les deux VMs peuvent joindre Internet en passant par le `Router`
* 🌞 **To Do** 
  * Tableau récapitulatif des IPs
  * Configuration (bref) de VM1 et VM2
  * Configuration routeur
  * Preuve que VM1 passe par le routeur pour joindre internet
  * Une (ou deux ? ;) ) capture(s) réseau ainsi que des explications qui mettent en évidence le routage effectué par le routeur

# IV. Autres applications et métrologie

Dans cette partie, on va jouer un peu avec de nouvelles commandes qui peuvent être utiles pour diagnostiquer un peu ce qu'il se passe niveau réseau.

---

## 1. Commandes

* jouer avec `iftop`
  * expliquer son utilisation et imaginer un cas où `iftop` peut être utile

---

## 2. Cockpit

* 🌞 mettre en place cockpit sur la VM1
  * c'est quoi ? C'est un service web. Pour quoi faire ? Vous allez vite comprendre en le voyant.
  * `sudo dnf install -y cockpit`
  * `sudo systemctl start cockpit`
  * trouver (à l'aide d'une commande shell) sur quel port (TCP ou UDP) écoute Cockpit 
  * vérifier que le port est ouvert dans le firewall
* 🌞 explorer Cockpit, plus spécifiquement ce qui est en rapport avec le réseau

---

## 3. Netdata

Netdata est un outil utilisé pour récolter des métriques et envoyer des alertes. Il peut aussi être utilisé afin de visionner ces métriques, à court terme. Nous allons ici l'utiliser pour observer les métriques réseau et mettre en place un service web supplémentaire.

* 🌞 mettre en place Netdata sur la VM1 et la VM2
  * se référer à la documentation officielle
  * repérer et ouvrir le port dédié à l'interface web de Netdata
* 🌞 explorer les métriques liées au réseau que récolte Netdata