# Notes


- Inspect the kubelet service and identify the container runtime endpoint value is set for Kubernetes.
    - Run the command: `ps -aux | grep kubelet | grep --color container-runtime-endpoint` and look at the configured `--container-runtime-endpoint` flag.

- What is the path configured with all binaries of CNI supported plugins?
    - `/opt/cni/bin`

- What is the CNI plugin configured to be used on this kubernetes cluster?
    - `/etc/cni/net.d`

- What is the type set for the binaries to execute.
    - Look at the type field in file `/etc/cni/net.d/10-flannel.conflist`.

- To send a curl from one pod to another in the same namespace
    - `kubectl exec -it <from-pod> -- curl -m 5 <to-pod-ip>`


- What is the range of IP addresses configured for PODs on this cluster?
    - `cat /etc/kubernetes/manifests/kube-controller-manager.yaml   | grep cluster-cidr`