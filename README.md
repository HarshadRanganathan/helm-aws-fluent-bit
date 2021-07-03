# helm-aws-fluentbit

Example Helm chart for setting up Fluentbit in your EKS cluster.

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

This trust relationship allows pods with serviceaccount `cert-manager` in `cert-manager` namespace to assume the role.

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
