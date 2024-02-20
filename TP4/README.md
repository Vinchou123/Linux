# TP4 : Real services

Dans ce TP4, on va s'approcher de plus en plus vers de la gestion de serveur, commen on le fait dans le monde réel.

Le but de ce TP :

➜ **monter un serveur de stockage** VM `storage.tp4.linux`

- le serveur de stockage possède une partition dédiée
- sur cette partition, plusieurs dossiers sont créés
- chaque dossier contient un site web
- ces dossiers sont partagés à travers le réseau pour rendre leur contenu disponible à notre serveur web

➜ **monter un serveur web** VM `web.tp4.linux`

- il accueillera deux sites web
- il ne sera (malheureusement) pas publié sur internet : c'est juste une VM
- les sites web sont stockés sur le serveur de stockage, le serveur web y accède à travers le réseau

---

➜ Plutôt que de monter des petits services de test, ou analyser les services déjà existants sur la machine, on va donc passer à l'étape supérieure et **monter des trucs vraiment utilisés dans le monde réel** 

Rien de sorcier cela dit, et **à la fin vous aurez appris à monter un petit serveur Web.** Ce serait exactement la même chose si vous voulez publier un site web, et que vous voulez gérer vous-mêmes le serveur Web.

**Le serveur de stockage** c'est pour rendre le truc un peu plus fun (oui j'ai osé dire *fun*), et voir un service de plus, qui est utilisé dans le monde réel. En plus, il est parfaitement adapté pour pratiquer et s'exercer sur le partitionnement de façon pertinente.

➜ On aura besoin de deux VMs dans ce TP : 🖥️ **VM `web.tp4.linux`** et 🖥️ **VM `storage.tp4.linux`**.

> Pour une meilleure lisibilité, j'ai éclaté le TP en 3 parties.

# Partie 1 : Partitionnement du serveur de stockage

> Cette partie est à réaliser sur 🖥️ **VM storage.tp4.linux**.

On va ajouter un disque dur à la VM, puis le partitionner, afin de créer un espace dédié qui accueillera nos sites web.

➜ **Ajouter deux disques durs de 2G à la VM**

- cela se fait via l'interface graphique de virtualbox
- il faut éteindre la VM pour ce faire

> [**Référez-vous au mémo LVM pour réaliser le reste de cette partie.**](../../../cours/memo/lvm.md)

**Le partitionnement est obligatoire pour que le disque soit utilisable.** Ici on va rester simple : une seule partition, qui prend toute la place offerte par le disque.

Comme vu en cours, le partitionnement dans les systèmes GNU/Linux s'effectue généralement à l'aide de *LVM*.

**Allons !**


🌞 **Partitionner le disque à l'aide de LVM**

- créer 2 *physical volumes (PV)* :
  - un PV ajouté pour chacun des disques
  - chaque PV fait donc 2G

  ```golang
  [vince@vm2 ~]$ sudo pvdisplay
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB5ad3db88-792a8c5c PVID NAAaUiQQdJVUNoa80HboRnaJlqi9Sh22 last seen on /dev/sda2 not found.
  "/dev/sdb" is a new physical volume of "2.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb
  VG Name
  PV Size               2.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               im3asG-6Qg9-O1Nf-eIot-2rTk-NMr1-RXtGM9

  "/dev/sdc" is a new physical volume of "2.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdc
  VG Name
  PV Size               2.00 GiB
  Allocatable           NO
  PE Size
  ```
- créer un nouveau *volume group (VG)*
  - il devra s'appeler `storage`
  - il doit contenir les deux PVs créés à l'étape précédente

  ```golang
  [vince@vm2 ~]$ sudo vgdisplay
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB5ad3db88-792a8c5c PVID NAAaUiQQdJVUNoa80HboRnaJlqi9Sh22 last seen on /dev/sda2 not found.
  --- Volume group ---
  VG Name               storage
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               3.99 GiB
  PE Size               4.00 MiB
  Total PE              1022
  Alloc PE / Size       0 / 0
  Free  PE / Size       1022 / 3.99 GiB
  VG UUID               FvMj9x-CQfT-NRo0-u8OG-SSJz-CLTx-mJl45s
  ```

