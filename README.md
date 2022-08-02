# Prévision de la consommation énergétique à l'horizon 2023 - 2024

![Version](https://img.shields.io/badge/version-1.0.0-blue) ![Python](https://img.shields.io/badge/python-3.10.5-red)

<p>Prévision du pic de consommation énergétique en France.<br />
<i>Publié le 1er Août 2022</i></p>

## Source
Cette étude se base sur le Dataset mis à disposition par ODRE : Open Data Réseaux Energie.
Groupement parmi lequel on retrouve RTE, et GRT Gaz.

Les données comportent les caractéristiques suivantes :
- Granularité à la demi-heure
- Période allant du 1er Janvier 2012, au 31 Mai 2022, au moment de la publication
- Détail fait entre la consommation de gaz, d'électricité, et consommation totale brute

Lien : [open_data_reseaux-energies](https://odre.opendatasoft.com/explore/dataset/consommation-quotidienne-brute/information/?sort=-date_heure&dataChart=eyJxdWVyaWVzIjpbeyJjaGFydHMiOlt7InR5cGUiOiJsaW5lIiwiZnVuYyI6Ik1BWCIsInlBeGlzIjoiY29uc29tbWF0aW9uX2JydXRlX2VsZWN0cmljaXRlX3J0ZSIsInNjaWVudGlmaWNEaXNwbGF5Ijp0cnVlLCJjb2xvciI6IiM2NmMyYTUifV0sInhBeGlzIjoiZGF0ZV9oZXVyZSIsIm1heHBvaW50cyI6bnVsbCwidGltZXNjYWxlIjoiaG91ciIsInNvcnQiOiIiLCJjb25maWciOnsiZGF0YXNldCI6ImNvbnNvbW1hdGlvbi1xdW90aWRpZW5uZS1icnV0ZSIsIm9wdGlvbnMiOnsic29ydCI6Ii1kYXRlX2hldXJlIn19fV0sInRpbWVzY2FsZSI6IiIsInNpbmdsZUF4aXMiOmZhbHNlLCJkaXNwbGF5TGVnZW5kIjp0cnVlLCJhbGlnbk1vbnRoIjp0cnVlfQ%3D%3D)

Aperçu : ![Aperçu](images/OpenData.JPG)

Le profil des données sources est le suivant.<br/>
Extrait à partir de 2018 pour une meilleure visualisation.<br/>
La crête de consommation journalière est en rouge. Les mesures étant à la demi-heure pour la consonmmation électrique.<br/> 
<i>Unité GW</i>.<br/><br/>
![Consommation](images/raw_data_viz.png)

<br/>

## Conclusions de l'étude prédictive
Le notebook intitulé / **4_Static_Prediction_Residuals.ipynb** / effectue une prédiction à l'horizon du 31/12/2013.<br/>
Sur la base du modèle hybride entraîné<sup>(1)</sup>, nous arrivons à l'estimation suivante :
>Le pic de consommation électrique de l'année 2022 aurait déjà bien eu lieu le 14 Janvier dernier à 87 GW.<br/>
>Le prochain pic hivernal de consommation électrique est estimé survenir **le 18 Janvier 2023, entre 11h et midi**.<br/>
>Il devrait être de **85,2 GW**, au minimum<sup>(2)</sup>.<br/>
>
>C'est une estimation, si elle se vérifie, qui sera en nette baisse par rapport à l'année 2022.<br/>
>Toutefois, elle reste plus elevée que l'hiver 2020, malgré une tendance de consommation énergétique ayant une tendance à la légère baisse.

<sup>(1)</sup> Ce modèle hybride, aussi appelé 'ensemble' selon la littérature, est une combinaison de deux modèles.<br/>
<sup>(2)</sup> A minima, car notre modèle à tendance à volontairement sous-evaluer les pics, bien qu'à s'en rapprocher avec le plus de précision possible.<br/>
<i>La MAE (Mean Average Error / Erreur Absolue Moyenne) est de +/- 3,02 GW pour ce modèle, en l'état actuel de ses paramètres.</i>

<br />

## Fonctionnement du modèle

Une série temporelle dite additive peut-être décomposée en plusieurs élements :<br/>
Une tendance, une saisonnalité, et un bruit résiduel.<br/>
En voici un exemple :<br/>
![Décomposition](images/time_series_decomposition.png)

<br/>
<p>
Le modèle hybride de ce projet fonctionne de la manière suivante :
<li>Identification de la **tendance** avec une régression linéaire</li>
<li>Identification et projection de la **saisonnalité**, avec une décomposition du signal en transformée de Fourrier. Et Regression sur celle-ci</li>
<li>Prédiction des **résidus** avec LightGBM, un framework d'amplification de gradient basé sur des arbres de décision.</li>
</p>

