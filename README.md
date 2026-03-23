# Projet-de-recherche-Deep-learning-for-reliable-study-of-erosion-of-Normandy-cliffs







## Résultats intermédiaires :
  Avant l'obtention des résultats joints et analysés dans le rapport, plusieurs essais ont été réalisés.
  Avec le premeir dataset sans stride, nous obtenions des résultats avec de mauvaise performances, que cela soit du point de vue d'une analyse qualitatif ou quantitatif.  
  Inférence pour 30 epochs du modèle UNet pour un dataset sans stride :
  ![alt text](https://github.com/Rahima-Z/Projet-de-recherche-Deep-learning-for-reliable-study-of-erosion-of-Normandy-cliffs/blob/main/images_readme/decoupe_mosaic_sans_stride.png)
  Evolution de la perte  pour 30 epochs du modèle UNet pour un dataset sans stride :
  ![alt text](https://github.com/Rahima-Z/Projet-de-recherche-Deep-learning-for-reliable-study-of-erosion-of-Normandy-cliffs/blob/main/images_readme/loss_unet_sans_stride.png)
  Diverses métriques d'évaluation pour 30 epochs du modèle UNet pour un dataset sans stride :
  ![alt text](https://github.com/Rahima-Z/Projet-de-recherche-Deep-learning-for-reliable-study-of-erosion-of-Normandy-cliffs/blob/main/images_readme/metriques_sans_stride.png)
  Courbe ROC pour 30 epochs du modèle UNet pour un dataset sans stride :
  ![alt text](https://github.com/Rahima-Z/Projet-de-recherche-Deep-learning-for-reliable-study-of-erosion-of-Normandy-cliffs/blob/main/images_readme/roc_curve_unet_sans_stride.png)

  La qualité de ces résultats peut être imputée au manques de tuiles présentant des pixels de la classe "côte" (soit des pixels blancs). Il conviendra également de noter que la courbe ROC ne    permets pas de bien rendre compte des capacité du modèle de par le désequilibre des classes dans le dataset.
  Nous avons donc augmenter la taille du dataset en découpant l'image d'entrée en tuiles avec un stride de 64. Nous effectuons également un filtre pour ne garder que les tuiles dont les         masque ne contiennent qu'un certains nombre de pixels blancs. Nous obtenous :  
  Inférence pour 30 epochs du modèle UNet pour un dataset avec stride et filtrage :
  ![alt text](https://github.com/Rahima-Z/Projet-de-recherche-Deep-learning-for-reliable-study-of-erosion-of-Normandy-cliffs/blob/main/images_readme/decoupe30_mosaic_stride.png)
  Evolution de la perte  pour 30 epochs du modèle UNet pour un dataset avec stride et filtrage :
  ![alt text](https://github.com/Rahima-Z/Projet-de-recherche-Deep-learning-for-reliable-study-of-erosion-of-Normandy-cliffs/blob/main/images_readme/loss_unet_v7_filtr%C3%A9_30_stride.png.png)
  Diverses métriques d'évaluation pour 30 epochs du modèle UNet pour un dataset avec stride et filtrage :
  ![alt text](https://github.com/Rahima-Z/Projet-de-recherche-Deep-learning-for-reliable-study-of-erosion-of-Normandy-cliffs/blob/main/images_readme/metriques_stride.png) 

  Visuellement, dans les deux cas nous observons des artefacts de bords sur les tuiles ainsi qu'une difficulté à bien détecter la ligne de c$ote. Cette difficulté est confirmée par la valeurs   des métriques qui montre un très grands déséquilibre entre la précision et le rappel.
  C'est l'utilisation de l'IPS ainsi qu'une calibration des paramètres évoqués dans le rapport qui permettent d'obtenir les résultats finaux.

 
