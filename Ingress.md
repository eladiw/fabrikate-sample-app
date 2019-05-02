# Simplifying Ingress and Deployments with Fabrikate

Fabrikate _high-level defintions_ allows you to compose helm charts, static manifests, and even other high-level defintions as subcomponents. 

Below we have a high-level defintion that composes a custom microservice and an ingress/deployment strategy. 

<pre>
name: custom-microservice-with-ingress
subcomponents:
- name: "custom-microservice"
  source: "https://github.com/andrebriggs/fabrikate-custom-microservice"
  method: "git"
- name: "ingress-deploy-strategy"
  source: "https://github.com/andrebriggs/fabrikate-ingress-deploy-strategy"
  method: "git"
</pre>

The _custom-microservice_ points to a git repo that have a helm chart. You can assume that helm chart only defines a Kubernetes _deployment_ object and would produce a `deployment.yaml` when generated.

Secondly the _ingress-deploy-strategy_ would point to a git repo that also has a helm chart. This helm chart would be produce a kubernetes objects that would correspond to the ingress option and deployment options that would be set in the `config/common.yaml` of the `custom-microservice-with-ingress` high-level definition. 

An example of the `config/common.yaml` file could look like the following:

<pre>
subcomponents:
  custom-microservice:
    config:
      deployment:
        name: microservice
      image:
        repository: andrebriggs.azurecr.io/my-image
        tag: "v1.0.1"
  ingress-deploy-strategy:
    config:
      ingress:
        strategy: <b>"traefik"</b> # Could also be "istio" or "nginx"
      deployment:
        strategy: <b>"blue-green"</b>
      blue-green:
        image:
          repository: andrebriggs.azurecr.io/my-image
          tag: "v1.0.2"
        <b>deploymentSlot: "blue"</b>
        ...
      ...  
</pre>

In the above configuration we are choosing [Traefik](https://traefik.io) as our ingress strategy and a [blue/green](https://martinfowler.com/bliki/BlueGreenDeployment.html) as our deployment type. Additionally we have described an image for the "green" part of our blue/green deployment in `subcomponents.ingress-deploy-strategy.config.blue-green.image` path. We also have defined a _deploymentSlot_ key and value that will allow us to define which deployment (blue or green) is currently active.

Below we have a permutation of the configuration that would allow us to use Istio for our ingress, while generating a canary deployment. Again the container image for the canary is defined at `subcomponents.ingress-deploy-strategy.config.canary.image`

<pre>
subcomponents:
  custom-microservice:
    config:
      deployment:
        name: microservice
      image:
        repository: andrebriggs.azurecr.io/my-image
        tag: "v1.0.1"
  ingress-deploy-strategy:
    config:
      ingress:
        strategy: <b>"istio"</b>
      deployment:
        <b>strategy: "canary"</b>
      <b>canary</b>:
        image:
          repository: andrebriggs.azurecr.io/my-image
          tag: "v1.0.2"
        ...
      ...  
</pre>

The Helm chart in the _ingress-deploy-strategy_ repository will use conditional logic to determin with Kubernetes object defintions get generated based upon the `config/common.yaml`.

One caveat about the current deisgn is that that the "blue" (existing) or the stable (as opposed to canary) would be defined in the _custom-microservice_ defintion. The _ingress-deploy-strategy_ chart would need to somehow retreive a config value outside of it's own Fabrikate subcomponent config section. 