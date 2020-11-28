### 1. Create cluster using:
```
kubeadm init --apiserver-advertise-address=13.235.196.56 --service-dns-domain=curl.local --control-plane-endpoint=13.235.196.56 --pod-network-cidr=10.244.0.0/16 # Domain name shold be local
```
