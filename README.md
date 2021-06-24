# Epitech - PXE Serveur

![Image de l'écran des menus du PXE](https://media.discordapp.net/attachments/769912532109754368/857567841380204574/unknown.png?width=1191&height=661)
## C'est quoi un PXE ?
Le PXE *(Preboot Execution Environment)* permet à l'utilisateur de pouvoir se connecter à l'aide d'une prise RJ45 au serveur PXE, par la suite lors du démarrage d'un ordinateur, il sufira d'appuillez sur F12 pour pouvoir acceder à l'interface du projet. Sur cette interface, l'utilisateur pourra choisir l'OS qu'il souhaite pour pouvoir l'installer sur son ordinateur sans avoir a télécharger d'ISO et de créer des clées USB Bootable.

## Liste des Distributions disponible :
```
   PXE
	├── Linux 
	│ 	└── Ubuntu 20.04 LTS - Focal Fossa!
	├── Debian
	│ 	├── Debian 10 Buster - Stable
	│	└── Debian 10 Buster - Testing
	├── Fedora
	│ 	├── Fedora 32 + Epitech Dump
	│ 	├── Fedora 32
	│	└── Fedora 34
	└── Windows
		└── Windows 10
```

# Configuration
## Comment ajouter une distribution au serveur PXE ?

Pour ce qui est de windows, vous allez devoir prendre l'ISO d'un installer de Windows classique, le monter et copier tout le contenue de l'ISO dans le serveur samba *(/dev/samba/<new_version>/)*

Concernant les systèmes UNIX vous devrez aller télécharger la version NetBoot de la distribution désirée. Une fois cela fait, vous l'ajoutez dans les dossiers de votre serveur WEB (*/var/www/http/*). Ensuite vous devrez aller ajouter la distribution dans la configuration du serveur PXE, elle se situe dans le serveur WEB aussi (/var/www/html/default_bootmenu.ipxe)

Vous ajoutez tout dabords un block qui chargera le kernel et l'initrd : 
```ipxe
:distribution_name
kernel http://<ip_du_serveur_web>/chemin/vers/le/kernel initrd=initrd_name
initrd http://<ip_du_serveur_web>/chemin/vers/le/initrd
```

Une fois que vous avez définis le block qui chargera la distribution, vous allez devoir rajouter une entrée dans le menu pour pouvoir la séléctionner . Pour ce faire dans le block `main_menu` de la configuration : 

```ipxe
item <block_name> Ma nouvelle distribution
```

Pour windows, la tâche se complique, vous pouvez ajouter un racourci de l'OS rajouté sur Samba mais pour cela il faudra modifier le boot.wim de Malekal, le plus simple est d'utiliser l'explorateur de fichier de Windows PE, et directement aller dans le dossier de Samba pour vous rendre dans le fichier que vous avez ajouter et lancé a la main le setup.exe !

Et c'est tout, une fois le fichier de configuration sauvegarder vous pouvez redémarrer un ordinateur sur le serveur PXE et votre nouvelle distribution sera là !

# Partie Technique

Ce projet peut supporter les système UEFI & BIOS. pour ce faire nous utilisons iPXE. Il y a deux version de la ROM d'iPXE, une pour les UEFI, et une pour les BIOS, mais eux deux chargerons le même script *(/home/tftp/script.ipxe)* pour l'affichage du menu. Il n'y aura donc pas deux configuration à gérer.
Des qu'un ordinateur tente de boot sur le résaux, DNSMASQ prend le relais *(/etc/dnsmasq.conf)*, et attribut une adresse IPV4 à la machine. Une fois que la machine a une IPV4, on lui envoie la ROM d'iPXE via le protocole TFTP, qui chargera elle même par le principe de "chainloading" le script qui est situé dans le serveur WEB.
Pour ce qui est des système UNIX on chargera tout les fichiers par le biais du serveur WEB, pour ce qui est de windows, on chargera dans un premier temps Malekal, une version de WindowsPE un peu custom pour apporter plus de facilité au niveau des pilotes. En démarrant cette version de WindowsPE, elle chargera en arrière plan la liaison avec le serveur Samba afficher sur le Bureau une icon "Windows 10 Installer", cette dernière lancera l'instalation de Windows 10.

### Listes des chemin d'accès :
Configuration de DNSMASQ : /etc/dnsmasq.conf
Interface de séléction par défaut : /var/www/html/default_bootmenu.ipxe
Dossier TFTP : /home/tftp
Samba : /srv/samba
Serveur WEB : /var/www/http/

