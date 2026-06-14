# Walmart Sales Prediction

## 1. Contexte

La prévision des ventes constitue un enjeu majeur pour les enseignes de distribution. Une estimation fiable des ventes futures permet notamment d'améliorer la gestion des stocks, l'organisation logistique et la planification commerciale.

Dans ce projet, l'objectif est de prédire les ventes hebdomadaires de magasins Walmart à partir de variables économiques, calendaires et propres aux magasins.

L'étude s'appuie sur un dataset historique contenant des informations relatives à plusieurs magasins Walmart ainsi que différents indicateurs susceptibles d'influencer les ventes.

---

## 2. Objectif

L'objectif du projet est de construire un modèle de Machine Learning supervisé capable de prédire la variable :

**Weekly_Sales**

à partir des variables explicatives disponibles dans le dataset.

Plusieurs approches de régression sont évaluées afin d'identifier le modèle offrant le meilleur compromis entre performance et capacité de généralisation.

---

## 3. Description du dataset

### Caractéristiques générales

* 150 observations initiales
* 8 variables
* 20 magasins Walmart

### Variables disponibles

| Variable     | Description                                            |
| ------------ | ------------------------------------------------------ |
| Store        | Identifiant du magasin                                 |
| Date         | Date de la semaine observée                            |
| Holiday_Flag | Présence ou non d'une semaine comportant un jour férié |
| Temperature  | Température moyenne observée                           |
| Fuel_Price   | Prix du carburant                                      |
| CPI          | Consumer Price Index                                   |
| Unemployment | Taux de chômage                                        |
| Weekly_Sales | Ventes hebdomadaires (variable cible)                  |

---

## 4. Préparation des données

Plusieurs étapes de nettoyage et de transformation ont été réalisées afin d'améliorer la qualité des données avant l'entraînement des modèles.

### Gestion des valeurs manquantes

* Suppression des lignes dont la variable cible (`Weekly_Sales`) est absente.
* Suppression des lignes dont la date est manquante.
* Remplacement des valeurs manquantes de `Holiday_Flag` par 0.
* Estimation métier des températures manquantes à partir :

  * des semaines voisines ;
  * de la même période sur les autres années disponibles.
* Interpolation linéaire des variables :

  * `Fuel_Price`
  * `CPI`
  * `Unemployment`

### Traitement des outliers

Les valeurs extrêmes ont été identifiées selon la règle :

* moyenne ± 3 écarts-types

et supprimées pour les variables :

* Temperature
* Fuel_Price
* CPI
* Unemployment

### Création de variables temporelles

La variable `Date` a été décomposée afin d'introduire une information de saisonnalité :

* Year
* Month
* Week

### Encodage

Les variables catégorielles suivantes ont été converties en catégories :

* Store
* Holiday_Flag

L'encodage final est réalisé automatiquement dans le pipeline de modélisation via un `OneHotEncoder`.

---

## 5. Analyse exploratoire des données

L'analyse exploratoire met en évidence plusieurs éléments importants.

### Corrélations

Aucune variable ne présente de corrélation très forte avec les ventes hebdomadaires.

Cela suggère que les ventes dépendent probablement de plusieurs facteurs combinés plutôt que d'une seule variable dominante.

### Influence du magasin

Le magasin apparaît comme le facteur explicatif le plus important du dataset.

Les niveaux moyens de ventes diffèrent fortement d'un magasin à l'autre, probablement en raison de caractéristiques non présentes dans les données :

* taille du magasin ;
* localisation ;
* zone de chalandise ;
* fréquentation.

### Saisonnalité

Une saisonnalité marquée est observée.

Les ventes augmentent significativement en fin d'année avec un pic particulièrement visible durant le mois de décembre.

### Température

La température présente une relation apparente avec les ventes.

Cependant, une analyse plus approfondie montre que cette relation est largement liée à la saisonnalité et ne semble pas constituer un facteur explicatif direct des ventes.

### CPI

L'analyse du CPI met en évidence un phénomène de composition :

* corrélation négative à l'échelle globale ;
* comportement plus complexe lorsque l'effet du magasin est pris en compte.

