# Projet : Robot intelligent d'assistance pour EHPAD



------------------------------------------------------------------------
# 1. Vision


Développer un robot autonome basé sur ROS2 capable d'assister les

infirmiers en EHPAD en livrant de manière sécurisée les médicaments et

les plateaux-repas aux patients, afin d'optimiser le temps du personnel

soignant et améliorer la qualité des soins.


------------------------------------------------------------------------


# 2. Objectifs et périmètre

  

## Objectifs


1.  Développer un robot autonome capable de se déplacer dans un EHPAD

2.  Permettre la livraison sécurisée de médicaments et de repas

3.  Fournir une interface permettant au personnel de commander le robot

4.  Assurer la sécurité des patients et du personnel

5.  Envoyer la position du robot en temps réel

  

## Objectifs secondaires

  

1.  Ajouter une fonctionnalité de sortie des poubelles (optionnel)

2.  Ajouter un système de notification de livraison

3.  Ajouter une caméra pour surveillance et navigation

  

## Non-objectifs

  

1.  Remplacer complètement le personnel soignant

2.  Transporter des charges lourdes (\>5kg)

3.  Utilisation en extérieur

  

------------------------------------------------------------------------

  

# 3. Use Cases

  

  ID      Nom                           Acteur      Description

  ------- ----------------------------- ----------- ------------------------

  UC-01   Livrer médicament             Infirmier   Robot livre médicament

  UC-02   Livrer repas                  Personnel   Robot livre repas

  UC-03   Navigation autonome           Robot       Se déplace seul

  UC-04   Éviter obstacles              Robot       Évite personnes

  UC-05   Retour base                   Robot       Retour automatique

  UC-06   Sortir poubelle (optionnel)   Personnel   Transport déchets

  

------------------------------------------------------------------------

  

# 4. Architecture Technique

  

    Interface Web → WiFi → Robot ROSMASTER M3 Pro → ROS2 → Capteurs / Lidar / Caméra

  

------------------------------------------------------------------------

  

# 5. Stack Technique

  

## Robot

  

-   ROSMASTER M3 Pro

-   ROS2

-   Jetson Nano / Orin NX

-   Ubuntu

  

## Software

  

-   Python

-   ROS2 Nodes

-   React

-   MQTT / WebSocket

-   OpenCV

  

------------------------------------------------------------------------

  

# 6. Fonctionnalités principales

  

## Obligatoires

  

-   Navigation autonome

-   Livraison médicaments

-   Livraison repas

-   Évitement obstacles

-   Retour automatique

  

## Optionnelle
  
### Sortie des poubelles

  Objectif : Permettre au robot de transporter les déchets vers une zone définie.

Fonctionnement :

  

1.  Chargement des déchets

2.  Navigation vers zone déchets

3.  Déchargement

4.  Retour base

  

------------------------------------------------------------------------

  

# 7. Architecture ROS2

  

Nodes :

  

-   navigation_node

-   delivery_node

-   safety_node

-   vision_node

-   interface_node

-   trash_node (optionnel)

  

------------------------------------------------------------------------

  

# 8. Risques

  

  Risque              Impact   Solution

  ------------------- -------- ----------

  Collision           Élevé    Lidar

  Bug ROS             Moyen    Tests

  Erreur navigation   Moyen    SLAM

  

------------------------------------------------------------------------

  

# 9. Roadmap

  

  Phase     Objectif

  --------- -------------------

  Phase 1   Installation ROS2

  Phase 2   Navigation

  Phase 3   Livraison

  Phase 4   Interface

  Phase 5   Tests

  Phase 6   Fonction poubelle

  

------------------------------------------------------------------------

  

# 10. Résultat attendu

  

Robot capable de :

  

-   Naviguer seul

-   Livrer médicaments

-   Livrer repas

-   Retourner base

-   Optionnel : sortir poubelle

  

------------------------------------------------------------------------

  

# 11. Structure du projet

  

    robot-boy/

    │

    ├── docs/

    │   └── cdc-technique.md

    │

    ├── ros2_ws/

    ├── interface/

    ├── backend/

    └── README.md