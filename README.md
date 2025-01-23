# NaaVRE-helm-charts

## Deployment

NaaVRE consists of several sub-charts, whose values need to be derived from a root values file.
This is not directly supported by Helm, so deployment is done in two steps:

- **Step 1:** render values for the sub-charts, using the `values` chart
  - copy the example root values
    ```shell
    cp ./values/values-example-single.yaml ./root-values/values-my-deployment.yaml
    ```
  - fill in your values
    ```shell
    vim ./root-values/values-my-deployment.yaml
    ```
  - render the `values` chart
    ```shell
    helm template values/ --output-dir values/rendered -f ./root-values/values-my-deployment.yaml
    ```
    > Note: never edit files in `values/rendered`. Instead, change `./root-values/values-my-deployment.yaml` or `values/templates/*.yaml` and re-render the `values` chart.


- **Step 2:** Deploy the sub-charts, using the `naavre` chart and the previously-rendered values.
  - get subcharts
  ```shell
  helm dependency build naavre
  ```
  - deploy subcharts
  ```shell
  helm -n naavre upgrade --create-namespace --install naavre naavre/ $(find values/rendered/values/templates -type f | xargs -I{} echo -n " -f {}")
  ```

## Uninstall

```shell
helm -n naavre uninstall naavre
```

## Advanced setups

### Several virtual labs

In this example, we deploy NaaVRE with two virtual labs. This is achieved by deploying three instances of the `naavre` chart: one instance running the common services (`argo-workflows`, `k8s-secret-creator`, `keycloak` and `vrepaas` sub-charts), and two instances running Jupyter (`jupyterhub` sub-chart; one for each virtual lab).

```shell
cp ./values/values-example-multi-*.yaml ./root-values/
# Edit the common values
vim ./root-values/values-example-multi-base.yaml
# Configure the virtual labs
vim ./root-values/values-example-multi-vl-1.yaml
vim ./root-values/values-example-multi-vl-2.yaml
# Get naavre sub-charts
helm dependency build naavre
# Render values and deploy the services (steps 1 and 2)
helm template values/ --output-dir values/rendered -f ./root-values/values-example-multi-base.yaml -f ./root-values/values-example-multi-services.yaml
helm -n naavre upgrade --create-namespace --install naavre-services naavre/ $(find values/rendered/values/templates -type f | xargs -I{} echo -n " -f {}")
# Render values and deploy vl 1 (steps 1 and 2)
helm template values/ --output-dir values/rendered -f ./root-values/values-example-multi-base.yaml -f ./root-values/values-example-multi-vl-1.yaml
helm -n naavre upgrade --create-namespace --install naavre-vl-1 naavre/ $(find values/rendered/values/templates -type f | xargs -I{} echo -n " -f {}")
# Render values and deploy vl 2 (steps 1 and 2)
helm template values/ --output-dir values/rendered -f ./root-values/values-example-multi-base.yaml -f ./root-values/values-example-multi-vl-2.yaml
helm -n naavre upgrade --create-namespace --install naavre-vl-2 naavre/ $(find values/rendered/values/templates -type f | xargs -I{} echo -n " -f {}")
```

## Limitations

- Assumes that all components are served from one domain
- Assumes that everything runs within a single k8s namespace
- Assumes it is the only NaaVRE instance running in said namespace
- Assumes that the [Ingress NGINX Controller](https://kubernetes.github.io/ingress-nginx/) is deployed on the cluster
- Does not deploy monitoring (see https://github.com/QCDIS/infrastructure/blob/main/doc/monitoring.md)
- Does not configure pod affinities (see https://github.com/QCDIS/infrastructure/blob/main/doc/pod-affinities.md)
- Does support argo workflow artifacts
