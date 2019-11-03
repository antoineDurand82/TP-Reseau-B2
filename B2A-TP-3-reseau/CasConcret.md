# Cas Concret

## Adressage IP

Chaque rôle est dans un réseau avec le même VLan.


### Salle R1

| Nom | Rôles | Net | IP | Vlan | NetMask
| ---- | ---- | ---- | ---- | ---- | ----
| Admin 3 | Admin | 192.168.45.0 | 192.168.45.1 | 10 | /29
| Printer 5 | Serveur | 192.168.40.0 | 192.168.40.2 | 50 | /28
| Console 1 | Console | 192.168.60.0 | 192.168.60.3 | 60 | /28



### Salle R2

| Nom | Rôles | Net | IP | Vlan | NetMask
| ---- | ---- | ---- | ---- | ---- | ----
| Serveur 1 | Serveur | 192.168.45.0 | 192.168.45.1 | 45 | /28
| Serveur 2 | Serveur | 192.168.40.0 | 192.168.40.2 | 40 | /28
| Serveur 3 | Serveur | 192.168.40.0 | 192.168.40.3 | 40 | /28
| Serveur 4 | Serveur | 192.168.40.0 | 192.168.40.4 | 40 | /28
| Serveur 5 | Serveur | 192.168.40.0 | 192.168.40.5 | 40 | /28
| Serveur 6 | Serveur | 192.168.45.0 | 192.168.45.6 | 45 | /28


### Salle R3

| Nom | Rôles | Net | IP | Vlan | NetMask
| ---- | ---- | ---- | ---- | ---- | ----
| User 1 | User | 192.168.20.0 | 192.168.20.1 | 20 | /27
| User 2 | User | 192.168.20.0 | 192.168.20.2 | 20 | /27
| User 3 | User | 192.168.20.0 | 192.168.20.3 | 20 | /27
| User 4 | User | 192.168.20.0 | 192.168.20.1 | 20 | /27
| User 5 | User | 192.168.20.0 | 192.168.20.2 | 20 | /27
| User 6 | User | 192.168.20.0 | 192.168.20.3 | 20 | /27
| Stagiaire 1 | Stagiaire | 192.168.30.0 | 192.168.30.2 | 30 | /28
| Stagiaire 2 | Stagiaire | 192.168.30.0 | 192.168.30.3 | 30 | /28
| Printer 1 | Imprimante | 192.168.50.0 | 192.168.50.1 | 50 | /28

### Salle R4

| Nom | Rôles | Net | IP | Vlan | NetMask
| ---- | ---- | ---- | ---- | ---- | ----
| Admin 1 | Admin | 192.168.10.0 | 192.168.10.1 | 10 | /29
| User 7 | User | 192.168.20.0 | 192.168.20.1 | 20 | /27
| User 8 | User | 192.168.20.0 | 192.168.20.2 | 20 | /27
| User 9 | User | 192.168.20.0 | 192.168.20.3 | 20 | /27
| User 10 | User | 192.168.20.0 | 192.168.20.4 | 20 | /27
| Stagiaire 3 | Stagiaire | 192.168.30.0 | 192.168.30.1 | 30 | /28
| Stagiaire 4 | Stagiaire | 192.168.30.0 | 192.168.30.2 | 30 | /28
| Stagiaire 5 | Stagiaire | 192.168.30.0 | 192.168.30.3 | 30 | /28
| Printer 2 | Imprimante | 192.168.50.0 | 192.168.50.1 | 50 | /28

### Salle R5

| Nom | Rôles | Net | IP | Vlan | NetMask
| ---- | ---- | ---- | ---- | ---- | ----
| Admin 2 | Admin | 192.168.10.0 | 192.168.10.1 | 10 | /29
| User 11 | User | 192.168.20.0 | 192.168.20.1 | 20 | /27
| User 12 | User | 192.168.20.0 | 192.168.20.2 | 20 | /27
| User 13 | User | 192.168.20.0 | 192.168.20.3 | 20 | /27
| User 14 | User | 192.168.20.0 | 192.168.20.4 | 20 | /27
| User 15 | User | 192.168.20.0 | 192.168.20.3 | 20 | /27
| User 16 | User | 192.168.20.0 | 192.168.20.4 | 20 | /27
| Stagiaire 6 | Stagiaire | 192.168.30.0 | 192.168.30.1 | 30 | /28
| Stagiaire 7 | Stagiaire | 192.168.30.0 | 192.168.30.2 | 30 | /28
| Stagiaire 8 | Stagiaire | 192.168.30.0 | 192.168.30.3 | 30 | /28
| Printer 3 | Imprimante | 192.168.50.0 | 192.168.50.1 | 50 | /28
| Printer 4 | Imprimante | 192.168.50.0 | 192.168.50.1 | 50 | /28


## Routeur et switchs

### Switchs

Il nous faudra 6 switchs, un pour chaque salle et un pour lier les switchs au routeur.
Concernant les switch par salle, ils seront situés en hauteur afin de ne pas avoir les câbles qui trainent par terre.

### Routeur

Il nous faudra un seul routeur qui sera placé dans la salle serveur.

## Câbles

Il nous faudra un câble de **moyenne** longueur pour chacune des machines à leur switch
Un câble *long* pour relier les switch au switch du routeur
Un câble *moyen* pour relier le switch du routeur au routeur


Cela fait donc 36 câbles **moyens** et 5 câbles **longs**
