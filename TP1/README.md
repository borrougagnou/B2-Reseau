
  

tableau:

  
|         |    NET1    |    NET2    |
|---------|:----------:|:----------:|
| Client1 | 10.10.1.10 | 10.10.2.10 |
| Client2 | 10.10.1.30 |      X     |
  

  

# I. Exploration du réseau d'une machine CentOS

  

Combien y a t'il d'adresse /24:

>254

  

Combien y a t'il d'adresse /30: Question de sécurité et de clarté, un masque /30 indique immédiatement seulement 2 ip simultanées sur le sous réseaux
>2

  

Pourquoi un /30 ?

>Parce que c'est une protection supplémentaire pour sécuriser un ordinateur dans un réseaux
>
>Il sert également pour une communication en Pair-à-Pair

  

Definition IP statique:

>une IP statique est une IP qui ne change jamais

  

  

l'erreur 301 c'est une redirection
```
➜ ~ curl google.fr

<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">

<TITLE>301 Moved</TITLE></HEAD><BODY>

<H1>301 Moved</H1>

The document has moved

<A HREF="[http://www.google.fr/](http://www.google.fr/)">here</A>.

</BODY></HTML>

```

  
```
➜ ~ ping 10.10.1.10

PING 10.10.1.10 (10.10.1.10) 56(84) bytes of data.

64 bytes from 10.10.1.10: icmp_seq=1 ttl=64 time=0.022 ms

64 bytes from 10.10.1.10: icmp_seq=2 ttl=64 time=0.088 ms

```

  
```
➜ ~ ping 10.10.2.10

PING 10.10.2.10 (10.10.2.10) 56(84) bytes of data.

64 bytes from 10.10.2.10: icmp_seq=1 ttl=64 time=0.021 ms

64 bytes from 10.10.2.10: icmp_seq=2 ttl=64 time=0.086 ms
```
  

  

// REMPLACER 10.10.1.10 et 10.10.2.10 en 10.1.1.1 et 10.1.2.1

  

  

afficher les routes que connaît votre machine **+ expliquer chacune des lignes**
```
➜ ~ ip route show

10.10.1.0/24 dev ens33 proto kernel scope link src 10.10.1.10 metric 100

10.10.2.0/24 dev ens37 proto kernel scope link src 10.10.2.10 metric 101
```
  

