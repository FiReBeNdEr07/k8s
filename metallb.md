## Setting up MetalLB 
If you’re using kube-proxy in IPVS mode, since Kubernetes v1.14.2 you have to enable strict ARP mode.
You can achieve this by editing kube-proxy config in current cluster:
```
kubectl edit configmap -n kube-system kube-proxy
```
set:
```
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true

```
#### Installing MetalLB
```
#Create MetalLB namespace
kubectl apply -f cka/ns-metallb.yaml
#Installing MetalLB
kubectl apply -f cka/metallb.yaml
# On first install only
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```

