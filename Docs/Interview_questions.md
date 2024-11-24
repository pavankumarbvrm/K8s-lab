# Top 10 Kubernetes Scenario-Based Questions 💡

1. **Scenario: Your application has multiple replicas, but only one pod is receiving traffic. What could be the issue?**
   - **Possible issue**: The most likely cause is that the Service is not load-balancing correctly. It could be due to:
     - Incorrect Service configuration (e.g., Service type set to `ClusterIP` without load balancer)
     - Pod selector issue (mismatch of labels)
     - Endpoint readiness (only one pod is ready)
   - **Action**: Verify labels, service type, and endpoints readiness with:
     ```bash
     kubectl get svc <service-name> -o yaml
     kubectl get endpoints <service-name>
     ```

2. **Scenario: During a rolling update, some pods fail to start. How would you troubleshoot this issue?**
   - **Steps**:
     - Inspect logs of failing pods using:
       ```bash
       kubectl logs <pod-name>
       ```
     - Describe the pod for more details:
       ```bash
       kubectl describe pod <pod-name>
       ```
     - Check for common issues like image pull errors, resource constraints, or misconfigured environment variables.
     - Verify liveness/readiness probes and correct rolling update strategy configuration in the Deployment spec.

3. **Scenario: You need to deploy a stateful database with persistent storage. What Kubernetes resources would you use and why?**
   - **Resources**:
     - Use a `StatefulSet` to ensure the uniqueness and ordering of the database pods.
     - Use `PersistentVolume` and `PersistentVolumeClaim` to provide stable, persistent storage.
     - Optionally, use `StorageClass` to dynamically provision storage based on your infrastructure provider.

4. **Scenario: Your application is experiencing intermittent downtime due to pod failures. How would you ensure high availability?**
   - **Steps**:
     - Ensure multiple replicas are running using a `Deployment` or `StatefulSet`.
     - Configure `PodDisruptionBudget` to prevent too many pods from being taken down simultaneously.
     - Implement health checks using `livenessProbe` and `readinessProbe`.
     - Use multiple `node pools` or spread replicas across nodes to improve resilience.

5. **Scenario: Your team needs to manage different environments (dev, staging, prod) in the same cluster. How would you achieve this?**
   - **Approach**:
     - Use `Namespaces` to isolate resources between environments.
     - Use `ResourceQuotas` and `LimitRanges` to control resource usage within each namespace.
     - Deploy different configurations for each environment by using `ConfigMaps` and `Secrets` with environment-specific values.

6. **Scenario: An application needs to access sensitive credentials securely. How would you store and inject these credentials into the pods?**
   - **Approach**:
     - Use Kubernetes `Secrets` to store sensitive credentials.
     - Inject secrets as environment variables or mounted volumes in the pod spec:
       ```yaml
       env:
         - name: DB_PASSWORD
           valueFrom:
             secretKeyRef:
               name: db-secret
               key: password
       ```
     - Ensure the proper access controls and RBAC policies are in place to limit access.

7. **Scenario: After scaling your pods, some of them are failing to communicate with each other. What could be causing this?**
   - **Possible causes**:
     - Network issues: Ensure the `CNI` (Container Network Interface) is properly configured.
     - Pod selectors or `NetworkPolicies` are blocking traffic between pods.
     - DNS resolution issue: Verify that `CoreDNS` is working properly by running:
       ```bash
       kubectl get pods -n kube-system -l k8s-app=kube-dns
       ```

8. **Scenario: Your cluster’s etcd is running out of storage. How would you resolve this situation?**
   - **Steps**:
     - **Monitor etcd**: Check etcd storage consumption using `kubectl top`.
     - **Compact etcd**: Perform an etcd compaction to reduce disk usage:
       ```bash
       etcdctl --endpoints=<etcd-endpoint> compact <revision>
       ```
     - **Add storage**: Increase the storage capacity for etcd or move it to a larger volume.
     - **Backup**: Ensure regular etcd backups to restore the cluster if needed.

9. **Scenario: You need to update a container image in production, but you want to avoid downtime. How would you do this?**
   - **Steps**:
     - Perform a **rolling update** by updating the Deployment spec with the new image:
       ```bash
       kubectl set image deployment/<deployment-name> <container-name>=<new-image>
       ```
     - Kubernetes will automatically update the pods with zero downtime using the rolling update strategy.
     - Use `readinessProbe` to ensure the new pods are fully ready before serving traffic.

10. **Scenario: An application needs to be highly available across multiple Kubernetes clusters. How would you architect this?**
   - **Approach**:
     - Use **multi-cluster** setups with Kubernetes Federation or external tools like Rancher, or Anthos.
     - Implement **cross-cluster service discovery** using external DNS or tools like Istio.
     - Ensure **data consistency** across clusters for stateful applications by using a multi-region database or managed service (e.g., Google Spanner, AWS Aurora Global DB).
     - Use **global load balancing** (e.g., AWS Route 53, GCP Global Load Balancer) to route traffic across clusters.
