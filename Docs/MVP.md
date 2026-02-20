# MVP

## Objectif :
 - Créer un robot de soutien logistique pour le personnel soignant des EHPAD
 - Libérer 2-3 heures par jour du personnel soignant en automatisant la livraison de matériel et repas
 - Améliorer la réactivité et la qualité des soins en réduisant les déplacements répétitifs

## Problème :
 - **Aide-soignant** : Manque de matériel (gants, serviettes, pansements, etc.) au moment d'une intervention
 - **Personnel soignant** : Passe 30% de son temps à chercher/livrer du matériel au lieu d'interagir avec les patients
 - **Personnel cuisine** : Repas qui refroidissent en route, double vérification manuelle de la livraison nécessaire
 - **Management** : Impossibilité de tracer les livraisons et optimiser les opérations

## Notre MVP en 5 points :

1. **Navigation autonome** (évitement des obstacles, cartographie)
   - Robot capable de se déplacer seul dans l'EHPAD sans intervention humaine
   - Utilisation du Lidar pour la cartographie et l'évitement d'obstacles en temps réel
   - Localisation par SLAM (Simultaneous Localization And Mapping)
   - Vitesse limitée à 0.5 m/s (0.3 m/s avec repas) pour respecter la sécurité

2. **Interface web simple de commande** (tablette/mobile)
   - Interface accessible sur tablette et téléphone (PWA React)
   - Authentification du personnel soignant
   - Sélection simple du type de livraison (matériel, repas) et chambre destination
   - Feedback immédiat de l'état (acceptée, en cours, livrée)

3. **Livraison de matériel et repas**
   - Robot transporte matériel léger (< 5kg : gants, serviettes, pansements, etc.)
   - Robot transporte plateaux-repas avec maintien thermique
   - Confirmation de livraison (capteur de charge, interaction utilisateur)

4. **Communication temps réel (MQTT + WebSocket)**
   - Suivi en direct de la position du robot et de la batterie
   - Notification immédiate d'événements (batterie faible, obstacles, livraision complète)
   - Latence < 1 seconde pour les commandes
   - Logs détaillés de toutes les operations

5. **Sécurité et arrêt d'urgence**
   - Détection automatique des obstacles (personnes, objets)
   - Arrêt immédiat en cas de détection de personne
   - Bouton d'arrêt d'urgence physique et numérique
   - Respect de la norme ISO 3691-4 (véhicules autonomes intérieur)
   - Batterie critique avec retour automatique à la base

## Critères de réussites :

1. **Fiabilité de la navigation**
   - Livraison réussit dans 95% des cas nominaux
   - Zéro collision avec les personnes ou les obstacles
   - Temps total < 3 minutes (de la commande à la livraison)
   - Robot capable de revenir à sa base de charge autonomement

2. **Disponibilité et performance**
   - Interface web répond en < 2 secondes
   - Commandes traitées en < 1 seconde
   - Robot opérationnel 8h par jour sans intervention
   - Batterie > 30% avant chaque mission

3. **Acceptation utilisateur**
   - 80% du personnel soignant utilise l'interface sans formation supplémentaire
   - Gain observable de 2+ heures par jour pour le personnel
   - Satisfaction utilisateur > 8/10
   - Interface accessible avec gants (gros boutons, actions simples)

## Contraintes :

1. **Matérielles**
   - Robot ROSMASTER M3 Pro : capacité de charge max 5kg
   - Batterie autonomie 4-6 heures (dépend du type de tâche)
   - Environnement intérieur uniquement (corridors EHPAD)
   - Vitesse max 0.5 m/s (sécurité)

2. **Logicielles**
   - ROS2 comme système d'exploitation robot
   - Infrastructure réseau WiFi stable dans l'EHPAD
   - Interface PWA (Progressive Web App) basée React
   - Communication MQTT pour la fiabilité

3. **Réglementaires**
   - Respect norme ISO 3691-4
   - Arrêt immédiat si personne détectée
   - Traçabilité complète des livraisons
   - Conformité RGPD pour les données logs

4. **Organisationnelles**
   - Acceptation du personnel (formation brève)
   - Espace de charge disponible en base
   - Maintenance quotidienne minimale

## Risques + plan B :

1. **Risque : Perte de connexion réseau WiFi**
   - Probabilité : Moyen | Impact : Moyen
   - **Plan B :** Robot passe en mode autonome local, retour automatique à la base
   - Reconnexion automatique avec recu des tâches en queue

