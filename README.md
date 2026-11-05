# 🏊 Pool Control Automation (PCA) pour Home Assistant

![Home Assistant Minimum Version](https://img.shields.io/badge/Home%20Assistant-2024.10.0%2B-blue?style=for-the-badge&logo=home-assistant)
![License](https://img.shields.io/badge/License-Apache_2.0-green.svg?style=for-the-badge)
![Version](https://img.shields.io/badge/Version-v2026.1-orange?style=for-the-badge)

**Pool Control Automation (PCA)** est une suite d'automatisations de niveau industriel pour Home Assistant, conçue pour piloter, optimiser et sécuriser la filtration de votre piscine. 

Là où la plupart des automatisations domotiques reposent sur de simples compteurs temporels fragiles, PCA utilise un **moteur mathématique sans état (Stateless)** couplé à un **module de sécurité électrique indépendant**.

---

## 🌟 Fonctionnalités Principales

*   🧠 **Moteur Prédictif Stateless** : Recalcule instantanément l'état attendu en fonction de l'heure absolue. Indifférent aux redémarrages de Home Assistant et immunisé contre les bugs de changement d'heure (DST).
*   🌡️ **Hystérésis Thermique Parfaite** : Utilisation de mémoires (`helpers`) pour les modes Canicule, Gel et Hivernage. Évite les cycles courts destructeurs pour le moteur de la pompe (ex: On entre en canicule à 28°C, on n'en sort qu'à 27.5°C).
*   ⏱️ **Ordonnancement en Cascade** : Remplissage prioritaire de la plage de Nuit (Heures Creuses) selon un ratio cible. Le débordement se fait automatiquement sur la plage de Jour (solaire).
*   🛡️ **PCA Safety (Garde du Corps Électrique)** : Coupe physiquement la pompe en cas de désamorçage (marche à sec) via la surveillance de la puissance réelle (W).
*   🔥 **Détection Hybride de Relais Collé** : Si le relais physique reste soudé après un ordre d'arrêt, le système le détecte via la consommation résiduelle et lance une alerte critique.
*   🔕 **Notifications Anti-Spam** : Mécanisme intelligent limitant les alertes non-critiques à une notification par heure.

---

## 🧠 Architecture du Système

PCA est divisé en deux Blueprints distincts et complémentaires :

### 1. PCA Logic (`pca_logic.yaml`)
Le cerveau du système. Il applique la courbe polynomiale d'Abacus pour calculer le temps de filtration idéal en fonction de la température de l'eau. Il ne possède aucun délai interne (`delay` ou `wait`). À tout instant *T* ou lors d'un changement de variable, il évalue mathématiquement si la pompe doit tourner. Les commandes manuelles hors-plage sont intelligemment gérées.

### 2. PCA Safety (`pca_safety.yaml`)
Le vigile matériel. Il tourne en parallèle de *PCA Logic* et a autorité absolue. Si la pompe tourne mais que la consommation (Watts) chute sous un seuil paramétrable (désamorçage), PCA Safety verrouille le système, coupe l'alimentation et envoie une alerte. Le déverrouillage nécessite une action humaine explicite.

---

## 📋 Prérequis Matériels

Pour tirer pleinement parti de PCA, votre installation Home Assistant doit remonter :
1.  **Un relais connecté** pilotant le contacteur de puissance de la pompe (`switch`).
2.  **Une sonde de température d'eau** (`sensor`).
3.  **Un capteur de puissance électrique** (`sensor` mesurant les Watts de la pompe). *(Optionnel mais strictement requis pour PCA Safety)*.
4.  **Une sonde de température extérieure** *(Optionnelle - agit en redondance pour le mode Hors-Gel)*.

---

## 🚀 Installation

### Étape 1 : Création des Helpers (Entités mémoires)
Le fonctionnement sans état nécessite quelques `Helpers` (Entrées) dans Home Assistant pour stocker les variables physiques.
Créez les entités suivantes depuis `Paramètres > Appareils et services > Entrées` :

*   **Interrupteurs booléens (`input_boolean`) :**
    *   `input_boolean.piscine_verrou_defaut` (Partagé entre Logic et Safety)
    *   `input_boolean.piscine_mem_canicule`
    *   `input_boolean.piscine_mem_gel`
    *   `input_boolean.piscine_mem_hivernage`
*   **Liste déroulante (`input_select`) :**
    *   `input_select.piscine_mode` (Options : *Courbe, Hivernage, Automatique, Forcé, Inactif*)
*   **Nombre (`input_number`) :**
    *   `input_number.piscine_duree_calculee` (Min: 0, Max: 24, Pas: 0.1)
*   **Date et Heure (`input_datetime`) :**
    *   `input_datetime.piscine_fin_filtration` (Mode: Date et Heure)
*   **Bouton (`input_button`) :**
    *   `input_button.piscine_recalculer`

### Étape 2 : Importation des Blueprints
Importez les deux Blueprints depuis ce dépôt GitHub vers votre instance Home Assistant :
1. Allez dans `Paramètres > Automatisations et Scènes > Blueprints`.
2. Importez `pca_logic.yaml` et `pca_safety.yaml`.

### Étape 3 : Création des automatisations
Créez une automatisation pour **PCA Logic** et une pour **PCA Safety** en associant vos capteurs physiques et les Helpers créés à l'Étape 1.

---

## 📊 Dashboard Lovelace Recommandé

Le code pour un tableau de bord optimal, utilisant la technique de la *divulgation progressive* (les alertes n'apparaissent qu'en cas de danger), est fourni dans le dossier `/dashboard`. 

---

## 📄 Licence

Ce projet est sous licence **Apache License 2.0**.
Vous êtes libre de l'utiliser, de le modifier et de le distribuer, sous réserve d'en mentionner la provenance technique. Consultez le fichier `LICENSE` pour plus de détails.

---
*Si ce projet vous a aidé à sécuriser votre bassin et optimiser votre facture d'électricité, n'hésitez pas à laisser une étoile ⭐ sur le dépôt !*
