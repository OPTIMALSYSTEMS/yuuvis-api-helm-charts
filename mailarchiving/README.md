### Install the yuuvis mailarchiving Helm chart

The mailarchiving consists of three services:
* mailarchivingsmtp
* mailarchivingmailbox
* mailarchivingstorage

Helm charts constructed in such a way that each service can be installed independently. 
In order for *mailarchivingsmtp* or *mailarchivingmailbox* to work *mailarchivingstorage* must be installed.
Installing **mailarchivingstorage** will create application-mas.yml.
That configuration file is required for services mas-mailbox and mas-storage to work together, where mailarchivingmailbox and mailarchivingstorage are relevant helm charts.

### Install the yuuvis mailarchivingstorage Helm chart

Please change needed values in **mailarchiving/mailarchivingstorage/values.yml**

```shell
# Check if yuuvis core services running (namespace yuuvis is example, yuuvis core services can be in namespace with another name)
kubectl get po -n yuuvis

# For every instance create new namespace e.g. xxxxx
kubectl create namespace xxxxx

# Make sure correct values are set in values.yml (credentials, ports, profile, tenant...)
helm install mailarchivingstorage ./mailarchiving/mailarchivingstorage --namespace xxxxx 
```

### Install the yuuvis mailarchivingsmtp Helm chart

Please change needed values in **mailarchiving/mailarchivingsmtp/values.yml**

```shell
# Check if yuuvis core services running (namespace yuuvis is example, yuuvis core services can be in namespace with another name)
kubectl get po -n yuuvis 

# Check if storage service running in xxxxx namespace
kubectl get po -n xxxxx

# Make sure correct values are set in values.yml (credentials, ports, profile, tenant...)
helm install mailarchivingsmtp ./mailarchiving/mailarchivingsmtp --namespace xxxxx 
```

### Install the yuuvis mailarchivingmailbox Helm chart

Please change needed values in **mailarchiving/mailarchivingmailbox/values.yml**

```shell
# Check if yuuvis core services running (namespace yuuvis is example, yuuvis core services can be in namespace with another name)
kubectl get po -n yuuvis

# Check if storage service running in xxxxx namespace
kubectl get po -n xxxxx

# Make sure correct values are set in values.yml (credentials, ports, profile, tenant...)
helm install mailarchivingmailbox ./mailarchiving/mailarchivingmailbox --namespace xxxxx 
```
