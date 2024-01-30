# I. Service SSH

> Pour rappel : un *service* c'est juste un programme lancé par l'OS à notre place. Donc quand on dit qu'un "service tourne", ça veut concrètement dire qu'il y a un programme en cours d'exécution.

Le service SSH est déjà installé sur la machine, et il est aussi déjà démarré. C'est par défaut sur Rocky.

> *En effet Rocky c'est un OS qui est spécialisé pour faire tourner des serveurs. Pas choquant d'avoir un serveur SSH préinstallé et dispo dès l'installation pour pouvoir s'y connecter à distance ! Ce serait pas le cas sur un OS Ubuntu (par exemple) que vous installeriez sur votre PC. C'est pas le cas non plus sur un Windows 11 ou un MacOS ou un Android ouuuu... bref t'as capté. Mais sur n'importe lequel de ces OS, on peut **ajouter** un service SSH si on le souhaite.*

- [I. Service SSH](#i-service-ssh)
  - [1. Analyse du service](#1-analyse-du-service)
  - [2. Modification du service](#2-modification-du-service)

## 1. Analyse du service

On va, dans cette première partie, analyser le service SSH qui est en cours d'exécution.

🌞 **S'assurer que le service `sshd` est démarré**

- avec une commande `systemctl status`

```
[vince@web ~]$ systemctl status
● web.tp5.b1
    State: running
    Units: 283 loaded (incl. loaded aliases)
     Jobs: 0 queued
   Failed: 0 units
    Since: Mon 2024-01-29 10:18:16 CET; 17min ago
  systemd: 252-13.el9_2
   CGroup: /
           ├─init.scope
           │ └─1 /usr/lib/systemd/systemd --switched-root --syste>           ├─system.slice
           │ ├─NetworkManager.service
           │ │ └─675 /usr/sbin/NetworkManager --no-daemon
           │ ├─auditd.service
lines 1-14...skipping...
```

> *Vous pouvez utiliser des commandes comme `sudo systemctl list-units -t service -a` pour lister tous les services de la machine. Voilà pour le fun, si t'as envie de regarder un peu tout ce qu'il y a qui tourne. C'est littéralement tous les constituants de l'OS que t'as sous les yeux si tu le fais, puisqu'on a ajouté aucun services nous-mêmes pour le moment.*

🌞 **Analyser les processus liés au service SSH**

- afficher les processus liés au service `sshd`
  - vous pouvez afficher la liste des processus en cours d'exécution avec une commande `ps`
  - pour rappel "un processus" c'est juste un programme qu'on a "lancé", qui a donc été déplacé en RAM et qui est en cours d'exécution
  - pour le compte-rendu, vous devez filtrer la sortie de la commande en ajoutant `| grep <TEXTE_RECHERCHE>` après une commande
    - au cas où un ptit rappel de l'utilisation de `| grep` :

```bash
# Exemple de manipulation de | grep

# admettons un fichier texte appelé "fichier_demo"
# on peut afficher son contenu avec la commande cat :
$ cat fichier_demo
bob a un chapeau rouge
emma surfe avec un dinosaure
eve a pas toute sa tête

# il est possible de filtrer la sortie de la commande cat pour afficher uniquement certaines lignes
$ cat fichier_demo | grep emma
emma surfe avec un dinosaure

$ cat fichier_demo | grep bob
bob a un chapeau rouge
```

> Il est possible de repérer le numéro des processus liés à un service avec la commande `systemctl status sshd`.

```
[vince@web ~]$ ps -ef | grep sshd
root         682       1  0 10:18 ?        00:00:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
root        1302     682  0 10:19 ?        00:00:00 sshd: vince [priv]
vince       1305    1302  0 10:19 ?        00:00:00 sshd: vince@pts/0
vince       1354    1306  0 10:43 pts/0    00:00:00 grep --color=auto sshd
```

🌞 **Déterminer le port sur lequel écoute le service SSH**

- avec une commande `ss`
  - il faudra ajouter des options à la commandes `ss` pour que ce soit des infos plus lisibles (encore une fois, [voir le mémoooooo y'a des exemples de commandes](../../cours/memo/shell.md).
- isolez les lignes intéressantes avec un `| grep <TEXTE>`

```
[vince@web ~]$ sudo ss -alnpt | grep sshd
LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=682,fd=3))
LISTEN 0      128             [::]:22           [::]:*    users:(("sshd",pid=682,fd=4))
```

🌞 **Consulter les logs du service SSH**

- les logs du service sont consultables avec une commande `journalctl`
  - donnez une commande `journalctl` qui permet de consulter les logs du service SSH
- AUSSI, il existe un fichier de log, dans lequel le service SSH enregistre toutes les tentatives de connexion
  - il est dans le dossier `/var/log`
  - utilisez une commande `tail` pour visualiser les 10 dernière lignes de ce fichier


```
[vince@web ~]$ journalctl -xe -u sshd
Jan 29 10:18:22 web.tp5.b1 systemd[1]: Starting OpenSSH server daemon...
░░ Subject: A start job for unit sshd.service has begun execution
░░ Defined-By: systemd
░░ Support: https://access.redhat.com/support
░░
░░ A start job for unit sshd.service has begun execution.
░░
░░ The job identifier is 239.
Jan 29 10:18:22 web.tp5.b1 sshd[682]: main: sshd: ssh-rsa algorithm is disabled
Jan 29 10:18:22 web.tp5.b1 sshd[682]: Server listening on 0.0.0.0 port 22.
Jan 29 10:18:22 web.tp5.b1 sshd[682]: Server listening on :: port 22.
Jan 29 10:18:22 web.tp5.b1 systemd[1]: Started OpenSSH server daemon.
░░ Subject: A start job for unit sshd.service has finished successfully
░░ Defined-By: systemd
░░ Support: https://access.redhat.com/support
░░
░░ A start job for unit sshd.service has finished successfully.
░░
░░ The job identifier is 239.
Jan 29 10:19:08 web.tp5.b1 sshd[1302]: main: sshd: ssh-rsa algorithm is disabled
Jan 29 10:19:08 web.tp5.b1 sshd[1302]: Accepted publickey for vince from 10.5.1.1 port 54213 ssh2: RSA SHA256:aW4hXbC1e5ItvZbypolAE4+DVpjL2goO40LeyELFvt0
Jan 29 10:19:08 web.tp5.b1 sshd[1302]: pam_unix(sshd:session): session opened for user vince(uid=1000) by (uid=0)
```
```
[vince@web ~]$ sudo tail -n 10 /var/log/secure
Jan 29 10:55:48 web sudo[1368]: pam_unix(sudo:session): session closed for user root
Jan 29 10:56:31 web sudo[1374]:   vince : TTY=pts/0 ; PWD=/home/vince ; USER=root ; COMMAND=/sbin/ss -alnpt
Jan 29 10:56:31 web sudo[1374]: pam_unix(sudo:session): session opened for user root(uid=0) by vince(uid=1000)
Jan 29 10:56:31 web sudo[1374]: pam_unix(sudo:session): session closed for user root
Jan 29 10:57:03 web sudo[1378]:   vince : TTY=pts/0 ; PWD=/home/vince ; USER=root ; COMMAND=/sbin/ss -alnpt
Jan 29 10:57:03 web sudo[1378]: pam_unix(sudo:session): session opened for user root(uid=0) by vince(uid=1000)
Jan 29 10:57:03 web sudo[1378]: pam_unix(sudo:session): session closed for user root
Jan 29 11:08:32 web sudo[1411]:   vince : TTY=pts/0 ; PWD=/home/vince ; USER=root ; COMMAND=/bin/tail -n 10 /var/log/auth.log
Jan 29 11:08:32 web sudo[1411]: pam_unix(sudo:session): session opened for user root(uid=0) by vince(uid=1000)
Jan 29 11:08:32 web sudo[1411]: pam_unix(sudo:session): session closed for user root
```
![When she tells you](./img/when_she_tells_you.png)

## 2. Modification du service

Dans cette section, on va aller visiter et modifier le fichier de configuration du serveur SSH.

Comme tout fichier de configuration, celui de SSH se trouve dans le dossier `/etc/`.

Plus précisément, il existe un sous-dossier `/etc/ssh/` qui contient toute la configuration relative à SSH

🌞 **Identifier le fichier de configuration du serveur SSH**

```
[vince@web home]$ cd /etc/ssh/
[vince@web ssh]$ ls
moduli      👉ssh_config.d  sshd_config.d       ssh_host_ecdsa_key.pub  ssh_host_ed25519_key.pub  ssh_host_rsa_key.pub
ssh_config  sshd_config   ssh_host_ecdsa_key  ssh_host_ed25519_key    ssh_host_rsa_key
```

🌞 **Modifier le fichier de conf**

- exécutez un `echo $RANDOM` pour **demander à votre shell de vous fournir un nombre aléatoire**
  - simplement pour vous montrer la petite astuce et vous faire manipuler le shell :)
  - pour un numéro de port valide, c'est entre 1 et 65535 ! 
  ```
  [vince@web ssh_config.d]$ echo $RANDOM RANDOM
    1786 RANDOM
  ```
- **changez le port d'écoute du serveur SSH** pour qu'il écoute sur ce numéro de port
  - il faut modifier le fichier avec `nano` ou `vim` par exemple
  - dans le compte-rendu je veux un `cat` du fichier de conf
  - filtré par un `| grep` pour mettre en évidence la ligne que vous avez modifié

  ```
  [vince@web ssh]$ sudo !!
    sudo cat cat /etc/ssh/sshd_config | grep "Port"
    cat: cat: No such file or directory
    Port 1786
    #GatewayPorts no
  ```
- **gérer le firewall**
  - fermer l'ancien port
  - ouvrir le nouveau port
  - vérifier avec un `firewall-cmd --list-all` que le port est bien ouvert
    - vous filtrerez la sortie de la commande avec un `| grep TEXTE`

    ```
    [vince@web ~]$ sudo firewall-cmd --list-all | grep ports
  ports: 80/tcp 1786/tcp
  forward-ports:
  source-ports:
    ```

🌞 **Redémarrer le service**

- avec une commande `systemctl restart`

🌞 **Effectuer une connexion SSH sur le nouveau port**

- depuis votre PC
- il faudra utiliser une option à la commande `ssh` pour vous connecter à la VM

```
PS C:\Users\vince> ssh -p1786 vince@10.5.1.12
Last login: Mon Jan 29 12:11:16 2024 from 10.5.1.1
```

> Je vous conseille de remettre le port par défaut une fois que cette partie est terminée.

✨ **Bonus : affiner la conf du serveur SSH**

- faites vos plus belles recherches internet pour améliorer la conf de SSH
- par "améliorer" on entend essentiellement ici : augmenter son niveau de sécurité
- le but c'est pas de me rendre 10000 lignes de conf que vous pompez sur internet pour le bonus, mais de vous éveiller à divers aspects de SSH, la sécu ou d'autres choses liées

![Such a hacker](./img/such_a_hacker.png)


# II. Service HTTP

**Dans cette partie, on ne va pas se limiter à un service déjà présent sur la machine : on va ajouter un service à la machine.**

➜ **On va faire dans le *clasico* et installer un serveur HTTP très réputé : NGINX**

Un serveur HTTP permet d'héberger des sites web.

➜ **Un serveur HTTP (ou "serveur Web") c'est :**

- un programme qui écoute sur un port (ouais ça change pas ça)
- il permet d'héberger des sites web
  - un site web c'est un tas de pages html, js, css
  - un site web c'est aussi parfois du code php, go, python, ou autres, qui indiquent comment le site doit se comporter
- il permet à des clients de visiter les sites web hébergés
  - pour ça, il faut un client HTTP (par exemple, un navigateur web, ou la commande `curl`)
  - le client peut alors se connecter au port du serveur (connu à l'avance)
  - une fois le tunnel de communication établi, le client effectuera des *requêtes HTTP*
  - le serveur répondra par des *réponses HTTP*

> Une requête HTTP c'est "donne moi tel fichier HTML". Une réponse c'est "voici tel fichier HTML" + le fichier HTML en question.

**Ok bon on y va ?**

- [II. Service HTTP](#ii-service-http)
  - [1. Mise en place](#1-mise-en-place)
  - [2. Analyser la conf de NGINX](#2-analyser-la-conf-de-nginx)
  - [3. Déployer un nouveau site web](#3-déployer-un-nouveau-site-web)

## 1. Mise en place

![nngijgingingingijijnx ?](./img/njgjgijigngignx.jpg)

> Si jamais, pour la prononciation, NGINX ça vient de "engine-X" et vu que c'était naze comme nom... ils ont choisi un truc imprononçable ouais je sais cherchez pas la logique.

🌞 **Installer le serveur NGINX**

- il faut faire une commande `dnf install`
- pour trouver le paquet à installer :
  - `dnf search <VOTRE RECHERCHE>`
  - ou une recherche Google (mais si `dnf search` suffit, c'est useless de faire une recherche pour ça)
  ```powershell
    [vince@web ~]$ sudo dnf install nginx
    Last metadata expiration check: 0:41:56 ago on Mon 29 Jan 2024 11:46:43 AM CET.
    Package nginx-1:1.20.1-14.el9_2.1.x86_64 is already installed.
    Dependencies resolved.
    Nothing to do.
    Complete!
    ```

🌞 **Démarrer le service NGINX**

```powershell
[vince@web ~]$ sudo systemctl start nginx
```
🌞 **Déterminer sur quel port tourne NGINX**

- vous devez filtrer la sortie de la commande utilisée pour n'afficher que les lignes demandées

```
[vince@web ~]$ sudo ss -alnpt | grep nginx
[sudo] password for vince:
LISTEN 0      511          0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=1444,fd=6),("nginx",pid=1443,fd=6))
LISTEN 0      511             [::]:80           [::]:*    users:(("nginx",pid=1444,fd=7),("nginx",pid=1443,fd=7))
```

- ouvrez le port concerné dans le firewall
```powershell
[vince@web ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
Warning: ALREADY_ENABLED: 80:tcp
success
```

> **NB : c'est bientôt la dernière fois que je signale explicitement la nécessité d'ouvrir un port dans le firewall.** Vous devrez vous-mêmes y penser lorsque nécessaire. C'est simple en vrai : dès qu'un truc écoute sur un port, faut ouvrir ce port dans le firewall. *Toutes les commandes liées au firewall doivent malgré tout figurer dans les compte-rendus.*

🌞 **Déterminer les processus liés au service NGINX**

- vous devez filtrer la sortie de la commande utilisée pour n'afficher que les lignes demandées

```powershell
[vince@web ~]$ ps -ef | grep nginx
root        1443       1  0 Jan29 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx       1444    1443  0 Jan29 ?        00:00:00 nginx: worker process
vince       1714    1646  0 03:59 pts/1    00:00:00 grep --color=auto nginx
```

🌞 **Déterminer le nom de l'utilisateur qui lance NGINX**

- vous devriez le voir dans la commande `ps` précédente
- si l'utilisateur existe, alors il est listé dans le fichier `/etc/passwd`
  - je veux un `cat /etc/passwd | grep <USER>` pour mettre en évidence l'utilisateur qui lance NGINX

  ```powershell
  [vince@web ~]$ cat /etc/passwd | grep nginx
    nginx:x:991:991:Nginx web server:/var/lib/nginx:/sbin/nologin
  ```

🌞 **Test !**

- visitez le site web
  - ouvrez votre navigateur (sur votre PC) et visitez `http://<IP_VM>:<PORT>`
  - vous pouvez aussi (toujours sur votre PC) utiliser la commande `curl` depuis un terminal pour faire une requête HTTP et visiter le site
- dans le compte-rendu, je veux le `curl` (pas un screen de navigateur)
  - utilisez Git Bash si vous êtes sous Windows (obligatoire parce que le `curl` de Powershell il fait des dingueries)
  - vous utiliserez `| head` après le `curl` pour afficher que les 7 premières lignes

  ```powershell
  [vince@web nginx]$ curl http://10.5.1.12:80
    <h1>MEOW</h1>
    ps: ça m'affiche ça car j'ai utliser une anicienne VM que j'avais déjà config pour un ancien TP.
  ```

## 2. Analyser la conf de NGINX

🌞 **Déterminer le path du fichier de configuration de NGINX**

- faites un `ls -al <PATH_VERS_LE_FICHIER>` pour le compte-rendu
- la conf c'est dans `/etc/` normalement, comme toujours !

```powershell
[vince@web /]$ ls -al /etc/nginx
total 84
drwxr-xr-x.  4 root root 4096 Nov 16 11:42 .
drwxr-xr-x. 79 root root 8192 Jan 30 03:06 ..
drwxr-xr-x.  2 root root   31 Nov 16 11:53 conf.d
drwxr-xr-x.  2 root root    6 Oct 16 20:00 default.d
-rw-r--r--.  1 root root 1077 Oct 16 20:00 fastcgi.conf
-rw-r--r--.  1 root root 1077 Oct 16 20:00 fastcgi.conf.default
-rw-r--r--.  1 root root 1007 Oct 16 20:00 fastcgi_params
-rw-r--r--.  1 root root 1007 Oct 16 20:00 fastcgi_params.default
-rw-r--r--.  1 root root 2837 Oct 16 20:00 koi-utf
-rw-r--r--.  1 root root 2223 Oct 16 20:00 koi-win
-rw-r--r--.  1 root root 5231 Oct 16 20:00 mime.types
-rw-r--r--.  1 root root 5231 Oct 16 20:00 mime.types.default
-rw-r--r--.  1 root root 2334 Oct 16 20:00 nginx.conf
-rw-r--r--.  1 root root 2656 Oct 16 20:00 nginx.conf.default
-rw-r--r--.  1 root root  636 Oct 16 20:00 scgi_params
-rw-r--r--.  1 root root  636 Oct 16 20:00 scgi_params.default
-rw-r--r--.  1 root root  664 Oct 16 20:00 uwsgi_params
-rw-r--r--.  1 root root  664 Oct 16 20:00 uwsgi_params.default
-rw-r--r--.  1 root root 3610 Oct 16 20:00 win-utf
```
🌞 **Trouver dans le fichier de conf**

- les lignes qui permettent de faire tourner un site web d'accueil (la page moche que vous avez vu avec votre navigateur)
  - ce que vous cherchez, c'est un bloc `server { }` dans le fichier de conf
  - vous ferez un `cat <FICHIER> | grep <TEXTE> -A X` pour me montrer les lignes concernées dans le compte-rendu
    - l'option `-A X` permet d'afficher aussi les `X` lignes après chaque ligne trouvée par `grep`
```powershell
[vince@web /]$ cat /etc/nginx/nginx.conf | grep "server {" -A 35
    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }

# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2;
#        listen       [::]:443 ssl http2;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers PROFILE=SYSTEM;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }

}
```
- une ligne qui commence par `include`
  - cette ligne permet d'inclure d'autres fichiers
  - bah ouais, on stocke pas toute la conf dans un seul fichier, sinon ça serait le bordel
  - encore un `cat <FICHIER> | grep <TEXTE>` pour ne montrer que cette ligne

```powershell
[vince@web /]$ cat /etc/nginx/nginx.conf | grep "include" -A 0
include /usr/share/nginx/modules/*.conf;
--
    include             /etc/nginx/mime.types;
--
    # See http://nginx.org/en/docs/ngx_core_module.html#include
--
    include /etc/nginx/conf.d/*.conf;
--
        include /etc/nginx/default.d/*.conf;
--
#        include /etc/nginx/default.d/*.conf;
```

## 3. Déployer un nouveau site web

🌞 **Créer un site web**

- bon on est pas en cours de design ici, alors on va faire simplissime
- créer un sous-dossier dans `/var/www/`
```powershell
[vince@web /]$ mkdir /var/www
mkdir: cannot create directory ‘/var/www’: Permission denied
[vince@web /]$ sudo !!
sudo mkdir /var/www
```
  - par convention, on stocke les sites web dans `/var/www/`
  - votre dossier doit porter le nom `tp2_linux`
```powershell
[vince@web www]$ ls
tp2_linux
```
- dans ce dossier `/var/www/tp2_linux`, créez un fichier `index.html`
```powershell
[vince@web tp2_linux]$ ls
index.html
```
  - il doit contenir `<h1>MEOW mon premier serveur web</h1>`
```powershell
[vince@web tp2_linux]$ sudo vim index.html
[sudo] password for vince:
[vince@web tp2_linux]$ cat index.html
<h1>MEOW mon premier serveur web</h1>
  ```

🌞 **Gérer les permissions**

- tout le contenu du dossier  `/var/www/tp2_linux` doit appartenir à l'utilisateur qui lance NGINX
```
[vince@web tp2_linux]$ sudo chown -R nginx /var//www/tp2_linux/
```

🌞 **Adapter la conf NGINX**

- dans le fichier de conf principal
  - vous supprimerez le bloc `server {}` repéré plus tôt pour que NGINX ne serve plus le site par défaut
```powershell
[vince@web tp2_linux]$ sudo nano /etc/nginx/nginx.conf
[sudo] password for vince:
```
  - redémarrez NGINX pour que les changements prennent effet
```powershell
[vince@web tp2_linux]$ sudo systemctl restart nginx
```
- créez un nouveau fichier de conf
  - il doit être nommé correctement
  - il doit être placé dans le bon dossier
  - c'est quoi un "nom correct" et "le bon dossier" ?
    - bah vous avez repéré dans la partie d'avant les fichiers qui sont inclus par le fichier de conf principal non ?
    - créez votre fichier en conséquence
```powershell
[vince@web conf.d]$ touch nginx2.conf
touch: cannot touch 'nginx2.conf': Permission denied
[vince@web conf.d]$ sudo !!
sudo touch nginx2.conf
```
  - redémarrez NGINX pour que les changements prennent effet
  - le contenu doit être le suivant :

```nginx
server {
  # le port choisi devra être obtenu avec un 'echo $RANDOM' là encore
  listen <PORT>;

  root /var/www/tp2_linux;
}
```

```powershell
[vince@web conf.d]$ echo $RANDOM
4402
```
```powershell
[vince@web conf.d]$ cat nginx2.conf
server {
  # le port choisi devra être obtenu avec un 'echo $RANDOM' là encore
  listen 4402;

  root /var/www/tp2_linux;
}
```

🌞 **Visitez votre super site web**

- toujours avec une commande `curl` depuis votre PC (ou un navigateur)

```powershell
[vince@web conf.d]$ curl 10.5.1.12:4402
<h1>MEOW mon premier serveur web</h1>
```