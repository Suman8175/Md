1. Apply pods 
   >When you define a pod's configuration in a YAML file and use `kubectl apply -f pods.yaml`, you are creating a pod resource in your Kubernetes cluster
   ```
   kubectl apply -f <pod-config-file>
   ```
  eg: ` kubectl apply -f pods.yaml`

2. Check if the pod is created
   ```
   kubectl get pods -o wide
   ```

3. As minikube is a VM,we cannot access the pod from outside the cluster.So we need to login to the minikube VM and run the container from there.
   ```
   minikube ssh
   ```
   ```
   curl <pod-ip>
   ```




