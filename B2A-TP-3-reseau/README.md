# TP3 : Routage INTER-VLAN + mise en situation

# Sommaire

* [I. *Router-on-a-stick*](#i-router-on-a-stick)
* [II. Cas concret](#ii-cas-concret)

# I. *Router-on-a-stick*


Schéma moche ftw :

```
             +--+
             |R1|
             +-++
               |
               |                    +---+
               |          +---------+PC4|
+---+        +-+-+      +---+       +---+
|PC1+--------+SW1+------+SW2|
+---+        +-+-+      +-+--+
               |          |  |
               |          |  +------+--+
               |          |         |P1|
             +-+-+      +-+-+       +--+
             |PC2|      |PC3|
             +---+      +---+
```

![infra 1](infra1.png)

**Tableau des réseaux utilisés**

Réseau | Adresse | VLAN | Description
--- | --- | --- | ---
`net1` | `10.3.10.0/24` | 10 | Utilisateurs
`net2` | `10.3.20.0/24` | 20 | Admins
`net3` | `10.3.30.0/24` | 30 | Visiteurs
`netP` | `10.3.40.0/24` | 40 | Imprimantes

**Tableau d'adressage**

Machine | VLAN | IP `net1` | IP `net2` | IP `net3` |  IP `netP`
--- | --- | --- | --- | --- | ---
PC1 | 10 | `10.3.10.1/24` | x | x | x
PC2 | 20 | x | `10.3.20.2/24` | x | x | x
PC3 | 20 | x | `10.3.20.3/24` | x | x | x
PC4 | 30 | x | x |  `10.3.30.4/24` | x | x
P1 | 40 | x | x | x | `10.3.40.1/24` 
R1 | x |  `10.3.10.254/24` | `10.3.20.254/24` | `10.3.30.254/24` | `10.3.40.254/24` 

**Qui peut joindre qui ?**

✅ = peuvent se joindre
❌ = ne peuvent pas se joindre

Réseaux | `net1` |  `net2` |  `net3` |  `netP`
--- | --- | --- | --- | ---
 `net1` | ✅ | ❌ | ❌ | ❌
 `net2` | ❌ | ✅ | ✅ | ✅
 `net3` | ❌ | ✅ | ✅ | ✅
 `netP` | ❌ | ✅ | ✅ | ✅

**Instructions** (pretty straightforward) :
* Setup this shit
* You'll need inter-VLAN routing to make it work properly
  * se référer au [mémo Cisco section sous-interface](/memo/cli-cisco.md#sous-interface)
  * pour la partie "Qui peut joindre qui ?" vous n'avez besoin que de trunks avec des VLANs spécifiques autorisés
* 🌞 Prove me that your setup is actually working
  * PC | `PC1` |  `PC2` |  `PC3` |  `PC4` | `Imprimante`
  --- | --- | --- | --- | --- | ---
  `PC1` | x | `PC-1> ping 10.3.20.2`<br> `host (10.3.10.254) not reachable`| `PC-1> ping 10.3.20.3`<br> `host (10.3.10.254) not reachable` | `PC-1> ping 10.3.30.4`<br> `host (10.3.10.254) not reachable` | `PC-1> ping 10.3.40.1`<br> `host (10.3.10.254) not reachable`
  `PC2` | `PC-2> ping 10.3.10.1` <br> `10.3.10.1 icmp_seq=1 timeout` | x | `PC-2> ping 10.3.20.3` <br> `84 bytes from 10.3.20.3 icmp_seq=1 ttl=64 time=1.083 ms` | `PC-2> ping 10.3.30.4` <br> `84 bytes from 10.3.30.4 icmp_seq=1 ttl=63 time=10.940 ms` |`PC-2> ping 10.3.40.1` <br> `84 bytes from 10.3.40.1 icmp_seq=1 ttl=63 time=12.049 ms`
  `PC3` | ❌ | ✅ | ✅ | ✅ |✅
  `PC4` | ❌ | ✅ | ✅ | ✅ |✅
  `Imprimante` | ❌ | ✅ | ✅ | ✅ |✅


**CHECK MATE !**

# II. Cas concret

> C'est cool si vous jouez un peu le jeu et que vous imaginez quelque chose d'original. *Vous pouvez (comme aux autres TPs) pomper sur les autres mais ça a encore moins d'intérêt que d'habitude n_n !*

**Creusez-vous un peu la tête.**  

Le but est de mettre en place une infra qui répond au besoin des bureaux représentés ci-dessous :

![Yo](./pics/schema-II.png)

* `R1` `R3` `R4` et `R5` sont des bureaux avec des utilisateurs
* `R2` est une salle serveur 
* le bâtiment a une taille de 20m x 20m (approximativement, vous en aurez besoin sur la fin)

**C'est quoi ces machines ?**

Type | Nom | Rôle | Dans GNS 
--- | --- | --- | ---
`A` | Admins | Accès à tout à frer. Full power. | VPCS
`U` | Users | Accès à un peu moins. | VPCS
`S` | Stagiaires | Encore un peu moins. | VPCS
`SRV` | Serveurs | Services hébergés en local. Ceux encadrés en rouge sont des **serveurs sensibles ou SS** | VPCS (ou autre si explicitement demandé)
`P` | Imprimantes | Imprimantes dispo en réseau

---

**Exceptions** *(ce sont des bonus, voir la fin du TP*)
* tous les postes ne peuvent joindre que l'imprimante de leur propre salle
* les serveurs sensibles n'ont pas accès à internet
* seul l'admin 1 (`A1`) a accès au serveur 4 (`SRV4`)

**Qui a accès à qui exactement ?** (à mettre en place dans un second temps)  

✅ = peuvent se joindre
❌ = ne peuvent pas se joindre

X | Admins | Users | Stagiaires | Serveurs | SS | Imprimantes
--- | --- | --- | --- | --- | --- | --- | 
Admins | ✅ | ❌ | ❌ | ✅ | ✅ | ✅ |
Users | ❌ | ✅ | ❌ | ✅ | ❌ | ✅ |
Stagiaires | ❌ | ❌ | ✅ | ❌ | ❌ | ✅ |
Serveurs | ✅ | ✅ | ❌ | ✅ | ❌ | ✅ |
Serveurs sensibles | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ |
Imprimantes | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ |

---

**TODO**
* setup this shit in GNS3
  * matériel autorisé : routeurs (Cisco 3640), switches (IOU L2 Cisco), VPCS
  * outils : routage statique, VLAN, votre talent
* pour la partie soft
  * 🌞 dimensionnez intelligemment les réseaux
    * prévoyez une augmentation légère
  * 🌞 permettre un accès internet à tout le monde
* pour la partie hard
  * 🌞 proposez un nombre de routeur et de switches et précisez à quel endroit physique ils se trouveront
  * 🌞 précisez le nombre de câbles nécessaires et une longueur (approximative)
    * court : moins de 1m
    * moyen : entre 1 et 5m
    * long : 5m+
    * **le but c'est d'avoir un ordre de grandeur**, on s'en fout complet des tailles exactes pour ce TP
* 🌞 livrer, en plus de l'infra, des éléments qui rendent compte de l'infra (de façon simple)
  * schéma réseau (screen GNS ?)
  * référez-vous à la partie I. (tableau des réseaux utilisés, tableau d'adressage)
* **être en mesure de prouver que l'infra fonctionne comme demandé**

**Dans un second temps :**
* 🌞 mettre en place "qui a accès à qui exactement ?"

**Conseils**
* **avant de vous lancer** réfléchissez aux différentes étapes qui vous permettront de réaliser le TP
  * je vous conseille par exemple de faire un schéma et un plan d'adressage **en premier**
* documentez ce que vous faites au fur et à mesure
* n'oubliez pas de sauvegarder la configuration des équipements réseau et celle des VPCS

---

**Bonus**
* 🐙 mettre en place les exceptions
  * documentez-vous, proposez des choses
* 🐙 mettre en place un serveur DHCP 
  * il devra 
    * s'intégrer à l'existant
    * être installé sur une VM dédiée (Virtualbox, Workstation)
    * permettre l'attribution d'IPs pour tous les PCs clients (admins, users, stagiaires)
    * libre choix de l'OS (m'enfin, déconnez pas, on va pas mettre un Windows Server 2016 si ?...)
  * mise en place d'un test avec l'ajout d'un nouveau client