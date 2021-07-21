# Monitoring

## Configuration

```
prometheus:
  prometheusSpec:    
    retention: 10d    
```
<em>retention</em>: Storage time for log data collected by prometheus operator

```
grafana:
  adminPassword: changeme
  service:
    enabled: true 
```
<em>adminPassword</em>: Password for login to grafana UI

<em>service.enabled</em>: Enabling/disabling Nodeport service for accessing grafana UI

## Dashboards

Dashboards are displayed in grafana UI.
To access grafana, enable nodeport service and navigate to port 3000.
```
http://{{ip}}:3000 
```

Dashboards available for import:
```
momentum/monitoring/dashboards/momentum-application-instances-(pods).json 
momentum/monitoring/dashboards/momentum-applications-(services).json 
```
Momentum Application Instances (pods): Oberservation of single yuuvis application instances on pod level.
Momentum Applications (services): Oberservation of yuuvis applications on service level.
