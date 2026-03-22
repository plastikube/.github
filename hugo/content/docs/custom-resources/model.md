---
weight: 10
title: "plastikube.dev/v1/Model"
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
| `engine` | string | Model engine to use (currently only supports "llamacpp") |
| `features` | []string | Model features |
| `resourceProfile` | string | Reference to a predefined resource profile |
| `entrypoint` | []string | Override the image entrypoint |
| `args` | []string | Override the image args |
| `env` | []object | Extra environment variables to set in the container |
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
| `type` | string | Download type (huggingface-dl \| http \| s3) |
| `source` | string | Download source URL / huggingface repo/filename |
| `job` | object | Configuration for the download job |

### job Object

| Property | Type | Description |
| -------- | ---- | ----------- |
| `image` | string | Container image to use for downloading the model |
| `imagePullSecret` | string | Secret to use for pulling the downloader image |
| `entrypoint` | []string | Override the image entrypoint for the downloader |
| `args` | []string | Override the image args for the downloader |
| `env` | []object | Extra environment variables to set in the downloader container |
| `securityContext` | object | Security settings for the downloader pod |

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
      job:
        env:
        - name: HUGGINGFACE_TOKEN
          valueFrom:
            secretKeyRef:
              name: huggingface-secret
              key: token
  autoscaling:
    minReplicas: 0
    maxReplicas: 1
    idleScaleDown: 300
    busyScaleUp:
      bucket: 60
      activePercent: 70
```

### Model with Custom Download Job

```yaml
apiVersion: plastikube.dev/v1
kind: Model
metadata:
  name: custom-download-model
spec:
  image: my-model:latest
  engine: llamacpp
  resourceProfile: my-resource-profile
  modelStorage:
    download:
      type: http
      source: https://example.com/model.bin
      job:
        image: curlimages/curl:latest
        args:
        - "-L"
        - "https://example.com/model.bin"
        - "-o"
        - "/model/model.bin"
        env:
        - name: CUSTOM_VAR
          value: "value"
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
