
# I. Mise en place du lab

## 2. Routage Statique
![https://media.discordapp.net/attachments/452443077467176971/549600613814632448/unknown.png](https://media.discordapp.net/attachments/452443077467176971/549600613814632448/unknown.png)

Il est impossible de ping de `router1` vers `serveur1` puisque `server1` ne connais pas la route 10.2.12.0

## 3. Visualisation du routage avec Wireshark

Ce qui change entre les 2 captures Wireshark, c'est l'adresse MAC:

via NET12: ![https://cdn.discordapp.com/attachments/452443077467176971/549621086535548928/unknown.png](https://cdn.discordapp.com/attachments/452443077467176971/549621086535548928/unknown.png)

via NET2 : ![https://cdn.discordapp.com/attachments/452443077467176971/549621646940700692/unknown.png](https://cdn.discordapp.com/attachments/452443077467176971/549621646940700692/unknown.png)


# II. NAT et services d'infra

## 1. Mise en place du NAT

Pour faire fonctionner le NAT sur `client1` via `router1`, je devais
- ajouter une route **default**
- ajouter un DNS vers 8.8.8.8 dans le fichier `/etc/resolv.conf`

vu que je n'ai pas configuré de route **default** sur `Server1`, je n'ai pas de ping vers internet, même si j'ajoutais un DNS.

```
CLIENT1➜  ~ curl google.com
curl: (6) Could not resolve host: google.com; Erreur inconnue
CLIENT1➜  ~ sudo vim /etc/resolv.conf
CLIENT1➜  ~ sudo ip route add default via 10.2.1.254 dev ens37
CLIENT1➜  ~ curl google.com                                   
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```

> **EDIT:** une mises à jours du sujet nous demande d'avoir `Server1` fonctionnel sur le NAT, j'ai donc ajouté une route et ajouté un DNS.


## 2. DHCP server

![https://cdn.discordapp.com/attachments/452443077467176971/554275895477796875/unknown.png](https://cdn.discordapp.com/attachments/452443077467176971/554275895477796875/unknown.png)
L'adresse IP commence à `10.2.1.50` puisque dans le fichier **dhcpd.conf**, on a explicitement dit que les IPs dynamique vont de `10.2.1.50` à `10.2.1.70`;
le serveur dhcp va délivrer un bail DHCP à l’ordinateur qui en fait la demande (ici avec la commande dhclient).
dans la configuration actuel, le bail est de 600 secondes par défaut, et peut monter jusqu'à 7200 secondes max.


## 3. NTP server

Pour ma part, j'utilise majoritairement les serveurs de debian pour la synchronisation NTP:
> 0.debian.pool.ntp.org
>
> 1.debian.pool.ntp.org
>
> 2.debian.pool.ntp.org
>
> 3.debian.pool.ntp.org

que je met en iburst, *iburst* qui signifie qu'en cas d'indisponibilité, il essayera plusieurs fois avant d'abandonner.

Après ajout des fichiers de configuration "clients" + **ouverture des ports 123 en UDP**, je me retrouve avec ce schéma la:
Server1:
```
SERVER1➜  /etc chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^? router1.tp2.b2                3   6     1     7  +6232us[+6232us] +/-   58ms

SERVER1➜  /etc chronyc tracking
Reference ID    : 0A0201FE (router1.tp2.b2)
Stratum         : 4
Ref time (UTC)  : Sun Mar 10 17:45:52 2019
System time     : 0.000000000 seconds slow of NTP time
Last offset     : +0.005812709 seconds
RMS offset      : 0.005812709 seconds
Frequency       : 3.218 ppm slow
Residual freq   : -0.000 ppm
Skew            : 207.711 ppm
Root delay      : 0.068785526 seconds
Root dispersion : 0.029253975 seconds
Update interval : 0.0 seconds
Leap status     : Normal
```

Router2:
```
ROUTER2➜  /etc chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* router1.tp2.b2                    4   6     7     9    -56us[ -804us] +/-   61ms
```

Mon `Router2` et mon `Server1` sont maintenant synchronisé avec l'horloge du `Router1`


## 4. Web server

Vu que nous avons **ouvert le port 80 TCP**
et que nous avons vérifié que NGINX est en mode "**Ecoute**"
```
SERVER1➜  /etc sudo ss -altnp4 
State       Recv-Q Send-Q       Local Address:Port                      Peer Address:Port              
LISTEN      0      128                      *:80                                   *:*                   users:(("nginx",pid=9801,fd=6),("nginx",pid=9800,fd=6))
LISTEN      0      128                      *:22                                   *:*                   users:(("sshd",pid=6926,fd=3))
LISTEN      0      100              127.0.0.1:25                                   *:*                   users:(("master",pid=7032,fd=13))
```

on peut alors tester si on peux acceder au site web avec les autres ordinateurs du réseau

via `nc`, `wget`, `curl`:
```
CLIENT2➜  ~ curl 10.2.2.10
<html>
<body>
<h1>Ma page NGINX personnalisé</h1>
<span>Lorem Ipsum Dolor Sit Amet</span>
</body>
</html>
```

Ou même via un navigateur web :

![https://cdn.discordapp.com/attachments/452443077467176971/554367694020018178/unknown.png](https://cdn.discordapp.com/attachments/452443077467176971/554367694020018178/unknown.png)
