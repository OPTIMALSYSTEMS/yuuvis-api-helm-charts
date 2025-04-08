# YUUVIS API HELM CHARTS

Yuuvis Api Helm Charts are a tool for accelerated development of tailored content and information management solutions.
Solutions build using Yuuvis Api Helm Charts are highly scalable, run either cloud native or on premises.

This repository contains a collection of helm charts for different components of yuuvis api.  
The helm charts are grouped by functionality.  
*Before* installation please check which components are required for your usecase.  
Usually a subset of components is sufficient.  

| helm chart          | yuuvis component                                                |
| ------------------- | --------------------------------------------------------------- |
| infrastructure      | examples for yuuvis dependencies such as database, redis, message queue, identiy provider etc. [dependencies](https://help.optimal-systems.com/yuuvis/Momentum/latest/index.html#_requirements_and_dependencies) |
| yuuvis              | core components of yuuvis api systems  [core services](https://help.optimal-systems.com/yuuvis/Momentum/latest/index.html#_overview_core_services) |
| rendition           | services for providing renditions and async textextraction [content_renditions](https://help.optimal-systems.com/yuuvis/Momentum/latest/index.html#_content_renditions) |
| client              | services for a reference [client](https://help.optimal-systems.com/yuuvis/Momentum/latest/index.html#_client_development) and [tenant management](https://help.optimal-systems.com/yuuvis/Momentum/latest/index.html#_administration_tools) |
| bpm                 | bpm/workflow engine [yuuvis momentum bpm](https://help.optimal-systems.com/yuuvis/Momentum/latest/index.html#_business_process_management)|
| mailarchiving       | e-mail archive solution with yuuvis as storage system [yuuvis mail archive](https://help.optimal-systems.com/yuuvis/Momentum/latest/index.html#_e_mail_archive) |
| repositorymanager   | different implementation of sap integration, for more info see [yuuvis momentum sap integration](https://help.optimal-systems.com/yuuvis/Momentum/latest/index.html#_sap_integration) |


## Table of Contents

* [Prerequisites](#prerequisites)
  * [Installation](#installation)
    - [Add required Helm repositorys:](#add-required-helm-repositorys-)
    + [Install steps for the infrastructure Helm chart](#install-the-infrastructure-helm-chart)
    + [Install steps for the yuuvis Helm chart](#install-the-yuuvis-helm-chart)
    + [Install steps for the yuuvis client Helm chart](#install-the-yuuvis-client-helm-chart)
    + [Install steps for the yuuvis bpm Helm chart](#install-the-yuuvis-bpm-helm-chart)
	+ [Install steps for the yuuvis rendition Helm chart](#install-the-yuuvis-rendition-helm-chart)
    + [Install steps for the yuuvis mailarchiving Helm chart](#install-the-yuuvis-mailarchiving-helm-chart)
    + [Install steps for the yuuvis repositorymanager Helm chart](#install-the-yuuvis-repositorymanager-helm-chart)
  * [Yuuvis version upgrades](#yuuvis-version-upgrades)
    + [2023 autumn](#2023-autumn)
    + [2023 summer](#2023-summer)
    + [2023 spring](#2023-spring)
	+ [2022 winter](#2022-winter)
  * [Uninstall](#uninstall)
  * [Changelog](#changelog)
- [License](#license)


## Prerequisites 

Please use helm version 3.

## Installation

First please **add your credentials for the docker.yuuvis.org** registry in **the values yaml** files of the helm charts.  For any questions about credentials please contact support@yuuvis.com.

Replace all **changeme** default passwords in the values.yaml of the charts you plan to use.   

**Important: an helm update with the infrastructure chart is not supported."**

#### Add required Helm repositorys:

```shell
helm repo add minio https://charts.min.io/
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add gitea-charts https://dl.gitea.io/charts/
helm repo add codecentric https://codecentric.github.io/helm-charts/
```

### Install steps for the infrastructure Helm chart

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


### Install steps for the yuuvis Helm chart

**Edit the yuuvis values.yaml and docker registry credentials**

```shell
kubectl create namespace yuuvis
helm install yuuvis ./yuuvis --namespace yuuvis
```

wait till all pods are ready 

```shell
kubectl get po -n yuuvis
```

### Install steps for the yuuvis client Helm chart

**Edit the client values.yaml and docker registry credentials**


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

### Install steps for the yuuvis bpm Helm chart

install bpm services with:
```shell
kubectl get po -n yuuvis
helm install bpm ./bpm --namespace yuuvis
```

### Install steps for the yuuvis rendition Helm chart

install rendition services with:
```shell
kubectl get po -n yuuvis
helm install rendition ./rendition --namespace yuuvis
```

### Install steps for the yuuvis mailarchiving Helm chart

Installation of the mailarchiving services described in README.md in mailarchiving folder.

### Install steps for the yuuvis repositorymanager Helm chart

> **_DEPRECATED:_**
> Repositorymanager is deprecated as of 2024 winter version. Please contact support regarding migration to new repositorymanager 5 version.

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


## Install steps for the monitoring helm chart

Installing monitoring chart
```
helm dep up monitoring
helm install monitoring ./monitoring -n monitoring --create-namespace --debug
```

Further information on configuration and available dashboards can be found in the [monitoring module readme](monitoring/README.md).


## Yuuvis version upgrades

Detailed informations reagrading the yuuvis update can be found in the documentation in the [update guide](https://help.optimal-systems.com/yuuvis/Momentum/latest/index.html#_update_guide).  

The upgrade of the infrastructure chart is **not supported** at the moment.  
Elasticsearch and Keycloak can be updated using the helm chart.  
For all the other depencencies (postgres, minio, redis, gitea) please refer to the documentation of the corresponding projects.  

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



# Changelog

## infrastructure chart


| Version | Changes                                                                                                                                            |
| ------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0.16.2  | fix default keycloak startup, readiness and liveness probe, if an selfsigned cert is used, the internal scheme must be 'https' |
| 0.16.1  | a switch to enable/disable the creation of a selfsigned cert for keycloak |
| 0.16.0  | the dependency versions are updated, upgrade for postgres, reddis, rabbitmq with the helm chart is not supported, start paramter for keycloak changed |
| 0.12.0  | codecentric keycloakx helm chart is used as a dependency  |
| 0.11.0  | minio https://charts.min.io repository is used as a dependency |
| 0.9.0   | gitea is used as an example git server |

## yuuvis chart


| Version | Changes                                                                                                                                            |
| ------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0.21.3  | update init container image in the system service statefulset |
| 0.21.0  | added security contexts for the deployment, added extra security context for the system statefulset |
| 0.20.0  | init job is removed in favor of an init container in the system service statefulset |
| 0.17.0  | an init job can be configured to create a realm |
| 0.13.0  | default configuration of keycloak is changed, in previous versions two test realms were imported *testyuuvis* and *yuuvistest*, with version *0.13.0* no realms will be imported by default  |
| 0.12.0  | yuuvis api version *2022 winter* uses keycloak version 19, keycloak configuration parameters changed |

## client chart


| Version | Changes                                                                                                                                            |
| ------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0.8.2   | added init job for textextraction (can be used to create a cross tenant user required by the serivce) |
| 0.8.1   | added security context |
| 0.12.0  | added security context, added parameters to enable/disable deployment of services |
| 0.6.0   | an app systemHookConfiguration.json is used for the sothook. The global systemHookConfiguration.json is no longer used/changed by the init script |

## bpm chart


| Version | Changes                                                                                                                                            |
| ------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0.8.0   | the bpm-admin is removed, added security contexts for the deployment |


# License

Copyright 2024 OPTIMAL SYSTEMS GmbH

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
