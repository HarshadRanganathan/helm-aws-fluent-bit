# helm-aws-fluent-bit

Example Helm chart for setting up Fluentbit in your EKS cluster.

We install two versions of FluentBit in the cluster -

[1] FluentBit App version for shipping application logs from specific namespaces

[2] FluentBit Infra version for shipping platform logs e.g. prometheus, external dns etc. excluding kube-system logs

## Pre-requisites

### Namespace

Create a new namespace `fluent-bit` where we will install the `aws-fluent-bit` service.

```bash
kubectl create namespace fluent-bit
```

### IAM

We will be using IRSA (IAM Roles for Service Accounts) to give the required permissions to FluentBit pod for publishing the logs to cloudwatch.

`Note: You need to create an OIDC provider for your cluster to make use of IRSA. Refer - https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html`

1. Create a new IAM policy `aws-fluent-bit-pol` with the policy document at `iam/policy.json`

2. Create a new IAM role `aws-fluent-bit-rol` and attach the IAM policy `aws-fluent-bit-pol`

3. Update the trust relationship of the IAM role `aws-fluent-bit-rol` as below replacing the `account_id`, `eks_cluster_id` and `region` with the appropriate values.

This trust relationship allows pods with serviceaccount `aws-fluent-bit` in `fluent-bit` namespace to assume the role.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<account_id>:oidc-provider/oidc.eks.us-east-1.amazonaws.com/id/<eks_cluster_id>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.<region>.amazonaws.com/id/<eks_cluster_id>:sub": "system:serviceaccount:fluent-bit:aws-fluent-bit"
        }
      }
    }
  ]
}
```

### Service Account

Create a new service account in the `fluent-bit` namespace and associate it with the IAM role which we had created earlier.

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: aws-fluent-bit
  namespace: fluent-bit
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<AWS_ACCOUNT_ID>:role/aws-fluent-bit-rol
EOF
```


## Install/Upgrade Chart

Run below commands to install/upgrade the chart.

```bash
helm upgrade -i aws-fluent-bit-infra . -n fluent-bit --values=stages/shared-values.yaml --values=stages/prod/prod-infra-values.yaml

helm upgrade -i aws-fluent-bit-app . -n fluent-bit --values=stages/shared-values.yaml --values=stages/prod/prod-app-values.yaml
```

