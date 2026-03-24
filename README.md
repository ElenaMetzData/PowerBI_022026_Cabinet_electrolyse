# 📊 Pilotage d’activité – Cabinet d’électrolyse (Power BI)

## Contexte

Un cabinet spécialisé en épilation définitive disposait de plusieurs sources de données (Planity, fichier client, transactions bancaires), mais sans outil structuré pour piloter son activité.

Les données existaient, mais restaient :
- dispersées,
- peu comparables,
- difficilement exploitables pour la prise de décision.

L’enjeu était de construire une vision claire de l’activité, à la fois **opérationnelle (planning, remplissage)** et **financière (chiffre d’affaires, rentabilité)**.

---

## Objectifs

- Centraliser les données issues de différentes sources  
- Suivre la performance du cabinet dans le temps  
- Comprendre les leviers de croissance (zones traitées, acquisition client)  
- Anticiper l’activité future à partir des rendez-vous déjà planifiés  
- Fournir un outil simple, lisible et actionnable pour la dirigeante  

---

## Données exploitées

- **Planity (RDV)**  
  Statuts des rendez-vous (réalisé, annulé, no-show…), durées, dates  

- **Fichier client**  
  Date de première consultation, canal d’acquisition  

- **Transactions bancaires**  
  Recettes et charges réelles  

- **Données web (Google Analytics)**  
  Trafic et sources d’acquisition  

---

## Approche

### 1. Structuration des données
- Création d’une table calendrier complète  
- Nettoyage et harmonisation des statuts  
- Centralisation des sources dans un modèle unique  

### 2. Modélisation métier
- Construction d’indicateurs clés :
  - Chiffre d’affaires  
  - Résultat net  
  - Taux de remplissage  
  - Valeur vie client (LTV)  
  - Durée moyenne de traitement  

- Création d’une table d’occupation permettant de regrouper :
  - RDV réalisés  
  - Temps administratif  
  - Annulations  
  - No-show  

### 3. Analyse temporelle et prévision
- Historique glissant sur 12 mois  
- Projection sur 6 mois intégrant :
  - les rendez-vous déjà planifiés  
  - la saisonnalité  
  - le taux d’abandon  
  - l’acquisition de nouveaux clients  

### 4. Visualisation
- Dashboard **one-page** avec navigation par signets  
- Alternance entre vues :
  - financière (CA / marge)  
  - opérationnelle (clients / remplissage)  

- Mise en place d’interactions entre visuels pour explorer les données :
  - par zone traitée  
  - par canal d’acquisition  
  - par période  

---

## Résultats & insights

- Mise en évidence d’un **décalage entre croissance du CA et dégradation de la marge**  
- Identification des zones les plus rentables  
- Visualisation de la **différence de valeur client selon le canal d’acquisition**  
- Anticipation des périodes creuses et des pics d’activité  
- Meilleure compréhension du poids des annulations et du temps non facturé  

---

## Outils

- Power BI (modélisation + visualisation)  
- DAX (mesures avancées, prévision)  
- Power Query (transformation des données)  
- Excel / CSV (sources)  

---

## Valeur apportée

- Passage d’une vision “opérationnelle / intuitive” à un **pilotage structuré**  
- Gain de temps dans le suivi des performances  
- Aide à la décision sur :
  - la gestion du planning  
  - l’acquisition client  
  - la rentabilité des prestations  

---

## Positionnement

Ce projet illustre ma manière de travailler :

> transformer des données existantes mais sous-exploitées en un outil simple, lisible et directement utile pour piloter une activité.
