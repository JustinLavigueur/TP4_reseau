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

## 1. Création des 2 VCN et des deux instances Ubuntu

### 1.1 — Création des VCN
- **VCN A** : CIDR `10.0.0.0/16`
- **VCN B** : CIDR `10.1.0.0/16`
- Chaque VCN contient un sous-réseau public :
  - `10.0.1.0/24` pour le VCN A  
  - `10.1.1.0/24` pour le VCN B  
- On ajoute une **Internet Gateway** et une **table de routage** par défaut à chaque VCN.

![Image Création VCN sur Oracle Cloud](imagesTP4/oracleVCN.png)

- 2. Créer deux instance avec Ubuntu 22.04



