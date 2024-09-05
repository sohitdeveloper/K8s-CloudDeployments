```markdown
# Resolving IP Exhaustion Issues in Kubernetes on AWS (t2.medium Nodes)

When running Kubernetes on AWS EC2 instances, particularly with instance types that have limited networking capabilities like `t2.medium`, it's common to encounter IP exhaustion issues when deploying a large number of pods. This guide walks through the issue, how to identify it, and possible solutions to resolve it.

## Table of Contents
1. [Understanding the Problem](#understanding-the-problem)
2. [Identifying the Issue](#identifying-the-issue)
3. [Causes of IP Exhaustion](#causes-of-ip-exhaustion)
4. [Solutions](#solutions)
   - [Option 1: Scale Nodes or Use Larger Instances](#option-1-scale-nodes-or-use-larger-instances)
   - [Option 2: Use VPC CNI Custom Networking](#option-2-use-vpc-cni-custom-networking)
   - [Option 3: Use Smaller Pods with More Nodes](#option-3-use-smaller-pods-with-more-nodes)
   - [Option 4: Adjust AWS CNI IP Allocation Parameters](#option-4-adjust-aws-cni-ip-allocation-parameters)
   - [Option 5: Review Other Pods Consuming IPs](#option-5-review-other-pods-consuming-ips)

## Understanding the Problem

### IP Exhaustion
On AWS, each EC2 instance type has a fixed number of Elastic Network Interfaces (ENIs) and a limited number of IP addresses that can be assigned to those ENIs. Kubernetes on AWS uses the **AWS VPC CNI plugin** to assign secondary IP addresses to pods, which are allocated from the pool of IPs associated with the node.

For instance, a **t2.medium** instance can support:
- **3 ENIs** 
- **9 secondary IP addresses** (3 secondary IPs per ENI)

Thus, a single `t2.medium` node can only host a limited number of pods before running out of IPs.

## Identifying the Issue

When Kubernetes cannot assign an IP address to a new pod due to IP exhaustion, you will see errors related to network setup failure. You can identify the issue by running:

```bash
kubectl describe pod <POD_NAME>
```

You will see an error like the following under the **Events** section:

```
Failed to create pod sandbox: rpc error: code = Unknown desc = failed to setup network for sandbox "<pod_id>": plugin type="aws-cni" failed (add): add cmd: failed to assign an IP address to container
```

This indicates that the AWS CNI plugin cannot assign a new IP address to the pod because all available IPs are in use on the node.

## Causes of IP Exhaustion

The primary causes of this issue are:
1. **Limited IP addresses per node**: Each node has a limited number of ENIs and secondary IPs, and once exhausted, new pods cannot be assigned an IP.
2. **Too many pods per node**: If the number of pods exceeds the available IPs, new pods will fail to start.
3. **Other infrastructure pods consuming IPs**: Kubernetes system pods (e.g., DNS, logging) also use IPs, reducing the total available for application pods.

## Solutions

### Option 1: Scale Nodes or Use Larger Instances

One simple solution is to either add more nodes to the cluster or switch to instance types with more ENIs and secondary IPs.

| Instance Type | Max ENIs | Max Secondary IPs per ENI | Total IPs (per node) |
|---------------|----------|---------------------------|----------------------|
| t2.medium     | 3        | 3                         | 9                    |
| t3.medium     | 3        | 6                         | 18                   |
| t3.large      | 3        | 15                        | 45                   |

**Steps:**
- Scale the cluster by adding more nodes using `kubectl scale nodes`.
- Upgrade to larger instance types (e.g., `t3.large`).

### Option 2: Use VPC CNI Custom Networking

AWS CNI supports custom networking, which allows you to allocate additional subnets specifically for your pods. This increases the number of IPs available to your pods.

To enable custom networking:
1. Create additional subnets within your VPC.
2. Annotate your node groups to use specific subnets for secondary IP allocation.
   
Refer to the [AWS CNI custom networking documentation](https://docs.aws.amazon.com/eks/latest/userguide/cni-custom-network.html) for detailed steps.

### Option 3: Use Smaller Pods with More Nodes

If your application can run efficiently on smaller pods (with fewer CPU/memory requests), consider using smaller instance types (e.g., `t3.small`) but increase the number of nodes in the cluster.

**Steps:**
- Adjust pod resource requests in your deployment YAML.
- Increase the number of nodes by editing your cluster's node auto-scaling group.

This distributes the workload across more nodes, thereby providing more IPs for your pods.

### Option 4: Adjust AWS CNI IP Allocation Parameters

AWS CNI allows fine-tuning of IP allocation with the following parameters:

- **`WARM_IP_TARGET`**: The number of free IP addresses to keep on the node. Increasing this value can reduce IP allocation delays.
- **`MINIMUM_IP_TARGET`**: The minimum number of IPs that should be allocated to the node.

Modify these parameters by updating the AWS CNI configuration:

```bash
kubectl edit daemonset aws-node -n kube-system
```

### Option 5: Review Other Pods Consuming IPs

Kubernetes system pods (such as DNS and logging) also use IPs. To reduce unnecessary IP consumption:
- Identify the system pods running on the nodes using `kubectl get pods -n kube-system`.
- If possible, move system-critical pods to dedicated nodes or reduce their resource requests.

## Conclusion

IP exhaustion is a common issue in AWS Kubernetes clusters, especially on instance types with limited network resources like `t2.medium`. By understanding the networking constraints and applying the appropriate solutions (scaling nodes, custom networking, etc.), you can prevent and resolve these issues.

---

If you continue to experience issues, consider reviewing AWS's [best practices for scaling IP addresses](https://docs.aws.amazon.com/eks/latest/userguide/cni-increase-ip-addresses.html) in Kubernetes.

```
