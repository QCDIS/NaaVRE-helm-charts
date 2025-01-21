# NaaVRE-helm-charts

NaaVRE consists of several sub-charts, whose values need to be derived from a central configuration file.
Deployment is done in two steps:
- render values for the sub-charts, using the `values` chart
- deploy the sub-charts, using the `naavre` chart and the previously-rendered values

```shell
helm dependency build naavre
helm template values/ --output-dir values/rendered/
helm -n naavre upgrade --install naavre naavre/ $(find values/rendered/values/templates -type f | xargs -I{} echo -n " -f {}")
```

Files in `values/rendered/` should never be edited directly. Instead, change `values/values.yaml` and `values/templates/*.yaml`, and re-render the `values` chart.
