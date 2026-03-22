---
weight: 20
title: "plastikube.dev/v1/ResourceProfile"
description: "Resource Profile Custom Resource Definition"
draft: false
---

The Resource Profile custom resource defines compute resource requirements that can be referenced by other resources in Plastikube.

## Specification

### Properties

| Property | Type | Description |
| -------- | ---- | ----------- |
| `resources` | object | Compute resources required for this profile |

### resources Object

The resources object defines the compute resources (CPU, memory, etc.) required for this profile. Keys are resource names and values are resource quantities.

Common resource names:

- `cpu` - CPU cores
- `memory` - Memory in bytes
- `nvidia.com/gpu` - NVIDIA GPUs via gpu-operator
- `hostdev.k8s.io/dev_kfd` - AMD GPUs via [k8s-hostdev-plugin](https://github.com/thehonker/k8s-hostdev-plugin)
- `amd.com/gpu` - AMD GPUs via official AMD GPU operator

## Example

```yaml
---
# cpu inference
apiVersion: plastikube.dev/v1
kind: ResourceProfile
metadata:
  name: default
spec:
  resources:
    cpu: "1"
    memory: "2Gi"

---
# nvidia gpu using nvidia-gpu-operator
apiVersion: plastikube.dev/v1
kind: ResourceProfile
metadata:
  name: nvidia-gpu-operator
spec:
  resources:
    cpu: "4"
    memory: "16Gi"
    nvidia.com/gpu: "1"

---
# amd gpu leveraging k8s-hostdev-plugin
apiVersion: plastikube.dev/v1
kind: ResourceProfile
metadata:
  name: amd-gpu-hostdev-plugin
spec:
  resources:
    cpu: "4"
    memory: "16Gi"
    hostdev.k8s.io/dev_kfd: 1

---
# amd gpu using official amd gpu operator
apiVersion: plastikube.dev/v1
kind: ResourceProfile
metadata:
  name: amd-gpu-official
spec:
  resources:
    cpu: "4"
    memory: "16Gi"
    amd.com/gpu: "1"
```
