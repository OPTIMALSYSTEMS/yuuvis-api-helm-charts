### Install the yuuvis mailarchiving Helm chart

The mailarchiving consists of three services:
* mailarchiving-smtp
* mailarchiving-mailbox
* mailarchiving-storage

Helm charts constructed in such a way that each service can be installed independently. 
In order for *mailarchiving-smtp* or *mailarchiving-mailbox* to work *mailarchiving-storage* must be installed.
Installing **mailarchiving-storage** will create application-mas.yml.
That configuration file is required for services mas-mailbox and mas-storage to work together, where mailarchiving-mailbox and mailarchiving-storage are relevant helm charts.

### Preinstallation requirements

```shell
# Check if yuuvis core services running (namespace yuuvis is example, yuuvis core services can be in namespace with another name)
kubectl get po -n yuuvis

# For every instance create new namespace e.g. mailarchiving
kubectl create namespace mailarchiving

# To securely manage sensitive data in the newly created mailarchiving namespace, create Kubernetes secrets for all sensitive values used by your services. 
# In the mailarchiving Helm charts, the following values must be stored as secrets: dms_password, exchange_onprem_secret and exchange_online_secret. 
# Please review the scripts below and add your specific values as needed.
kubectl create secret generic dms-secret-name --from-literal=dms-secret-key="YOUR_DMS_SECRET_VALUE" -n YOUR_NAMESPACE
# If exchange_onprem_secret and exchange_online_secret are not needed, you can skip creating these secrets.
# In values.yml to use these secrets, set in values.yml key enabled to true and set the following values:
kubectl create secret generic exchange-onprem-secret-name --from-literal=exchange-onprem-secret-key="YOUR_ONPREM_SECRET_VALUE" -n YOUR_NAMESPACE
kubectl create secret generic exchange-online-secret-name --from-literal=exchange-online-secret-key="YOUR_ONLINE_SECRET_VALUE" -n YOUR_NAMESPACE
```

### Install the yuuvis mailarchivingstorage Helm chart
Please change needed values in **mailarchiving/storage/values.yml**

```shell
# Make sure correct values are set in values.yml (credentials, ports, profile, tenant...)
helm install mailarchivingstorage ./mailarchiving/storage --namespace mailarchiving 
```

### Install the yuuvis mailarchivingsmtp Helm chart

Please change needed values in **mailarchiving/smtp/values.yml**

```shell
# Check if storage service running in mailarchiving namespace
kubectl get po -n mailarchiving

# Make sure correct values are set in values.yml (credentials, ports, profile, tenant...)
helm install mailarchivingsmtp ./mailarchiving/smtp --namespace mailarchiving 
```

### Install the yuuvis mailarchivingmailbox Helm chart

Please change needed values in **mailarchiving/mailbox/values.yml**

```shell
# Check if storage service running in mailarchiving namespace
kubectl get po -n mailarchiving

# Make sure correct values are set in values.yml (credentials, ports, profile, tenant...)
helm install mailarchivingmailbox ./mailarchiving/mailbox --namespace mailarchiving 
```
