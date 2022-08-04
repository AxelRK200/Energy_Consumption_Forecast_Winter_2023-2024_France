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
<i>La MAE (Mean Absolute Error / Erreur Absolue Moyenne) est de +/- 3,02 GW pour ce modèle, en l'état actuel de ses paramètres.</i>

<br />

## Fonctionnement du modèle

Une série temporelle dite additive peut-être décomposée en plusieurs élements :<br/>
Une tendance, une saisonnalité, et un bruit résiduel.<br/>
En voici un exemple :<br/>
![Décomposition](images/time_series_decomposition.png)

<br/>
<p>
Le modèle hybride de ce projet fonctionne de la manière suivante :
  <li>Identification de la <b>tendance</b> avec une régression linéaire</li>
<li>Identification et projection de la <b>saisonnalité</b>, avec une décomposition du signal en transformée de Fourrier. Et régression sur celle-ci</li>
</p>
<br/>
### Décomposition du signal avec transformée de Fourier
L'objet de cette étape, et le point fort de cette approche, est de décompser notre saisonnalité en plusieurs fonctions sinusoïdales, d'amplitudes différentes, afin que leur consolidation s'ajuste au mieux à la saisonnalité que nous essayons de capter.<br/>
En image, voici par exemple une décomposition en 4 séries, qui s'adapte à une saisonnalité annuelle.<br/>
![Fourier](images/fourier.png)<br/>
<i>(chaque courbe de couleur représentant une année)</i>
</p>

<br/>

### Prédiction statique ou dynamique
A l'issue du Notebook 3 (Saisonnalité), deux approches pour la prédiction des résidus,et donc du résultat final sont alors possibles :<br/>
<li>Une prévision dynamique, basée sur la correlation partielle et l'auto correlation des données (dans notre cas) de l'heure précédente, du jour précédent, et de la semaine précédente.<br/>La limite de cette approche est qu'il n'est pas possible de faire une prédiciton à l'horizon 2024, sur une base horaire, ou alors avec une incertitude (ou erreurs cumulées au fur et à mesure des prédictions) qui n'est pas pertinente.<br/>
Ce modèle est conçu pour être joué toutes les heures, ajuster ses prévisions, et détecter un éventuel emballement de consommation à court-moyen terme.</li><br/>
<li>Le second modèle, qui utilise <b>LightGBM</b>, un framework d'amplification de gradient basé sur des arbres de décision.pour la prédiction des résidus peut lui se projeter à bien plus long terme, au prix d'une précision moindre.</li>

<br/>

### Etapes réalisées dans les Notebooks

Identification de la Tendance : 
![Tendance](images/trend.png)

<br/>

Identification de la Saisonnalité : 
![Saisonnalité](images/saisonnalité.png)

<br/>

Prédiction dynamique des Rédidus : 
![Résidus dynamiques](images/résidus_dynamiques.png)


<br/>

Prédiction statique des Résidus : 
![Résidus statitiques](images/résidus.png)

<br/>

Prédiction 31/12/2023 (statique) : 
![Prédiction finale](images/forecast_final.png)
