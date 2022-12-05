# Monitoring

## Configuration
```
monitoring:
  address: grafana.${DNS_SUFFIX} 

grafana:
  adminPassword: changeme
  service:
    enabled: true 
```
<em>monitoring.address</em>: DNS record for accessing Grafana-UI

<em>adminPassword</em>: Password for login to grafana UI

<em>service.enabled</em>: Enabling/disabling Nodeport service for accessing grafana UI

## Dashboards

Dashboards available for import:
```
momentum/monitoring/dashboards/momentum-application-instances-(pods).json 
momentum/monitoring/dashboards/momentum-applications-(services).json 
```
Momentum Application Instances (pods): Displaying Momentum pod metrics.
Momentum Applications (services): Displaying Momentumg service metrics.
