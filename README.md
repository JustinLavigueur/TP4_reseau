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
On crée **une instance Ubuntu 22.04** dans chaque VCN.

#### Étapes de création d'une instance Ubuntu 22.04
- 1. On se connecte à notre compte Oracle Cloud (compte déjà créé dans mon cas)
  
![Connexion oracle cloud](imagesTP4/connexionOracle.png)
- 2. On accède au menu principal et on regarde sur accueuil → Compute → Instances.
  
![allerSurInstance](imagesTP4/allerSurInstance.png)
- 3. On clique sur Créer une instance.
  
- 4. On renseigne le nom de l’instance (ex. Ubuntu-VCN1) et on sélectionne le compartiment où sera créée l'instance.

![nomInstance](imagesTP4/nomInstance.png)
- 5. On choisit l’image Ubuntu 22.04 LTS dans la section Image et forme.

![imageUbuntu](imagesTP4/imageUbuntu.png)
![22.04](imagesTP4/22.04.png)
- 6. À l'étape 3 de service du réseau, on sélectionne (si c'est votre cas, un réseau en nuage virtuel existant)

![vNIcMax](imagesTP4/vNIcMax.png)
- 7. On ajoute sa clé SSH publique pour pouvoir se connecter à l’instance. Si vous n'avez pas de clé SSH en main, juste à cliquer sur **télécharger la clé privée** et **télécharger la clé publique**

![cleSSH](imagesTP4/cleSSH.png)
- 8. On vérifie les configurations réseau : IP publique si nécessaire et règles du NSG ou Security List pour autoriser SSH.

- 9. On clique sur Créer l’instance et on attend que son statut devienne Running.

![running](imagesTP4/running.png)
- 10. On se connecte à l’instance depuis son terminal avec ssh:

![Connexion oracle cloud](imagesTP4/connexionSSH.png)

- 11. On répète les mêmes étapes pour la création de la deuxième instance.
![Les instances](imagesTP4/lesintances.png)


---

**Petite erreur avant d'analyser:** Mon collègue a nommé les instances communément (instance-TP3A) et (instance-TP3B), alors qu'on est sur le travail pratique 4. C'est juste un problème de nommage, svp ne pas nous faire perdre des points là-dessus.      
- **instance-TP3A** : rattachée au vcn1.
![Instance A](imagesTP4/instance-a-vcn.png)
  
- **Instance-TP3B** : rattachée au vcn2.  
![Instance B](imagesTP4/instance-b-vcn.png)

- On ouvre les ports suivants :
  - **ICMP (Ping)** pour les tests de communication
  - **22 (SSH)** pour la connexion


## Étape 2 — Création de la passerelle DRG

1. On va dans **Networking → Dynamic Routing Gateways**
2. On clique sur **Create DRG**
3. On nomme la passerelle : `DRG-TP4`

Voici la preuve:
![Création DRG](imagesTP4/drg.png)

## Étape 3 — Attachement des VCN créés prédécemment à la passerelle DRG

1. Dans la section **Attachments**, on crée une attache pour :
   - `vcn1`
   - `vcn2`
2. On vérifie que les deux VCN apparaissent dans la liste des attachements de la DRG.

![Attachment VCN A](imagesTP4/attachement-drg-vcna.png)
![Attachment VCN B](imagesTP4/attachement-drg-vcnb.png)

## Étape 4 — Configuration des tables de routage

### 4.1 — Dans le VCN A
- On ajoute une route dans la table :
  - **Destination CIDR** : `10.1.0.0/16`
  - **Target Type** : DRG

![Routage Instance A](imagesTP4/routage-instance-a.png)

### 4.2 — Dans le VCN B
- On ajoute une route :
  - **Destination CIDR** : `10.0.0.0/16`
  - **Target Type** : DRG

![Routage Instance B](imagesTP4/routage-instance-b.png)

---

## Étape 5 — Mise à jour des règles de sécurité

### 5.1 — Dans le vcn1
- On autorise :
  - **Type** : ICMP (Ping)
  - **Source CIDR** : `10.1.0.0/16`

![Ingress A](imagesTP4/ingress-rules-instance-a.png)
![Egress A](imagesTP4/egress-rules-instance-a.png)

### 5.2 — Dans le vcn2
- On autorise :
  - **Type** : ICMP (Ping)
  - **Source CIDR** : `10.0.0.0/16`

![Ingress B](imagesTP4/ingress-rules-instance-b.png)
![Egress B](imagesTP4/egress-rules-instance-b.png)

---

# === Évaluation de la performance réseau ===

## Étape 6 — Évaluation de la performance réseau

On pourra procéder à un test de performance à l’aide de l’utilitaire **iperf3**.  
Pour installer et utiliser iperf3, on suit les étapes ci-dessous.

### 6.1 — Installation de iperf3
On installe iperf3 sur **les deux instances** :

- `sudo apt update`
- `sudo apt install iperf3 -y`

### 6.2 — Lancement de iperf3 en mode serveur (sur l’instance du vcn1)

- `iperf3 -s`

Cette action permet le démarrage de iperf3 en mode serveur.
L’instance TP3_A attend les connexions de test provenant de l’autre VCN.

### 6.3 — Lancement de iperf3 en mode client (sur l’instance du vcn2)
Sur l’instance du vcn2 (instance-TP3B), on lance iperf3 en mode client en visant l’adresse privée de l’instance A :

- `iperf3 -c <IP_privée_instance_VCN1>`

### 6.4 — Résultats du test

iperf3 affiche automatiquement :

- la bande passante (Mbits/sec),
- la quantité de données transmises,
- la durée du test,
- un résumé global de la performance.

Ce test permet de confirmer :

- que le routage inter-VCN fonctionne,
- que les deux instances peuvent communiquer sans restriction,
- et d’évaluer la performance réseau fournie par Oracle Cloud.

# === Mise en place un serveur DHCP ===

