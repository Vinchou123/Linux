# TP1 : Casser avant de construire

**Dans ce TP on va casser des VMs.**

Pour le plaisir ("for fun and profit" comme on dit) mais aussi pour :

- vous pousser à explorer l'environnement par vous-mêmes
- appréhender par vous-mêmes le début de certaines notions
- vous rendre compte qu'on casse pas une machine par hasard, et donc ne pas avoir peur de taper des commandes !

Au début je vous indique des façons de casser la machine, et dans un deuxième temps, je vous laisse être créatifs pour trouver par vos propres moyens d'autres façons de péter la machine.

## Sommaire

- [TP1 : Casser avant de construire](#tp1--casser-avant-de-construire)
  - [Sommaire](#sommaire)
- [I. Setup](#i-setup)
- [II. Casser](#ii-casser)
  - [1. Objectif](#1-objectif)
  - [2. Fichier](#2-fichier)
  - [3. Utilisateurs](#3-utilisateurs)
  - [4. Disques](#4-disques)
  - [5. Malware](#5-malware)
  - [6. You own way](#6-you-own-way)

# I. Setup

➜ **Machine virtuelle avec un OS Linux**

- l'OS de votre choix (Rocky Linux ou autre chose)

➜ **Accès à un terminal sur la machine**

- avec une session SSH
- ou directement l'interface de la VM si ça vous convient

➜ **Effectuer un snapshot**

- c'est une fonctionnalité de VirtualBox (ou de n'importe quel autre hyperviseur)
- ça permet de faire un "instantané" d'une VM à un instant T
- on peut, plus tard, restaurer la VM dans cet état
- je vous recommande donc de faire un snapshot de votre VM avant de la détruire
- comme ça, vous pouvez revenir en arrière, avant qu'elle soit cassée pour continuer à bosser

# II. Casser

## 1. Objectif

➜ **Vous devez rendre la VM inutilisable. Par inutilisable on entend :**

- elle ne démarre (boot) même plus
- ça boot, mais on ne peut plus se connecter de aucune façon
- ça boot, on peut se co, mais on arrive sur un environnement tellement dégradé qu'il est pas utilisable

![Dead yet ?](./img/dead_yet.gif)

➜ **Tout doit être fait depuis le terminal de la VM**

> Si vous avez des doutes sur la validité de votre cassage, demandez-moi !

## 2. Fichier

🌞 **Supprimer des fichiers**

```
[vince@Marcel ~]$ pstree
[vince@Marcel ~]$ ps -ef
[vince@Marcel ~]$ sudo find / -name systemd -type f
/usr/lib/systemd/systemd
[vince@Marcel ~]$ sudo rm -f /usr/lib/systemd/systemd

```

## 3. Utilisateurs

🌞 **Mots de passe**

```
[vince@Marcel ~]$ sudo cat /etc/shadow
[vince@Marcel ~]$ sudo awk -F: '($2 != "!!" && $2 != "*" && $2 != "") {print $1}' /etc/shadow
[vince@Marcel ~]$ sudo vim /etc/shadow

```
- changez le mot de passe de tous les utilisateurs qui en ont déjà un
- trouvez donc un moyen de lister les utilisateurs, et trouver ceux qui ont déjà un mot de passe

> *On peut parfaitement avoir un utilisateur sans mot de passe dans un système Linux : il ne peut pas se connecter du tout. On verra en quoi c'est utile plus tard dans les cours.*

🌞 **Another way ?**

- sans toucher aux mots de passe, faites en sorte qu'aucun utilisateur ne soit utilisable
- trouver un autre moyen donc, en restant sur les utilisateurs !

## 4. Disques

🌞 **Effacer le contenu du disque dur**


- ici on parle pas de toucher aux fichiers et dossiers qui existent au sein du disque dur de la VM
- mais de toucher directement au disque dur
- essayez de remplir le disque de zéros

```
[vince@Marcel ~]$ sudo fdisk -l
[vince@Marcel ~]$ sudo dd if=/dev/zero of=/dev/sda bs=1M
```
## 5. Malware

🌞 **Reboot automatique**
```
[vince@Marcel ~]$ vim reboot_vm_sh
Dans le vim j'ai ajouté ça :
vboxmanage controlvm "marcel.tp3" reset
 pkill -KILL -u vince
 [vince@Marcel ~]$ chmod +x /home/vince/reboot_vm_sh
 [vince@Marcel ~]$ nano ~/.bash_profile
 Dans le bash profile j'ai ajouté le fichier
```

- faites en sorte que si un utilisateur se connecte, ça déclenche un reboot automatique de la machine

## 6. You own way

🌞 **Trouvez 4 autres façons de détuire la machine**

- tout doit être fait depuis le terminal de la machine
- pensez à ce qui constitue un ordi/un OS
- l'idée c'est de supprimer des trucs importants, modifier le comportement de trucs existants, surcharger tel ou tel truc...
