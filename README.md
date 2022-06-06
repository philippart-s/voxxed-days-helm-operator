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

## 🤖 Nginx operator
 - la branche `03-nginx-operator` contient le résultat de cette étape
 - lancer l'opérateur en mode local : `make install run`
```bash
$ make install run
/Users/stef/Talks/operators-for-all-dev/helm-operator-template/bin/helm-operator run
{"level":"info","ts":1654255365.359911,"logger":"cmd","msg":"Version","Go Version":"go1.17.10","GOOS":"darwin","GOARCH":"arm64","helm-operator":"v1.20.1","commit":"181212ef5575492ca093592333b88d09ec582172"}
{"level":"info","ts":1654255365.361741,"logger":"cmd","msg":"Watch namespaces not configured by environment variable WATCH_NAMESPACE or file. Watching all namespaces.","Namespace":""}
{"level":"info","ts":1654255367.612032,"logger":"controller-runtime.metrics","msg":"Metrics server is starting to listen","addr":":8080"}
{"level":"info","ts":1654255367.614935,"logger":"helm.controller","msg":"Watching resource","apiVersion":"charts.fr.wilda/v1","kind":"NgnixOperatorHelmChart","namespace":"","reconcilePeriod":"1m0s"}
{"level":"info","ts":1654255367.615459,"msg":"Starting server","path":"/metrics","kind":"metrics","addr":"[::]:8080"}
{"level":"info","ts":1654255367.615628,"msg":"Starting server","kind":"health probe","addr":"[::]:8081"}
{"level":"info","ts":1654255367.6157079,"logger":"controller.ngnixoperatorhelmchart-controller","msg":"Starting EventSource","source":"kind source: *unstructured.Unstructured"}
{"level":"info","ts":1654255367.615736,"logger":"controller.ngnixoperatorhelmchart-controller","msg":"Starting Controller"}
{"level":"info","ts":1654255367.716397,"logger":"controller.ngnixoperatorhelmchart-controller","msg":"Starting workers","worker count":8}
```
 - créer le namespace `test-nginx-operator`: `kubectl create ns test-nginx-operator`
 - appliquer la CR d'exemple présente dans `./config/samples`sur Kubernetes: `kubectl apply -f ./config/samples/charts_v1_ngnixoperatorhelmchart.yaml -n test-nginx-operator`
 - l'opérateur devrait créer le pod Nginx et son service:
```bash
$ kubectl get pod,svc,ngnixoperatorhelmchart  -n test-nginx-operator
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-557d859bff-g5zqg   1/1     Running   0          81s

NAME                    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/nginx-service   NodePort   10.3.237.138   <none>        80:30080/TCP   83s

NAME                                                                   AGE
ngnixoperatorhelmchart.charts.fr.wilda/ngnixoperatorhelmchart-sample   2m16s
```
 - tester dans un navigateur ou par un curl l'accès à `http://<node external ip>:30080`, pour récupérer l'IP externe du node : `kubectl cluster-info`

 ## ✏️ Update CR
 - la branche `04-update-cr` contient le résultat de cette étape
 - changer le port et le nombre de replicas dans la CR `config/samples/charts_v1_ngnixoperatorhelmchart.yaml`:
```yaml
apiVersion: charts.fr.wilda/v1
kind: NgnixOperatorHelmChart
metadata:
  name: ngnixoperatorhelmchart-sample
spec:
  # Default values copied from <project_dir>/helm-charts/ngnix-operator-helm-chart/values.yaml
  replicaCount: 2
  service:
    port: 30081
```
 - appliquer la CR: `kubectl apply -f ./config/samples/charts_v1_ngnixoperatorhelmchart.yaml -n test-nginx-operator`
 - vérifier que le nombre de pods et le port ont bien changés:
```bash
$ kubectl get pod,svc  -n test-nginx-operator
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-557d859bff-g5zqg   1/1     Running   0          6m25s
pod/nginx-deployment-557d859bff-s5v5t   1/1     Running   0          11s

NAME                    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/nginx-service   NodePort   10.3.237.138   <none>        80:30081/TCP   6m27s
```
 - tester dans un navigateur ou par un curl l'accès à `http://<node external ip>:30081`

## 👀 Watch service deletion
 - la branche `05-watch-service-deletion` contient le résultat de cette étape
 - supprimer le service : `kubectl delete svc/nginx-service -n test-nginx-operator`
 - constater qu'il est recréé: `kubectl get svc  -n test-nginx-operator`
```bash
$ kubectl get svc  -n test-nginx-operator
NAME            TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.3.148.182   <none>        80:30081/TCP   93s
```
 - supprimer la CR : `kubectl delete ngnixoperatorhelmcharts.charts.fr.wilda/ngnixoperatorhelmchart-sample -n test-nginx-operator`
 - vérifier que tout a été supprimé:
```bash
$ kubectl get pod,svc  -n test-nginx-operator
No resources found in test-nginx-operator namespace.
```

## 🐳 Packaging & deployment to K8s
 - la branche `06-package-deploy` contient le résultat de cette étape
 - modifier le Makefile:
```makefile
## unmodified code ...

IMAGE_TAG_BASE ?= wilda/helm-operator-template

## unmodified code ...

IMG ?= $(IMAGE_TAG_BASE):$(VERSION)

## unmodified code ...

.PHONY: docker-build
docker-build: ## Build docker image with the manager.
## ⚠️ A ne modifier que si vous êtes sous MacOs ⚠️
	docker buildx build --platform linux/amd64 -t ${IMG} . 

## unmodified code ...
```
 - lancer la création de l'image: `make docker-build`
 - s'authentifier sur le docker hub : `docker login`
 - push de l'image : `make docker-push`:
```bash
$ make docker-push
docker push wilda/helm-operator-template:0.0.1
The push refers to repository [docker.io/wilda/helm-operator-template]
ba799744b273: Pushed 
c2478431b8aa: Pushed 
b8b93056b2b4: Pushed 
f17b8b04ea46: Pushed 
0f5e46be1279: Pushed 
f67325c917d2: Pushed 
8fdd60a624e2: Pushed 
0.0.1: digest: sha256:4cbaf35830abe9f37dd109b7e2cdfc7b5ea67efdc73e85015ad7e9e8799b8582 size: 1778
```
 - déployer l'opérateur dans Kubernetes : `make deploy`:
```bash
$ kubectl get deployment -n helm-operator-template-system

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
helm-operator-template-controller-manager   1/1     1            1           5m57s
```
 - créer la CR : `kubectl apply -f ./config/samples/charts_v1_ngnixoperatorhelmchart.yaml -n test-nginx-operator`
 - vérifier que l'opérateur a fait le nécessaire: `kubectl get pod,svc  -n test-nginx-operator`
```bash
$ kubectl get pod,svc  -n test-nginx-operator
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-557d859bff-f62vh   1/1     Running   0          20s
pod/nginx-deployment-557d859bff-ht8hh   1/1     Running   0          20s

NAME                    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/nginx-service   NodePort   10.3.100.194   <none>        80:30081/TCP   21s
```
 - supprimer la CR : `kubectl delete ngnixoperatorhelmcharts.charts.fr.wilda/ngnixoperatorhelmchart-sample -n test-nginx-operator`
 - undeploy de l'opérateur : `make undeploy`
 - supprimer les namespaces: `kubectl delete ns test-nginx-operator`