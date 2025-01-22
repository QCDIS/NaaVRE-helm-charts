# NaaVRE-helm-charts

## Deployment

NaaVRE consists of several sub-charts, whose values need to be derived from a root values file.
This is not directly supported by Helm, so deployment is done in two steps:

- **Step 1:** render values for the sub-charts, using the `values` chart
  - copy the example root values
    ```shell
    cp ./values/values-example.yaml ./root-values/values-my-deployment.yaml
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

## Limitations

- Assumes that all components are served from one domain
- Assumes that everything runs within a single k8s namespace
- Assumes it is the only NaaVRE instance running in said namespace
- Assumes that the [Ingress NGINX Controller](https://kubernetes.github.io/ingress-nginx/) is deployed on the cluster
- Deploys a single JupyterHub instance
- Does not deploy monitoring (see https://github.com/QCDIS/infrastructure/blob/main/doc/monitoring.md)
- Does not configure pod affinities (see https://github.com/QCDIS/infrastructure/blob/main/doc/pod-affinities.md)
- Does support argo workflow artifacts
