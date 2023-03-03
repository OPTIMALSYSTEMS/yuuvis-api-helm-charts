# REPOSITORYMANAGER HELM CHART

Repositorymanager has ClusterIp type of port that means that outside of cluster it is not reachable. 
We suggest that depends on cloud provider you should choose to use cloud provider load balancer or to implement appropriate Ingress controller.
In next section we show basic nginx Ingress controller implementation, that can help if nginx Ingress controller suits to your needs.

> **_NOTE: CORS Ingress_**
In Ingress controller because of communication with SAP protocols, please disable CORS e.g.  nginx.ingress.kubernetes.io/enable-cors: "false", or if you use cloud provider you should disable there.

##Example of Nginx Ingress controller 

1. Add helm repository for nginx Ingress controller and repository for cert-manager

```shell
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo add jetstack https://charts.jetstack.io
helm repo update
```
2. Install cert-manager to automate TLS certificate management and install ingress

```shell
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.8.0 \
  --set installCRDs=true
  
helm install nginx-ingress nginx-stable/nginx-ingress --set rbac.create=true

# Validate that nginx is running
kubectl get pods --all-namespaces -l app=nginx-ingress-nginx-ingress
  
```
3. Change values in yaml file and add it to template folder 

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: repositorymanager.yourdns.net # Change value
spec:
  secretName: repositorymanager-yourdns-net-cert # Change value
  dnsNames:
    - repositorymanager.yourdns.net # Change value
  issuerRef:
    group: cert-manager.io
    name: letsencrypt-prod
    kind: ClusterIssuer
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    ingress.kubernetes.io/ssl-redirect: "true"
    meta.helm.sh/release-name: repositorymanager
    meta.helm.sh/release-namespace: repositorymanagerwinter # Change value
    nginx.ingress.kubernetes.io/enable-cors: "false"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /
  name: repositorymanager-ingress
spec:
  rules:
    - host: repositorymanager.yourdns.net # Change value
      http:
        paths:
          - backend:
              service:
                name: repositorymanager
                port:
                  number: 80
            path: /
            pathType: ImplementationSpecific
  tls:
  - hosts:
      - repositorymanager.yourdns.net # Change value
    secretName: repositorymanager-yourdns-net-cert # Change value

```