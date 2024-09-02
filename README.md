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
    + [Install the yuuvis mailarchiving Helm chart](#install-the-yuuvis-mailarchiving-helm-chart)
    + [Install the yuuvis rendition Helm chart](#install-the-yuuvis-rendition-helm-chart)
    + [Install the yuuvis repositorymanager Helm chart](#install-the-yuuvis-repositorymanager-helm-chart)
    + [Install the yuuvis repositorymanager AL Helm chart](#install-the-yuuvis-repositorymanager-al-helm-chart)
    + [Install the yuuvis repositorymanager ILM Helm chart](#install-the-yuuvis-repositorymanager-ilm-helm-chart)
    + [Install the yuuvis repositorymanager CMIS Helm chart](#install-the-yuuvis-repositorymanager-cmis-helm-chart)
  * [Version upgrades](#version-upgrades)
    + [2023 winter](#2023-winter)
    + [2023 autumn](#2023-autumn)
    + [2023 summer](#2023-summer)
    + [2023 spring](#2023-spring)
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
The yuuvis api version *2022 winter* uses keycloak version 19.  

#### Changes with version 0.13

The default configuration of keycloak is changed.  
In previous versions two test realms were imported *testyuuvis* and *yuuvistest*.  
Since version *0.13.0* no realms will be imported by default.  
In the yuuvis chart starting with version *0.17.0* an init job can be configured to create a realm.  

The versions of the chart dependencies have been updated.  

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

### Install the yuuvis bpm Helm chart

**Edit the bpm values.yaml and docker registry credentials**

install bpm services with:
```shell
kubectl get po -n yuuvis
helm install bpm ./bpm --namespace yuuvis
```
### Install the yuuvis mailarchiving Helm chart

Installation of the mailarchiving services described in README.md in mailarchiving folder.

### Install the yuuvis rendition Helm chart

install rendition services with:
```shell
kubectl get po -n yuuvis
helm install rendition ./rendition --namespace yuuvis
```

### Install the yuuvis repositorymanager Helm chart

**Edit the repositorymanager values.yaml and docker registry credentials**

```shell
# Check if yuuvis core services running
kubectl get po -n yuuvis

# For every instance create new namespace e.g. repositorymanager
kubectl create namespace repositorymanager

# Make sure correct values are set in values.yml (credentials, ports, profile, tenant...)
helm install repositorymanager ./repositorymanager --namespace repositorymanager 
```

_It is possible to have **more than one instance** of repositorymanager.
To use that possibility repositorymanager will not be part of yuuvis namespace and for **every instance** it is needed to be created **new namespace**._

> **_NOTE: CORS Ingress_**
In Ingress controller because of communication with SAP protocols, please disable CORS e.g.  nginx.ingress.kubernetes.io/enable-cors: "false", or if you use cloud provider you should disable there.

> **NOTE : Update/Upgrade Repository Manager from artifact (docker image tag) 4.3.3**
> If, in the webapps/cs folder, one of the default folders is missing (e.g., conf, META-INF, and/or WEB-INF), the missing ones will be extracted during the installation/upgrades of the repository manager.
Please check whether this step is advised through the **RELEASE NOTES**; for example:
If the KGS version is not compatible with an old version, then delete the WEB-INF folder before upgrading to a new version of the repository manager (old configuration will remain).

### Install the yuuvis repositorymanager AL Helm chart

```shell
# Check if yuuvis core services running (namespace for core yuuvis services can have another name e.g. 2023winter)
kubectl get po -n yuuvis
  
# Create namespace for repositorymanager AL service e.g. repositorymanageral
kubectl create namespace repositorymanageral
  
# Before running please make sure that correct values are set in values.yml
helm install repositorymanageral ./repositorymanageral --namespace repositorymanageral
```
### Install the yuuvis repositorymanager ILM Helm chart

```shell
# Check if yuuvis core services running (namespace for core yuuvis services can have another name e.g. 2023winter)
kubectl get po -n yuuvis
  
# Create namespace for repositorymanager ILM service e.g. repositorymanagerilm
kubectl create namespace repositorymanagerilm
  
# Before running please make sure that correct values are set in values.yml
helm install repositorymanagerilm ./repositorymanagerilm --namespace repositorymanagerilm
```
### Install the yuuvis repositorymanager CMIS Helm chart

```shell
# Check if yuuvis core services running (namespace for core yuuvis services can have another name e.g. 2023winter)
kubectl get po -n yuuvis
  
# Create namespace for repositorymanager CMIS service e.g. repositorymanagercmis
kubectl create namespace repositorymanagercmis
  
# Before running please make sure that correct values are set in values.yml
helm install repositorymanagercmis ./repositorymanagercmis --namespace repositorymanagercmis
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
helm upgrade repositorymanager ./repositorymanager --namespace repositorymanager
helm upgrade repositorymanagerilm ./repositorymanagerilm --namespace repositorymanagerilm
helm upgrade repositorymanagercmis ./repositorymanagercmis --namespace repositorymanagercmis
```
Check version of upgraded helm chart

```shell
helm list -n yuuvis 
```

### 2023 autumn

With version *2023 autumn* yuuvis api uses Keycloak 22.  
Since Keycloak version *19.0.2* a *scope* parameter is mandatory in the oauth2 client configuration.  
See Keycloak documentation *user-endpoint-changes - Other Changes*.  
[keycloak openid required](https://www.keycloak.org/docs/latest/upgrading/index.html#userinfo-endpoint-changes)  
Since Keycloak version 20 login will fail without the scope parameter.  

The yuuvis momentum elasticsearch connection configuration is changed with *2023autumn*.  

More information can be found here:  
[yuuvis 2023 autumn changes](https://yuuvisdevelop.atlassian.net/wiki/spaces/YMY/pages/320047771/Breaking+Changes)

An optional update helm upgrade job *pre-upgrade-job-2023autumn* is provided with the yuuvis helm chart.  
The job will run during a helm upgrade before after the templates are rendered and before kubernetes resources are changed.  
[helm upgrade hooks](https://helm.sh/docs/topics/charts_hooks/#the-available-hooks)  
The update job can be enabled/disabled in the yuuvis values yaml.  
```yaml
yuuvis:
  update:
    autumn2023:
      enable: true
```

If configured the job will try to load the application-oauth2.yml and add 
the paramter *scope: openid* to the configurations if not present.  

Optionally the update job will load the application-es.yml and map the parameters to the new format.  
This job assumes the existing application-es.yml used in previous helm chart versions.  

With *2023 autumn* the metricsservice is removed.  

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

Copyright 2023 OPTIMAL SYSTEMS GmbH

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
