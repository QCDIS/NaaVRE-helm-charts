# NaaVRE-helm-charts

## Deployment

NaaVRE consists of several sub-charts, whose values need to be derived from a central configuration file.
Deployment is done in two steps:

**Step 1:** render values for the sub-charts, using the `values` chart

```shell
helm template values/ --output-dir values/rendered/
```

**Step 2:** deploy the sub-charts, using the `naavre` chart and the previously-rendered values

```shell
helm dependency build naavre
helm -n naavre upgrade --install naavre naavre/ $(find values/rendered/values/templates -type f | xargs -I{} echo -n " -f {}")
```

Files in `values/rendered/` should never be edited directly. Instead, change `values/values.yaml` and `values/templates/*.yaml`, and re-render the `values` chart.


## Limitations

- Assumes that all components are served from one domain
- Assumes that everything runs within a single k8s namespace
- Assumes it is the only NaaVRE instance running in said namespace
- Assumes that the [Ingress NGINX Controller](https://kubernetes.github.io/ingress-nginx/) is deployed on the cluster
- Deploys a single JupyterHub instance
- Does not deploy monitoring (see https://github.com/QCDIS/infrastructure/blob/main/doc/monitoring.md)
- Does not configure pod affinities (see https://github.com/QCDIS/infrastructure/blob/main/doc/pod-affinities.md)
- Does support argo workflow artifacts
