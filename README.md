# _nom du projet :_ Cartographie 3D Paraview

## Table des matières
[1. Description](#1-description)

[2. Technologies utilisées](#2-technologies-utilisées)

[3. Description du livrable](#3-description-du-livrable)

[4. Installation](#4-installation)

[5. Utilisation](#5-utilisation)

[6. Architecture du livrable](#6-architecture-du-livrable)

[7. Remerciements](#7-remerciements)

[8. Auteur](#8-auteur)
  
## 1. Description
 
Ce projet constitue un pipeline semi-automatisé de traitement et de visualisation 3D des résultats neutroniques issus de simulations MCNP. L’objectif est de passer, en quelques étapes guidées, de fichiers bruts `.msht` à des visualisations exploitables dans ParaView, en utilisant FreeCAD pour l’import géométrique et des scripts Python pour orchestrer le tout.

Il s'adresse au service calcul de l'entreprise CEGELEC-CEM, et l'équipe sûreté/radioprotection. 

Le flux de traitement suit la séquence suivante :

**MCNP (.msht) + FreeCAD (.FCStd)** ⟶ **Script Python (main.py)** ⟶ **Conversion en VTK et STEP respectivement** ⟶ **Ouverture automatique dans ParaView** ⟶ **Application de `vue_brute.py`** ⟶ **Visualisation et post-traitement libre à l'aide des macros proposées**

## 2. Technologies utilisées

- [ParaView 5.13.3](https://www.paraview.org/) – pour la visualisation 3D
- [FreeCAD 1.0.0](https://www.freecad.org/) – pour l’exportation en STEP
- [Python 3.11.10](https://www.python.org/) – pour les scripts de post-traitement (.py)

## 3. Description du livrable

Ce livrable est un dossier de travail appelé **`CARTO3D/`**. Il contient l’ensemble des éléments nécessaires à la procédure de post traitement :

### `Convertisseur_Fichier_MSHT_to_vtk/`  
_Dossier contenant le convertisseur `./bin/Release/mt2vtk.exe` de MSHT vers VTK._

### `input_files/`  
_Dossier où l’on place les fichiers d’entrée FreeCAD (`.FCStd`) et MCNP (`.msht`). On y trouvera également la géométrie STEP, les fichiers contenant les résultats et erreurs sous VTK, et le CSV._

### `scripts/`  
_Dossier contenant les scripts Python principaux :_

* **`util.py`** : Contient les fonctions utilitaires et les chemins d’accès utilisés globalement. Toutes les macros et scripts y  accèdent via l’import `import util as u`. Il est **essentiel** de renseigner l’adresse d’installation de ParaView au début de ce fichier.

* **`main.py`** : _Script principal orchestrant les opérations suivantes :_
   * Appel du convertisseur `mt2vtk.exe`
   * Déplacement des fichiers nécessaires dans `input_files`
   * Lancement automatique de ParaView avec :
      * le fichier `.step`
      * les fichiers `.vtk` (valeurs + erreurs)
      * le script `vue_brute.py`

* **`vue_brute.py`** : _Script exécuté automatiquement à l’ouverture de ParaView. Il applique un traitement initial commun :_
  * création d’un layout lisible
  * application du filtre `CellDataToPointData`
  * création de la Sonde

_NB :_ Les scripts `main.py` et `vue_brute.py` sont exécutés automatiquement par la macro FreeCAD. Aucune action utilisateur n’est nécessaire.

### `macro/`
_Dossier contenant les macros à placer dans FreeCAD ou ParaView._

#### `macro/FreeCAD/`
* **`macro_FreeCAD_CARTO3D.py`** : _C’est la seule macro utilisée dans FreeCAD. Elle constitue le point d’entrée de la procédure :_
   * Sélectionner le fichier `.FCStd` placé dans `input_files`.
   * Exportation automatique du fichier au format `.step`.
   * Exécution du script principal `main.py`.

#### `macro/Paraview/`
* **`macro_coupe_2D.py`** : _Permet de créer une coupe 2D de la source MSHT :_

   * Depuis le layout principal, lancer la macro.
   * Saisir l’origine du plan, le vecteur normal, et le nom de la coupe.
   * La macro génère la coupe et l’affiche dans un nouveau layout.
   * Une légende (nom de la coupe) est créée à l’origine ; elle peut être repositionnée en sélectionnant `legend_<nom de coupe>` et modifiant les coordonnées dans `Billboard Position` ou en pressant `P`, elle sera alors placée au niveau du pointeur.

* **`macro_iso_2D.py`** : _Génère des lignes isodoses sur un plan de coupe 2D :_

   * À lancer depuis le layout d’une coupe.
   * Entrer une ou plusieurs valeurs d’isodoses (ex. : "0.0025", "0.0025, 2").
   * Les lignes s’affichent automatiquement ; leur couleur peut être personnalisée via `Display > Coloring > Solid Color`.

* **`macro_isodose.py`** : _Crée des isodoses sous forme de surfaces 3D (isosurfaces) à partir de valeurs choisies, affichées dans un nouveau layout (même syntaxe que macro_iso_2D.py)._

* **`macro_legende.py`** : _Gère les échelles de couleur et les titres des légendes :_

  * se placer dans le layout dans lequel la source d'interêt est affichée puis lancer la macro
  * entrer le nom de la source d'interêt (_exemples : pour recalibrer sur l'ensemble des valeurs, entrer MSHT dans le layout principal. Pour recalibrer sur une coupe particulière, entrer le nom de la coupe dans le layout de la coupe_)
  * Mode automatique (couvre toutes les valeurs de la source) : entrer 'A'. Mode manuel (plage définie par l’utilisateur) : entrer 'M'.
  * Le titre de la légende peut aussi être modifié localement dans le layout actif. Entrer 'O' pour le modifier, 'N' pour ne rien changer.

* **`macro_max_plan.py`** : _Recherche du maximum d’un champ scalaire sur une coupe nommée :_
  * Se placer dans la coupe d'interêt, et lancer la macro
  * Entrer le nom de la coupe
  * Affiche dans la console la position du maximum, sa valeur, et l’erreur relative.
  * Option d’ajout d’une zone de texte contenant ces données.

* **`macro_règle.py`** : _Permet de tracer une règle graduée entre deux points :_
  * Commencer par entrer manuellement les coordonnées des deux points. 
  * Entrer le nom souhaité pour la règle.
  * Il est ensuite possible de placer les points à la souris (touche `1`, `2`). Pour cela, cliquer sur la source `règle`(_via le nom donné_), cliquer dans la zone `Properties` encadrée en bleu, cocher `Show line`. Une fois les points correctement placés, cliquer sur `Apply`. 

* **`macro_datas.py`** : _Permet de récupérer les donnees en un point de la géométrie. S'utilise en complément avec `Sonde`._
  * Commencer par sélectionner avec la Sonde le point d'interêt. Pour cela, renseigner ses coordonnées (en mm !) ou utiliser la touche `P`. **Attention**, le point sélectionné est sur la surface directe, et pas en profondeur. Penser à toujours vérifier les coordonnées du point étudié. S'aider des plans de coupe peut être utile.
  * Une fois le point correctement placé, lancer la macro. Le résultat s'affiche dans la fenêtre de dialogue `Output Messages`. 
  * De plus, le résultat est stocké dans un fichier `resultats_sonde.csv` dans le dossier `input_files`, créé lors de la première utilisation, puis modifié pour ajouter chaque résultat.

## 4. Installation

1. Installez les logiciels nécessaires - voir [2. Technologies utilisées](#2-technologies-utilisées) 
2. Dans le fichier util.py, renseigner l'adresse du dossier du logiciel Paraview (et pas l'executable).
3. Importer les différentes macros :
* les macros destinées à FreeCAD et Paraview sont situées dans le dossier `macro/<nom_logiciel>`
* les coller dans le dossier macro des logiciels : `AppData/Roaming/<nom_logiciel>/macro` (_pour accéder à AppData, taper `%AppData%` dans le menu démarrer windows_ )
* relancer les logiciels si nécessaire
* Dans Paraview, pour afficher les macros dans la barre d'outils : `View`>`Toolbars`> sélectionner `Macros Toolbars`.

## 5. Utilisation

1. Placer les fichiers FreeCAD `.FCStd` et meshtally `.msht` dans `input_files/`.
2. Ouvrir FreeCAD, exécuter `macro_FreeCAD_carto3D.py`
3. Sélectionner votre fichier FreeCAD, puis patientez jusqu'à l'ouverture de Paraview. Vous pouvez fermer le terminal quand cela est proposé.
4. Paraview est ouvert. Cliquez dans la fenêtre à gauche sur `Apply`. Les deux sources `.vtk` sont alors affichées. Cliquez sur l'oeil pour les cacher : elles sont inutiles à l'affichage.
5. Vous pouvez choisir de voir la boite de dialogue dans `View`>`Output Message`. Elle est utilisée dans les macros.
6. Utiliser les différentes macros et Paraview pour le post traitement souhaité (_référez vous à [3. Description du livrable](#3-description-du-livrable) pour la correcte utilisation des macros, et le guide essentiel Paraview pour ce dernier_) 

## 6. Architecture du livrable

### CARTO3D/  
├── Convertisseur_Fichier_MSHT_to_VTK/  
│   └── ... mt2vtk.exe  
│  
├── input_files/  
│   ├── nom_fichier_geometrie.FCStd  
│   ├── nom_fichier_geometrie.step  
│   ├── fichier2.msht  
│   ├── nom_fichier_geometrie.vtk  
│   ├── nom_fichier_geometrie_error.vtk  
│   └── resulats_sonde.csv  
│  
├── macro/  
│   ├── FreeCAD/  
│   │   └── macro_FreeCAD_carto3D.py  
│   │  
│   └── Paraview/  
│       ├── macro_max_plan.py  
│       ├── macro_coupe_2D.py  
│       ├── macro_datas.py  
│       ├── macro_isodose.py  
│       ├── macro_règle.py  
│       ├── macro_iso_2D.py  
│       └── macro_legende.py  
│  
├── scripts/  
│   ├── main.py  
│   ├── util.py  
│   └── vue_brute.py                        

## 7. Remerciements

Je tiens tout d'abord à remercier mon maitre de stage, Christophe Mattera qui m'a encadré pour ce projet, et Héloïse Morin, ils ont été présent pour répondre à toutes mes questions, et ont eu la patience de m'expliquer précisemment ce qu'ils attendaient de cet outil. Je remercie également Copilot, camadarade quotidien, spécialiste Python et Paraview, qui m'a appuyé sur la partie code du projet. Il n'a jamais manqué à l'appel, et a même supporté mon impatience et mon agacement certain quand il me générait cinq fois les mêmes scripts bugués avec des erreurs différentes. Je remercie toute l'équipe CEGELEC-CEM pour son accueil, cette opportunité qu'elle m'a offerte et j'espère avoir digne de ces gens qui m’ont tendu la main, peut-être à un moment où je ne pouvais pas, où j’étais seul chez moi. Et c’est assez curieux de se dire que les hasards, les rencontres forgent une destinée… Je m'égard. Je remercie CEGELEC pour ses cafés open-bar qui ont été nécessaires, et pour les 30°C dans l'open space qui ont boosté mon summer body mais peut être pas ma productivité. Je remercierai finalement mes collègues d'open space du service calcul, et particulièrement Philippe et Mathéo pour leur bonne humeur.

## 8. Auteur

Arthur Allovon  
Grenoble-INP Phelma, promo diplomée en 2026  
Spécialité GEN  
