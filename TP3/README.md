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
