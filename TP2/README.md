# Partie : Files and users

- [Partie : Files and users](#partie--files-and-users)
- [I. Fichiers](#i-fichiers)
  - [1. Find me](#1-find-me)
- [II. Users](#ii-users)
  - [1. Nouveau user](#1-nouveau-user)
  - [2. Infos enregistrées par le système](#2-infos-enregistrées-par-le-système)
  - [3. Hint sur la ligne de commande](#3-hint-sur-la-ligne-de-commande)
  - [3. Connexion sur le nouvel utilisateur](#3-connexion-sur-le-nouvel-utilisateur)

# I. Fichiers

## 1. Find me

🌞 **Trouver le chemin vers le répertoire personnel de votre utilisateur**

```
[vince@web ~]$ pwd
/home/vince
````

🌞 **Trouver le chemin du fichier de logs SSH**

```
[vince@web ~]$ sudo journalctl -xe -u sshd
~
~
~
~
~
~
~
~
~
~
~
~
Jan 22 10:25:17 web.tp5.b1 systemd[1]: Starting OpenSSH server daemon...
░░ Subject: A start job for unit sshd.service has begun execution
░░ Defined-By: systemd
░░ Support: https://access.redhat.com/support
░░
░░ A start job for unit sshd.service has begun execution.
░░
░░ The job identifier is 236.
Jan 22 10:25:17 web.tp5.b1 sshd[682]: main: sshd: ssh-rsa algorithm is disabled
Jan 22 10:25:17 web.tp5.b1 sshd[682]: Server listening on 0.0.0.0 port 22.
Jan 22 10:25:17 web.tp5.b1 sshd[682]: Server listening on :: port 22.
Jan 22 10:25:17 web.tp5.b1 systemd[1]: Started OpenSSH server daemon.
░░ Subject: A start job for unit sshd.service has finished successfully
░░ Defined-By: systemd
░░ Support: https://access.redhat.com/support
░░
░░ A start job for unit sshd.service has finished successfully.
░░
░░ The job identifier is 236.
Jan 22 10:37:45 web.tp5.b1 sshd[1338]: main: sshd: ssh-rsa algorithm is disabled
Jan 22 10:37:49 web.tp5.b1 unix_chkpwd[1340]: password check failed for user (vince)
Jan 22 10:37:49 web.tp5.b1 sshd[1338]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tt>
Jan 22 10:37:51 web.tp5.b1 sshd[1338]: Failed password for vince from 10.5.1.1 port 61826 ssh2
Jan 22 10:37:55 web.tp5.b1 sshd[1338]: Accepted password for vince from 10.5.1.1 port 61826 ssh2
Jan 22 10:37:55 web.tp5.b1 sshd[1338]: pam_unix(sshd:session): session opened for user vince(uid=1000) by (u>
lines 1-25/25 (END)
````

🌞 **Trouver le chemin du fichier de configuration du serveur SSH**

```
[vince@web ~]$ sudo cat /etc/ssh/sshd_config
#       $OpenBSD: sshd_config,v 1.104 2021/07/02 05:11:21 dtucker Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

# To modify the system-wide sshd configuration, create a  *.conf  file under
#  /etc/ssh/sshd_config.d/  which will be automatically included below
Include /etc/ssh/sshd_config.d/*.conf

# If you want to change the port on a SELinux system, you have to tell
# SELinux about this change.
# semanage port -a -t ssh_port_t -p tcp #PORTNUMBER
#
#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
#PermitRootLogin prohibit-password
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

#PubkeyAuthentication yes

# The default is to check both .ssh/authorized_keys and .ssh/authorized_keys2
# but this is overridden so installations will only check .ssh/authorized_keys
AuthorizedKeysFile      .ssh/authorized_keys

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication yes
#PermitEmptyPasswords no

# Change to no to disable s/key passwords
#KbdInteractiveAuthentication yes

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no
#KerberosUseKuserok yes

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no
#GSSAPIEnablek5users no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the KbdInteractiveAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via KbdInteractiveAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and KbdInteractiveAuthentication to 'no'.
# WARNING: 'UsePAM no' is not supported in RHEL and may cause several
# problems.
#UsePAM no

#AllowAgentForwarding yes
#AllowTcpForwarding yes
#GatewayPorts no
#X11Forwarding no
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
#PrintMotd yes
#PrintLastLog yes
#TCPKeepAlive yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS no
#PidFile /var/run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

# no default banner path
#Banner none

# override default of no subsystems
Subsystem       sftp    /usr/libexec/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
#       X11Forwarding no
#       AllowTcpForwarding no
#       PermitTTY no
#       ForceCommand cvs server
```

# II. Users

## 1. Nouveau user

🌞 **Créer un nouvel utilisateur**

```
   33  sudo useradd marmotte -m -d  /home/papier_alu/
   34  sudo passwd marmotte
   35  history
```
## 2. Infos enregistrées par le système

➜ **Pour le compte-rendu**, et pour vous habituer à **utiliser le terminal de façon pratique**, petit hint :

```bash
# supposons un fichier "nul.txt", on peut afficher son contenu avec la commande :
$ cat /chemin/vers/nul.txt
salut
à
toi

# il est possible en une seule ligne de commande d'afficher uniquement une ligne qui contient un mot donné :
$ cat /chemin/vers/nul.txt | grep salut
salut

# à l'aide de `| grep xxx`, on a filtré la sortie de la commande cat
# ça n'affiche donc que la ligne qui contient le mot demandé : "salut"
```

🌞 **Prouver que cet utilisateur a été créé**

```
[vince@web ~]$ sudo cat /etc/passwd | grep marmotte
marmotte:x:1001:1001::/home/papier_alu/:/bin/bash

```

🌞 **Déterminer le *hash* du password de l'utilisateur `marmotte`**

```
[vince@web ~]$ sudo cat /etc/shadow | grep marmotte
marmotte:$6$j0Rz/wdEZDahV/oN$79ZXuTgW6lKbs9LorabtlxnidvSPxyUBlbHkcQjbBYyCd6Lwk.b8Q2DDTUHrK5DTjgsUqQYOzkaaNpHXQpNg0/:19744:0:99999:7:::
```

> **On ne stocke JAMAIS le mot de passe des utilisateurs** (sous Linux, ou ailleurs) mais **on stocke les *hash* des mots de passe des users.** Un *hash* c'est un dérivé d'un mot de passe utilisateur : il permet de vérifier à l'avenir que le user tape le bon password, mais sans l'avoir stocké ! On verra ça une autre fois en détails.

![File ?](./img/file.jpg)

## 3. Hint sur la ligne de commande

> *Ce qui est dit dans cette partie est valable pour tous les OS.*

**Quand on donne le chemin d'un fichier à une commande, on peut utiliser soit un *chemin relatif*, soit un *chemin absolu* :**

➜ **chemin absolu**

- c'est le chemin complet vers le fichier
  - il commence forcément par `/` sous Linux ou MacOS
  - il commence forcément par `C:/` (ou une autre lettre) sous Windows
- peu importe où on l'utilise, ça marche tout le temps
- par exemple :
  - `/etc/ssh/sshd_config` est un chemin absolu
  - *et c'est le chemin vers le fichier de conf du serveur SSH sous Linux en l'occurrence*
- mais parfois c'est super long et chiant à taper/utiliser donc on peut utiliser...

➜ ... un **chemin relatif**

- on écrit pas le chemin en entier, mais uniquement le chemin depuis le dossier où se trouve
- par exemple :
  - si on se trouve dans le dossier `/etc/ssh/`
  - on peut utiliser `./sshd_config` : c'est le chemin relatif de `sshd_config` quand on se trouve dans `/etc/ssh/`
  - un chemin relatif commence toujours par un `.`
  - `.` c'est "le dossier actuel"

➜ **Exemples :**

```bash
# on se déplace dans un répertoire spécifique, ici le répertoire personnel du user it4
$ cd /home/it4

# on affiche (parce que pourquoi pas) le fichier de conf du serveur SSH
# en utilisant le chemin absolu du fichier
$ cat /etc/ssh/sshd_config
[...] # ça fonctionne

# cette fois chemin relatif ???
$ cat ./sshd_config
cat: sshd_config: No such file or directory
# on a une erreur car le fichier "sshd_config" n'existe pas dans "/home/it4"

# on se déplace dans le bon dossier
$ cd /etc/ssh

# et là
$ cat ./sshd_config
[...] # ça fonctionne

# en vrai pour permettre d'aller plus vite, ça marche aussi si on met pas le ./ au début
$ cat sshd_config
[...] # ça fonctionne
```

## 3. Connexion sur le nouvel utilisateur

🌞 **Tapez une commande pour vous déconnecter : fermer votre session utilisateur**

```
[vince@web ~]$ exit
logout
Connection to 10.5.1.12 closed.
```

🌞 **Assurez-vous que vous pouvez vous connecter en tant que l'utilisateur `marmotte`**

```
[marmotte@web home]$ ls vince
ls: cannot open directory 'vince': Permission denied
```

> **On verra en détails la gestion des droits très vite.**

# Partie 2 : Programmes et paquets

- [Partie 2 : Programmes et paquets](#partie-2--programmes-et-paquets)
- [I. Programmes et processus](#i-programmes-et-processus)
  - [1. Run then kill](#1-run-then-kill)
  - [2. Tâche de fond](#2-tâche-de-fond)
  - [3. Find paths](#3-find-paths)
  - [4. La variable PATH](#4-la-variable-path)
- [II. Paquets](#ii-paquets)

# I. Programmes et processus

➜ Dans cette partie, afin d'avoir quelque chose à étudier, on va utiliser le programme `sleep`

- c'est une commande (= un programme) dispo sur tous les OS
- ça permet de... ne rien faire pendant X secondes
- la syntaxe c'est `sleep 90` pour attendre 90 secondes
- on s'en fout de `sleep` en doit, c'est une commande utile parmi plein d'autres, elle est pratique pour étudier les trucs que j'veux vous montrer

## 1. Run then kill

🌞 **Lancer un processus `sleep`**

```
[vince@web ~]$ ps aux | grep sleep
vince       1730  0.0  0.1   5584  1016 pts/0    S+   12:13   0:00 sleep 1000
vince       1732  0.0  0.2   6408  2300 pts/1    S+   12:13   0:00 grep --color=auto sleep
```

🌞 **Terminez le processus `sleep` depuis le deuxième terminal**

```
[vince@web ~]$ ps aux | grep sleep
vince       1748  0.2  0.1   5584  1016 pts/0    S+   12:21   0:00 sleep 1000
vince       1750  0.0  0.2   6408  2280 pts/1    S+   12:21   0:00 grep --color=auto sleep
[vince@web ~]$ kill 1748
```

![Kill it](./img/killit.jpg)

## 2. Tâche de fond

🌞 **Lancer un nouveau processus `sleep`, mais en tâche de fond**

```
[vince@web ~]$ sleep 1000 &
[1] 1751
```

🌞 **Visualisez la commande en tâche de fond**

```
[vince@web ~]$ jobs -p
1751
```

## 3. Find paths

➜ La commande `sleep`, comme toutes les commandes, c'est un programme

- sous Linux, on met pas l'extension `.exe`, s'il y a pas d'extensions, c'est que c'est un exécutable généralement

🌞 **Trouver le chemin où est stocké le programme `sleep`**

```
[vince@web ~]$ ls -al /usr/bin/sleep | grep sleep
-rwxr-xr-x. 1 root root 36312 Apr 24  2023 /usr/bin/sleep
```

🌞 Tant qu'on est à chercher des chemins : **trouver les chemins vers tous les fichiers qui s'appellent `.bashrc`**

```
[vince@web ~]$ sudo find / -name ".bashrc"
[sudo] password for vince:
/etc/skel/.bashrc
/root/.bashrc
/home/vince/.bashrc
/home/marmotte/.bashrc
/home/papier_alu/.bashrc
```

➜ `find` est une commande de ouf qui permet de trouver des fichiers ou dossiers selon plein de critères

```bash
# quelques exemples d'utilisation de find

# trouver tous les .jpg sur tout le disque dur
$ sudo find / -name "*.jpg"

# trouver tous les .jpg sur tout le disque dur, s'ils font + de 10Mo
$ sudo find / -name "./jpg" -size +10M

# c'est un tout ptit aperçu, un peut chercher en fonction de plein plein de critères, et c'est hyper rapide
```

## 4. La variable PATH

**Sur tous les OS (MacOS, Windows, Linux, et autres) il existe une variable `PATH` qui est définie quand l'OS démarre.** Elle est accessible par toutes les applications qui seront lancées sur cette machine.

**On dit que `PATH` est une variable d'environnement.** C'est une variable définie par l'OS, et accessible par tous les programmes.

> *Il existe plein de variables d'environnement définie dès que l'OS démarre, `PATH` n'est pas la seule. On peut aussi en créer autant qu'on veut. Attention, suivant avec quel utilisateur on se connecte à une machine, les variables peuvent être différentes : parfait pour avoir chacun sa configuration !*

**`PATH` contient la liste de tous les dossiers qui contiennent des commandes/programmes.**

Ainsi quand on tape une commande comme `ls /home`, il se passe les choses suivantes :

- votre terminal consulte la valeur de la variable `PATH`
- parmi tous les dossiers listés dans cette variable, il cherche s'il y en a un qui contient un programme nommé `ls`
- si oui, il l'exécute
- sinon : `command not found`

Démonstration :

```bash
# on peut afficher la valeur de la variable PATH à n'importe quel moment dans un terminal
$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/bin:/bin
# ça contient bien une liste de dossiers, séparés par le caractère ":"

# si la commande ls fonctionne c'est forcément qu'elle se trouve dans l'un de ces dossiers
# on peut savoir lequel avec la commande which qui interroge aussi la variable PATH
$ which ls
/usr/bin/ls
```

🌞 **Vérifier que**

- les commandes `sleep`, `ssh`, et `ping` sont bien des programmes stockés dans l'un des dossiers listés dans votre `PATH`
```
[vince@web ~]$ echo $PATH
/home/vince/.local/bin:/home/vince/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin
[vince@web ~]$ which sleep
/usr/bin/sleep
[vince@web ~]$ which ssh
/usr/bin/ssh
[vince@web ~]$ which ping
/usr/bin/ping
```

# II. Paquets

➜ **Tous les OS Linux sont munis d'un store d'application**

- c'est natif
- quand tu fais `apt install` ou `dnf install`, ce genre de commandes, t'utilises ce store
- on dit que `apt` et `dnf` sont des gestionnaires de paquets
- ça permet aux utilisateurs de télécharger des nouveaux programmes (ou d'autres trucs) depuis un endroit safe

🌞 **Installer le paquet `firefox`**
```
[vince@web ~]$ sudo dnf install git
```

🌞 **Utiliser une commande pour lancer Firefox**

```
[vince@web ~]$ sudo find / -name "git"
/var/lib/selinux/targeted/active/modules/100/git
```

🌞 **Installer le paquet `nginx`**

```
[vince@web ~]$ sudo dnf install nginx
Last metadata expiration check: 0:11:24 ago on Tue 23 Jan 2024 10:26:41 AM CET.
Package nginx-1:1.20.1-14.el9_2.1.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```

🌞 **Déterminer**

- le chemin vers le dossier de logs de NGINX
- le chemin vers le dossier qui contient la configuration de NGINX
```
[vince@web ~]$ sudo ls /var/log/nginx
[sudo] password for vince:
access.log  access.log-20240123  error.log  error.log-20240123
```
```
[vince@web ~]$ ls /etc/nginx
conf.d                  koi-utf             scgi_params
default.d               koi-win             scgi_params.default
fastcgi.conf            mime.types          uwsgi_params
fastcgi.conf.default    mime.types.default  uwsgi_params.default
fastcgi_params          nginx.conf          win-utf
fastcgi_params.default  nginx.conf.default
```
🌞 **Mais aussi déterminer...**

- l'adresse `http` ou `https` du serveur où vous avez téléchargé le paquet
- une commande `apt install` ou `dnf install` permet juste de faire un téléchargement HTTP
- ma question c'est donc : sur quel URL tu t'es connecté pour télécharger ce paquet
- il existe un dossier qui contient la liste des URLs consultées quand vous demandez un téléchargement de paquets
```
 cd /etc/yum.repos.d/
  111  ls
  112  cat rocky
  113  cat rocky.repo
  114  grep http rocky.repo
  115  grep http -r
  116  grep http -rn
  117  grep mirrorlist -rn
  118  grep mirrorlist -rn -E '^mirrorlist'
  119  grep -rn -E '^mirrorlist'
```

# Partie 3 : Poupée russe

Pour finir de vous exercer avec le terminal, je vous ai préparé une poupée russe :D

➜ **De mon côté, pour préparer l'exercice :**

- j'ai créé un dossier `dawa/`, qui a plein de sous-dossiers, sous-fichiers, etc
  - y'a des fichiers particuliers, les autres c'est juste du random
- je l'ai archivé/compressé plusieurs fois
- j'ai volontairement supprimé les extensions des fichiers compressés
  - comme ça tu peux pas savoir à l'aide du nom si c'est un `.zip` ou un autre format

➜ **Le but de l'exercice pour vous :**

- je vous file le fichier que j'ai compressé/archivé plusieurs fois, que vous téléchargez dans la VM
- vous devez :
  - déterminer le type du fichier
  - renommer le fichier correctement (si c'est une archive zip, il faut ajouter `.zip` à son nom)
  - extraire l'archive
  - répéter l'opération (car j'ai mis une archive, dans une archive, dans une archive... meow)
- bien sûr que c'est stupide
  - mais c'est pour vous faire pratiquer le terminal, et voir des commandes usuelles
- une fois que vous avez trouvé le dossier `dawa/`, vous avez fini le jeu de poupées russes
- vous allez devoir trouver les fichiers spécifiques que je vous demande à l'intérieur de ce dossier

🌞 **Récupérer le fichier `meow`**

- [il est dispo là](./meow) (juste à côté de ce README.md)
- téléchargez-le à l'aide d'une commande

```
       ls -al
  157  mkdir tp2
  158  cd tp2/
  159  ls

       wget https://gitlab.com/it4lik/b1-linux-2023/-/raw/master/tp/2/meow?inline=false
  161  sudo dnf install -y wget
  162  sudo vim /etc/resolv.conf
  163  sudo dnf install -y wget*
  164  wget https://gitlab.com/it4lik/b1-linux-2023/-/raw/master/tp/2/meow?inline=false
  165  wget  https://gitlab.com/it4lik/b1-linux-2023/-/raw/master/tp/2/meow
```

🌞 **Trouver le dossier `dawa/`**

- le fichier `meow` récupéré est une archive compressée
- utilisez la commande `file /path/vers/le/fichier` pour déterminer le type du fichier
- renommez-le fichier correctement (si c'est une archive compressée ZIP, il faut ajouter `.zip` à son nom)
- extraire l'archive avec une commande
- répétez ces opérations jusqu'à trouver le dossier `dawa/`

> *Dans les OS Linux, le format d'archivage/compression qu'on voit le plus c'est `.tar.gz` (archivage tar avec une compression gz) et moins de `.zip`.*
```
       ls
  168  file meow
  169  mv meow meow.zip
  170  unzip meow.zip
  171  sudo dnf install -y unzip
  172  unzip meow.zip
  173  ls
  174  file meow
  175  mv meow.xz
  176  mv meow meow.xz
  177  ls
  178  xz --help

```

🌞 **Dans le dossier `dawa/`, déterminer le chemin vers**

- le seul fichier de 15Mo
- le seul fichier qui ne contient que des `7`
- le seul fichier qui est nommé `cookie`
- le seul fichier caché (un fichier caché c'est juste un fichier dont le nom commence par un `.`)
- le seul fichier qui date de 2014
- le seul fichier qui a 5 dossiers-parents
  - je pense que vous avez vu que la structure c'est 50 `folderX`, chacun contient 50 dossiers `X`, et chacun contient 50 `fileX`
  - bon bah là y'a un fichier qui est contenu dans `folderX/X/X/X/X/` et c'est le seul qui 5 dossiers parents comme ça

![Matryoshka](./img/dolls.png)