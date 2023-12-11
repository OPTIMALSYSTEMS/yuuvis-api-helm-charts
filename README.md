# YUUVIS API HELM CHARTS

Yuuvis Api Helm Charts are tool for accelerated development of tailored content and information management solutions.
Solutions build using Yuuvis Api Helm Charts are highly scalable, run either cloud native or on premises and exhibit outstanding performance.

## Table of Contents

* [Prerequisites](#prerequisites)
  * [Installation](#installation)
    - [Add required Helm repositorys:](#add-required-helm-repositorys-)
    + [Install the infrastructure Helm chart](#install-the-infrastructure-helm-chart)
    + [Install the yuuvis Helm chart](#install-the-yuuvis-helm-chart)
    + [Install the yuuvis client Helm chart](#install-the-yuuvis-client-helm-chart)
    + [Install the yuuvis bpm Helm chart](#install-the-yuuvis-bpm-helm-chart)
    + [Install the yuuvis rendition Helm chart](#install-the-yuuvis-rendition-helm-chart)
    + [Install the yuuvis management Helm chart](#install-the-yuuvis-management-helm-chart)
    + [Install the yuuvis repositorymanager Helm chart](#install-the-yuuvis-repositorymanager-helm-chart)
  * [Version upgrades](#version-upgrades)
    + [2021 autumn](#2021-autumn)
    + [2021 summer](#2021-summer)
	+ [2022 winter](#2022-winter)
  * [Uninstall](#uninstall)
- [License](#license)


## Prerequisites 

Please use helm version 3.

## Installation

First please **add your credentials for the docker.yuuvis.org** registry in **the values yaml** files of the helm charts.  For any questions about credentials please contact support@yuuvis.com.

Replace all **changeme** default passwords in the values.yaml of the charts you plan to use.   

**Important: an helm update with the infrastructure chart is not supported."

#### Add required Helm repositorys:

```shell
helm repo add minio https://charts.min.io/
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add gitea-charts https://dl.gitea.io/charts/
helm repo add codecentric https://codecentric.github.io/helm-charts/
```

### test realm init change

With the infrastructure chart version *0.12.1* and yuuvis chart version *0.15.3* the initialization of the test realm is changed.  
The infrastructure chart will not import/create *testyuuvis* and *yuuvistest* realms anymore.  
Existing realms are not changed.  
The yuuvis config init job is changed and will now optionally create a test realm with the name and user from the yuuvis values yaml.  
This job is only run once.  

### Install the infrastructure Helm chart

**Update infrastructure dependencies**

```shell
cd infrastructure
helm dep up
helm repo add stable https://charts.helm.sh/stable
cd ..
```

**Edit the infrastructure values.yaml** 

* Edit the docker registry credentials. 
* Optionally change passwords
* Optionally change the used storage classes

*Since version 0.9.0 of the infrastructure helm chart gitea is used as an example git server.*

*Since version 0.11.0 of the infrastructure helm chart the minio https://charts.min.io repository is used.*

*Since version 0.12.0 of the infrastructure helm chart the codecentric keycloakx helm chart is used.*

**Install infrastructure services**

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
gitea-init                        1/1           83s        8m4s
keycloak-create-selfsigned-cert   1/1           8m4s       8m4s
```

#### Changes with version 0.12

Starting with version *0.12.0* of the infrastructure helm chart the codecentric keycloakx helm chart is used.  
Thus the configuration paramters for the keycloak changed.  
The yuuvis api version *2022 winter* uses keycloak1 version 19.  

### Install the yuuvis Helm chart

**Edit the yuuvis values.yaml and docker registry credentials**

```shell
kubectl create namespace yuuvis
helm install yuuvis ./yuuvis --namespace yuuvis
```

wait till all pods are ready 

```shell
kubectl get po -n yuuvis
```

### Install the yuuvis client Helm chart

**Edit the client values.yaml and docker registry credentials**

*With version 0.6.0 of the client helm chart an app systemHookConfiguration.json is used for the sothook. The global systemHookConfiguration.json is no longer used/changed by the init script.*

```shell
helm install client ./client --namespace yuuvis
```

wait till all pods are ready 

```shell
kubectl get po -n yuuvis
```

**Post-install tasks for the client**

The client helm chart will change the *systemHookConfiguration.json*.  
Services that use this configuration will only read it once at startup.  
For the changes to be noticeable the corresponding services must be restart.  
The changes in the *systemHookConfiguration.json* affect the api gateway.  
To restart the api gateway:  

```shell
kubectl rollout restart deployment api -n yuuvis
```

A role *YUUVIS_CREATE_OBJECT* must be created and assigned to users who should be able to create objects in the client.


### Install the yuuvis bpm Helm chart

**Edit the bpm values.yaml and docker registry credentials**

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

### Install the yuuvis repositorymanager Helm chart

**Edit the repositorymanager values.yaml and docker registry credentials**

The repositorymanager is connector that provides solution for **ArchiveLink** and **ILM** protocols of SAP.

_It is possible to have **more than one instance** of repositorymanager.
To use that possibility repositorymanager will not be part of yuuvis namespace and for **every instance** it is needed to be created **new namespace**._

> **_NOTE: CORS Ingress_**
In Ingress controller because of communication with SAP protocols, please disable CORS e.g.  nginx.ingress.kubernetes.io/enable-cors: "false", or if you use cloud provider you should disable there.

Install the yuuvis repositorymanager chart:

```shell

# Check if yuuvis core services running
kubectl get po -n yuuvis

# For every instance create new namespace e.g. xxxxx
kubectl create namespace xxxxx

# Make sure correct values are set in values.yml (credentials, ports, profile, tenant...)
helm install repositorymanager ./repositorymanager --namespace xxxxx 
```

## Version upgrades

The upgrade of the infrastructure chart is not supported at the moment.

For upgrading the yuuvis or monitoring components get the new Helm charts version, edit the values.yaml of each chart with your modifications and the upgrade the Helm deployments:

Check version of deployed helm chart

```shell
helm list -n yuuvis 
helm list -n monitoring
```

```shell
helm upgrade yuuvis ./yuuvis --namespace yuuvis 
helm upgrade client ./client --namespace yuuvis 
helm upgrade bpm ./bpm --namespace yuuvis
helm upgrade monitoring ./monitoring --namespace monitoring 
helm upgrade repositorymanager ./repositorymanager --namespace xxxxx
```
Check version of upgraded helm chart

```shell
helm list -n yuuvis 
```

### 2023 spring

With version *2023 spring* the management helm chart has been removed.  
Before updating to *2023 spring* please delete the helm chart with the previous used version.  

```shell
helm del management  --namespace yuuvis
``` 

Since version *2022 winter* the tenant-management-api service is required for the client.  
Thus the service is moved into the client helm chart.  
The metricsservice is depcrecated.  
For this release the metricsservice is included in the client helm chart.  

### 2022 winter

With version *2022 winter* yuuvis api uses keycloak 19.  
It is required to manually adjust the endSessionUri parameter for each tenant in the *application-oauth2.yml* configuration file.  
The previously used parameter *redirect_uri* must be removed.  

Further the db connection format used in the *application-dbs.yml* changed.  


More information can be found here:
[yuuvis 2022 winter changes](https://yuuvisdevelop.atlassian.net/wiki/spaces/YMY/pages/320047771/Breaking+Changes)

### 2021 winter and 2022spring

With the yuuvis helm chart version *0.14.0* and the docker tags *4.9.9 (2021winter)* and *4.10.1 (2022spring)* the functionality of the configuration service is changed.  
Starting with these versions the configservice applies all changes to configuration files to its local resources first. At regular intervals of 5 minutes, the remote resources on the git server are synchronized.  
Thus since version *0.14.0* of the yuuvis helm chart the configuration service is deployed as an statefulset.  
For more informations on the change, please refer to the documentaion at: 
[configservice changes](https://help.optimal-systems.com/yuuvis_develop/display/YM21WI/Breaking+Changes)  

More information on the configuration of the configservice can be found here:
[configservice config](https://help.optimal-systems.com/yuuvis_develop/display/YM21WI/CONFIGSERVICE)


### 2021 autumn

The example git service in the infrastructure helm chart is changed from gogs to gitea.  

In the management helm charts the deployments and services are renamed to match the docker image names.  


### 2021 summer

The configuration files will not be changed during an upgrade.  
Please follow the instructions provied at:

* [breaking changes](https://help.optimal-systems.com/yuuvis_develop/display/YMY/Breaking+Changes)
* [update instructions 2021 summer version](https://help.optimal-systems.com/yuuvis_develop/display/YMY/Update+Instructions+2021+Summer)

With the 2021 summer version the webhook type *dms.request.update.metadata* is deprecated.  
The type is still functional in this version, but will be removed in later versions.  
Please migrate your config to use the new webhook type *dms.request.objects.upsert.storage-before*.  

[deprecated webhook](https://help.optimal-systems.com/yuuvis_develop/pages/viewpage.action?pageId=40144034)

## Installing the Monitoring Helm chart

Installing monitoring chart
```
helm dep up monitoring
helm install monitoring ./monitoring -n monitoring --create-namespace --debug
```

Further information on configuration and available dashboards can be found in the [monitoring module readme](monitoring/README.md).

If the follwing error appears:
```shell
Error: unable to build kubernetes objects from release manifest: error validating "": error validating data: [
  ValidationError(Prometheus.spec): unknown field "hostNetwork" in com.coreos.monitoring.v1.Prometheus.spec,
  ValidationError(Prometheus.spec): unknown field "tsdb" in com.coreos.monitoring.v1.Prometheus.spec
]
```

Execute the following commands and deploy the monitoring helm charts again
```shell
kubectl replace -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.63.0/example/prometheus-operator-crd/monitoring.coreos.com_alertmanagerconfigs.yaml
kubectl replace -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.63.0/example/prometheus-operator-crd/monitoring.coreos.com_alertmanagers.yaml
kubectl replace -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.63.0/example/prometheus-operator-crd/monitoring.coreos.com_podmonitors.yaml
kubectl replace -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.63.0/example/prometheus-operator-crd/monitoring.coreos.com_probes.yaml
kubectl replace -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.63.0/example/prometheus-operator-crd/monitoring.coreos.com_prometheuses.yaml
kubectl replace -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.63.0/example/prometheus-operator-crd/monitoring.coreos.com_prometheusrules.yaml
kubectl replace -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.63.0/example/prometheus-operator-crd/monitoring.coreos.com_servicemonitors.yaml
kubectl replace -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.63.0/example/prometheus-operator-crd/monitoring.coreos.com_thanosrulers.yaml
```

## Uninstall

```shell
 helm uninstall infrastructure  --namespace infrastructure
 helm uninstall prometheus-operator --namespace infrastructure
 helm uninstall yuuvis  --namespace yuuvis
 helm uninstall client  --namespace yuuvis
 helm uninstall bpm  --namespace yuuvis
 helm uninstall repositorymanager  --namespace xxxx
 helm uninstall monitoring  --namespace monitoring
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


# License

Copyright 2022 OPTIMAL SYSTEMS GmbH

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
