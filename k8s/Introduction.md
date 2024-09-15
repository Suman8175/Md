<img src="https://kubernetes.io/images/docs/kubernetes-cluster-architecture.svg" alt="k8 architecture">


# Worker Plane
Node are called worker plane

- Kube-proxy: handles `load balancing` by maintaining network rules on each node to forward traffic to the appropriate Pod endpoints, `whether the Pods are on the same node or different nodes`.
  >How? 
  >
  `kube-proxy` watches for changes to Services and Endpoints in real-time. If a new Pod for Container1 is created on Node2 or Node3, the `kube-API server` updates the Endpoints list for the `Container1` Service, and `kube-proxy on all nodes is notified of this change`.

- Kublet:Continuously `monitors` the state of the Pods running on its node.

>Example:
##
##
##

***1 Pod Failure:***

A container in a Pod crashes or fails a health check`.
`Kubelet detects the failure` and kills the container.

***2 Notification to API Server:***

`Kubelet reports the container's failure` and the Pod's new status (e.g., "Failed" or "Terminating") to the `API server.`

***3 API Server Updates Endpoints:***

If the failed Pod was part of a Service, the `API server updates the Endpoints for that Service`, `removing the failed Pod from the list of healthy Pods`.

***4 kube-proxy Reacts:***

`kube-proxy on all nodes receives this update from the API server` and `updates its routing rules to stop sending traffic to the failed Pod`.

##
##
##


- `etcd` :Contains all `backup of cluster`.***If we want to restore cluster api server gets it from etcd***


