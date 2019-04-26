# Blue-Green Deployment with Traefik

Using [Traefik](https://traefik.io/) along with [Fabrikate](https://github.com/microsoft/fabrikate)Fabrikate and Project Jackson, a Blue-Green Deployment can be configured for Project Jackson.

The following instructions have been tested with an [AKS Cluster](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough) but should work with another cloud provider with minor adjustments.

### Requirements
-AKS Cluster
-[Traefik Fabrikate](https://github.com/evanlouie/fabrikate-traefik) component

### Traefik Fabrikate Component
In the `common.yaml` file add a domain to the dashboard and a service annotation.
Use the `service.beta.kubernetes.io/azure-dns-label-name` label. In the `<name>` placeholder specify your own name for example `stagingendpoint`, making the full domain `stagingendpoint.cloudapp.azure.com`.

```
config:
  namespace: traefik
  dashboard:
    enabled: true
    domain: <name>.cloudapp.azure.com
  service:
    annotations: 
      service.beta.kubernetes.io/azure-dns-label-name: <name>
  rbac:
    enabled: true
subcomponents:
```



