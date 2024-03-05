# Partie 1 : Mise en place et maîtrise du serveur Web

Dans cette partie on va installer le serveur web, et prendre un peu la maîtrise dessus, en regardant où il stocke sa conf, ses logs, etc. Et en manipulant un peu tout ça bien sûr.

On va installer un serveur Web très très trèèès utilisé autour du monde : le serveur Web Apache.

- [Partie 1 : Mise en place et maîtrise du serveur Web](#partie-1--mise-en-place-et-maîtrise-du-serveur-web)
  - [1. Installation](#1-installation)
  - [2. Avancer vers la maîtrise du service](#2-avancer-vers-la-maîtrise-du-service)

![Tipiii](../img/linux_is_a_tipi.jpg)

## 1. Installation

🖥️ **VM web.tp6.linux**

**N'oubliez pas de dérouler la [📝**checklist**📝](../README.md#checklist).**

| Machine         | IP            | Service     |
|-----------------|---------------|-------------|
| `web.tp6.linux` | `10.6.1.11` | Serveur Web |

🌞 **Installer le serveur Apache**

- paquet `httpd`
- la conf se trouve dans `/etc/httpd/`
  - le fichier de conf principal est `/etc/httpd/conf/httpd.conf`
  - je vous conseille **vivement** de virer tous les commentaire du fichier, à défaut de les lire, vous y verrez plus clair
    - avec `vim` vous pouvez tout virer avec `:g/^ *#.*/d`

```golang
[vince@web ~]$ sudo cat /etc/resolv.conf
[sudo] password for vince:
# Generated by NetworkManager
search tp6.linux
nameserver 8.8.8.8
[vince@web ~]$ sudo nano /etc/resolv.conf
[vince@web ~]$ sudo yum install httpd
[vince@web ~]$ cd /etc/httpd/
[vince@web httpd]$ sudo vim conf/httpd.conf
```

> Ce que j'entends au-dessus par "fichier de conf principal" c'est que c'est **LE SEUL** fichier de conf lu par Apache quand il démarre. C'est souvent comme ça : un service ne lit qu'un unique fichier de conf pour démarrer. Cherchez pas, on va toujours au plus simple. Un seul fichier, c'est simple.  
**En revanche** ce serait le bordel si on mettait toute la conf dans un seul fichier pour pas mal de services.  
Donc, le principe, c'est que ce "fichier de conf principal" définit généralement deux choses. D'une part la conf globale. D'autre part, il inclut d'autres fichiers de confs plus spécifiques.  
On a le meilleur des deux mondes : simplicité (un seul fichier lu au démarrage) et la propreté (éclater la conf dans plusieurs fichiers).

🌞 **Démarrer le service Apache**

- le service s'appelle `httpd` (raccourci pour `httpd.service` en réalité)
  - démarrez-le
  - faites en sorte qu'Apache démarre automatiquement au démarrage de la machine
    - ça se fait avec une commande `systemctl` référez-vous au mémo
  - ouvrez le port firewall nécessaire
    - utiliser une commande `ss` pour savoir sur quel port tourne actuellement Apache
    - une portion du mémo commandes est dédiée à `ss`

```golang
[vince@web ~]$ sudo systemctl start httpd
sudo systemctl enable httpd
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
[vince@web httpd]$ sudo systemctl restart httpd
[vince@web httpd]$ sudo ss -alnpt
[vince@web httpd]$ sudo firewall-cmd --add-port=80/tcp --permanent
success
[vince@web httpd]$ sudo firewall-cmd --reload
success
```



**En cas de problème** (IN CASE OF FIIIIRE) vous pouvez check les logs d'Apache :

```bash
# Demander à systemd les logs relatifs au service httpd
$ sudo journalctl -xe -u httpd

# Consulter le fichier de logs d'erreur d'Apache
$ sudo cat /var/log/httpd/error_log

# Il existe aussi un fichier de log qui enregistre toutes les requêtes effectuées sur votre serveur
$ sudo cat /var/log/httpd/access_log
```

🌞 **TEST**

- vérifier que le service est démarré
- vérifier qu'il est configuré pour démarrer automatiquement
- vérifier avec une commande `curl localhost` que vous joignez votre serveur web localement
- vérifier depuis votre PC que vous accéder à la page par défaut
  - avec votre navigateur (sur votre PC)
  - avec une commande `curl` depuis un terminal de votre PC (je veux ça dans le compte-rendu, pas de screen)

```golang
[vince@web httpd]$ sudo systemctl status httpd  
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; preset: disabled)
     Active: active (running) since Tue 2024-03-05 10:20:53 CET; 10min ago
       Docs: man:httpd.service(8)
   Main PID: 1924 (httpd)
     Status: "Total requests: 1; Idle/Busy workers 100/0;Requests/sec: 0.00167; Bytes served/sec:  13>
      Tasks: 213 (limit: 4674)
     Memory: 23.0M
        CPU: 599ms
     CGroup: /system.slice/httpd.service
             ├─1924 /usr/sbin/httpd -DFOREGROUND
             ├─1925 /usr/sbin/httpd -DFOREGROUND
             ├─1926 /usr/sbin/httpd -DFOREGROUND
             ├─1927 /usr/sbin/httpd -DFOREGROUND
             └─1928 /usr/sbin/httpd -DFOREGROUND

Mar 05 10:20:53 web.tp6.linux systemd[1]: Starting The Apache HTTP Server...
Mar 05 10:20:53 web.tp6.linux systemd[1]: Started The Apache HTTP Server.
Mar 05 10:20:53 web.tp6.linux httpd[1924]: Server configured, listening on: port 80

[vince@web httpd]$ sudo systemctl is-enabled httpd     
enabled
[vince@web httpd]$ curl 10.6.1.11
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/

      html {
        height: 100%;
        width: 100%;
      }
        body {
  background: rgb(20,72,50);
  background: -moz-linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%)  ;
  background: -webkit-linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%) ;
  background: linear-gradie...
```

## 2. Avancer vers la maîtrise du service

🌞 **Le service Apache...**

- affichez le contenu du fichier `httpd.service` qui contient la définition du service Apache

```golang
[vince@web httpd]$ cat /usr/lib/systemd/system/httpd.service
# See httpd.service(8) for more information on using the httpd service.

# Modifying this file in-place is not recommended, because changes
# will be overwritten during package upgrades.  To customize the
# behaviour, run "systemctl edit httpd" to create an override unit.

# For example, to pass additional options (such as -D definitions) to
# the httpd binary at startup, create an override unit (as is done by
# systemctl edit) and enter the following:

#       [Service]
#       Environment=OPTIONS=-DMY_DEFINE

[Unit]
Description=The Apache HTTP Server
Wants=httpd-init.service
After=network.target remote-fs.target nss-lookup.target httpd-init.service
Documentation=man:httpd.service(8)

[Service]
Type=notify
Environment=LANG=C

ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
# Send SIGWINCH for graceful stop
KillSignal=SIGWINCH
KillMode=mixed
PrivateTmp=true
OOMPolicy=continue

[Install]
WantedBy=multi-user.target
```

🌞 **Déterminer sous quel utilisateur tourne le processus Apache**

- mettez en évidence la ligne dans le fichier de conf principal d'Apache (`httpd.conf`) qui définit quel user est utilisé

```golang
[vince@web httpd]$ grep "User" /etc/httpd/conf/httpd.conf
User apache
```
- utilisez la commande `ps -ef` pour visualiser les processus en cours d'exécution et confirmer que apache tourne bien sous l'utilisateur mentionné dans le fichier de conf
  - filtrez les infos importantes avec un `| grep`
  ```golang
[vince@web httpd]$ ps -ef | grep apache
apache      1925    1924  0 10:20 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache      1926    1924  0 10:20 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache      1927    1924  0 10:20 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache      1928    1924  0 10:20 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache      2189    1924  0 10:36 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
vince       2331    1281  0 11:06 pts/0    00:00:00 grep --color=auto apache
```
- la page d'accueil d'Apache se trouve dans `/usr/share/testpage/`
  - vérifiez avec un `ls -al` que tout son contenu est **accessible en lecture** à l'utilisateur mentionné dans le fichier de conf
```golang
[vince@web httpd]$ ls -al /usr/share/testpage/
total 12
drwxr-xr-x.  2 root root   24 Mar  5 10:10 .
drwxr-xr-x. 83 root root 4096 Mar  5 10:10 ..
-rw-r--r--.  1 root root 7620 Feb 21 14:12 index.html
```

🌞 **Changer l'utilisateur utilisé par Apache**

- créez un nouvel utilisateur
  - pour les options de création, inspirez-vous de l'utilisateur Apache existant
    - le fichier `/etc/passwd` contient les informations relatives aux utilisateurs existants sur la machine
    ```golang
    [vince@web httpd]$ grep apache /etc/passwd
    apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
    ```
    - servez-vous en pour voir la config actuelle de l'utilisateur Apache par défaut (son homedir et son shell en particulier)
    ```golang
    [vince@web httpd]$ sudo useradd -d /var/www -s /sbin/nologin Vince
    [sudo] password for vince:
    useradd: warning: the home directory /var/www already exists.
    useradd: Not copying any file from skel directory into it.
    ```

- modifiez la configuration d'Apache pour qu'il utilise ce nouvel utilisateur
```golang
[vince@web httpd]$ sudo vim /etc/httpd/conf/httpd.conf
```
  - montrez la ligne de conf dans le compte rendu, avec un `grep` pour ne montrer que la ligne importante
    ```golang
    [vince@web httpd]$ grep -E '^(User|Group)' /etc/httpd/conf/httpd.conf
    User Vince
    ```  
- redémarrez Apache
```golang
[vince@web httpd]$ sudo systemctl restart httpd
```
- utilisez une commande `ps` pour vérifier que le changement a pris effet
  - vous devriez voir un processus au moins qui tourne sous l'identité de votre nouvel utilisateur
```golang
[vince@web httpd]$ ps aux | grep httpd
root        1903  0.0  1.0  19300  8624 pts/0    T    10:14   0:00 sudo vim conf/httpd.conf
root        1905  0.0  1.2  14436  9624 pts/0    T    10:14   0:00 vim conf/httpd.conf
root        2365  0.0  1.4  20396 11708 ?        Ss   11:25   0:00 /usr/sbin/httpd -DFOREGROUND
Vince       2366  0.0  0.9  21676  7496 ?        S    11:25   0:00 /usr/sbin/httpd -DFOREGROUND
Vince       2367  0.0  2.4 1079480 19344 ?       Sl   11:25   0:00 /usr/sbin/httpd -DFOREGROUND
Vince       2368  0.0  2.4 1079480 19344 ?       Sl   11:25   0:00 /usr/sbin/httpd -DFOREGROUND
Vince       2369  0.0  2.1 1210616 17304 ?       Sl   11:25   0:00 /usr/sbin/httpd -DFOREGROUND
vince       2582  0.0  0.2   6408  2296 pts/0    S+   11:26   0:00 grep --color=auto httpd
```


🌞 **Faites en sorte que Apache tourne sur un autre port**

- modifiez la configuration d'Apache pour lui demander d'écouter sur un autre port de votre choix
```golang
[vince@web httpd]$ sudo vim /etc/httpd/conf/httpd.conf
```
  - montrez la ligne de conf dans le compte rendu, avec un `grep` pour ne montrer que la ligne importante
  ```golang
  [vince@web httpd]$ grep -E '^Listen ' /etc/httpd/conf/httpd.conf
Listen 7777
```
- ouvrez ce nouveau port dans le firewall, et fermez l'ancien
```golang
[vince@web httpd]$ sudo firewall-cmd --add-port=7777/tcp --permanent
success
[vince@web httpd]$ sudo firewall-cmd --remove-port=80/tcp --permanent
success
```
- redémarrez Apache
```golang
[vince@web httpd]$ sudo firewall-cmd --reload
success
```
- prouvez avec une commande `ss` que Apache tourne bien sur le nouveau port choisi
```golang
[vince@web httpd]$ sudo ss -alnpt | grep :7777
LISTEN 0      511                *:7777             *:*    users:(("httpd",pid=2628,fd=4),("httpd",pid=2627,fd=4),("httpd",pid=2626,fd=4),("httpd",pid=2624,fd=4))
```
- vérifiez avec `curl` en local que vous pouvez joindre Apache sur le nouveau port
```
[vince@web httpd]$ curl http://localhost:77
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/

      html {
        height: 100%;
        width: 100%;
      }
        body {
  background: rgb(20,72,50);
  background: -moz-linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%)  ;
  background: -webkit-linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%) ;
  background: linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%);
  background-repeat: no-repeat;
  background-attachment: fixed;
  filter: progid:DXImageTransform.Microsoft.gradient(startColorstr="#3c6eb4",endColorstr="#3c95b4",GradientType=1);
        color: white;
        font-size: 0.9em;
       
       ...
       ...
       ...

        <p><strong>For systems using
        <a href="https://nginx.org">Nginx</strong></a>:
        You can add your content in a location of your
        choice and edit the <code>root</code> configuration directive
        in <code>/etc/nginx/nginx.conf</code>.</p>

        <div id="logos">
          <a href="https://rockylinux.org/" id="rocky-poweredby"><img src="icons/poweredby.png" alt="[ Powered by Rocky Linux ]" /></a> <!-- Rocky -->
          <img src="poweredby.png" /> <!-- webserver -->
        </div>
      </div>
      </div>

      <footer class="col-sm-12">
      <a href="https://apache.org">Apache&trade;</a> is a registered trademark of <a href="https://apache.org">the Apache Software Foundation</a> in the United States and/or other countries.<br />
      <a href="https://nginx.org">NGINX&trade;</a> is a registered trademark of <a href="https://">F5 Networks, Inc.</a>.
      </footer>

  </body>
</html>
```

- vérifiez avec votre navigateur que vous pouvez joindre le serveur sur le nouveau port

📁 **Fichier `/etc/httpd/conf/httpd.conf`**

➜ **Si c'est tout bon vous pouvez passer à [la partie 2.](../part2/README.md)**