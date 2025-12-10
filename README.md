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
| client              | services for a reference [client](https://help.optimal-systems.com/yuuvis/Momentum/latest/index.html#_client_development) and [tenant management](https://help.optimal-systems.com/yuuvis/Momentum/latest/index.html#_administration_tools) and bpm/workflow engine [yuuvis momentum bpm](https://help.optimal-systems.com/yuuvis/Momentum/latest/index.html#_business_process_management) |
| mailarchiving       | e-mail archive solution with yuuvis as storage system [yuuvis mail archive](https://help.optimal-systems.com/yuuvis/Momentum/latest/index.html#_e_mail_archive) |


## Table of Contents

* [Prerequisites](#prerequisites)
  * [Installation](#installation)
    - [Add required Helm repositorys:](#add-required-helm-repositorys-)
    + [Install steps for the infrastructure Helm chart](#install-the-infrastructure-helm-chart)
    + [Install steps for the yuuvis Helm chart](#install-the-yuuvis-helm-chart)
    + [Install steps for the yuuvis client Helm chart](#install-the-yuuvis-client-helm-chart)
    + [Install steps for the yuuvis rendition Helm chart](#install-the-yuuvis-rendition-helm-chart)
    + [Install steps for the yuuvis mailarchiving Helm chart](#install-the-yuuvis-mailarchiving-helm-chart)
  * [Uninstall](#uninstall)
  * [Changelog](#changelog)
    + [2025 spring client changes](#2025spring-client-changes)
- [License](#license)


## Prerequisites 

Please use helm version 3.

With *2025summer* the bpm chart was integrated into the client chart since the client 'clients' depends 
on the bpm-engine.  

With *2025spring* the helm client helm chart is signifcantly changed.  
A new client is available.   
The init jobs for the client services require an yuuvis user.   
Please refer to section [2025spring client changes](#2025spring-client-changes) for more information.   

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

The init jobs for the client services require a yuuvis user 
(*tenant*, *username*, and *password*) to configure the
system via the yuuvis api.   
Adjust the parameters in the values.yaml before installing the helm chart.   

To use an existing secret set *yuuvis.init.configuser.createnewsecret* to *false* 
and set the *yuuvis.init.configuser.secretname*.   

```yaml
yuuvis:
  init:
    configuser:
      secretname: configure-apps
      createnewsecret: true
      tenant: myfirsttenant
      username: root
      password: changeme
```

The new client (clients with 's') uses the bpm-engine.  
By default the init job tries to add an bpm workflow.  
This requires a running instance of the bpm-engine.  
If you plan to use this client and feature you have to install the bpm chart first.    
The configuration of the workflow is controlled in the values yaml with:    

```yaml
yuuvis:
  taskflow:
    enabled: true
```

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

### Install steps for the yuuvis rendition Helm chart

install rendition services with:
```shell
kubectl get po -n yuuvis
helm install rendition ./rendition --namespace yuuvis
```

### Install steps for the yuuvis mailarchiving Helm chart

Installation of the mailarchiving services described in README.md in mailarchiving folder.

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
```
Check version of upgraded helm chart

```shell
helm list -n yuuvis 
```

## Uninstall

```shell
 helm uninstall infrastructure  --namespace infrastructure
 helm uninstall prometheus-operator --namespace infrastructure
 helm uninstall yuuvis  --namespace yuuvis
 helm uninstall client  --namespace yuuvis
 helm uninstall bpm  --namespace yuuvis
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

### 2025spring client changes

The client chart is significantly changed with 2025spring.  
A new client (clients with 's') is available.   
In the client deployments an init container is used to add
the required configuration to the yuuvis system.  

It is recommanded to install the *bpm* chart before the *client* chart.  
If *yuuvis.taskflow.enabled* is **true** (by default) the bpm engine is **required**.   
```yaml
yuuvis:
  taskflow:
    enabled: true
```

The init jobs for the client services require an yuuvis user.   
In previous version of the helm chart, the required configuration files for the clients were added 
directly into the git.   
With this version the yuuvis api endpoints are used.   
For that an yuuvis user is required.    
It is possible to use an existing secret with the keys *tenant*, *username*, and *password*.   
To use an existing secret *yuuvis.init.configuser.createnewsecret* to *false* 
and set the *yuuvis.init.configuser.secretname*.   

```yaml
yuuvis:
  init:
    configuser:
      secretname: configure-apps
      createnewsecret: true
      tenant: myfirsttenant
      username: root
      password: changeme
```

## bpm chart


| Version | Changes                                                                                                                                            |
| ------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0.8.0   | the bpm-admin is removed, added security contexts for the deployment |


# License

Copyright 2025 OPTIMAL SYSTEMS GmbH

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