- créer un nouveau *logical volume (LV)* : ce sera la partition utilisable
  - elle doit être dans le VG `storage`
  - elle doit occuper tout l'espace libre

```
[vince@vm2 ~]$ sudo lvdisplay
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB5ad3db88-792a8c5c PVID NAAaUiQQdJVUNoa80HboRnaJlqi9Sh22 last seen on /dev/sda2 not found.
  --- Logical volume ---
  LV Path                /dev/storage/storage
  LV Name                storage
  VG Name                storage
  LV UUID                hH7KsP-BmUB-whcs-80HS-cG3D-dowO-CsUTm9
  LV Write Access        read/write
  LV Creation host, time vm2.tp4.b1, 2024-02-20 09:28:00 +0100
  LV Status              available
  # open                 0
  LV Size                3.99 GiB
  Current LE             1022
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2
  ```


🌞 **Formater la partition**

- vous formaterez la partition en ext4 (avec une commande `mkfs`)
  - le chemin de la partition, vous pouvez le visualiser avec la commande `lvdisplay`
```golang
  [vince@vm2 ~]$ mkfs -t ext4 /dev/storage/storage
mke2fs 1.46.5 (30-Dec-2021)
mkfs.ext4: Permission denied while trying to determine filesystem size
[vince@vm2 ~]$ sudo !!
sudo mkfs -t ext4 /dev/storage/storage
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 1046528 4k blocks and 261632 inodes
Filesystem UUID: 85610353-103f-4031-8320-078a49aff167
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
``` 

  - pour rappel un *Logical Volume (LVM)* **C'EST** une partition

🌞 **Monter la partition**

- montage de la partition (avec la commande `mount`)
  - la partition doit être montée dans le dossier `/storage`

  - preuve avec une commande `df -h` que la partition est bien montée
```golang
  [vince@vm2 ~]$ df -h
Filesystem                   Size  Used Avail Use% Mounted on
devtmpfs                     4.0M     0  4.0M   0% /dev
tmpfs                        386M     0  386M   0% /dev/shm
tmpfs                        155M  3.7M  151M   3% /run
/dev/mapper/rl-root          6.2G  1.2G  5.0G  20% /
/dev/sda1                   1014M  220M  795M  22% /boot
tmpfs                         78M     0   78M   0% /run/user/1000
/dev/mapper/storage-storage  3.9G   24K  3.7G   1% /mnt/storage
```

  - utilisez un `| grep` pour isoler les lignes intéressantes

```golang
[vince@vm2 ~]$ df -h | grep storage
/dev/mapper/storage-storage  3.9G   24K  3.7G   1% /mnt/storage
```
  - prouvez que vous pouvez lire et écrire des données sur cette partition

  ```golang
[vince@vm2 ~]$ cat /mnt/storage/fichier_vince
coucou
```
- définir un montage automatique de la partition (fichier `/etc/fstab`)
  - vous vérifierez que votre fichier `/etc/fstab` fonctionne correctement

```golang
  [vince@vm2 ~]$ sudo umount /mnt/storage
[vince@vm2 ~]$ sudo mount -av
/                        : ignored
/boot                    : already mounted
none                     : ignored
mount: /mnt/storage does not contain SELinux labels.
       You just mounted a file system that supports labels which does not
       contain labels, onto an SELinux box. It is likely that confined
       applications will generate AVC messages and not be allowed access to
       this file system.  For more details see restorecon(8) and mount(8).
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
/mnt/storage             : successfully mounted
```

Ok ! Za, z'est fait. On a un espace de stockage dédié pour stocker nos sites web.

⭐**BONUS**

- utilisez une commande `dd` pour remplir complètement la nouvelle partition
- prouvez que la partition est remplie avec une commande `df`
- ajoutez un nouveau disque dur de 2G à la machine
- ajoutez ce new disque dur à la conf LVM
- agrandissez la partition pleine à l'aide du nouveau disque
- prouvez aavec un `df` que la partition a bien été agrandie

**Passons à [la partie 2 : installation du serveur de partage de fichiers](./../part2/README.md).**