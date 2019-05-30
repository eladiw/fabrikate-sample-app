# Simplifying Ingress and Deployments with Fabrikate

Fabrikate _high-level defintions_ allows you to compose helm charts, static manifests, and even other high-level defintions as subcomponents. 

Below we have a high-level defintion that composes a custom microservice and an ingress/deployment strategy. 

<pre>
name: custom-microservice-with-ingress-blue-green
subcomponents:
- name: "ingress-deploy-strategy"
  source: "https://github.com/andrebriggs/fabrikate-ingress-deploy-strategy"
  method: "git"
</pre>

The _ingress-deploy-strategy_ would point to a git repo that has a helm chart following the common k8s pattern of ingress -> service -> deployment. This helm chart would be produce a kubernetes objects that would correspond to the ingress option and deployment options that would be set in the `config/common.yaml` of the `custom-microservice-with-ingress` high-level definition. 

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
        strategy: <b>"traefik"</b> # Could also be "istio", "linkerd" or "nginx"
      deployment:
        strategy: <b>"blue-green"</b>
      blue-green:
        image: my-custom-image
        repository: andrebriggs.azurecr.io/my-image-repo
        <b>productionSlot: blue</b>
          blue:
            enabled: true
            <b>tag: "v.1.0.1"</b>
          green:
            enabled: false
            <b>tag: "v1.0.2"</b>
        productionSlot: bue
        ...
      ...  
</pre>

In the above configuration we are choosing [Traefik](https://traefik.io) as our ingress strategy and a [blue/green](https://martinfowler.com/bliki/BlueGreenDeployment.html) as our deployment type. Additionally we have described a repo and image for the for our deployment in `subcomponents.ingress-deploy-strategy.config.blue-green.image` path. We also have defined a _productionSlot_ key and value that will allow us to define which deployment(s) (blue or green) is currently active.

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

The Helm chart in the _ingress-deploy-strategy_ repository will use conditional logic to determine which Kubernetes object defintions get generated based upon the `config/common.yaml`.

One caveat about the current design is that that the "blue" (existing) or the stable (as opposed to canary) is that all the deployed resources are defined in the chart. The _ingress-deploy-strategy_ chart be opinionated in the supported resources that are deployed.

# Links

+ [Traefik Blue/Green Implementation](https://github.com/andrebriggs/fabrikate-sample-app/blob/traefik_bg/Traefik.bg.README.md)
+ [Istio Blug/Green Implementation](https://github.com/andrebriggs/fabrikate-sample-app/pull/9)
