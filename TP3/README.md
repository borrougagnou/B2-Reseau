# Mise en place du lab

Schema:

![image](https://user-images.githubusercontent.com/10796546/55687528-1da2e480-596e-11e9-925e-6d2c680a21c6.png)

comme on peut le voir, tout ce ping bien:

![image](https://user-images.githubusercontent.com/10796546/55687365-7ec9b880-596c-11e9-8e46-c6115f7d018d.png)

# Configuration des VLANs
## Configuration du trunk
Sur **IOU1**, on commence déjà a configurer le hostname puis le port **eth0/0** en mode **TRUNK**
```
IOU1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
IOU1(config)#hostn SW1
SW1(config)#
```
```
SW1(config)# int Eth 0/0
SW1(config-if)# switchport trunk encapsulation dot1q
SW1(config-if)# switchport mode trunk
```
sur **IOU2** on fait fait exactement pareil, sauf qu'il faut changer le hostname en **SW2**

## Configuration access
Sur SW1, on ajoute l'accès **VLAN 10** au port **eth1/1** et on fait la même chose sur **SW2** avec le port **1/0**:
```
SW1(config)# vlan 10
SW1(config-vlan)# name vlan_10
SW1(config-vlan)# exit
SW1(config)# int Eth1/1
SW1(config-if)# sw mode acc
SW1(config-if)# sw acc vlan 10
```

Ensuite on refait la même manipulation pour le **Client2** à la différence on mettra `vlan 20` et `int Eth2/0`

On teste, et on vois que Client1 peut ping Client3 mais pas Client2

![image](https://user-images.githubusercontent.com/10796546/55688123-79bd3700-5975-11e9-892c-834e0f5f00c0.png)

En traceroute -I on obtient cela:

![image](https://user-images.githubusercontent.com/10796546/55688282-1c29ea00-5977-11e9-8679-799e06ac1768.png)

d'ailleur traceroute -I permet d'envoyer des packet ICMP et non UDP
il envoie une "demande d'écho ICMP" avec modification de la valeur TTL et attend que la requette ICMP soit dépassée.
il peut-être utilisé lorsque les packets UDP soient bloqués

# Manipulation simple de routeurs
Schema:
![image](https://user-images.githubusercontent.com/10796546/55695271-2b7d5780-59b8-11e9-8074-3842c28870e1.png)

Le seul PC qui ne pourra pas communiquer sera **client1** puisqu'il n'a pas de passerelle

Ensuite, pour faire communiquer les 2 réseaux, il suffit d'ajouter les IPs au port ethernet, puis les routes comme ceci:
```
R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#hostn router1
router1(config)#
router1(config)#int Fa0/0
router1(config-if)#ip add 10.2.12.1 255.255.255.252
router1(config-if)#no sh
router1(config-if)#
*Mar  1 00:03:58.715: %LINK-3-UPDOWN: Interface FastEthernet0/0, changed state to up
*Mar  1 00:03:59.715: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to up
router1(config-if)#exit
router1(config)#int Fa3/0
router1(config-if)#ip add 10.1.2.254 255.255.255.0
router1(config-if)#no sh
router1(config-if)#
*Mar  1 00:04:32.331: %LINK-3-UPDOWN: Interface FastEthernet3/0, changed state to up
*Mar  1 00:04:33.331: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet3/0, changed state to up
router1(config-if)#exit
router1(config)#ip route 10.2.2.0 255.255.255.0 10.2.12.2
router1(config)#

```

on fait la même chose sur le router 2 sauf qu'on lui ajoute 2 autres IP fa2/0:`10.2.2.254` et fa0/0:`10.2.12.2`
puis une route pointant vers le routeur 1: `ip route 10.1.2.0 255.255.255.0 10.2.12.1`

sur les machines ont fait pareil, sauf qu'on ajoute une route default vers le routeur auquel on est relié, ici sur **SERVER1**:
```
# sudo ip route add default via 10.2.2.254 ens33
```

un petit ping, et ***PAF !!!*** *ça fait des chocapics !* :
![image](https://user-images.githubusercontent.com/10796546/55696781-d264f200-59be-11e9-9e41-dfee41a7ca83.png)

SERVER1 ping bien le client mais pas le réseaux entre les 2 routeurs


### Petit info supplémentaire:
les ports du routeur sont des port **Fast Ethernet**, cela veux dire qu'ils peuvent aller jusqu'à 100 Mbits comparé au port **Ethernet** qui eux vont à 10 Mbits. Sur le marché actuel, on trouve également du **Gigabit Ethernet** (1Gbits) et du **10 Gigabit Ethernet** qui peux monter jusqu'à 10 Gbits !!!

Et sur un Routeur ou Switch, cela change tout !

un port Ethernet de base, sera représenté par **Ethernet** ou **e** en abrégé

un port Fast Ethernet sera représenté par **FastEthernet** ou **fa** en abrégé

un port Gigabit Ethetnet sera représenté par **GigabitEthernet** ou **ga** en abrégé

un port 10 Gigabit Ethernet sera représenté par **tengigabitethernet** mais je ne connais pas son abrégé.

Ensuite pour finir cet étape, si je voulais que tout communique, il aurait fallut que je mette un switch relier au routeur, qui prend Client1 et Client2 comme cela:

![image](https://user-images.githubusercontent.com/10796546/55697537-173e5800-59c2-11e9-9528-ceb979a64f47.png)


# Mise en place d'OSPF
Schéma:

![image](https://user-images.githubusercontent.com/10796546/55698456-32ab6200-59c6-11e9-9f71-92d7429014c8.png)

On commence par ajouter chaque IP à chaque interface de chaque routeur seul l'interface fa3/0 est utilisé sur **R1** et **R4**:
```
R1#
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#int fa3/0
R1(config-if)#ip add 10.3.102.254 255.255.255.0
R1(config-if)#no sh
R1(config-if)#
*Mar  1 00:01:59.135: %LINK-3-UPDOWN: Interface FastEthernet3/0, changed state to up
*Mar  1 00:02:00.135: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet3/0, changed state to up
R1(config-if)#exit
R1(config)#int fa0/0
R1(config-if)#ip add 10.3.100.1 255.255.255.252
R1(config-if)#no sh
R1(config-if)#
*Mar  1 00:03:09.483: %LINK-3-UPDOWN: Interface FastEthernet0/0, changed state to up
*Mar  1 00:03:10.483: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to up
R1(config-if)#exit
R1(config)#int fa1/0
R1(config-if)#ip add 10.3.100.2 255.255.255.252
% 10.3.100.0 overlaps with FastEthernet0/0
R1(config-if)#ip add 10.3.100.22 255.255.255.252
R1(config-if)#no sh
R1(config-if)#exit
R1(config)#
*Mar  1 00:04:26.691: %LINK-3-UPDOWN: Interface FastEthernet1/0, changed state to up
*Mar  1 00:04:27.691: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet1/0, changed state to up
R1(config)#
```
On ajoute un `router-id <router>` et on relie chaque `network 10.3.100.8 0.0.0.3 area 0` au port que le routeur en question possède, comme on peut le voir, le router as pu détecter les autres routeurs qui sont relier à eux:

![image](https://user-images.githubusercontent.com/10796546/55699405-0f36e600-59cb-11e9-9dc5-c41e841f77b1.png)

une fois cela fait, on sauvegarde !!!
```
R1#copy running-config startup-config
Destination filename [startup-config]?
Building configuration...
[OK]
R1#
```

on fait un petit test ping pong directement avec router1 sur router4 (quoi de mieux que les 2 réseaux les plus éloigés pour faire des tests):

![image](https://user-images.githubusercontent.com/10796546/55713230-83d04b80-59f0-11e9-982e-c7ab1f65c3ae.png)

on vérifie également qu'ils sont ajouté:
```
R2#show ip ospf neigh

Neighbor ID     Pri   State           Dead Time   Address         Interface
3.3.3.3           1   FULL/DR         00:00:35    10.3.100.6      FastEthernet0/0
1.1.1.1           1   FULL/BDR        00:00:35    10.3.100.1      FastEthernet1/0

```

And voila !

# LabFinal

![image](https://user-images.githubusercontent.com/10796546/55314285-1e1d1600-546a-11e9-94ae-c8b39d652052.png)

## Configuration OSPF
Pour la configuration OSPF, on avais le droit d'utiliser qu'une seule area, j'ai donc choisi l'area 0,
j'ai relié les réseaux appartenant à chaque routeur avec la commande "network"
```
R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#router ospf 1
R2(config-router)#router-id 1.1.1.1
R2(config-router)#network 0.10.1.10  0.255.255.255 area 0
% OSPF: Invalid address/mask combination
R2(config-router)#network 10.1.10.0  0.255.255.255 area 0
R2(config-router)#
```

j'ai ensuite un peu attendu pour qu'il broadcast au autres routeur les routes qu'il possède

et ma table c'est tout de suite rempli
```
R2#ip route show
     10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
C       10.1.10.0/24 is directly connected, FastEthernet3/0
S       10.1.2.0/24 [1/0] via 10.10.10.6
                    [1/0] via 10.10.10.1
C       10.10.10.0/30 is directly connected, FastEthernet1/0
S       10.1.1.0/24 [1/0] via 10.10.10.6
C       10.10.10.4/30 is directly connected, FastEthernet0/0
```

## Creation des vlans

Pour commencer, sur le switch j'ai eu le droit à ce message `*Apr  1 09:40:18.406: %CDP-4-DUPLEX_MISMATCH: duplex mismatch discovered on Ethernet0/0 (not full duplex), with R3 FastEthernet3/0 (full duplex).`
il m'a suffit de dire que ce port est un port en duplex full pour corriger le truc
```
IOU1(config)#
IOU1(config)#int e0/0
IOU1(config-if)#duplex full
```


j'ai crée 2 vlans, nommé `vlan_10` et `vlan_20`puis j'ai configurer les 3 interfaces de IOU1 de manière suivante:

```
IOU1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
IOU1(config)#vlan 10
IOU1(config-vlan)#name vlan_10
IOU1(config-vlan)#exit
IOU1(config)#vlan 20
IOU1(config-vlan)#name vlan_20


# configuration
IOU1(config-vlan)#int e1/0
IOU1(config-if)#switchport mode trunk
Command rejected: An interface whose trunk encapsulation is "Auto" can not be configured to "trunk" mode.
```
pour ce message d'erreur
je n'ai eu d'autre choix que d'utiiliser l'encapsulation:
`IOU1(config-if)#switchport trunk encapsulation dot1q`

ensuite j'ai pu continuer

```
IOU1(config-if)#switchport trunk encapsulation dot1q
IOU1(config-if)#switchport mode trunk
IOU1(config-if)#
*Apr  1 09:44:52.985: %LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet1/0, changed state to down
IOU1(config-if)#
*Apr  1 09:44:55.994: %LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet1/0, changed state to up
IOU1(config-if)#switchport trunk allowed vlan 20
IOU1(config-if)#no sh
IOU1(config-if)#exit
IOU1(config-if)#
IOU1(config)#int e2/0
IOU1(config-if)#switchport trunk encapsulation dot1q
IOU1(config-if)#switchport mode trunk
*Apr  1 09:50:28.386: %LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet2/0, changed state to down
IOU1(config-if)#
*Apr  1 09:50:31.398: %LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet2/0, changed state to up
IOU1(config-if)#switchport trunk allowed vlan 10
IOU1(config-if)#exit
IOU1(config-if)#
IOU1(config)#int e2/1
IOU1(config-if)#switchport trunk encapsulation dot1q
IOU1(config-if)#switchport mode trunk
IOU1(config-if)#switchport trunk allowed vlan 10
IOU1(config-if)#
*Apr  1 09:51:07.576: %LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet2/1, changed state to down
IOU1(config-if)#
*Apr  1 09:51:10.585: %LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet2/1, changed state to up
IOU1(config-if)#no sh
IOU1(config-if)#exit
```

voila, je viens de créer un vlan sur IOU1
j'ai reproduit a peu près la même chose sur IOU2 sauf que IOU2 n'a pas de vlan20

comme on peux voir, les tests réalisé montrent que client 1 ne ping pas client 2 (et donc client3 non plus)
![image](https://user-images.githubusercontent.com/10796546/55325920-bde89d00-5486-11e9-84aa-94086a247677.png)

client3 ping bien client2
![image](https://user-images.githubusercontent.com/10796546/55325950-d48ef400-5486-11e9-86f7-940624ced6d8.png)

parce que client3 et 2 sont dans le même VLAN et pas client1


## Ajout du NAT
Pour le nat, on configure en premier celui en externe, et ensuite ceux en interne

```
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#int fa3/0
R1(config-if)#ip nat outside
*Mar  1 00:01:04.563: %LINEPROTO-5-UPDOWN: Line protocol on Interface NVI0, changed state to up
R1(config-if)#exit
R1(config)#int fa0/0
R1(config-if)#ip nat inside
R1(config-if)#no sh
R1(config-if)#exit
R1(config)#int fa1/0
R1(config-if)#ip nat inside
R1(config-if)#no sh
R1(config-if)#exit
R1(config)#ip nat inside source list 1 interface fa3/0 overload
R1(config)#access-list 1 permit any
R1(config)#
```

mais pour je ne sais quel raison,
```
R1#ping 8.8.8.8

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
```
le NAT ne répond pas à une connexion internet...
