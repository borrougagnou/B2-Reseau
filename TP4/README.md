# TP 4 Réseaux

## Menu: Infra Campus
1. [Description](#description)
2. [Documents](#documents)
	- [Détails des utilisateur et équipements](#détails-des-utilisateur-et-équipements)
	- [Schéma de la topologie](#schéma-de-la-topologie)
	- [Plan d'adressage IP](#plan-dadressage-ip)
	- [Plan des VLAN](#plan-des-vlan)
	- [Matériels supplémentaires](#matériels-supplémentaires)
3. [Maquette GNS3](#maquette-gns3)
4. [Information Supplémentaires](#information-supplémentaires)

## Description:
Mise en Place d'une infrastructure réseaux à l'échelle d'un campus étudiants:
#### Locaux
-   3 bâtiments
    -   2 étages, 5 salles/étages
    -   minimum 30 clients par salle

####  Clients du réseau
-   ~200 étudiants
    -   en moyenne, ils ont 1,5 équipements
        -   tous ont un PC
        -   certains utilisent smartphone/tablette avec la wifi
-   ~50 profs
    -   en moyenne, ils ont 1,5 équipements
        -   tous ont un PC
        -   certains utilisent smartphone/tablette avec la wifi
-   ~20 serveurs
-   15 caméras
-   2 admins

## Documents:

### Détails des utilisateur et équipements
![image](https://raw.githubusercontent.com/borrougagnou/B2-Reseau/master/TP4/plan/plan_adressage.PNG)
Source: ![Excel logo](https://icon-icons.com/icons2/1156/PNG/32/1486565571-microsoft-office-excel_81549.png "Excel logo")[Excel Online](https://auvencecom-my.sharepoint.com/:x:/g/personal/boris_rougagnou_ynov_com/EVJisEHuij9PhnuAtfTp8AcBm2xnpknZKRhpwgXTqD71Pw?e=NpB8fq)

### Schéma de la topologie
PDF: https://github.com/borrougagnou/B2-Reseau/blob/master/TP4/plan/TP4-2-batiment.pdf

Source: ![Visio logo](https://icon-icons.com/icons2/1156/PNG/32/1486565580-microsoft-office-ms-visio_81554.png "Visio logo")[Visio Online](https://auvencecom-my.sharepoint.com/:x:/g/personal/boris_rougagnou_ynov_com/EVJisEHuij9PhnuAtfTp8AcBm2xnpknZKRhpwgXTqD71Pw?e=NpB8fq)

### Plan d'adressage IP
PDF: https://github.com/borrougagnou/B2-Reseau/blob/master/TP4/plan/TP4-2-ip.pdf

Source: ![Visio logo](https://icon-icons.com/icons2/1156/PNG/32/1486565580-microsoft-office-ms-visio_81554.png "Visio logo")[Visio Online](https://auvencecom-my.sharepoint.com/:u:/g/personal/boris_rougagnou_ynov_com/EQLM_L9bcMZBvBKLz-xysqQBHS7EeT_E4G46CW6BuZLDrA?e=IhFPns)

### Plan des VLAN
Source: ![Visio logo](https://icon-icons.com/icons2/1156/PNG/32/1486565580-microsoft-office-ms-visio_81554.png "Visio logo")[Visio Online](https://auvencecom-my.sharepoint.com/:u:/g/personal/boris_rougagnou_ynov_com/ESpNieUurVhDuXon-_Eq7KoBGbIqMOrDiZ1-qMulP6Pf4A?e=lr1Pjt)

### Matériels supplémentaires:
- 16 Switchs
- 1 ou 4 Routeur(s)

## Maquette GNS3
![image](https://raw.githubusercontent.com/borrougagnou/B2-Reseau/master/TP4/gns3_plan/plan.png)

## Information Supplémentaires
- Le réseaux caméra est un réseaux totalement déconnecté
- Petite hésitation entre le plan Visio, et le plan GNS3 concernant les routeurs, faire de la Haute Disponibilité ou non
