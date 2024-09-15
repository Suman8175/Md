# Commands of kubectl commands

1. Check the version of kubectl
   ```
   kubectl version
   ```

2. Check the nodes in the cluster
   ```
   kubectl get nodes
   ```

3. Check the pods in the cluster
   ```
   kubectl get pods
   kubectl get pods -o wide
   ```

4. Check the services in the cluster
   ```
   kubectl get services
   ```

5. To debug a pod
   ```
   kubectl describe pod <pod-name>
   ```
  eg: `kubectl describe pod nginx-pod`

6. To delete a pod
   ```
   kubectl delete pod <pod-name>
   ```
   eg: `kubectl delete pod nginx-pod`






# Commands of minikube

1. Start minikube
   ```
   minikube start
   ```

2. Stop minikube
   ```
   minikube stop
   ```

3. Delete minikube
   ```
   minikube delete
   ```
4. Check the status of minikube
   ```
   minikube status
   ```

5. Check the logs of minikube
   ```
   minikube logs
   ```