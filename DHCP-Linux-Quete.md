Résumé suite à cette quête que j’ai réussie :

# Sur le Debian qui sert de serveur DHCP :

## Il y a 3 fichiers clés à modifier :

## ⇒ Le fichier etc/dhcp/dhcpd.conf

En haut de l'impression d'écran, c’est pour configurer l’adresse du réseau et la plage.
En bas c’est pour réserver une adresse IP et la lier à une adresse MAC :

<br><p align="center"><img width="70%" src="https://github.com/user-attachments/assets/29a6589e-80b9-4b0c-88a7-405d120ae3b0" alt=""></p>

<br><p align="center"><img width="70%" src="https://github.com/user-attachments/assets/6df242aa-f98d-426b-a89b-ad683c9a577c" alt=""></p><br><br><br>

## ⇒ le fichier etc/network/interface

<br><p align="center"><img width="70%" src="https://github.com/user-attachments/assets/55f25f3a-81be-4e34-b79e-9dd799d3b10d" alt=""></p><br><br><br>

Permet d’activer une adresse IP statique et un masque pour le serveur DHCP (on supprime “DHCP” écrit par défaut et on ajoute les infos comme ci-dessus)

## Le fichier etc/default/isc-dhcp-server

<br><p align="center"><img width="70%" src="https://github.com/user-attachments/assets/f25d38c7-e696-4332-be8d-bf1d91a3cc73" alt=""></p><br><br><br>

On précise juste l’interface réseau de la machine

## Commandes utilisées

### `systemctl restart isc-dhcp-server` : permet de redémarrer le service DHCP après une modification de fichier

### `systemctl status isc-dhcp-server` : permet d’afficher l’état du service DHCP

### `dhcpd -t -cf /ect/dhcp/dhcpd.conf` : ne fonctionnait pas sur mon Debian (dhcpd introuvable) mais commande qui permet de tester la validité du fichier dhcpd.conf

### `ip a` : donne les infos sur les interfaces réseau et leurs adresse IP

# Sur le Ubuntu qui sert de client :

### ⇒ Installer le paquet isc-dhcp-client avec la commande `sudo apt install isc-dhcp-client`

Pour obtenir la connexion internet qui permet d’installer le paquet, on peut choisir NAT dans les paramètres réseau de la VM et choisir une IP automatique.

⇒ Une fois le paquet installé, configurer une adresse IP statique, par exemple :

<br><p align="center"><img width="70%" src="https://github.com/user-attachments/assets/2a3520f5-a305-4a76-97f5-431fa64aa59b" alt=""></p><br><br><br>

Et régler le mode d’accès réseau (dans les options de la VM Ubuntu) sur le même paramétrage que le serveur DHCP (ici en Réseau Interne). Enfin on lance la commande ping vers le serveur DHCP pour vérifier qu’il répond bien :

<br><p align="center"><img width="70%" src="https://github.com/user-attachments/assets/582cc72b-2ec0-4b1f-a8f8-a97464311f14" alt=""></p><br><br><br>

⇒ A présent, dans les options réseau d’Ubuntu configurer l’interface réseau en DHCP :

<br><p align="center"><img width="70%" src="https://github.com/user-attachments/assets/30f49fd2-765e-4f98-b477-3f0daa9b9372" alt=""></p><br><br><br>

⇒ Lorsqu’on a configuré une plage d’adresses IP sur le serveur Debian, comme pour Windows on utilise des commandes pour libérer l’adresse IP et demander une nouvelle adresse sur le PC client. Ces commandes sont :

`sudo dhclient -r enp0s3` : pour libérer l’adresse IP en cours (enp0s3 est le nom de l’interface réseau sur cette machine)

`sudo dhclient enp0s3` :  pour demander une nouvelle adresse (du coup c’est le serveur Debian qui la donne)

Si la machine Ubuntu communique bien avec le serveur Debian DHCP, il se voit attribuer une adresse IP située dans la plage d’adresse configurée dans le Debian. Dans notre cas  c’est la première adresse de la plage, soit 172.10.0.5 :

<br><p align="center"><img width="70%" src="https://github.com/user-attachments/assets/fd38b0a1-19af-41cb-bd22-b4facc7622ac" alt=""></p><br><br><br>

Dans la suite du challenge, lorsqu’on a rentré l’adresse MAC de l’interface réseau du Ubuntu sur le fichier `dhcpd.conf` du Debian, on refait la même procédure pour appliquer les nouvelles règles (libérer IP / demander IP) 

On a bien l’adresse IPv4 liée à l’adresse MAC définie sur le serveur DHCP (soit  `172.10.0.22`) :

<br><p align="center"><img width="70%" src="https://github.com/user-attachments/assets/80b67148-010a-4c05-a8c2-b13007b1ad46" alt=""></p><br><br><br>

## A noter

Pour supprimer le lien avec l’adresse MAC le fait de commenter ce paramètre dans le fichier etc/dhcp/dhcpd.conf n’a pas suffit :

<br><p align="center"><img width="70%" src="https://github.com/user-attachments/assets/af078907-fd6b-4ae7-a7af-ca5349c42faf" alt=""></p><br><br><br>

En effet, même si ce paramètre a été désactivé sur le serveur Debian, la machine Ubuntu gardait systématiquement l’adresse 172.10.0.22, que ce soit :

- en libérant l’adresse IP et en redemandant une adresse au serveur DHCP
- et même en remettant une IP fixe, puis en rebasculant sur une IP DHCP après redémarrage de la machine.

Pour vraiment supprimer le lien avec l’adresse MAC, j’ai du définir une nouvelle plage d’adresses IP sur le serveur DHCP pour le forcer à attribuer une nouvelle adresse IPv4 à la machine Ubuntu.
Cela a marché, et ensuite j’ai modifié une nouvelle fois la plage d’adresse sur le serveur DHCP pour la remettre telle qu’elle était initialement (`172.10.0.5` → `172.10.0.30`), et dans ce cas l’adresse IPv4 attribuée à la machine Ubuntu était la première de la plage (soit  `172.10.0.5`)

