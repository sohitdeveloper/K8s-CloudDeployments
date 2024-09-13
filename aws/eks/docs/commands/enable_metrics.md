1. **Install or Update Metrics Server**:
   If the Metrics Server is not installed, you can deploy it using the following command:

   ```sh
   kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
   ```

   This YAML file will deploy the Metrics Server in your cluster.

2. **Check Metrics Server Logs**:
   If the Metrics Server is installed but not working correctly, check the logs for any errors:

   ```sh
   kubectl logs -n kube-system deployment/metrics-server
   ```

4. **Verify RBAC Permissions**:
   Ensure that the Metrics Server has the necessary RBAC (Role-Based Access Control) permissions. The YAML file from the above step should handle this, but double-check if you have custom RBAC configurations.

5. **Wait for Metrics Server to Initialize**:
   After deploying the Metrics Server, it might take a few minutes for it to start collecting metrics. Wait a few minutes and then try running `kubectl top pods` again.

6. **Check Cluster Health**:
   Ensure that your cluster is healthy and that all nodes and pods are in a ready state. Sometimes, issues with cluster health can affect the Metrics Server's ability to function.