Cette variable doit donc être interprétée avec prudence.

---

## 6. Modélisation

### Pipeline de prétraitement

Le pipeline de modélisation comprend :

#### Variables numériques

* Imputation par médiane
* Standardisation

#### Variables catégorielles

* One-Hot Encoding
* Suppression de la première modalité (`drop="first"`)

---

## 7. Modèles évalués

### 1. Régression linéaire (Baseline)

Premier modèle de référence utilisé pour évaluer les performances du dataset.

Objectif :

* disposer d'une base de comparaison ;
* identifier d'éventuels phénomènes d'overfitting.

---

### 2. Ridge Regression

La régularisation Ridge pénalise les coefficients de grande amplitude.

Objectifs :

* réduire la variance du modèle ;
* améliorer la généralisation ;
* limiter l'overfitting.

---

### 3. Lasso Regression

La régularisation Lasso applique une pénalisation L1.

Objectifs :

* réduire la complexité du modèle ;
* supprimer automatiquement certaines variables peu informatives ;
* améliorer la robustesse.

---

### 4. Lasso optimisé par GridSearchCV

Une recherche systématique du meilleur paramètre `alpha` est réalisée via validation croisée.

Configuration :

* Validation croisée : 5 folds
* Grille logarithmique de valeurs d'alpha

Cette approche permet d'obtenir une sélection d'hyperparamètres plus robuste que le réglage manuel.

---

## 8. Résultats

Les résultats montrent que :

* la régression linéaire présente un surapprentissage important ;
* Ridge améliore la généralisation ;
* Lasso améliore encore légèrement les performances ;
* la sélection automatique d'alpha via GridSearchCV fournit le modèle le plus robuste.

### Modèle retenu

**Lasso optimisé par GridSearchCV**

### Justification

* bonne capacité de généralisation ;
* réduction de l'overfitting ;
* méthodologie rigoureuse grâce à la validation croisée ;
* modèle plus simple et plus interprétable.

---

## 9. Limites du projet

Plusieurs limites doivent être prises en compte lors de l'interprétation des résultats.

### Taille du dataset

Après nettoyage, le dataset contient seulement un peu plus d'une centaine d'observations.

Cette taille limitée réduit la capacité du modèle à apprendre des relations complexes.

### Faible nombre d'observations par magasin

Certains magasins disposent de très peu d'observations, ce qui limite la robustesse des analyses spécifiques à chaque magasin.

### Peu de semaines festives

La variable `Holiday_Flag` ne comporte qu'un faible nombre d'occurrences positives, ce qui limite la fiabilité des conclusions associées.

### Variables explicatives manquantes

Plusieurs facteurs susceptibles d'influencer les ventes ne sont pas disponibles :

* promotions ;
* surface des magasins ;
* nombre de clients ;
* concurrence locale ;
* événements commerciaux.

---

## 10. Perspectives d'amélioration

Plusieurs pistes pourraient permettre d'améliorer les performances du modèle.

### Enrichissement des données

Ajout de nouvelles variables :

* promotions ;
* surface commerciale ;
* fréquentation ;
* données géographiques ;
* événements marketing.

### Historique plus long

Disposer de plusieurs années supplémentaires permettrait :

* d'améliorer l'apprentissage de la saisonnalité ;
* de stabiliser les estimations.

### Modèles avancés

Tester des modèles non linéaires :

* Random Forest Regressor
* XGBoost Regressor
* Gradient Boosting Regressor

Ces modèles pourraient capturer des relations plus complexes entre les variables.

---

## 11. Conclusion

Ce projet met en œuvre l'ensemble des étapes classiques d'un workflow de Machine Learning supervisé :

* préparation des données ;
* analyse exploratoire ;
* ingénierie des variables ;
* modélisation ;
* évaluation ;
* sélection du modèle final.

L'analyse met en évidence l'importance de la saisonnalité et des caractéristiques propres aux magasins dans l'explication des ventes hebdomadaires.

Parmi les modèles évalués, le Lasso optimisé par validation croisée apparaît comme le meilleur compromis entre performance, simplicité et robustesse.
