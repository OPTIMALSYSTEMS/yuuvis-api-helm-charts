# License

Copyright 2021 OPTIMAL SYSTEMS GmbH

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

# yuuvis api helm charts

## Prerequisites

Please use helm version [v3.2.4](https://github.com/helm/helm/releases/tag/v3.2.4), newer versions may not be compatible with some of the helm charts.

## yuuvis installation

First please add your credentials for the docker.yuuvis.org registry in the values yaml files of the helm charts.  For any questions about credentials please contact support@yuuvis.com.

Replace all **changeme** default passwords in the values.yaml of the charts you plan to use.   

### Add required Helm repositorys:

```shell
helm repo add minio https://helm.min.io/
helm repo add bitnami https://charts.bitnami.com/bitnami
```
### Install the infrastructure Helm chart

#### Update infrastructure dependencies

```shell
cd infrastructure
helm dep up
helm repo add stable https://charts.helm.sh/stable
cd ..
```

#### Edit the infrastructure values.yaml

* Edit the docker registry credentials. 
* Optionally change passwords
* Optionally change the used storage classes

#### Install infrastructure services

```shell
kubectl create namespace infrastructure
helm install infrastructure ./infrastructure --namespace infrastructure
```

wait till jobs are done

```shell
kubectl get jobs -n infrastructure
```

There are 2 jobs that prepare the git server and the keycloak environment that need to be completed.

```shell
NAME                              COMPLETIONS   DURATION   AGE
gogsrepo-init                     1/1           83s        8m4s
keycloak-create-selfsigned-cert   1/1           8m4s       8m4s
```

#### Installing Monitoring services

Installing Prometheus Operator
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/release-0.38/example/prometheus-operator-crd/monitoring.coreos.com_alertmanagers.yaml
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/release-0.38/example/prometheus-operator-crd/monitoring.coreos.com_podmonitors.yaml
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/release-0.38/example/prometheus-operator-crd/monitoring.coreos.com_prometheuses.yaml
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/release-0.38/example/prometheus-operator-crd/monitoring.coreos.com_prometheusrules.yaml
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/release-0.38/example/prometheus-operator-crd/monitoring.coreos.com_servicemonitors.yaml
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/release-0.38/example/prometheus-operator-crd/monitoring.coreos.com_thanosrulers.yaml

helm install prometheus-operator prometheus-community/prometheus-operator  --namespace infrastructure --values ./monitoring/values.yaml --set prometheusOperator.createCustomResource=false
```

Installing the monitoring chart
```shell
helm install monitoring ./monitoring --namespace infrastructure
```

#### Edit the yuuvis values.yaml

* Edit the docker registry credentials.


### Install the yuuvis Helm chart

```shell
kubectl create namespace yuuvis
helm install yuuvis ./yuuvis --namespace yuuvis
```

wait till all pods are ready 

```shell
kubectl get po -n yuuvis
```

#### Edit the client values.yaml

* Edit the docker registry credentials.

### Install the yuuvis client Helm chart:

```shell
helm install client ./client --namespace yuuvis
```

wait till all pods are ready 

```shell
kubectl get po -n yuuvis
```

#### Post-install tasks for the client

The client helm chart will change the *systemHookConfiguration.json*.  
Services that use this configuration will only read it once at startup.  
For the changes to be noticeable the corresponding services must be restart.  
The changes in the *systemHookConfiguration.json* affect the api gateway.  
To restart the api gateway:  

```shell
kubectl rollout restart deployment api -n yuuvis
```

A role *YUUVIS_CREATE_OBJECT* must be created and assigned to users who should be able to create objects in the client.  

#### Edit the bpm values.yaml

* Edit the docker registry credentials.


### Install the yuuvis bpm Helm chart

install bpm services with:
```shell
kubectl get po -n yuuvis
helm install bpm ./bpm --namespace yuuvis
```

### Install the yuuvis rendition Helm chart

install rendition services with:
```shell
kubectl get po -n yuuvis
helm install rendition ./rendition --namespace yuuvis
```

#### Using Grafana 
* Make Grafana pod accessible

```shell
kubectl port-forward prometheus-operator-grafana-xxxxx 3000 -n infrastructure
```

* Dashboards need to be exported via share -> export -> export for sharing externally

## Version upgrades

The upgrade of the infrastructure chart is not supported at the moment.

For upgrading the yuuvis components get the new Helm charts version, edit the values.yaml of each chart with your modifications and the upgrade the Helm deployments:

Check version of deployed helm chart

```shell
helm list -n yuuvis 
```


```shell
helm upgrade yuuvis ./yuuvis --namespace yuuvis 
helm upgrade client ./client --namespace yuuvis 
helm upgrade bpm ./bpm --namespace yuuvis 
```
Check version of upgraded helm chart

```shell
helm list -n yuuvis 
```

## Uninstall

```shell
 helm uninstall infrastructure  --namespace infrastructure
 helm uninstall prometheus-operator --namespace infrastructure
 helm uninstall monitoring  --namespace infrastructure
 helm uninstall yuuvis  --namespace yuuvis
 helm uninstall client  --namespace yuuvis
 helm uninstall bpm  --namespace yuuvis
```

```shell
kubectl delete crd alertmanagerconfigs.monitoring.coreos.com
kubectl delete crd alertmanagers.monitoring.coreos.com
kubectl delete crd podmonitors.monitoring.coreos.com
kubectl delete crd probes.monitoring.coreos.com
kubectl delete crd prometheuses.monitoring.coreos.com
kubectl delete crd prometheusrules.monitoring.coreos.com
kubectl delete crd servicemonitors.monitoring.coreos.com
kubectl delete crd thanosrulers.monitoring.coreos.com
```

```shell
kubectl delete statefulset elasticsearch -n infrastructure
kubectl delete statefulset rabbitmq -n infrastructure
kubectl delete jobs keycloakaddrole-yuuvis  -n infrastructure
kubectl delete jobs keycloak-create-selfsigned-cert -n infrastructure
kubectl delete job gogsrepo-init -n infrastructure
kubectl delete pvc gogs -n infrastructure
kubectl delete pv name(replace with pv from gogs --check value with kubectl get pv -n infrastructure) -n infrastructure
```

Before deleting the persistent volumes and persistent volume claims, please delete corresponding pods.
