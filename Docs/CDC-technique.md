# Projet : Robot intelligent d’assistance pour EHPAD

## Robot : ROSMASTER M3 Pro — ROS2 — Jetson Nano / Orin NX

---

# 1. Vision

 Un robot autonome basé sur ROS2 capable d’assister les employers (infirmiers et aides-soignants) en EHPAD en livrant le matériel(gants,serviettes, etc, ...) et les plateaux-repas aux patients, afin d’optimiser le temps du personnel et améliorer la qualité des soins.

---

# 2. Objectifs et périmètre

## Objectifs

1. Permettre la livraison autonome des repas et du matériel
2. Permettre à l’infirmier de commander le robot via une interface web (tablette ou telephone mobile)
3. Permettre une communication temps réel pour le suivi du robot (Batterie faible, collisions avec patients ou personnel)
4. Suivi de la position du robot en temps réel et de sa navigation pour conaitre sa progression dans sa tâche
---
## Objectifs secondaires

1. Sortie des poubelles (fonctionnalité optionnelle)
---

## Non-objectifs

1. Remplacer le personnel soignant
2. Transporter des charges supérieures à 5kg
3. Utilisation en extérieur

---

## Personas

### Persona 1 — Personnel soignant

Nom : Marie  
Age : 34 ans

Objectifs :

- Livrer le materiel manquant rapidement
- Réduire déplacements

Utilise :

- Interface web pour commander robot

---

### Persona 2 — Personnel EHPAD

Nom : Paul  
Age : 45 ans

Objectifs :

- Livrer repas efficacement
- Surveiller robot

Utilise :

- Interface web pour commander et surveiller robot

---

# 3. Use Cases

## Tableau des Use Cases

|ID|Nom|Acteur|Description|
|---|---|---|---|
|UC-01|Livrer materiel|Infirmier|Robot livre materiel|
|UC-02|Livrer repas|Personnel|Robot livre repas|
|UC-03|Navigation autonome|Robot|Robot se déplace seul|
|UC-04|Éviter obstacles|Robot|Robot évite obstacles|
|UC-05|Retour base|Robot|Robot retourne base|
|UC-06|Sortie poubelle|Personnel|Robot transporte déchets|

---

## UC-01 : Livrer materiel (détaillé)

Acteur : Infirmier

Préconditions :

- Robot connecté

- Carte navigation chargée


Scénario nominal :

1. Infirmier sélectionne chambre

2. Interface envoie commande

3. Serveur MQTT transmet commande

4. Robot navigue

5. Robot livre materiel

6. Robot envoie confirmation


Postconditions :

- Livraison terminée


---

## UC-02 : Livrer repas (détaillé)

Processus identique au UC-01.

---

# 4. Architecture technique

Architecture globale :

```
Infirmier
   │
   ▼
Interface Web
   │ WebSocket
   ▼
Serveur MQTT
   │ MQTT
   ▼
Robot ROSMASTER M3 Pro
   │
   ├── Navigation (ROS2 Nav2)
   ├── Vision (OpenCV)
   ├── Lidar Dual ToF
   └── Capteurs sécurité
```

---

# 5. Diagrammes UML :


![[Pasted image 20260217150427.png]]

![[Pasted image 20260217152148.png]]

---

# 6. Stack technique

|Composant|Technologie|Justification|
|---|---|---|
|Robot OS|ROS2|Standard robotique|
|Robot compute|Jetson Nano / Orin NX|IA embarquée|
|Langage|Python|Compatible ROS2|
|Vision|OpenCV|Vision ordinateur|
|Communication|MQTT|Communication robot fiable|
|Communication|WebSocket|Communication temps réel|
|Interface|React|Interface moderne|
|Navigation|Nav2|Navigation autonome|

---

## Alternatives écartées

| Technologie | Raison                 |
| ----------- | ---------------------- |
| Socket.io   | trop lourd             |
| Arduino     | puissance insuffisante |

---

# 7. Risques et contraintes

## Risques

| Risque              | Probabilité | Impact   | Mitigation           |
| ------------------- | ----------- | -------- | -------------------- |
| Collision           | Moyen       | Critique | Lidar + Vision       |
| Perte connexion     | Moyen       | Moyen    | Reconnexion MQTT     |
| Bug ROS2            | Faible      | Moyen    | Tests                |
| Erreur de livraison | Moyen       | Critique | Révisions régulières |

---

## Contraintes

Hardware :

- ROSMASTER M3 Pro obligatoire


Software :

- ROS2 obligatoire


Environnement :

- Utilisation intérieure uniquement


---

# 8. Sécurité

Mesures :

- Détection obstacles

- Arrêt urgence

- Surveillance capteurs

- Vision OpenCV

---

# 9. Conventions équipe

Git :

Branches :

```
feature/navigation
feature/mqtt
feature/interface
```

Commits :

```
[FEAT]:
[FIX]:
[DOCS]:
```
- Conflits : rebase sur main, resolution en binome
- Review obligatoire avant merge

---

# 10. Architecture ROS2

Nodes :

```
navigation_node
delivery_node
vision_node
obstacle_avoidance_node
mqtt_node
interface_node
safety_node
trash_node
```

---

# 11. Roadmap

|Phase|Objectif|Dates|
|---|---|---|
|Phase 1|Installation ROS2|23/02/2026 - 08/03/2026|
|Phase 2|Navigation autonome|09/03/2026 - 22/03/2026|
|Phase 3|Communication MQTT|23/03/2026 - 05/04/2026|
|Phase 4|Interface|06/04/2026 - 19/04/2026|
|Phase 5|Livraison|20/04/2026 - 03/05/2026|
|Phase 6|Sortie poubelles|04/05/2026 - 17/05/2026|

---

# 12. Questions ouvertes

-  Faut-il ajouter une reconnaissance via QR code ?
-  Faut-il ajouter l'IA détection personnes ?
-  Faut-il créer une flotte dans l'idéal ?

---

# 13. Résultat attendu

Robot capable de :

- recevoir les commandes données par le personnel soignant(infirmier,aides-soignants)

- déplacements autonomes

- éviter les obstacles

- livrer le matériel

- livrer les repas

- communiquer temps réel (suivi du robot)


