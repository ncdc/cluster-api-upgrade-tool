# cluster-api-upgrade-tool

## WARNING

This tool is a work in progress. It may have bugs. **DO NOT** use it on a production Kubernetes cluster or any cluster you can't live without.

To date, we have only tested this with [Cluster API Provider for AWS](http://github.com/kubernetes-sigs/cluster-api-provider-aws) (CAPA)

## Overview

This is a standalone tool to orchestrate upgrading Kubernetes clusters created by Cluster API v0.2.x / API version v1alpha2.

Our goal is to ultimately add this upgrade logic to Cluster API itself, but given that v1alpha2 doesn't easily lend itself to
handling upgrades of control plane machines, we decided to build a temporarily tool that can fill that gap. Once Cluster API
supports full lifecycle management of control planes, we plan to sunset this tool.

## Try it out

Build: Run `make bin` from the root directory of this project.

Run `bin/cluster-api-upgrade-tool` against an existing cluster.

The following examples assume you have `$KUBECONFIG` set.


### Specify options with flags

```
./bin/cluster-api-upgrade-tool \
  --cluster-namespace <Target cluster namespace> \
  --cluster-name <Target cluster name> \
  --kubernetes-version <Desired kubernetes version>
```

### Specify options with a config file

Given the following `my-file` config file:

```yaml
targetCluster:
  namespace: example
  name: cluster1
kubernetesVersion: v1.15.3
upgradeID: 1234
patches:
  infrastructure: '[{ "op": "replace", "path": "/spec/ami", "value": "ami-123456789" }]'
```

Run an upgrade with the following command:

```
./bin/cluster-api-upgrade-tool --config my-file
```

Both JSON and YAML formats are supported.

### Prerequisites

* Cluster created using Cluster API v0.2.x / API version v1alpha2
* Nodes bootstrapped with kubeadm
* Control plane Machine resources have the following labels:
  * `cluster.k8s.io/cluster-name=<cluster name>`
  * `cluster.x-k8s.io/control-plane=true`
* Control plane is comprised of individual Machines

## Documentation

### Usage

```
Usage:
  ./bin/cluster-api-upgrade-tool [flags]

Flags:
      --bootstrap-patches string        JSON patch expression of patches to apply to the machine's bootstrap resource (optional)
      --cluster-name string             The name of target cluster (required)
      --cluster-namespace string        The namespace of target cluster (required)
      --config string                   Path to a config file in yaml or json format
  -h, --help                            help for ./bin/cluster-api-upgrade-tool
      --infrastructure-patches string   JSON patch expression of patches to apply to the machine's infrastructure resource (optional)
      --kubeconfig string               The kubeconfig path for the management cluster
      --kubernetes-version string       Desired kubernetes version to upgrade to (required)
      --upgrade-id string               Unique identifier used to resume a partial upgrade (optional)
```

### Patches

The tool supports using [JSON Patch](https://tools.ietf.org/html/rfc6902) to modify fields in the infrastructure and
bootstrap resources. An example use for this could be to change the image ID in an infrastructure provider's
machine resource. For example, to change the AMI for an `AWSMachine` for the AWS provider, you would use the following:

```
[{ "op": "replace", "path": "/spec/ami", "value": "ami-123456789" }]
```

When specifying patches, make sure you do not replace an entire field such as `metadata` or `spec` unless you include
the full content for the field. For example, if you want to add a label while retaining the existing ones, do **not**
use this patch, as it will replace the entire `metadata` field, potentially losing critical information:

```
[{ "op": "add", "path": "/metadata", "value": {"labels":{ "hello":"world"}} }]
```

Instead, do this:

```
[{ "op": "add", "path": "/metadata/labels/hello", "value": "world" }]
```

This requires that there are already labels present in the original resource. If not, you need to add the entire
labels field:

```
[{ "op": "add", "path": "/metadata/labels", "value": {"hello":"world"} }]
```

## Contributing

The cluster-api-upgrade-tool project team welcomes contributions from the community. If you wish to contribute code and you have not signed our contributor license agreement (CLA), our bot will update the issue when you open a Pull Request. For any questions about the CLA process, please refer to our [FAQ](https://cla.vmware.com/faq).

## License
Apache 2.0
