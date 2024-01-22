# TP1 : Casser avant de construire



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




🌞 **Another way ?**

- sans toucher aux mots de passe, faites en sorte qu'aucun utilisateur ne soit utilisable
- trouver un autre moyen donc, en restant sur les utilisateurs !

## 4. Disques

🌞 **Effacer le contenu du disque dur**




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


## 6. You own way

🌞 **Trouvez 4 autres façons de détuire la machine**

- tout doit être fait depuis le terminal de la machine
- pensez à ce qui constitue un ordi/un OS
- l'idée c'est de supprimer des trucs importants, modifier le comportement de trucs existants, surcharger tel ou tel truc...
