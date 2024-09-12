```markdown
# EKS Cluster and Resource Cleanup

This document outlines the steps to delete all resources within an EKS cluster, including namespaces and ingress controllers, followed by deleting the EKS cluster itself.

## Step 1: List all resources in all namespaces

To list all resources across all namespaces, run the following command:

```bash
kubectl get all --all-namespaces
```

This provides a comprehensive list of all running resources in the cluster.

## Step 2: Delete all resources in all namespaces

To delete all resources (such as pods, services, deployments, etc.) in all namespaces:

```bash
kubectl delete all --all --all-namespaces
```

This will remove all resources from all namespaces.

## Step 3: Delete specific namespaces

### Delete the `ingress-nginx` namespace: (optional) only execute if you have

```bash
kubectl delete namespace ingress-nginx
```

### Delete the `default` namespace: (optional) it can be deleted when you execute command for deleting the cluster

```bash
kubectl delete namespace default
```

Ensure that the namespaces and their resources are no longer required before deleting them.

## Step 4: Delete all ingress resources across all namespaces (optional) only execute if you have

To remove all ingress resources in all namespaces, use:

```bash
kubectl delete -A ingress --all
```

This ensures that all ingress controllers and rules are deleted.

## Step 5: Delete the EKS cluster

Finally, delete the EKS cluster itself using `eksctl`. Replace `<cluster-name>` and `<region-name>` with the appropriate values:

```bash
eksctl delete cluster --name <cluster-name> --region <region-name>
```

This will completely remove the EKS cluster along with associated infrastructure.

---

**Note**: Be sure to back up any important data or configurations before proceeding with the deletion commands, as this process is irreversible.
```