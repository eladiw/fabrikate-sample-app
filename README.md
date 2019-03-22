# sample app

This repository is a high level deployment defintion for Kubernetes. The repo contains [Fabrikate](https://github.com/Microsoft/fabrikate) defintions for a boilerplate [Cloud Native stack](https://github.com/timfpark/fabrikate-cloud-native) and Helm charts for a demo application called [Project Jackson](https://github.com/CatalystCode/kubemalt/tree/master/demo).

## Features
* Istio based canary deployment configuration of the Project Jackson API microservice
* Multicluster support

## How to adjust the canary deployment traffic wights with Fabrikate

1. Download version 0.3.0 or greater of [Fabrikate](https://github.com/Microsoft/fabrikate/releases)
2. From the root of this repository run the following command

`fab set --subcomponent services.project-jackson.jackson-api service.canaryWeight=15`

## How to adjust the canary deployment traffic weights manually

1. Navigate to the [common.yaml](services/project-jackson/jackson-api/config/common.yaml) file.
2. Modifiy the `stableWeight` and `canaryWeight` values. (Below sure they total to 100)
<pre>
config:
  service:
      name: jackson-api
      port: 8080
      labelName: jackson
      canaryWeight: <b>10</b>
      stableWeight: <b>90</b>
  appName: spring-boot-api
  enableVirtualService: true
  enableDestinationRule: true
</pre>

## How to use TLS with Istio Ingress
1. See [Istio TLS Documentation](https://istio.io/docs/tasks/traffic-management/secure-ingress/mount/#configure-a-tls-ingress-gateway-with-a-file-mount-based-approach) to generate ingress certificate secrets, and deploy them to your cluster.

2. Uncomment the [TLS configuration](./services/project-jackson/manifests/jackson-gateway.yaml) (lines 15-27).

## How to generate manifest YAML with Fabrikate on this repository
1. Run Fabrikate at the root level of this repository
   1. `fab install .`
   2. `fab generate $ENV_CONFIG_NAME`

## Example of manifested YAML
This repository is linked to a manifest repository that has the output YAML
* [GitHub version](https://github.com/andrebriggs/sample_app_manifests)
* [Az DevOps version](https://dev.azure.com/abrig/bedrock_gitops/_git/sample_app_manifests?path=%2F&version=GBmaster)

## Deploying to Cluster
Make sure to apply [jackson-secrets](https://github.com/CatalystCode/kubemalt/tree/master/demo#create-and-deploy-kube-pod-with-secrets-keyvault-tbd) to your cluster.


