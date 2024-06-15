# configuration-routage-dynamique

 les étapes et commandes spécifiques nécessaires :

### 1. Découpage du réseau et configuration des interfaces

#### 1.1. Déterminer les sous-réseaux pour l'interconnexion des routeurs

Nous avons un réseau initial 200.170.13.0/24 et nous devons créer des sous-réseaux avec un masque /30 pour les interconnexions point à point. Chaque sous-réseau fournira 2 adresses utilisables.

Les sous-réseaux seront :
- **200.170.13.0/30** : IP utilisables 200.170.13.1, 200.170.13.2
- **200.170.13.4/30** : IP utilisables 200.170.13.5, 200.170.13.6
- **200.170.13.8/30** : IP utilisables 200.170.13.9, 200.170.13.10

#### 1.2. Remplir les tables de configuration des interfaces

##### Routeur DNTS1

```plaintext
interface Serial0/0/0
 ip address 200.170.13.1 255.255.255.252
 clock rate 64000
 no shutdown

interface Serial0/0/1
 ip address 200.170.13.5 255.255.255.252
 clock rate 128000
 no shutdown

interface Ethernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
```

##### Routeur DNTS2

```plaintext
interface Serial0/0/0
 ip address 200.170.13.6 255.255.255.252
 no shutdown

interface Serial0/0/1
 ip address 200.170.13.9 255.255.255.252
 no shutdown

interface Ethernet0/0
 ip address 192.168.12.1 255.255.255.0
 no shutdown
```

##### Routeur DNTS3

```plaintext
interface Serial0/0/0
 ip address 200.170.13.10 255.255.255.252
 no shutdown

interface Serial0/0/1
 ip address 200.170.13.2 255.255.255.252
 clock rate 64000
 no shutdown

interface Ethernet0/0
 ip address 192.168.11.1 255.255.255.0
 no shutdown
```

### 2. Configurer le routage dynamique avec RIP

Configurez RIP sur chaque routeur :

##### Routeur DNTS1

```plaintext
router rip
 version 2
 network 192.168.10.0
 network 200.170.13.0
 no auto-summary
```

##### Routeur DNTS2

```plaintext
router rip
 version 2
 network 192.168.12.0
 network 200.170.13.4
 network 200.170.13.8
 no auto-summary
```

##### Routeur DNTS3

```plaintext
router rip
 version 2
 network 192.168.11.0
 network 200.170.13.0
 network 200.170.13.8
 no auto-summary
```

### 3. Test de connectivité et chemin emprunté

1. **Traceroute entre PC lan10 (192.168.10.2) et PC lan11 (192.168.11.2) :**
   - Commande : `tracert 192.168.11.2`

2. **Relever le TTL d'un ping continu entre les deux PCs :**
   - Commande : `ping -t 192.168.11.2`

3. **Débrancher le câble entre DNTS3 et DNTS1**

4. **Chronométrer le temps de rétablissement du ping :**
   - Utiliser un chronomètre pour mesurer le temps nécessaire pour que le ping recommence à répondre.

### 4. Remplacer RIP par OSPF

Configurez OSPF sur chaque routeur, en tenant compte des bandes passantes indiquées :

##### Routeur DNTS1

```plaintext
router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 200.170.13.0 0.0.0.3 area 0
 network 200.170.13.4 0.0.0.3 area 0
```

##### Routeur DNTS2

```plaintext
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
 network 200.170.13.4 0.0.0.3 area 0
 network 200.170.13.8 0.0.0.3 area 0
```

##### Routeur DNTS3

```plaintext
router ospf 1
 network 192.168.11.0 0.0.0.255 area 0
 network 200.170.13.0 0.0.0.3 area 0
 network 200.170.13.8 0.0.0.3 area 0
```

### 5. Test avec OSPF

Répétez les tests de connectivité avec `tracert` et `ping -t` après avoir configuré OSPF, puis mesurez de nouveau le temps de récupération après déconnexion.

### Notes finales

 sauvegarder la configuration sur chaque routeur après chaque modification avec la commande `write memory` ou `copy running-config startup-config`.