2. **Risque : Obstacle imprévu non franchissable**
   - Probabilité : Moyen | Impact : Moyen
   - **Plan B :** Robot s'arrête et notifie le personnel, suggestion de route alternative
   - Personnel peut déplacer l'obstacle ou annuler la commande

3. **Risque : Collision avec patient ou personnel**
   - Probabilité : Faible | Impact : Critique
   - **Plan B :** Lidar + vision pour détection des personnes, arrêt immédiat
   - Bouton d'arrêt d'urgence physique et numérique
   - Tests de sécurité avant déploiement en zone patient

4. **Risque : Batterie insuffisante en cours de mission**
   - Probabilité : Moyen | Impact : Moyen
   - **Plan B :** Alerte à 20%, retour automatique à la base
   - Batterie rechargée en 30-45 minutes

5. **Risque : Non-acceptation du personnel**
   - Probabilité : Moyen | Impact : Moyen
   - **Plan B :** Interface très simple, formation courte, amélioration continue basée sur feedback
   - Itérations rapides avec le personnel pour améliorer l'UX

6. **Risque : Repas refroidi après livraison**
   - Probabilité : Faible | Impact : Moyen
   - **Plan B :** Compartiment isolé thermiquement, réduction vitesse (0.3 m/s), routes optimisées
   - Capteur de température pour alerte si dégradation excessive

## Objectifs primaires du MVP

1. **Permettre la livraison autonome des repas et du matériel**
   - Le robot doit naviguer automatiquement jusqu'à la chambre désignée
   - Livrer le matériel ou le plateau-repas sans intervention humaine
   - Confirmer la livraison via l'interface
   - Capacité à gérer plusieurs types de livraisons (matériel léger, repas chauds)

2. **Permettre au personnel soignant de commander le robot via une interface web**
   - Interface accessible sur tablette et téléphone mobile
   - Sélection simple de la chambre destination via QR code ou liste
   - Types de livraison sélectionnables (matériel, repas, autres)
   - Accès limité au personnel soignant authentifié
   - Feedback immédiat de la commande acceptée (< 2 secondes)

3. **Permettre une communication temps réel pour le suivi du robot**
   - Suivi de la batterie (alerte si < 20%)
   - Détection et signalement des obstacles avec patients ou personnel
   - Notification de l'état de livraison (en cours, terminée, erreur)
   - Localisation temps réel du robot dans l'EHPAD
   - Alertes de problèmes (obstacle non franchissable, erreur navigation)

4. **Assurer la sécurité et la fiabilité**
   - Respect de la norme ISO 3691-4 (véhicules autonomes en environnement intérieur)
   - Arrêt d'urgence déclenchable manuellement (bouton) ou automatiquement
   - Détection des obstacles et arrêt immédiat
   - Vitesse limitée à 0.5 m/s maximum en zone patients (0.3 m/s avec repas chauds)
   - Taux de succès de livraison ≥ 95%

5. **Garantir la traçabilité et la conformité**
   - Logs détaillés de toutes les livraisons (timestamp, chambre, type, durée)
   - Conformité RGPD pour les données collectées
   - Maintenance facile avec mode diagnostic

---

## Objectifs secondaires (Phase 2+)

1. **Reconnaissance automatique des destinations**
   - Reconnaissance des numéros de chambre par QR code
   - Reconnaissance optique de caractères (OCR) sur plaques chambre
   - Navigation vers destination sans intervention humaine

2. **Optimisations de la navigation et évitement**
   - Détection des personnes avec IA (OpenCV/TensorFlow)
   - Évitement prédictif des obstacles mobiles
   - Optimisation dynamique des routes en cas de congestion

3. **Gestion de flotte multi-robots**
   - Coordination de plusieurs robots
   - Répartition intelligente des livraisons
   - Planification centralisée des tâches

4. **Commande vocale optionnelle**
   - Activation par commande vocale simple
   - Reconnaissance de destinations courantes

---

## Non-objectifs (hors scope MVP)

1. **Remplacer le personnel soignant** — Le robot est un outil d'assistance, pas de substitution
2. **Transporter des charges supérieures à 5kg** — Limitations matérielles du ROSMASTER M3 Pro
3. **Utilisation en extérieur** — Conçu exclusivement pour environnement intérieur contrôlé
4. **Manipulation d'objets fragiles** — Pas de bras robotisé, pas de préhension fine
5. **Communication vocale bidirectionnelle avec patients** — Pas de speaker/micro patient-facing
6. **Contrôle automatique des équipements EHPAD** — Pas d'intégration portes, lumières, etc.
7. **Gestion des stocks et ERP** — Intégration managériale future seulement
8. **Téléconférence vidéo** — Pas d'équipement vidéo embarqué
