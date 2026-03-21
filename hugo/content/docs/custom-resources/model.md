---
weight: 10
title: "Model"
description: "Model Custom Resource Definition"
draft: false
---

The Model custom resource defines a machine learning model deployment in Plastikube.

## Specification

### Properties

| Property | Type | Description |
| -------- | ---- | ----------- |
| `image` | string | Container image to use for the model |
| `imagePullSecret` | string | Secret to use for pulling the image |
| `modelStorage` | object | Storage configuration for the model |
| `engine` | string | Model engine to use |
| `features` | []string | Model features |
| `resourceProfile` | string | Reference to a predefined resource profile |
| `entrypoint` | []string | Override the image entrypoint |
| `args` | []string | Override the image args |
| `env` | []object | Extra environment variables to set in the container |
| `envFrom` | []object | Extra environment variables from sources |
| `replicas` | integer | Default number of replicas |
| `autoscaling` | object | Autoscaling configuration |
| `nodeSelector` | object | Node labels for pod assignment |
| `tolerations` | []object | Taints that the pod can tolerate |

### modelStorage Object

| Property | Type | Description |
| -------- | ---- | ----------- |
| `PersistentVolumeClaim` | object | New PVC to create for storing the model |
| `existingVolume` | string | Existing PVC to use for storing the model |
| `path` | string | Path within the PVC to load the model from |
| `download` | object | How to download the model |

### download Object

| Property | Type | Description |
| -------- | ---- | ----------- |
| `type` | string | Download type (huggingface-dl \| wget) |
| `source` | string | Download source URL / huggingface repo/filename |
| `auth` | object | Authentication for the download |

### auth Object

| Property | Type | Description |
| -------- | ---- | ----------- |
| `secretKeyRef` | object | Secret key for HUGGINGFACE_TOKEN or HTTP basic auth |

### autoscaling Object

| Property | Type | Description |
| -------- | ---- | ----------- |
| `minReplicas` | integer | Minimum replicas when autoscaling is enabled |
| `maxReplicas` | integer | Maximum replicas when autoscaling is enabled |
| `idleScaleDown` | integer | Seconds model can be idle before scaling to minReplicas |
| `busyScaleUp` | object | Scale-up configuration |

### busyScaleUp Object

| Property | Type | Description |
| -------- | ---- | ----------- |
| `bucket` | integer | Seconds to size the bucket |
| `activePercent` | integer | Percentage of the bucket we have to be busy to trigger scaling up |

## Examples

### Basic Model with Download

```yaml
apiVersion: plastikube.dev/v1
kind: Model
metadata:
  name: example-model
spec:
  image: ghcr.io/plastikube/llamacpp:latest-rocm63 # other tags available for other libraries
  engine: llamacpp
  resourceProfile: amd-gpu-hostdev-plugin
  env:
  # example: force gfx906 on rocm
  - name: HSA_OVERRIDE_GFX_VERSION
    value: "9.0.6"
  modelStorage:
    download:
      type: huggingface-dl
      source: my-repo/my-model
      auth:
        secretKeyRef:
          name: huggingface-token
          key: token
  autoscaling:
    minReplicas: 0
    maxReplicas: 1
    idleScaleDown: 300
    busyScaleUp:
      bucket: 60
      activePercent: 70
```

### Model with Existing PVC

```yaml
apiVersion: plastikube.dev/v1
kind: Model
metadata:
  name: existing-pvc-model
spec:
  image: my-model:latest
  engine: llamacpp
  resourceProfile: my-resource-profile
  modelStorage:
    existingVolume: model-pvc
    path: /models/my-model
```

### Model with New PVC

```yaml
apiVersion: plastikube.dev/v1
kind: Model
metadata:
  name: new-pvc-model
spec:
  image: my-model:latest
  engine: llamacpp
  resourceProfile: my-resource-profile
  modelStorage:
    PersistentVolumeClaim:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi
    path: /models
```
