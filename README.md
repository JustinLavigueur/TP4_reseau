# TP4 – Réseau
**Cours : Réseau**  
**Travail pratique 4 : Routage inter-VCN Oracle Cloud, configuration d’un serveur DHCP et évaluation de la performance d’un réseau**

---

# === Configuration du routage inter-VCN ===

## **BUT**
Créer plusieurs VCN dans Oracle Cloud avec des blocs CIDR **qui ne se chevauchent pas**.  
Pour ce faire, il faut consulter la section *« Sommaire des composants de réseau pour l’appairage au moyen d’une passerelle DRG »*.

---

# 1. Création des 2 VCN et des deux instances Ubuntu 22.04

## 1.1 — Création des VCN
- **vcn1** : CIDR `10.0.0.0/16`  
- **vcn2** : CIDR `10.1.0.0/16`

![Création des VCN](imagesTP4/lesvcn.png)

Chaque VCN contient un sous-réseau public :
- `10.0.0.0/24` pour **vcn1**

![VCN 1](imagesTP4/vcn1.png)

- `10.1.0.0/24` pour **vcn2**

![VCN 2](imagesTP4/vcn2.png)

Une **Internet Gateway** et une **table de routage par défaut** sont ajoutées à chaque VCN.

---

## 1.2 — Création des instances Ubuntu 22.04

On crée **une instance Ubuntu 22.04 par VCN**.

### Étapes :
1. Connexion à Oracle Cloud

![Connexion oracle cloud](imagesTP4/connexionOracle.png)

2. Menu principal → Accueil → Compute → Instances

![allerSurInstance](imagesTP4/allerSurInstance.png)

3. Cliquer sur **Créer une instance**
4. Donner un nom à l’instance (ex. *Ubuntu-VCN1*) et choisir le compartiment

![nomInstance](imagesTP4/nomInstance.png)

5. Sélectionner l’image **Ubuntu 22.04 LTS**

![imageUbuntu](imagesTP4/imageUbuntu.png)
![22.04](imagesTP4/22.04.png)

6. Dans l’étape réseau, sélectionner le VCN existant

![vNIcMax](imagesTP4/vNIcMax.png)

7. Ajouter sa clé SSH publique  
   Si vous n’en avez pas, télécharger la paire de clés directement.

![cleSSH](imagesTP4/cleSSH.png)

8. Vérifier les règles réseau (NSG/Security List pour SSH)
9. Cliquer sur **Créer l’instance** et attendre le statut **Running**

![running](imagesTP4/running.png)

10. Connexion à l’instance avec SSH

![Connexion oracle cloud](imagesTP4/connexionSSH.png)

11. Répéter pour la deuxième instance

![Les instances](imagesTP4/lesinstances.png)

---

### Instances créées
- **instance-TP3A** : attachée à **vcn1**

![Instance A](imagesTP4/instance-a-vcn.png)

- **instance-TP3B** : attachée à **vcn2**

![Instance B](imagesTP4/instance-b-vcn.png)

---

# 2. Création de la passerelle DRG

1. Aller dans **Networking → Dynamic Routing Gateways**
2. Cliquer sur **Create DRG**
3. Nommer la passerelle : `DRG-TP4`

![Création DRG](imagesTP4/drg.png)

---

# 3. Attachement des VCN à la DRG

Dans **Attachments**, créer une attache pour :
- **vcn1**
- **vcn2**

![Attachment VCN A](imagesTP4/attachement-drg-vcna.png)
![Attachment VCN B](imagesTP4/attachement-drg-vcnb.png)

---

# 4. Configuration des tables de routage

## 4.1 — VCN A
Route à ajouter :
- **Destination CIDR** : `10.1.0.0/16`
- **Target** : **DRG**

![Routage Instance A](imagesTP4/routage-instance-a.png)

## 4.2 — VCN B
Route à ajouter :
- **Destination CIDR** : `10.0.0.0/16`
- **Target** : **DRG**

![Routage Instance B](imagesTP4/routage-instance-b.png)

---

# 5. Mise à jour des règles de sécurité

## 5.1 — Subnet A  
Autoriser :
- **Type** : ICMP (ping)
- **Source CIDR** : `10.1.0.0/24`

![Ingress A](imagesTP4/ingress-rules-instance-a.png)

## 5.2 — Subnet B  
Autoriser :
- **Type** : ICMP (ping)
- **Source CIDR** : `10.0.0.0/24`

![Ingress B](imagesTP4/ingress-rules-instance-b.png)

---

# 6. Preuve de ping

## Vers l’instance TP3_A
![PingVersA](imagesTP4/pingversa.png)

## Vers l’instance TP3_B
![PingVersB](imagesTP4/pingversb.png)

---

# === Évaluation de la performance réseau ===

# 7. Tests de performance avec iPerf3

## 7.1 — Installation de iPerf3

Sur **les deux instances** :

```bash
sudo apt update
sudo apt install iperf3 -y
```

---

## 1. Réinitialisation du pare-feu (iptables)

![iptables reset](imagesTP4/iperf3-iptables.png)

---

## 2. Règles de sécurité OCI

### Instance A (10.0.0.88)
![Security List A](imagesTP4/ingress-rules-instance-a.png)

### Instance B (10.1.0.96)
![Security List B](imagesTP4/ingress-rules-instance-b.png)

---

## 3. Lancement du serveur iPerf3 (Instance B)

```bash
iperf3 -s -B 10.1.0.96
```

![iperf server](imagesTP4/iperf3-instance-b.png)

---

## 4. Lancement du client iPerf3 (Instance A)

```bash
iperf3 -c 10.1.0.96
```

![iperf client](imagesTP4/iperf3-instance-a.png)

---

## 5. Résultats

| Instance   | Rôle     | Débit observé |
|------------|----------|----------------|
| Instance A | Client   | ~502 Mbits/s   |
| Instance B | Serveur  | ~497 Mbits/s   |

---

# === Mise en place d’un serveur DHCP ===

# 1. Serveur DHCP (Instance B – 10.1.0.96)

## Installation

```bash
sudo apt install isc-dhcp-server -y
```

![Installation DHCP Server](imagesTP4/installation-DHCP-Server.png)

---

## Fichier `/etc/default/isc-dhcp-server`

```conf
INTERFACESv4="ens3"
INTERFACESv6=""
```

![isc-dhcp-server](imagesTP4/isc-dhcp-server.png)

---

## Fichier `/etc/dhcp/dhcpd.conf`

![dhcpd.conf](imagesTP4/dhcpd.png)

---

## Statut du service DHCP

![Statut DHCP Server](imagesTP4/statut-server-dhcp.png)

---

## Règles de pare-feu DHCP

![Security List DHCP](imagesTP4/ingress-rules-instance-b.png)

---

# 2. DHCP Relay (Instance A – 10.0.0.49)

## Installation

```bash
sudo apt install isc-dhcp-relay -y
```

![Installation relay](imagesTP4/installation-relay-client.png)

---

## Fichier `/etc/default/isc-dhcp-relay`

![Config relay](imagesTP4/isc-dhcp-relay.png)

---

## Statut du relais

![Statut relay](imagesTP4/statut-relay.png)

---
