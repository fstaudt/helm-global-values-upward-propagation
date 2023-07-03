# helm-global-values-upward-propagation
Reproducer repository for upward propagation of Helm global values

Chart **template** defines a template to display global values from 2 different collections:
- `global.data.*`
- `global.configMap.data.*`

Chart **subchart-1** depends on chart **template**.\
It also defines one key in each collection in its `values.yaml`:
- `global.data.subchart-1`
- `global.configMap.data.subchart-1`

Chart **subchart-2** also depends on chart **template**.\
It also defines one key in each collection in its `values.yaml`:
- `global.data.subchart-2`
- `global.configMap.data.subchart-2`

Chart **bundle** depends on charts **subchart-1** and **subchart-2**.\ 
It also defines one key in each collection in its `values.yaml`:
- `global.data.bundle`
- `global.configMap.data.bundle`

Following [rules of propagation for global values](https://helm.sh/docs/topics/charts/#global-values), 
the following data are expected in generated resources for bundle chart:
```yaml
---
# Source: bundle/charts/subchart-1/charts/template/templates/configMap.yaml
apiVersion: "v1"
kind: ConfigMap
metadata:
  name: subchart-1
data:
  global.data.bundle: "test"
  global.data.subchart-1: "test"
  global.configMap.data.bundle: "test"
  global.configMap.data.subchart-1: "test"
---
# Source: bundle/charts/subchart-2/charts/template/templates/configMap.yaml
apiVersion: "v1"
kind: ConfigMap
metadata:
  name: subchart-2
data:
  global.data.bundle: "test"
  global.data.subchart-2: "test"
  global.configMap.data.bundle: "test"
  global.configMap.data.subchart-2: "test"
```

The following data are found in generated resources with Helm v3.12.1 (or lower):
```yaml
---
# Source: bundle/charts/subchart-1/charts/template/templates/configMap.yaml
apiVersion: "v1"
kind: ConfigMap
metadata:
  name: subchart-1
data:
  global.data.bundle: "test"
  global.data.subchart-1: "test"
  global.configMap.data.bundle: "test"
  global.configMap.data.subchart-1: "test"
  global.configMap.data.subchart-2: "test"
---
# Source: bundle/charts/subchart-2/charts/template/templates/configMap.yaml
apiVersion: "v1"
kind: ConfigMap
metadata:
  name: subchart-2
data:
  global.data.bundle: "test"
  global.data.subchart-2: "test"
  global.configMap.data.bundle: "test"
  global.configMap.data.subchart-1: "test"
  global.configMap.data.subchart-2: "test"
```

Global values for `global.configMap.data` have leaked from sub-charts to the bundle chart and have been propagated everywhere.\
Global values for `global.data` have however not leaked from sub-charts and are only propagated downward.
