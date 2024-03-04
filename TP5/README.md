# TP5

# Partie 1 : Script carte d'identité

- [Partie 1 : Script carte d'identité](#partie-1--script-carte-didentité)
  - [Rendu](#rendu)
  - [Bonus](#bonus)

Vous allez écrire **un script qui récolte des informations sur le système et les affiche à l'utilisateur.** Il s'appellera `idcard.sh` et sera stocké dans `/srv/idcard/idcard.sh`.

> `.sh` est l'extension qu'on donne par convention aux scripts réalisés pour être exécutés avec `sh` ou `bash`.

➜ **L'emoji 🐚** est une aide qui indique une commande qui est capable de réaliser le point demandé

➜ **Testez les commandes à la main avant de les incorporer au script.**

➜ Ce que doit faire le script. Il doit afficher :

- le **nom de la machine**
  - 🐚 `hostnamectl`
- le **nom de l'OS** de la machine
  - regardez le fichier `/etc/redhat-release` ou `/etc/os-release`
  - 🐚 `source`
- la **version du noyau** Linux utilisé par la machine
  - 🐚 `uname`
- l'**adresse IP** de la machine
  - 🐚 `ip`
- l'**état de la RAM**
  - 🐚 `free`
  - espace dispo en RAM (en Go, Mo, ou Ko)
  - taille totale de la RAM (en Go, Mo, ou Ko)
- l'**espace restant sur le disque dur**, en Go (ou Mo, ou Ko)
  - 🐚 `df`
- le **top 5 des processus** qui pompent le plus de RAM sur la machine actuellement. Procédez par étape :
  - 🐚 `ps`
  - listez les process
  - affichez la RAM utilisée par chaque process
  - triez par RAM utilisée
  - isolez les 5 premiers
- la **liste des ports en écoute** sur la machine, avec le programme qui est derrière
  - préciser, en plus du numéro, s'il s'agit d'un port TCP ou UDP
  - 🐚 `ss`
  - je vous recommande d'utiliser une [syntaxe `while read`](../../cours/scripting/README.md#8-itérer-proprement-sur-plusieurs-lignes)
- la liste des dossiers disponibles dans la variable `$PATH`
- un **lien vers une image/gif** random de chat
  - 🐚 `curl`
  - il y a de très bons sites pour ça hihi
  - avec [celui-ci](https://thecatapi.com/) par exemple, une simple requête HTTP vous retourne l'URL d'une random image de chat
    - une requête sur cette adresse retourne directement l'image, il faut l'enregistrer dans un fichier
    - parfois le fichier est un JPG, parfois un PNG, parfois même un GIF
    - 🐚 `file` peut vous aider à déterminer le type de fichier

Pour vous faire manipuler les sorties/entrées de commandes, votre script devra sortir **EXACTEMENT** :

```bash
$ /srv/idcard/idcard.sh
Machine name : ...
OS ... and kernel version is ...
IP : ...
RAM : ... memory available on ... total memory
Disk : ... space left
Top 5 processes by RAM usage :
  - ...
  - ...
  - ...
  - ...
  - ...
Listening ports :
  - 22 tcp : sshd
  - ...
  - ...
PATH directories :
  - /usr/local/bin
  - ...
  - ...

Here is your random cat (jpg file) : ./cat.jpg
```

> ⚠️ **Votre script doit fonctionner peu importe les conditions** : peu importe le nom de la machine, ou son adresse IP (genre il est interdit de récupérer pile 10 char sous prétexte que ton adresse IP c'est `10.10.10.1` et qu'elle fait 10 char de long)

## Rendu

📁 **Fichier `/srv/idcard/idcard.sh`**

```bash
voici mon fichier :
#!/bin/bash

echo "Machine name : $(hostnamectl hostname)"
source /etc/os-release
echo "OS $NAME and kernel version is $VERSION_ID"
ip_address=$(ip a show dev enp0s3 | awk '/inet / {print $2}' | cut -d '/' -f 1)
echo "IP : $ip_address"
echo "RAM : $(df -h | awk '/^\/dev\// {printf "%s", $4}') memory available on $(df -h | awk '/^\/dev\// {printf "%s", $2}') ttotal memory"
echo "Disk : $(df -h / | awk 'NR==2 {print $4}') space left"
echo -e "Top 5 processes by RAM usage:"
ps aux --sort=-%mem | awk 'NR <= 6 && NR > 1 {print "  - "gensub(/^.+\//, "", 1, $11)}'
echo -e "Listening ports:"
ss -tulpn | awk '/^(tcp|udp)/ {print $1, $5, $7}' | while read -r type local_address program; do
  port=$(echo "$local_address" | awk -F':' '{print $NF}')
  echo "  - $port $type: $program"
done
echo "PATH directories :"
IFS=':' read -ra path_directories <<< "$PATH"

for directory in "${path_directories[@]}"; do
  echo "  - $directory"
done
image_url="https://cdn2.thecatapi.com/images/MTk0MDk3MA.jpg"
file_extension=$(curl -sI "$image_url" | grep -i '^Content-Type:' | awk -F'/' '{print $2}' | tr -d '\r')
if [ -z "$file_extension" ]; then
  echo "Failed to determine file type. Exiting."
  exit 1
fi
file_name="random_cat"
file_with_extension="$file_name.$file_extension"
curl -sL "$image_url" -o "$file_with_extension"
echo "Here is your random cat ($file_name.$file_extension): $image_url"
```

🌞 **Vous fournirez dans le compte-rendu Markdown**, en plus du fichier, **un exemple d'exécution avec une sortie**

- genre t'exécutes ton script et tu copie/colles ça dans le compte-rendu

```bash
voici le rendu dans le terminal : 

[vince@vm idcard]$ ./idcard.sh
Machine name : vm.tp5
OS Rocky Linux and kernel version is 9.2
IP : 10.5.1.2
RAM : 4.9G795M memory available on 6.2G1014M ttotal memory
Disk : 4.9G space left
Top 5 processes by RAM usage:
  - node
  - node
  - node
  - python3
  - NetworkManager
Listening ports:
  - 323 udp: 
  - 323 udp: 
  - 22 tcp: 
  - 39515 tcp: users:(("node",pid=1414,fd=19))
  - 22 tcp: 
PATH directories :
  - /home/vince/.vscode-server/bin/903b1e9d8990623e3d7da1df3d33db3e42d80eda/bin/remote-cli
  - /home/vince/.local/bin
  - /home/vince/bin
  - /usr/local/bin
  - /usr/bin
  - /usr/local/sbin
  - /usr/sbin
Here is your random cat (random_cat.jpeg): https://cdn2.thecatapi.com/images/MTk0MDk3MA.jpg
```

## Bonus

⭐ **Bonus 1 : arguments**

- si vous avez utilisé le lien que j'ai filé pour les images de chats, on est limités à 10 requêtes avant de devoir créer un compte
- créer un compte, ça donne un token
- ajoutez la possibilité de passer le token en argument

C'est à dire :

```bash
$ /srv/idcard/idcard.sh live_gxXl7Z0PjPMFVDdIM77T2c6beemFCHy8KRimDi0BGwZVOIqwCHbksr2Bva8VWUUV 
[...]
Here is your random cat (jpg file) : ./cat.jpg
```

Si aucun token est précisé, il y en a un par défaut qui est utilisé et c'est affiché dans la sortie :

```bash
$ /srv/idcard/idcard.sh  
[...]
Here is your random cat (jpg file) : ./cat.jpg (using default token)
```

⭐ **Bonus 2 : votre script ne doit PAS d'exécuter en `root`**

> Vous apprend à mieux gérer vos permissions : un script donné qui a un but donné, c'est une bonne pratique que de créer un utilisateur dédié. Ainsi, on limite les privilèges donnés au script pendant son exécution.

- vous créez un utilisateur dédié appelé `id`
- le script appartient à l'utilisateur `id`
- vous exécutez le script avec une commande `sudo -u id`
- si le script est lancé en `root`, il quitte
  - vous pouvez utiliser la commande 🐚 `exit` pour que le script s'arrête

---

⭐ **Bonus 3 : l'image du chat est téléchargée sur la machine**

> *Nécessite d'avoir fait le bonus 2. Vous apprend à gérer un téléchargement HTTP et quelques tricks shell en + pour la gestion d'ID.*

- **dans le dossier `/srv/cats/`**
  - le script quitte si ce dossier n'existe pas
- **le dossier `/srv/cats/` doit appartenir à l'utilisateur `id`**
  - le script quitte si c'est pas le cas
- **le dossier `/srv/cats/` doit être *writable* pour l'utilisateur `id`**
  - sinon il pourrait pas créer de fichiers dedans...
  - le script quitte si c'est pas le cas
- **renommer l'image avec un id**
  - l'image est renommée `1.jpg` pour la première image, puis `2.jpg` pour la deuxième, etc.
  - à chaque fois qu'on appelle le script, faut donc déterminer quel numéro est le prochain
  - gérer les IDs vaquants
    - si on télécharge 3 photos, mais qu'on supprime `2.jpg`, la prochaine photo téléchargée devra s'appeler `2.jpg`
    - on récupère l'ID vaquant `2` qui avait été libéré

---

⭐ **Bonus 4 : afficher + d'infos**

> *Vous demande d'aller creuser un peu plus et au passage apprendre un peu plus sur la façon dont l'OS fonctionne. Petit bout par petit bout.*

En plus des infos que votre script `/srv/idcard/idcard.sh` affiche déjà, ajoutez les infos suivantes :

- nombre d'inodes dispos sur la partition montée sur `/` et nombre total possible
  - afficher sous la forme :

```bash
$ /srv/idcard/idcard.sh
[...]
inodes on / partition : xxx/yyy
# où `xxx` c'est le nombre dispo et `yyy` le nombre total dans votre affichage
[...]
```

- liste des fichiers qui sont en SUID `root` sur la machine
  - afficher sous la forme :

```bash
$ /srv/idcard/idcard.sh
[...]
SUID files :
  - /path/to/suid
  - ...
[...]
```

- vérifier que la connexion en SSH en root est désactivée
  - il faut regarder le contenu du fichier de conf du serveur SSH pour ça
  - y'a une ligne qui permet d'autoriser ou non la connexion en root
  - afficher sous la forme :

```bash
$ /srv/idcard/idcard.sh
[...]
SSH root connection enabled : yes/no
# affiche seulement yes ou seulement no chez vous, en fonction de si c'est activé ou non
[...]
```



# II. Script youtube-dl

**Dans cette partie, vous allez coder un petit script qui télécharge des vidéos Youtube.** On lui file une URL d'une vidéo en argument, et il la télécharge !

Dans un deuxième temps on automatisera un peu le truc, en exécutant notre script à l'aide d'un *service*.

![emoboi](./img/emoboy.png)

## Sommaire

- [II. Script youtube-dl](#ii-script-youtube-dl)
  - [Sommaire](#sommaire)
  - [1. Premier script youtube-dl](#1-premier-script-youtube-dl)
    - [A. Le principe](#a-le-principe)
    - [B. Rendu attendu](#b-rendu-attendu)
  - [2. MAKE IT A SERVICE](#2-make-it-a-service)
    - [A. Adaptation du script](#a-adaptation-du-script)
    - [B. Le service](#b-le-service)
    - [C. Rendu](#c-rendu)
  - [3. Bonus](#3-bonus)

## 1. Premier script youtube-dl

### A. Le principe

**Un petit script qui télécharge des vidéos Youtube.** Vous l'appellerez `yt.sh`. Il sera stocké dans `/srv/yt/yt.sh`.

**Pour ça on va avoir besoin d'une commande : `youtube-dl`.** Suivez les instructions suivantes pour récupérer la commande `youtube-dl` :

```bash
# si vous êtes sur Rocky, sinon ¯\_(ツ)_/¯ 
$ curl -SL https://github.com/ytdl-org/ytdl-nightly/releases/download/2024.02.23/youtube-dl -o /usr/local/bin/youtube-dl
$ chmod +x /usr/local/bin/youtube-dl
```

Comme toujours, **PRENEZ LE TEMPS** de manipuler la commande et d'explorer un peu le `youtube-dl --help`.

Le contenu de votre script :

➜ **1. Permettre le téléchargement d'une vidéo youtube dont l'URL est passée au script**

- **la vidéo devra être téléchargée dans le dossier `/srv/yt/downloads/`**
  - tester que le dossier existe
  - vous pouvez utiliser la commande 🐚 `exit` pour que le script s'arrête
- plus précisément, **chaque téléchargement de vidéo créera un dossier**
  - `/srv/yt/downloads/<NOM_VIDEO>`
  - il vous faudra donc, avant de télécharger la vidéo, exécuter une commande pour récupérer son nom afin de créer le dossier en fonction
- la vidéo sera téléchargée dans
  - `/srv/yt/downloads/<NOM_VIDEO>/<NOM_VIDEO>.mp4`
- **la description de la vidéo sera aussi téléchargée**
  - dans `/srv/yt/downloads/<NOM_VIDEO>/description`
  - on peut récup la description avec une commande `youtube-dl`
- **la commande `youtube-dl` génère du texte dans le terminal, ce texte devra être masqué**
  - vous pouvez utiliser une redirection de flux vers `/dev/null`, c'est ce que l'on fait généralement pour se débarasser d'une sortie non-désirée

Il est possible de récupérer les arguments passés au script dans les variables `$1`, `$2`, etc.

```bash
$ cat script.sh
echo $1

$ ./script.sh toto
toto
```

➜ **2. Le script produira une sortie personnalisée**

- utilisez la commande `echo` pour écrire dans le terminal
- la sortie **DEVRA** être comme suit :

```bash
$ /srv/yt/yt.sh https://www.youtube.com/watch?v=sNx57atloH8
Video https://www.youtube.com/watch?v=sNx57atloH8 was downloaded. 
File path : /srv/yt/downloads/tomato anxiety/tomato anxiety.mp4`
```

➜ **3. A chaque vidéo téléchargée, votre script produira une ligne de log dans le fichier `/var/log/yt/download.log`**

- votre script doit s'assurer que le dossier `/var/log/yt/` existe
  - tester que le dossier existe
  - sinon quitter en appelant la commande `exit`
- la ligne doit être comme suit :

```
[yy/mm/dd hh:mm:ss] Video <URL> was downloaded. File path : <PATH>`
```

Par exemple :

```
[21/11/12 13:22:47] Video https://www.youtube.com/watch?v=sNx57atloH8 was downloaded. File path : /srv/yt/downloads/tomato anxiety/tomato anxiety.mp4`
```

> Hint : La commande `date` permet d'afficher la date et de choisir à quel format elle sera affichée. Idéal pour générer des logs. [J'ai trouvé ce lien](https://www.geeksforgeeks.org/date-command-linux-examples/), premier résultat google pour moi, y'a de bons exemples (en bas de page surtout pour le formatage de la date en sortie).

### B. Rendu attendu

📁 **Le script `/srv/yt/yt.sh`**

```bash
voici mon fichier script :

#!/bin/bash

# Vérifier si youtube-dl est installé
if ! command -v youtube-dl &> /dev/null; then
    echo "Installing youtube-dl..."
    sudo curl -L https://github.com/ytdl-org/ytdl-nightly/releases/download/2024.02.23/youtube-dl -o /usr/local/bin/youtube-dl
    sudo chmod +x /usr/local/bin/youtube-dl
fi

# Vérifier si l'URL YouTube est fournie
if [ -z "$1" ]; then
    echo "Usage: $0 <YouTube_URL>"
    exit 1
fi

# Dossier de stockage
base_folder="/srv/yt/downloads"

# Créer le dossier de base s'il n'existe pas
if [ ! -d "$base_folder" ]; then
    echo "Creating base folder: $base_folder"
    sudo mkdir -p "$base_folder"
fi

# Dossier de logs
log_folder="/var/log/yt"

# Créer le dossier de logs s'il n'existe pas
if [ ! -d "$log_folder" ]; then
    echo "Creating log folder: $log_folder"
    sudo mkdir -p "$log_folder"
fi

# Emplacement du fichier de log
log_file="$log_folder/download.log"

# Créer le fichier de log s'il n'existe pas
if [ ! -f "$log_file" ]; then
    sudo touch "$log_file"
    sudo chmod 644 "$log_file"
fi

# Exécuter youtube-dl pour récupérer le nom de la vidéo et la description
video_info=$(youtube-dl --get-title --get-description "$1" 2>/dev/null)

# Extraire le nom de la vidéo et la description
video_name=$(echo "$video_info" | head -n 1)
video_description=$(echo "$video_info" | tail -n 1)

# Créer un dossier pour la vidéo
video_folder="$base_folder/$video_name"
mkdir -p "$video_folder"

# Télécharger la vidéo dans le dossier créé
echo "[$(date +"%y/%m/%d %H:%M:%S")] Video $1 was downloaded."
echo "File path: $video_folder/$video_name.mp4"
youtube-dl --output "$video_folder/$video_name.%(ext)s" "$1" > /dev/null 2>&1

# Enregistrer la description dans un fichier
echo "$video_description" > "$video_folder/description"

# Enregistrer la ligne de log dans le fichier de log
echo "[$(date +"%y/%m/%d %H:%M:%S")] Video $1 was downloaded. File path : $video_folder/$video_name.mp4" >> "$log_file"

# Afficher le contenu du fichier de log
echo "File content:"
cat "$log_file"

echo "Video downloaded successfully. Check $video_folder for the files."

```

📁 **Le fichier de log `/var/log/yt/download.log`**, avec au moins quelques lignes

```
voici mon fichier : 
[24/03/04 09:23:28] Video https://www.youtube.com/watch?v=sNx57atloH8 was downloaded. File path : /srv/yt/downloads/tomato anxiety/tomato anxiety.mp4
[24/03/04 10:19:49] Video https://www.youtube.com/watch?v=sNx57atloH8 was downloaded. File path : /srv/yt/downloads/tomato anxiety/tomato anxiety.mp4
[24/03/04 10:22:57] Video https://www.youtube.com/watch?v=2iOs6YJW8cQ was downloaded. File path : /srv/yt/downloads/L'Amour est Mort./L'Amour est Mort..mp4
[24/03/04 10:23:46] Video https://www.youtube.com/watch?v=2iOs6YJW8cQ was downloaded. File path : /srv/yt/downloads/L'Amour est Mort./L'Amour est Mort..mp4
[24/03/04 10:24:26] Video https://www.youtube.com/watch?v=hAjtLxNAs04 was downloaded. File path : /srv/yt/downloads/50 ans de pratique du zen ? A quoi bon ?/50 ans de pratique du zen ? A quoi bon ?.mp4
```


🌞 Vous fournirez dans le compte-rendu, en plus du fichier, **un exemple d'exécution avec une sortie**

- comme pour le script précédent : juste tu le lances, et tu cme copie/colles ça dans le rendu

```bash
[vince@vm ~]$ [24/03/04 10:24:18] Video https://www.youtube.com/watch?v=hAjtLxNAs04 was downloaded.
File path: /srv/yt/downloads/50 ans de pratique du zen ? A quoi bon ?/50 ans de pratique du zen ? A quoi bon ?.mp4
File content:
[24/03/04 10:24:26] Video https://www.youtube.com/watch?v=hAjtLxNAs04 was downloaded. File path : /srv/yt/downloads/50 ans de pratique du zen ? A quoi bon ?/50 ans de pratique du zen ? A quoi bon ?.mp4
Video downloaded successfully. Check /srv/yt/downloads/50 ans de pratique du zen ? A quoi bon ? for the files.
```

## 2. MAKE IT A SERVICE

### A. Adaptation du script

YES. Yet again. **On va en faire un *service*.**

L'idée :

➜ plutôt que d'appeler la commande à la main quand on veut télécharger une vidéo, **on va créer un service qui les téléchargera pour nous**

➜ **le service s'exécute en permanence en tâche de fond**

- il surveille un fichier précis
- s'il trouve une nouvelle ligne dans le fichier, il vérifie que c'est bien une URL de vidéo youtube
  - si oui, il la télécharge, puis enlève la ligne
  - sinon, il enlève juste la ligne

➜ **qui écrit dans le fichier pour ajouter des URLs ? Bah vous !**

- vous pouvez écrire une liste d'URL, une par ligne, et le service devra les télécharger une par une

---

Pour ça, procédez par étape :

- **partez de votre script précédent** (gardez une copie propre du premier script, qui doit être livré dans le dépôt git)
  - le nouveau script s'appellera `yt-v2.sh`
- **adaptez-le pour qu'il lise les URL dans un fichier** plutôt qu'en argument sur la ligne de commande
- **faites en sorte qu'il tourne en permanence**, et vérifie le contenu du fichier toutes les X secondes
  - boucle infinie qui :
    - lit un fichier
    - effectue des actions si le fichier n'est pas vide
    - sleep pendant une durée déterminée
- **il doit marcher si on précise une vidéo par ligne**
  - il les télécharge une par une
  - et supprime les lignes une par une

### B. Le service

➜ **une fois que tout ça fonctionne, enfin, créez un service** qui lance votre script :

- créez un fichier `/etc/systemd/system/yt.service`. Il comporte :
  - une brève description
  - un `ExecStart` pour indiquer que ce service sert à lancer votre script
  - une clause `User=` pour indiquer que c'est l'utilisateur `yt` qui lance le script
    - créez l'utilisateur s'il n'existe pas
    - faites en sorte que le dossier `/srv/yt` et tout son contenu lui appartienne
    - le dossier de log doit lui appartenir aussi
    - l'utilisateur `yt` ne doit pas pouvoir se connecter sur la machine

```bash
[Unit]
Description=<Votre description>

[Service]
ExecStart=<Votre script>
User=yt

[Install]
WantedBy=multi-user.target
```

> Pour rappel, après la moindre modification dans le dossier `/etc/systemd/system/`, vous devez exécuter la commande `sudo systemctl daemon-reload` pour dire au système de lire les changements qu'on a effectué.

Vous pourrez alors interagir avec votre service à l'aide des commandes habituelles `systemctl` :

- `systemctl status yt`
- `sudo systemctl start yt`
- `sudo systemctl stop yt`

![Now witness](./img/now_witness.png)

### C. Rendu

📁 **Le script `/srv/yt/yt-v2.sh`**

📁 **Fichier `/etc/systemd/system/yt.service`**

🌞 Vous fournirez dans le compte-rendu, en plus des fichiers :

- un `systemctl status yt` quand le service est en cours de fonctionnement
- un extrait de `journalctl -xe -u yt`

> Hé oui les commandes `journalctl` fonctionnent sur votre service pour voir les logs ! Et vous devriez constater que c'est vos `echo` qui pop. En résumé, **le STDOUT de votre script, c'est devenu les logs du service !**

🌟**BONUS** : get fancy. Livrez moi un gif ou un [asciinema](https://asciinema.org/) (PS : c'est le feu asciinema) de votre service en action, où on voit les URLs de vidéos disparaître, et les fichiers apparaître dans le fichier de destination

## 3. Bonus

Quelques bonus pour améliorer le fonctionnement de votre script et votre skill sur `bash` :

➜ **en accord avec les règles de [ShellCheck](https://www.shellcheck.net/)**

- bonnes pratiques, sécurité, lisibilité

➜ **votre script a une gestion d'options :**

- `-q` (comme *quality*) pour préciser la qualité des vidéos téléchargées (on peut choisir avec `youtube-dl`)
- `-o` (comme *output*) pour préciser un dossier autre que `/srv/yt/downloads/`
- `-h` (comme *help*) affiche l'usage

➜ **si votre script utilise des commandes non-présentes à l'installation** (`youtube-dl`, `jq` éventuellement, etc.)

- vous devez TESTER leur présence et refuser l'exécution du script
- le script refuse de s'exécuter si des commandes qu'il utilise ne sont pas installée, normal quoi !

➜  **contrôle d'existence et des permissions des dossiers**

- vous devez tester leur présence, sinon refuser l'exécution du script
- ajoutez des tests de contrôles sur les dossiers :
  - vérifier que `/srv/yt/downloads/` existe c'est bien
  - vérifier en + qu'on peut écrire à l'intérieur, c'est mieux

➜ **contrôle d'URL**

- contrôlez à l'aide d'une expression régulière que les strings saisies dans le fichier sont bien des URLs de vidéos Youtube
- si c'est pas une URL de vidéo youtube valide, supprimez la ligne, et générez une ligne de log qui indique que l'URL saisie n'était pas valide

➜ ***service* + *timer***

- plutôt que faire un script avec une boucle infinie et un `sleep` dégueulasses
- enlevez complètement la boucle infinie, et faites en sorte que le script soit juste one-shot :
  - il se lance, regarde le fichier, télécharge les vidéos indiquées, et quitte
- créez un *timer* pour lancer le service à intervalles réguliers
- un *timer* permet de déclencher l'exécution d'un service à intervalles réguliers
- appelez-moi si vous faites cette partie, je vous montre comment faire ça en deux-deux

> Notre OS a déjà la capacité de lancer un truc à intervalles réguliers, pourquoi faire une boucle infinie dégueue, et avoir notre programme qui passe le plus clair de son temps à ne rien faire ?

➜  **fonction `usage`**

- le script comporte une fonction `usage`
- c'est la fonction qui est appelée lorsque l'on appelle le script avec une erreur de syntaxe
- ou lorsqu'on appelle le `-h` du script