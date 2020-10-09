# License

Copyright 2020 OPTIMAL SYSTEMS GmbH

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

## install

First please add your credentials for the docker.yuuvis.org registry in the values yaml files of the helm charts.  

```shell
cd infrastructure
helm dep up
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
cd ..
kubectl create namespace infrastructure
helm install infrastructure ./infrastructure  --set yuuvis.authentication.ip=(CLUSTER_IP or LOAD_BALANCER_IP von authentication) --namespace infrastructure
```

wait till jobs are done (`kubectl get jobs -n infrastructure`)

```shell
kubectl get po -n infrastructure
kubectl create namespace yuuvis
helm install yuuvis ./yuuvis --set yuuvis.keycloak.ip=(CLUSTER_IP or LOAD_BALANCER_IP von keycloak) --namespace yuuvis
```

wait till all pods are ready (`kubectl get pods -n yuuvis`)

```shell
kubectl get po -n yuuvis
helm install client ./client --namespace yuuvis
```

## uninstall

```shell
 helm uninstall infrastructure  --namespace infrastructure
 helm uninstall yuuvis  --namespace yuuvis
 helm uninstall client  --namespace yuuvis
```

## upgrade

```shell
cd infrastructure
helm dep up
cd ..

helm upgrade yuuvis ./yuuvis --namespace yuuvis 
```
Check version of deployed helm chart
```shell
helm list -n yuuvis 
```

The upgrade of the infrastructure chart is not supported at the moment.