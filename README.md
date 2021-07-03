# helm-aws-fluentbit

Example Helm chart for setting up Fluentbit in your EKS cluster.

## Pre-requisites

### Namespace

Create a new namespace `fluent-bit` where we will install the `aws-fluent-bit` service.

```bash
kubectl create namespace fluent-bit
```
