# commands to configure IAM OIDC provider

**Important:** Configuring an IAM OIDC provider in AWS allows you to securely connect and grant access to AWS resources for users or services that are authenticated by an external identity provider (like Google or Okta). This setup is useful for integrating external systems with AWS, managing access for Kubernetes workloads, or securely running CI/CD pipelines without needing to store AWS credentials directly.

```
export cluster_name=<CLUSTER-NAME>
```

```
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
```

## Check if there is an IAM OIDC provider configured already

```
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
```

If not, run the below command

```
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```
