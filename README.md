# TP4 – Réseau
**Cours : Réseau**  
**Travail pratique 4 : Routage inter-VCN oracle cloud,
configuration d'un Serveur DHCP et évaluation de
la performance d'un réseau**

---

# === Configuration du routage inter-VCN ===

## BUT :
Créer plusieurs VCN's dans Oracle Cloud ayant des blocs CIDR ne se
chevauchent pas. À cette fin, allez lire la section `Sommaire des composants de
réseau pour l'appairage au moyen d'une passerelle DRG` à ce lien.

## 1. Création des 2 VCN et des deux instances avec Ubuntu 22.04

### 1.1 — Création des VCN
- **vcn1** : CIDR `10.0.0.0/16`
- **vcn2** : CIDR `10.1.0.0/16`

![Création des VCN](imagesTP4/lesvcn.png)

Chaque VCN contient un sous-réseau public :
  - `10.0.0.0/24` pour le vcn1

Photo qui montre le sous-réseau du vcn1:
![VCN 1](imagesTP4/vcn1.png)
  - `10.1.0.0/24` pour le vcn2

Photo qui montre le sous-réseau du vcn2:
![VCN 2](imagesTP4/vcn2.png)
    
- On ajoute une **Internet Gateway** et une **table de routage** par défaut à chaque VCN.




### 1.2 — Création des instances Ubuntu 22.04
- Créer **une instance Ubuntu 22.04** dans chaque VCN.

**Petite erreur avant d'analyser:** Mon collègue a nommé les instances communément (instance-TP3A) et (instance-TP3B), alors qu'on est sur le travail pratique 4. C'est juste un problème de nommage, svp ne pas nous faire perdre des points là-dessus.      
- **instance-TP3A** : rattachée au vcn1.
![Instance A](imagesTP4/instance-a-vcn.png)
  
- **Instance-TP3B** : rattachée au vcn2.  
![Instance B](imagesTP4/instance-b-vcn.png)

- Ouvrir les ports suivants :
  - **ICMP (Ping)** pour les tests de communication

![Les instances](imagesTP4/lesintances.png)

## Étape 2 — Création de la passerelle DRG

1. On va dans **Networking → Dynamic Routing Gateways**
2. On clique sur **Create DRG**
3. On nomme la passerelle : `allo`

![Image Création VCN sur Oracle Cloud](imagesTP4/oracleVCN.png)
