# Déploiement de l'outil EnR

Ce dépot permet le déploiement de l'outil EnR (versions en [python](https://github.com/blenzi/enr-app) et en [R-Shiny](https://gitlab.com/blenzi/enr_reseaux_teo)) sur un cluster Kubernetes. La configuration présentée est spécifiquement adaptée au [SSP Cloud](https://datalab.sspcloud.fr/home).

Les différentes branches de ce package correspondent au déploiement des différentes applications (shiny pour l'outil en R-Shiny, main pour l'outil en python).

### Chart Helm

Le déploiement de l'application se fait via un chart Helm, basé sur le [template de l'application shiny-app](https://github.com/InseeFrLab/sspcloud-tutorials/blob/main/deployment/shiny-app.md).

Le fichier `Chart.yaml` contient les métadonnées du chart (nom, version) ainsi que ses dépendances, i.e. les potentiels autres charts Helm dont il hérite. Dans notre cas, on voit que le chart hérite du [chart Shiny](https://github.com/InseeFrLab/helm-charts/tree/master/charts/shiny) d'InseeFrLab. Ce chart spécifie généralement les ressources Kubernetes nécessaires au déploiement d'une application Shiny, de sorte à ce que l'on ait qu'à modifier les valeurs d'instanciation pour déployer notre application.

Le fichier `values.yaml` contient précisément les valeurs que l'on modifie par rapport au chart général. Les modifications à apporter dépendent naturellement de ce que réalise en pratique l'application, car cela conditionne les ressources dont elle a besoin. Dans un premier temps, il nous faut modifier : 
- le chemin et nom de l'image (paramètre `shiny.image.repository`)
- le tag de l'image, i.e. sa version (paramètre `shiny.image.tag`)
- l'hostname de l'Ingress l'URL à laquelle l'application sera accessible une fois déployée (paramètre `shiny.ingress.hostname`)

### Accès au stockage objet S3

Pour avoir accès aux données stockées sur MinIO / S3 il faut l'activer dans `values.yaml` et fournir à l'application les informations d'authentification au service (voir [cette recette](https://github.com/InseeFrLab/sspcloud-tutorials/blob/main/deployment/shiny-app.md#utilisation-du-stockage-de-donn%C3%A9es-s3-avec-minio)):

```yaml
  s3:
    enabled: true
    existingSecret: app-s3
```


### Déploiement avec helm

Depuis la racine du _package_ :

```shell
helm install enr helm/  # ou enr-app pour l'outil en python
```

Pour mettre à jour l'image utilisée (faut avoir `pullPolicy: Always` dans `values.yaml`):

```shell
kubectl rollout restart deployment enr-shiny  # ou enr-app-shiny pour l'outil en python
```
