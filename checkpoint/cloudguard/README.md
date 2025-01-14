#  Check Point Cloudguard agents

## Introduction

This chart deploys the agents required by [Check Point CloudGuard](https://portal.checkpoint.com/) to provide Inventory Management, Posture Management, Image Assurance, Visibility, Threat Intelligence, Runtime Protection, Admission Control, and Monitoring capabilities.

Note: notice that some of the above capabilities require enrollment in the Early Availability program (contact a Check Point representative for more details).

## Prerequisites

General
- Kubernetes 1.12+, all nodes should have the same container runtime (docker, containerd or cri-o)
- Helm 3.0+
- Check Point CloudGuard account credentials

For the Admission Control feature
- Kubernetes 1.16+

For the Threat Intelligence feature
- Kernel 4.1+

For the Runtime Protection feature
- Kernel 4.14
- Kubernetes 1.16+


## Installing the Chart

To install the chart with the chosen release name (e.g. `my-release`), run:

```bash
$ helm repo add checkpoint https://raw.githubusercontent.com/CheckPointSW/charts/master/repository/
$ helm install my-release checkpoint/cloudguard --set credentials.user=[CloudGuard API Key] --set credentials.secret=[CloudGuard API Secret] --set clusterID=[Cluster ID] --namespace [Namespace] --create-namespace
```

These are the additional optional flags to enable add-ons:

```bash
$
$ --set addons.imageScan.enabled=true 
$ --set addons.flowLogs.enabled=true
$ --set addons.admissionControl.enabled=true
$ --set addons.runtimeProtection.enabled=true
```

This command deploys an inventory agent as well as optional add-on agents.

**Note**: the following add-ons require enrollment in the Early Availability program:
* Threat Intelligence (flowLogs)
* Runtime Protection (runtimeProtection)

> **Tip**: List all releases using `helm list --namespace [Namespace]`


## Upgrading the chart

To upgrade the deployment and/or to add/remove additional feature run:

```bash
$ helm repo update
$ helm upgrade my-release checkpoint-ea/cloudguard --set credentials.user=[CloudGuard API Key] --set credentials.secret=[CloudGuard API Secret] --set clusterID=[Cluster ID] --set addons.imageScan.enabled=[true/false] --set addons.flowLogs.enabled=[true/false] --namespace [Namespace]
```

## Uninstalling the Chart

To uninstall the `my-release` deployment:

```bash
$ helm uninstall my-release --namespace [Namespace]
```

This command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

In order to get the [Check Point CloudGuard](https://portal.checkpoint.com/) Cluster ID & credentials, you must first complete the Kubernetes Cluster onboarding process in [Check Point CloudGuard](https://portal.checkpoint.com/) website.

Refer to [defaults.yaml](defaults.yaml) for the full run-down on default values. These are a mixture of Kubernetes and CloudGuard directives that map to environment variables.

Specify each parameter by adding `--set key=value[,key=value]` to the `helm install`. For example,

```bash
$ helm install my-release checkpoint/cloudguard --set varname=value
```

For parameters which are dictionaries or arrays, make sure to use the proper syntax, for example:

```bash
$ ... --set addons.admissionControl.enforcer.podAnnotations.custom."aa\.bb/cc-dd"="ee\.ff/gg-hh"
```

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```bash
$ helm install my-release checkpoint/cloudguard -f values.yaml
```

> **Tip**: You can use the default [values.yaml](values.yaml)

**Maximal image size for Image Assurance**

For Image Assurance feature the default maximal image size to scan is 2GB, and the relevant imageScan-engine pod memory limit is 2.5GB. In order to configure a different maximal image size, *addons.imageScan.maxImageSizeMb* parameter should be set with the maximal image size in MB. Pay attention, using this flag defines also the memory limit of imagescan-engine pod to this value + 500MB. E.g., to scan images of size of up to 3000MB, helm install command should be appended with:
```bash
     --set addons.imageScan.maxImageSizeMb=3000
```

It will define memory limit for *imagescan-engine* pod to be 3.5GB.

**Number of Image Assurance Scanners**

The number of Image Assurance scanners can be increased to add parallelism and reduce the time it takes to scan multiple images. By default there is one such scanner.
Modifying the number of Image Assurance scanners can be done by setting the addons.imageScan.engine.replicaCount parameter. E.g. to set the number of scanning pods to 2, helm install command should be appended with:
```bash
--set addons.imageScan.engine.replicaCount=2
```

Note that each additional scanner will require additional resources.

## Configurable parameters

The following table list the configurable parameters of this chart and their default values.

| Parameter                                                  | Description                                                     | Default                                          |
| ---------------------------------------------------------- | --------------------------------------------------------------- | ------------------------------------------------ |
| `clusterID`                                                | Cluster Unique identifier in CloudGuard system                  | `CHANGEME`                                       |
| `datacenter`                                               | CloudGuard datacenter (usea1, euwe1 apse1, apse2, apso1)        | `usea1`                                          |
| `credentials.secret`                                       | CloudGuard APISecret (Note: mandatory unless `credentials.secretName` is specified) | `CHANGEME`                                       |
| `credentials.user`                                         | CloudGuard APIID  (Note: mandatory unless `credentials.secretName` is specified) | `CHANGEME`                                       |
| `credentials.secretName`                                    | Name of an existing Kubernetes Secret that contains CloudGuard APIID (data.username) and APISecret (data.secret) | None                                       |
| `rbac.pspEnabled`                                          | Specifies whether PSP resources should be created               | `false`                                          |
| `imageRegistry.url`                                        | Image registry                                                  | `quay.io`                                        |
| `imageRegistry.authEnabled`                                | Whether or not Image Registry access is password-protected      | `true`                                           |
| `imageRegistry.user`                                       | Image registry username                                         | `CHANGEME`                                       |
| `imageRegistry.password`                                   | Image registry password                                         | `CHANGEME`                                       |
| `imagePullPolicy`                                          | Image pull policy                                               | `Always`                                         |
| `proxy`                                                    | Proxy settings (e.g. http://my-proxy.com:8080)                  | `{}`                                             |
| `containerRuntime`                                         | Container runtime (docker/containerd/cri-o) overriding auto-detection | ``                                         |
| `containerRuntimeSocket`                                   | Container runtime socket path overriding auto-detection         | ``                                               |
| `platform`                                                 | Kubernetes platform (kubernetes/tanzu/openshift/openshift.v3/eks/eks.bottlerocket/gke.cos/gke.autopilot/k3s) overriding auto-detection | `kubernetes`                                |
| `seccompProfile`                                           | Computer Security facility profile. (to be used in kubernetes 1.19 and up) | `RuntimeDefault`                                |
| `podAnnotations.seccomp`                                   | Computer Security facility profile. (to be used in kubernetes below 1.19) | `runtime/default`                                |
| `podAnnotations.apparmor`                                  | Apparmor Linux kernel security module profile.                  | `{}`                                             |
| `autoUpgrade`                                              | Enable auto-upgrade (true or false). 'major.minor' tags will be set for images rather than 'major.minor.patch'" | `false`    |
| `podAnnotations.custom`                                    | Custom Pod annotations (for all agent Pods)                     | `{}`                                             |
| `priorityClassName`                                        | Specifies custom priorityClassName                              | ``                                               |
| `daemonSetStrategy.rollingUpdate.maxUnavailable`           | Maximum unavailable daemonset pods during a rolling update                 | `50%`                                            |
| `inventory.agent.image`                                    | Specify image for the agent                                     | `checkpoint/consec-inventory-agent`              |
| `inventory.agent.tag`                                      | Specify image tag for the agent                                 | see defaults.yaml                                          |
| `inventory.agent.serviceAccountName`                       | Specify custom Service Account for the Inventory agent          | ``                                               |
| `inventory.agent.replicaCount`                             | Number of Inventory agent instances to be deployed              | `1`                                              |
| `inventory.agent.env`                                      | Additional environmental variables for Inventory agent          | `{}`                                             |
| `inventory.agent.resources`                                | Resources restriction (e.g. CPU, memory) for Inventory agent    | see defaults.yaml                                             |
| `inventory.agent.nodeSelector`                             | Node labels for pod assignment for Inventory agent              | see below                                             |
| `inventory.agent.tolerations`                              | List of node taints to tolerate for Inventory agent             | `[]`                                             |
| `inventory.agent.affinity`                                 | Affinity settings for Inventory agent                           | `{}`                                             |
| `inventory.agent.podAnnotations.custom`                    | Custom Pod annotations (for Pods of this agent)                 | `{}`                                             |
| `inventory.priorityClassName`                              | Specifies custom priorityClassName                              | `system-cluster-critical`                                               |
| `addons.imageScan.enabled`                                 | Specifies whether the ImageScan addon should be installed      | `false`                                          |
| `addons.imageScan.mountPodman`                             | Should be set to true if ImageScan fails to export image on CRI-O | `false`                                           |
| `addons.imageScan.priorityClassName`                       | Specifies custom priorityClassName                              | `system-cluster-critical`                        |
| `addons.imageScan.maxImageSizeMb`                          | Specifies in MiBytes maximal image size to scan, its value + 500MB will be imageScan.engine main container memory limit | ``                                               |
| `addons.imageScan.daemon.image`                            | Specify image for the agent                                     | `checkpoint/consec-imagescan-daemon`             |
| `addons.imageScan.daemon.tag`                              | Specify image tag for the agent                                 | see defaults.yaml                                           |
| `addons.imageScan.daemon.serviceAccountName`               | Specify custom Service Account for the agent                    | ``                                               |
| `addons.imageScan.daemon.env`                              | Additional environmental variables for the agent                | `{}`                                             |
| `addons.imageScan.daemon.resources`                        | Resources restriction (e.g. CPU, memory)                        | see defaults.yaml                                             |
| `addons.imageScan.daemon.nodeSelector`                     | Node labels for pod assignment                                  | see below                                             |
| `addons.imageScan.daemon.tolerations`                      | List of node taints to tolerate                                 | `operator: Exists`                               |
| `addons.imageScan.daemon.affinity`                         | Affinity setting                                                | `{}`                                             |
| `addons.imageScan.daemon.podAnnotations.custom`            | Custom Pod annotations (for Pods of this agent)                 | `{}`                                             |
| `addons.imageScan.daemon.priorityClassName`                | Specifies custom priorityClassName (for Pods of this daemonset) | `system-node-critical`                           |
| `addons.imageScan.daemon.shim.image`                       | Specify image for the shim container                            | `checkpoint/consec-imagescan-shim`               |
| `addons.imageScan.daemon.shim.tag`                         | Specify image tag for the shim container                        | see defaults.yaml                                           |
| `addons.imageScan.daemon.shim.env`                         | Additional environmental variables for the shim container       | `{}`                                             |
| `addons.imageScan.daemon.shim.resources`                   | Resources restriction (e.g. CPU, memory)                        | see defaults.yaml                                             |
| `addons.imageScan.engine.image`                            | Specify image for the agent                                     | `checkpoint/consec-imagescan-engine`             |
| `addons.imageScan.engine.tag`                              | Specify image tag for the agent                                 | see defaults.yaml                                           |
| `addons.imageScan.engine.serviceAccountName`               | Specify custom Service Account for the agent                    | ``                                               |
| `addons.imageScan.engine.replicaCount`                     | Number of scanning engine instances to be deployed              | `1`                                              |
| `addons.imageScan.engine.env`                              | Additional environmental variables for the agent                | `{}`                                             |
| `addons.imageScan.engine.resources`                        | Resources restriction (e.g. CPU, memory)                        | see defaults.yaml                                             |
| `addons.imageScan.engine.nodeSelector`                     | Node labels for pod assignment                                  | see below                                             |
| `addons.imageScan.engine.tolerations`                      | List of node taints to tolerate                                 | `[]`                                             |
| `addons.imageScan.engine.affinity`                         | Affinity setting                                                | `{}`                                             |
| `addons.imageScan.engine.podAnnotations.custom`            | Custom Pod annotations (for Pods of this agent)                 | `{}`                                             |
| `addons.imageScan.list.image`                              | Specify image for the agent                                     | `checkpoint/consec-imagescan-engine`             |
| `addons.imageScan.list.tag`                                | Specify image tag for the agent                                 | see defaults.yaml                                          |
| `addons.imageScan.list.serviceAccountName`                 | Specify custom Service Account for the agent                    | ``                                               |
| `addons.imageScan.list.env`                                | Additional environmental variables for the agent                | `{}`                                             |
| `addons.imageScan.list.resources`                          | Resources restriction (e.g. CPU, memory)                        | see defaults.yaml                                             |
| `addons.imageScan.list.nodeSelector`                       | Node labels for pod assignment                                  | see below                                             |
| `addons.imageScan.list.tolerations`                        | List of node taints to tolerate                                 | `[]`                                             |
| `addons.imageScan.list.affinity`                           | Affinity setting                                                | `{}`                                             |
| `addons.imageScan.list.podAnnotations.custom`              | Custom Pod annotations (for Pods of this agent)                 | `{}`                                             |
| `addons.imageScan.daemonConfigurationOverrides`            | Overrides for multiple daemonSets with different configuration values                | see below                                              |
| `addons.flowLogs.enabled`                                  | Specifies whether the Flow Logs addon should be installed       | `false`                                          |
| `addons.flowLogs.priorityClassName`                        | Specifies custom priorityClassName                              | `system-cluster-critical`                        |
| `addons.flowLogs.daemon.image`                             | Specify image for the agent                                     | `checkpoint/consec-flowlogs-daemon`              |
| `addons.flowLogs.daemon.tag`                               | Specify image tag for the agent                                 | see defaults.yaml                                           |
| `addons.flowLogs.daemon.serviceAccountName`                | Specify custom Service Account for the agent                    | ``                                               |
| `addons.flowLogs.daemon.logLevel`                          | What should be logged. (info, debug)                            | `info`                                           |
| `addons.flowLogs.daemon.env`                               | Additional environmental variables for the agent                | `{}`                                             |
| `addons.flowLogs.daemon.resources`                         | Resources restriction (e.g. CPU, memory)                        | see defaults.yaml                                             |
| `addons.flowLogs.daemon.nodeSelector`                      | Node labels for pod assignment                                  | see below                                             |
| `addons.flowLogs.daemon.tolerations`                       | List of node taints to tolerate                                 | `operator: Exists`                               |
| `addons.flowLogs.daemon.affinity`                          | Affinity setting                                                | `{}`                                             |
| `addons.flowLogs.daemon.podAnnotations.custom`             | Custom Pod annotations (for Pods of this agent)                 | `{}`                                             |
| `addons.flowLogs.daemon.priorityClassName`                 | Specifies custom priorityClassName (for Pods of this daemonset) | `system-node-critical`                           |
| `addons.flowLogs.daemonConfigurationOverrides`    | Overrides for multiple daemonSets with different configuration values                | see below                                              |
| `addons.admissionControl.enabled`                          | Specify whether the Admission Control addon should be installed | `false`                                          |
| `addons.admissionControl.priorityClassName`                | Specifies custom priorityClassName                              | `system-cluster-critical`                        |
| `addons.admissionControl.policy.image`                     | Specify image for the agent                                     | `checkpoint/consec-admission-policy`             |
| `addons.admissionControl.policy.tag`                       | Specify image tag for the agent                                 | see defaults.yaml                                           |
| `addons.admissionControl.policy.serviceAccountName`        | Specify custom Service Account for the agent                    | ``                                               |
| `addons.admissionControl.policy.env`                       | Additional environmental variables for the agent                | `{}`                                             |
| `addons.admissionControl.policy.resources`                 | Resources restriction (e.g. CPU, memory)                        | see defaults.yaml                                             |
| `addons.admissionControl.policy.nodeSelector`              | Node labels for pod assignment                                  | see below                                             |
| `addons.admissionControl.policy.tolerations`               | List of node taints to tolerate                                 | `[]`                                             |
| `addons.admissionControl.policy.affinity`                  | Affinity setting                                                | `{}`                                             |
| `addons.admissionControl.policy.podAnnotations.custom`     | Custom Pod annotations (for Pods of this agent)                 | `{}`                                             |
| `addons.admissionControl.enforcer.image`                   | Specify image for the agent                                     | `checkpoint/consec-admission-enforcer`           |
| `addons.admissionControl.enforcer.tag`                     | Specify image tag for the agent                                 | see defaults.yaml                                           |
| `addons.admissionControl.enforcer.serviceAccountName`      | Specify custom Service Account for the agent                    | ``                                               |
| `addons.admissionControl.enforcer.replicaCount`            | Number of Inventory agent instances to be deployed              | `2`                                              |
| `addons.admissionControl.enforcer.env`                     | Additional environmental variables for the agent                | `{}`                                             |
| `addons.admissionControl.enforcer.resources`               | Resources restriction (e.g. CPU, memory)                        | see defaults.yaml                                             |
| `addons.admissionControl.enforcer.nodeSelector`            | Node labels for pod assignment                                  | see below                                             |
| `addons.admissionControl.enforcer.tolerations`             | List of node taints to tolerate                                 | `[]`                                             |
| `addons.admissionControl.enforcer.affinity`                | Affinity setting                                                | `{}`                                             |
| `addons.admissionControl.enforcer.podAnnotations.custom`   | Custom Pod annotations (for Pods of this agent)                 | `{}`                                             |
| `addons.runtimeProtection.enabled`                         | Specifies whether the Runtime Protection addon should be installed | `false`                                          |
| `addons.runtimeProtection.priorityClassName`               | Specifies custom priorityClassName                              | `system-cluster-critical`                        |
| `addons.runtimeProtection.daemon.image`                    | Specify image for the agent                                     | `checkpoint/consec-runtime-daemon`               |
| `addons.runtimeProtection.daemon.tag`                      | Specify image tag for the agent                                 | see defaults.yaml                                         |
| `addons.runtimeProtection.daemon.serviceAccountName`       | Specify custom Service Account for the agent                    | ``                                               |
| `addons.runtimeProtection.daemon.env`                      | Additional environmental variables for the agent                | `{}`                                             |
| `addons.runtimeProtection.daemon.resources`                | Resources restriction (e.g. CPU, memory)                        | see defaults.yaml                             |
| `addons.runtimeProtection.daemon.probe.image`              | Specify image for the agent                                     | `checkpoint/consec-runtime-probe`                |
| `addons.runtimeProtection.daemon.probe.tag`                | Specify image tag for the agent                                 | see defaults.yaml                                     |
| `addons.runtimeProtection.daemon.probe.resources`          | Resources restriction (e.g. CPU, memory)                        | `{}`                                             |
| `addons.runtimeProtection.daemon.nodeSelector`             | Node labels for pod assignment                                  | see below                  |
| `addons.runtimeProtection.daemon.tolerations`              | List of node taints to tolerate                                 | `operator: Exists`                               |
| `addons.runtimeProtection.daemon.affinity`                 | Affinity setting                                                | `{}`                                             |
| `addons.runtimeProtection.daemon.podAnnotations.custom`    | Custom Pod annotations (for Pods of this agent)                 | `{}`                                             |
| `addons.runtimeProtection.daemon.priorityClassName`        | Specifies custom priorityClassName (for Pods of this daemonset) | `system-node-critical`                           |
| `addons.runtimeProtection.policy.image`                    | Specify image for the agent                                     | `checkpoint/consec-runtime-policy`               |
| `addons.runtimeProtection.policy.tag`                      | Specify image tag for the agent                                 | see defaults.yaml                                           |
| `addons.runtimeProtection.policy.serviceAccountName`       | Specify custom Service Account for the agent                    | ``                                               |
| `addons.runtimeProtection.policy.env`                      | Additional environmental variables for the agent                | `{}`                                             |
| `addons.runtimeProtection.policy.resources`                | Resources restriction (e.g. CPU, memory)                        | see defaults.yaml                                             |
| `addons.runtimeProtection.policy.nodeSelector`             | Node labels for pod assignment                                  | see below                                             |
| `addons.runtimeProtection.policy.tolerations`              | List of node taints to tolerate                                 | `[]`                                             |
| `addons.runtimeProtection.policy.affinity`                 | Affinity setting                                                | `{}`                                             |
| `addons.runtimeProtection.policy.podAnnotations.custom`    | Custom Pod annotations (for Pods of this agent)                 | `{}`                                             |
| `addons.runtimeProtection.daemonConfigurationOverrides`    | Overrides for multiple daemonSets with different configuration values                | see below                                              |

The default nodeSelector for all agents is:
 - kubernetes.io/os: "linux"
 - kubernetes.io/arch: "amd64"

The `daemonConfigurationOverrides` object should have one or more objects with unique names (case insensitive), each object must then have a `nodeSelector` data and any additional overrides, such as resource limits and requests. The values defined in `daemon` object are used as a basis for the overrides.\
In the following example, there are two configurations: "sizeNormalConfig" and "sizeLargeConfig". The two Configurations use different values of the "size" label on the nodes and have different resource limits.


```yaml
addons:
  imageScan:
  enabled: true
  daemon:
     env:
     - name: commonENV
       value: commonValue
    daemonConfigurationOverrides:
     sizeNormalConfig:
        nodeSelector:
          size: normal
        resources:
          limits:
            cpu: 255m
            memory: 128Mi
          requests:
            cpu: 100m
            memory: 128Mi

      sizeLargeConfig:
        nodeSelector:
          size: large
        resources:
          limits:
            cpu: 455m
            memory: 128Mi
          requests:
            cpu: 250m
            memory: 128Mi
```
