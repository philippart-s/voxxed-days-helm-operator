# voxxed-days-helm-operator
Source code for the Helm operator talk at Voxxed Days Lux 2022

# Déroulé de la démo

**⚠️ Au moment de l'écriture de ce tuto, la dernière version de Go est 1.18.1, hors le SDK (1.20.1) n'est pas compatible avec cette version il faut utiliser une version 1.17.x maximum ⚠️**

## 🎉 Init project
 - la branche `01-init-project` contient le résultat de cette étape
 - [installer / mettre](https://sdk.operatorframework.io/docs/installation/) à jour la dernière version du [Operator SDK](https://sdk.operatorframework.io/) (v1.20.1 au moment de l'écriture du readme)
 - créer le répertoire `voxxed-days-helm-operator`
 - dans le répertoire `voxxed-days-helm-operator`, scaffolding du projet : `operator-sdk init --plugins helm --domain fr.wilda --version v1 --helm-chart=https://github.com/philippart-s/ngnix-operator-helm-chart/releases/download/1.0.0/ngnix-operator-helm-chart-1.0.0.tgz`
 - A ce stage une arborescence complète a été générée, notamment la partie configuration dans `config` et un `Makefile` permettant le lancement des différentes commandes de build

## 📄 CRD generation
 - la branche `02-crd-generation` contient le résultat de cette étape
 - création de la CRD dans Kubernetes : `make install`
 - vérification de la création de la CRD : `kubectl get crd ngnixoperatorhelmcharts.charts.fr.wilda`
```bash
$ kubectl get crd ngnixoperatorhelmcharts.charts.fr.wilda
NAME                                      CREATED AT
ngnixoperatorhelmcharts.charts.fr.wilda   2022-06-03T11:15:55Z
```