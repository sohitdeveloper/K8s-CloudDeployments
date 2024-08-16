````markdown
# EKS Cluster Setup Guide

This guide provides the steps to create an Amazon EKS (Elastic Kubernetes Service) cluster using `eksctl` and AWS CLI, along with an example of configuring the cluster using a YAML file.

**Important:** I have created this repository based on my requirements, but Kubernetes (K8s) doesn't care about which application it is. You just need to understand and follow the structure and requirements as needed; the rest will work.

## Prerequisites

Before you start, ensure you have the following:

- AWS CLI installed and configured with the necessary permissions.
- `eksctl` installed on your machine.
- An SSH key pair created in the AWS region where you plan to deploy the cluster.

## Step 1: Create EKS Cluster using `eksctl`

To create an EKS cluster with a managed node group using `eksctl`, run the following command:

```bash
eksctl create cluster \
  --name daas-cluster \
  --region us-west-2 \
  --nodegroup-name daas-nodegroup \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 3 \
  --managed
```
````

### Explanation of the Command:

- `--name daas-cluster`: Specifies the name of the EKS cluster.
- `--region us-west-2`: Specifies the AWS region where the cluster will be created.
- `--nodegroup-name daas-nodegroup`: Specifies the name of the node group.
- `--node-type t3.medium`: Specifies the EC2 instance type for the nodes.
- `--nodes 2`: Specifies the number of nodes to create in the node group.
- `--nodes-min 2`: Specifies the minimum number of nodes in the node group.
- `--nodes-max 3`: Specifies the maximum number of nodes in the node group.
- `--managed`: Creates a managed node group.

## Step 2: Create EKS Cluster using AWS CLI

Alternatively, you can create an EKS cluster using the AWS CLI with the following command:

```bash
aws eks create-cluster --name daas-cluster --region us-west-1 --nodegroup-name daas-node1 --nodegroup-name daas-node2 --node-type t2.medium,t2.medium --nodes 2
```

### Explanation of the Command:

- `--name daas-cluster`: Specifies the name of the EKS cluster.
- `--region us-west-1`: Specifies the AWS region where the cluster will be created.
- `--nodegroup-name daas-node1 --nodegroup-name daas-node2`: Specifies the names of the node groups.
- `--node-type t2.medium,t2.medium`: Specifies the EC2 instance types for the node groups.
- `--nodes 2`: Specifies the number of nodes to create in each node group.

## Step 3: Create EKS Cluster using YAML Configuration

You can also define your EKS cluster configuration in a YAML file. Below is an example configuration:

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: daas-cluster
  region: us-west-1

nodeGroups:
  - name: daas-node1
    instanceType: t2.medium
    desiredCapacity: 2
    minSize: 2
    maxSize: 2
    ssh:
      publicKeyName: your-ssh-key # Replace with your SSH key name

  - name: daas-node2
    instanceType: t2.medium
    desiredCapacity: 2
    minSize: 2
    maxSize: 2
    ssh:
      publicKeyName: your-ssh-key # Replace with your SSH key name
```

### Explanation of the YAML File:

- `apiVersion: eksctl.io/v1alpha5`: Specifies the API version for `eksctl`.
- `kind: ClusterConfig`: Specifies the kind of resource being created.
- `metadata`: Contains metadata for the cluster, such as its name and region.
- `nodeGroups`: Defines the configuration for the node groups, including instance types, capacity, and SSH key configuration.

## Step 4: Apply the YAML Configuration

To create the EKS cluster using the YAML configuration, run the following command:

```bash
eksctl create cluster -f your-cluster-config.yaml
```

Replace `your-cluster-config.yaml` with the path to your YAML configuration file.

## Conclusion

This guide provided you with three methods to create an EKS cluster: using `eksctl` commands, AWS CLI, and a YAML configuration file. Choose the method that best fits your workflow and infrastructure needs.

Note : First, you need to go inside the cluster, then play with these files.

```bash
aws eks --region <region-name> update-kubeconfig --name <cluster-name>
```