![https://cdn.discordapp.com/attachments/452443077467176971/545642171966029824/wireshark_capture.png](https://cdn.discordapp.com/attachments/452443077467176971/545642171966029824/wireshark_capture.png)

### SUPPRESSION DE ROUTE

```
➜ ~ ip route show

10.10.1.0/24 dev ens33 proto kernel scope link src 10.10.1.10 metric 100

10.10.2.0/24 dev ens37 proto kernel scope link src 10.10.2.10 metric 101

➜ ~ sudo ip route del 10.10.2.0

RTNETLINK answers: No such process

➜ ~ sudo ip route del 10.10.2.0/24

➜ ~ ip route show

10.10.1.0/24 dev ens33 proto kernel scope link src 10.10.1.10 metric 100

➜ ~ ping 10.10.2.1

connect: Le réseau n'est pas accessible
```
  

### AJOUT DE ROUTE
```
➜ ~ sudo ip route add 10.10.2.0/24 via 10.10.2.10 dev ens37

➜ ~ ping 10.10.2.1

PING 10.10.2.1 (10.10.2.1) 56(84) bytes of data.

64 bytes from 10.10.2.1: icmp_seq=1 ttl=64 time=0.093 ms

64 bytes from 10.10.2.1: icmp_seq=1 ttl=64 time=0.093 ms

64 bytes from 10.10.2.1: icmp_seq=1 ttl=64 time=0.093 ms

64 bytes from 10.10.2.1: icmp_seq=1 ttl=64 time=0.093 ms

^C

--- 10.10.2.1 ping statistics ---

4 packets transmitted, 4 received, 0% packet loss, time 0ms

rtt min/avg/max/mdev = 0.093/0.093/0.093/0.000 ms

➜ ~ ip route show

10.10.1.0/24 dev ens33 proto kernel scope link src 10.10.1.10 metric 100

10.10.2.0/24 via 10.10.2.10 dev ens37
```
  

  

### ON AFFICHE LA TABLE ARP
```
➜ ~ ip neigh show

10.10.2.1 dev ens37 lladdr 00:50:56:c0:00:02 STALE

10.10.1.1 dev ens33 lladdr 00:50:56:c0:00:03 DELAY
```
  

// ON VIDE LA TABLE ARP
```
➜ ~ sudo ip neigh flush all

  

➜ ~ ip neigh show

10.10.1.1 dev ens33 lladdr 00:50:56:c0:00:03 REACHABLE

  

**reachable** the neighbour entry is valid until the reachability timeout expires.
```
  

il en affiche qu'une parce que l'autre est utilisé par ma connexion SSH
```
➜ ~ sudo ss -tnp4

State Recv-Q Send-Q Local Address:Port Peer Address:Port

ESTAB 0 0 10.10.1.10:22 10.10.1.1:55578
```


  

### CAPTURE RÉSEAU

  
```
➜ ~ sudo tcpdump -i ens33 -w capture.pcap

tcpdump: listening on ens33, link-type EN10MB (Ethernet), capture size 262144 bytes

^C10 packets captured

10 packets received by filter

0 packets dropped by kernel
```
  

  

Explication trame PING REQUEST (en gros détail):

![https://cdn.discordapp.com/attachments/452443077467176971/545653545865707573/ping.png](https://cdn.discordapp.com/attachments/452443077467176971/545653545865707573/ping.png)

  

  

# II. Communication simple entre deux machines

  
```
➜ ~ ping -c 4 10.10.1.10

PING 10.10.1.10 (10.10.1.10) 56(84) bytes of data.

64 bytes from 10.10.1.10: icmp_seq=1 ttl=64 time=0.475 ms

64 bytes from 10.10.1.10: icmp_seq=2 ttl=64 time=0.892 ms

64 bytes from 10.10.1.10: icmp_seq=3 ttl=64 time=0.578 ms

64 bytes from 10.10.1.10: icmp_seq=4 ttl=64 time=0.822 ms

  

--- 10.10.1.10 ping statistics ---

4 packets transmitted, 4 received, 0% packet loss, time 3002ms

rtt min/avg/max/mdev = 0.475/0.691/0.892/0.174 ms
```
  

  

Ajout du Client2 : 10.10.1.30

![https://cdn.discordapp.com/attachments/452443077467176971/545661632815169545/unknown.png](https://cdn.discordapp.com/attachments/452443077467176971/545661632815169545/unknown.png)

  

Ensuite il fallait ouvrir le port 8888 en UDP puisque par défaut il n'était pas ouvert

  

En UDP, les messages sont tous envoyé les uns à la suite des autres :

![https://cdn.discordapp.com/attachments/452443077467176971/549550905213583380/Capture_decran_2019-02-25_12-17-33.png](https://cdn.discordapp.com/attachments/452443077467176971/549550905213583380/Capture_decran_2019-02-25_12-17-33.png)

  

en TCP on vois bien la communication entre les 2 ainsi que le mot "Bonjour" qui a été envoyé, et que chaque message envoyé possède un "accusé de réception":

![https://cdn.discordapp.com/attachments/452443077467176971/549537730779021324/Capture_decran_2019-02-25_11-24-44.png](https://cdn.discordapp.com/attachments/452443077467176971/549537730779021324/Capture_decran_2019-02-25_11-24-44.png)

  

  

Après avoir fermé uniquement le Firewall côté serveur, on s’aperçoit qu'il lui envoi une requete ICMP de refus de connexion:

![https://cdn.discordapp.com/attachments/452443077467176971/549551622087245844/Capture_decran_2019-02-25_12-21-01.png](https://cdn.discordapp.com/attachments/452443077467176971/549551622087245844/Capture_decran_2019-02-25_12-21-01.png)

  

Après fermeture des ports du Firewall sur le TCP des 2 machines, on se retrouve avec un message "Cannot assign requested address. QUITTING" et aucun message sur Wireshark apparait

  

  

# III. Routage statique simple

  

activation de l'ip forwarding sur client 01
```
->sudo echo 1 > /proc/sys/net/apv4/ip_forward
```
  

définition d'un route statique définitive dans client 1

  
```
-> sudo echo "10.2.0.0/24 via 10.2.0.254 dev ifcfg-ens34" > /etc/sysconfig/network-scripts/route-ens34
```
  

actualisation de la configuration des interfaces
-> reboot

  

vérification de la route: depuis client 2
```
-> ping 10.1.2.1

PING 10.1.2.1 (10.1.2.1) 56(84) bytes od data.

64 bytes from 10.1.2.1: icmp_seq=1 ttl=128 time=1.00 ms

64 bytes from 10.1.2.1: icmp_seq=1 ttl=128 time=0.619 ms

64 bytes from 10.1.2.1: icmp_seq=1 ttl=128 time= 0.535 ms

¨C

---10.1.2.1 ping statistics ---

3 packets transmitted, 3 received, 0% packet loss, time 2002ms

rtt min/avg/max/mdev =0.535/0.718/1.000/0.202ms
```
  

  

Vérification: depuis client2
```
-> traceroute 10.1.2.1

  

traceroute to 10.1.2.1 (10.1.2.1), 30 hops max, 60byte packets

1 gateway from 192.168.19.2) 0.270ms 0.134ms 0.069

2 * * *

3 * * *

4 * * *

5 * * *

6 * * *

7 * * *

8 * * *

9 * * *

10 * * *

11 * * *

12 * * *

13 * * *

14 * * *

15 * * *

16 * * *

17 * * *

18 * * *

19 * * *

20 * * *

21 * * *

22 * * *

23 * * *

24 * * *

25 * * *

26 * * *

27 * * *

28 * * *

29 * * *

30 * * *

-->
```
