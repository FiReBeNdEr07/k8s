# k8s
## Steps for setting up your own K8s cluster in On-Prem:
* Disable swap partition

   Run `swapoff -a` to turn off swap, then comment swapfile config line in `/etc/fstab`
* Install Container Runtime(Prefered `docker`)

* Install kubectl kubelet kubeadm in all master and nodes. [Refer](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

* Setuping up Loadbalancer for control panel(Need to be done in loadbalancer host)

   * Install Nginx `sudo apt-get update && sudo apt-get install nginx`
   * Create the directory `/etc/nginx`
   * Add and edit the file `/etc/nginx/nginx.conf`
   * Add Following config 
      ```
      events { }
   
      stream {
        upstream stream_backend {
            least_conn;
            server 172.16.0.10:6443;
            server 172.16.1.2:6443;
        }
           
        server {
            listen        6443;
            proxy_pass    stream_backend;
            proxy_timeout 3s;
            proxy_connect_timeout 1s;
        }
           
        }
      ```
   * Start nginx conatiner
      ```
      docker run --name proxy \
          -v /etc/nginx/nginx.conf:/etc/nginx/nginx.conf:ro \
          -p 6443:6443 \
          -d --restart unless-stopped nginx
      ```

* Init cluster
   ```
   root@storage:~# kubeadm init --pod-network-cidr=192.168.0.0/16 --control-plane-endpoint=172.16.0.7   --service-dns-domain=curlhg.local --cri-socket=/var/run/dockershim.sock --upload-certs
   W0628 14:32:11.197037    2927 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.  config.k8s.io kubeproxy.config.k8s.io]
   [init] Using Kubernetes version: v1.18.5
   [preflight] Running pre-flight checks
   	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd".    Please follow the guide at https://kubernetes.io/docs/setup/cri/
   [preflight] Pulling images required for setting up a Kubernetes cluster
   [preflight] This might take a minute or two, depending on the speed of your internet connection
   [preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
   [kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
   [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
   [kubelet-start] Starting the kubelet
   [certs] Using certificateDir folder "/etc/kubernetes/pki"
   [certs] Generating "ca" certificate and key
   [certs] Generating "apiserver" certificate and key
   [certs] apiserver serving cert is signed for DNS names [storage kubernetes kubernetes.default kubernetes.default.svc   kubernetes.default.svc.curlhg.local] and IPs [10.96.0.1 172.16.0.10 172.16.0.7]
   [certs] Generating "apiserver-kubelet-client" certificate and key
   [certs] Generating "front-proxy-ca" certificate and key
   [certs] Generating "front-proxy-client" certificate and key
   [certs] Generating "etcd/ca" certificate and key
   [certs] Generating "etcd/server" certificate and key
   [certs] etcd/server serving cert is signed for DNS names [storage localhost] and IPs [172.16.0.10 127.0.0.1 ::1]
   [certs] Generating "etcd/peer" certificate and key
   [certs] etcd/peer serving cert is signed for DNS names [storage localhost] and IPs [172.16.0.10 127.0.0.1 ::1]
   [certs] Generating "etcd/healthcheck-client" certificate and key
   [certs] Generating "apiserver-etcd-client" certificate and key
   [certs] Generating "sa" key and public key
   [kubeconfig] Using kubeconfig folder "/etc/kubernetes"
   [kubeconfig] Writing "admin.conf" kubeconfig file
   [kubeconfig] Writing "kubelet.conf" kubeconfig file
   [kubeconfig] Writing "controller-manager.conf" kubeconfig file
   [kubeconfig] Writing "scheduler.conf" kubeconfig file
   [control-plane] Using manifest folder "/etc/kubernetes/manifests"
   [control-plane] Creating static Pod manifest for "kube-apiserver"
   [control-plane] Creating static Pod manifest for "kube-controller-manager"
   W0628 14:32:14.125671    2927 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,   RBAC"
   [control-plane] Creating static Pod manifest for "kube-scheduler"
   W0628 14:32:14.126379    2927 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,   RBAC"
   [etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
   [wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/  manifests". This can take up to 4m0s
   [apiclient] All control plane components are healthy after 22.051744 seconds
   [upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
   [kubelet] Creating a ConfigMap "kubelet-config-1.18" in namespace kube-system with the configuration for the kubelets in the   cluster
   [upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
   [upload-certs] Using certificate key:
   5fdf248e94e790144de731ec05bccf5add388ab8db1b303cc47a5c0374593121
   [mark-control-plane] Marking the node storage as control-plane by adding the label "node-role.kubernetes.io/master=''"
   [mark-control-plane] Marking the node storage as control-plane by adding the taints [node-role.kubernetes.io/  master:NoSchedule]
   [bootstrap-token] Using token: yeu1oi.8n0iiqo84evixjld
   [bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
   [bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
   [bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term    certificate credentials
   [bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap   Token
   [bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
   [bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
   [kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
   [addons] Applied essential addon: CoreDNS
   [addons] Applied essential addon: kube-proxy
   
   Your Kubernetes control-plane has initialized successfully!
   
   To start using your cluster, you need to run the following as a regular user:
   
     mkdir -p $HOME/.kube
     sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
     sudo chown $(id -u):$(id -g) $HOME/.kube/config
   
   You should now deploy a pod network to the cluster.
   Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
     https://kubernetes.io/docs/concepts/cluster-administration/addons/
   
   You can now join any number of the control-plane node running the following command on each as root:
   
     kubeadm join 172.16.0.7:6443 --token yeu1oi.8n0iiqo84evixjld \
       --discovery-token-ca-cert-hash sha256:44175bd8c5956df594f4d0a9db3fa79c59adb1fed6c0ec0459b6166de1c5eb3d \
       --control-plane --certificate-key 5fdf248e94e790144de731ec05bccf5add388ab8db1b303cc47a5c0374593121
   
   Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
   As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
   "kubeadm init phase upload-certs --upload-certs" to reload certs afterward.
   
   Then you can join any number of worker nodes by running the following on each as root:
   
   kubeadm join 172.16.0.7:6443 --token yeu1oi.8n0iiqo84evixjld \
       --discovery-token-ca-cert-hash sha256:44175bd8c5956df594f4d0a9db3fa79c59adb1fed6c0ec0459b6166de1c5eb3d 
   
   ```
* Remove the taints on the master so that you can schedule pods on it.
   ```
   kubectl taint nodes --all node-role.kubernetes.io/master-
   ```
   It should return the following.
   ```
   node/<your-hostname> untainted
   ```
* Deploying pod network for cluster 
   ```
   kubectl apply -f cka/cilium.yaml
   ```
* Adding Second Master 

   ```
   luffy@devsecops-1  ~/wrdir/cka  sudo kubeadm join 172.16.0.7:6443 --token yeu1oi.8n0iiqo84evixjld \
       --discovery-token-ca-cert-hash sha256:44175bd8c5956df594f4d0a9db3fa79c59adb1fed6c0ec0459b6166de1c5eb3d \
       --control-plane --certificate-key 5fdf248e94e790144de731ec05bccf5add388ab8db1b303cc47a5c0374593121 --v=5
   I0628 14:46:17.427877   23131 join.go:371] [preflight] found NodeName empty; using OS hostname as NodeName
   I0628 14:46:17.427909   23131 join.go:375] [preflight] found advertiseAddress empty; using default interface's IP address as   advertiseAddress
   I0628 14:46:17.427934   23131 initconfiguration.go:103] detected and using CRI socket: /var/run/dockershim.sock
   I0628 14:46:17.428072   23131 interface.go:400] Looking for default routes with IPv4 addresses
   I0628 14:46:17.428079   23131 interface.go:405] Default route transits interface "enp1s0"
   I0628 14:46:17.428248   23131 interface.go:208] Interface enp1s0 is up
   I0628 14:46:17.428296   23131 interface.go:256] Interface "enp1s0" has 2 addresses :[172.16.1.2/16 fe80::efed:cafe:588d:5748/  64].
   I0628 14:46:17.428312   23131 interface.go:223] Checking addr  172.16.1.2/16.
   I0628 14:46:17.428318   23131 interface.go:230] IP found 172.16.1.2
   I0628 14:46:17.428324   23131 interface.go:262] Found valid IPv4 address 172.16.1.2 for interface "enp1s0".
   I0628 14:46:17.428329   23131 interface.go:411] Found active IP 172.16.1.2 
   [preflight] Running pre-flight checks
   I0628 14:46:17.428374   23131 preflight.go:90] [preflight] Running general checks
   I0628 14:46:17.428406   23131 checks.go:249] validating the existence and emptiness of directory /etc/kubernetes/manifests
   I0628 14:46:17.428439   23131 checks.go:286] validating the existence of file /etc/kubernetes/kubelet.conf
   I0628 14:46:17.428448   23131 checks.go:286] validating the existence of file /etc/kubernetes/bootstrap-kubelet.conf
   I0628 14:46:17.428454   23131 checks.go:102] validating the container runtime
   I0628 14:46:17.509746   23131 checks.go:128] validating if the service is enabled and active
   I0628 14:46:17.586514   23131 checks.go:335] validating the contents of file /proc/sys/net/bridge/bridge-nf-call-iptables
   I0628 14:46:17.586555   23131 checks.go:335] validating the contents of file /proc/sys/net/ipv4/ip_forward
   I0628 14:46:17.586575   23131 checks.go:649] validating whether swap is enabled or not
   I0628 14:46:17.586601   23131 checks.go:376] validating the presence of executable conntrack
   I0628 14:46:17.586620   23131 checks.go:376] validating the presence of executable ip
   I0628 14:46:17.586633   23131 checks.go:376] validating the presence of executable iptables
   I0628 14:46:17.586652   23131 checks.go:376] validating the presence of executable mount
   I0628 14:46:17.586684   23131 checks.go:376] validating the presence of executable nsenter
   I0628 14:46:17.586701   23131 checks.go:376] validating the presence of executable ebtables
   I0628 14:46:17.586715   23131 checks.go:376] validating the presence of executable ethtool
   I0628 14:46:17.586734   23131 checks.go:376] validating the presence of executable socat
   I0628 14:46:17.586747   23131 checks.go:376] validating the presence of executable tc
   I0628 14:46:17.586758   23131 checks.go:376] validating the presence of executable touch
   I0628 14:46:17.586773   23131 checks.go:520] running all checks
   I0628 14:46:17.658470   23131 checks.go:406] checking whether the given node name is reachable using net.LookupHost
   I0628 14:46:17.658570   23131 checks.go:618] validating kubelet version
   I0628 14:46:17.707908   23131 checks.go:128] validating if the service is enabled and active
   I0628 14:46:17.724797   23131 checks.go:201] validating availability of port 10250
   I0628 14:46:17.724928   23131 checks.go:432] validating if the connectivity type is via proxy or direct
   I0628 14:46:17.724954   23131 join.go:441] [preflight] Discovering cluster-info
   I0628 14:46:17.724970   23131 token.go:78] [discovery] Created cluster-info discovery client, requesting info from "172.16.0.  7:6443"
   I0628 14:46:17.737435   23131 token.go:116] [discovery] Requesting info from "172.16.0.7:6443" again to validate TLS against   the pinned public key
   I0628 14:46:17.752648   23131 token.go:133] [discovery] Cluster info signature and contents are valid and TLS certificate    validates against pinned roots, will use API Server "172.16.0.7:6443"
   I0628 14:46:17.752666   23131 discovery.go:51] [discovery] Using provided TLSBootstrapToken as authentication credentials    for the join process
   I0628 14:46:17.752676   23131 join.go:455] [preflight] Fetching init configuration
   I0628 14:46:17.752680   23131 join.go:493] [preflight] Retrieving KubeConfig objects
   [preflight] Reading configuration from the cluster...
   [preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
   I0628 14:46:17.769061   23131 interface.go:400] Looking for default routes with IPv4 addresses
   I0628 14:46:17.769073   23131 interface.go:405] Default route transits interface "enp1s0"
   I0628 14:46:17.769223   23131 interface.go:208] Interface enp1s0 is up
   I0628 14:46:17.769264   23131 interface.go:256] Interface "enp1s0" has 2 addresses :[172.16.1.2/16 fe80::efed:cafe:588d:5748/  64].
   I0628 14:46:17.769276   23131 interface.go:223] Checking addr  172.16.1.2/16.
   I0628 14:46:17.769282   23131 interface.go:230] IP found 172.16.1.2
   I0628 14:46:17.769287   23131 interface.go:262] Found valid IPv4 address 172.16.1.2 for interface "enp1s0".
   I0628 14:46:17.769291   23131 interface.go:411] Found active IP 172.16.1.2 
   I0628 14:46:17.769317   23131 preflight.go:101] [preflight] Running configuration dependant checks
   [preflight] Running pre-flight checks before initializing the new control plane instance
   I0628 14:46:17.769334   23131 checks.go:577] validating Kubernetes and kubeadm version
   I0628 14:46:17.769345   23131 checks.go:166] validating if the firewall is enabled and active
   I0628 14:46:17.775725   23131 checks.go:201] validating availability of port 6443
   I0628 14:46:17.775767   23131 checks.go:201] validating availability of port 10259
   I0628 14:46:17.775786   23131 checks.go:201] validating availability of port 10257
   I0628 14:46:17.775804   23131 checks.go:286] validating the existence of file /etc/kubernetes/manifests/kube-apiserver.yaml
   I0628 14:46:17.775814   23131 checks.go:286] validating the existence of file /etc/kubernetes/manifests/   kube-controller-manager.yaml
   I0628 14:46:17.775820   23131 checks.go:286] validating the existence of file /etc/kubernetes/manifests/kube-scheduler.yaml
   I0628 14:46:17.775826   23131 checks.go:286] validating the existence of file /etc/kubernetes/manifests/etcd.yaml
   I0628 14:46:17.775834   23131 checks.go:432] validating if the connectivity type is via proxy or direct
   I0628 14:46:17.775854   23131 checks.go:471] validating http connectivity to first IP address in the CIDR
   I0628 14:46:17.775868   23131 checks.go:471] validating http connectivity to first IP address in the CIDR
   I0628 14:46:17.775876   23131 checks.go:201] validating availability of port 2379
   I0628 14:46:17.775896   23131 checks.go:201] validating availability of port 2380
   I0628 14:46:17.775919   23131 checks.go:249] validating the existence and emptiness of directory /var/lib/etcd
   [preflight] Pulling images required for setting up a Kubernetes cluster
   [preflight] This might take a minute or two, depending on the speed of your internet connection
   [preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
   I0628 14:46:17.815430   23131 checks.go:838] image exists: k8s.gcr.io/kube-apiserver:v1.18.5
   I0628 14:46:17.850336   23131 checks.go:838] image exists: k8s.gcr.io/kube-controller-manager:v1.18.5
   I0628 14:46:17.883680   23131 checks.go:838] image exists: k8s.gcr.io/kube-scheduler:v1.18.5
   I0628 14:46:17.915397   23131 checks.go:838] image exists: k8s.gcr.io/kube-proxy:v1.18.5
   I0628 14:46:17.947329   23131 checks.go:838] image exists: k8s.gcr.io/pause:3.2
   I0628 14:46:17.979899   23131 checks.go:838] image exists: k8s.gcr.io/etcd:3.4.3-0
   I0628 14:46:18.012213   23131 checks.go:838] image exists: k8s.gcr.io/coredns:1.6.7
   [download-certs] Downloading the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
   [certs] Using certificateDir folder "/etc/kubernetes/pki"
   I0628 14:46:18.020615   23131 certs.go:38] creating PKI assets
   [certs] Generating "apiserver" certificate and key
   [certs] apiserver serving cert is signed for DNS names [devsecops-1 kubernetes kubernetes.default kubernetes.default.svc   kubernetes.default.svc.curlhg.local] and IPs [10.96.0.1 172.16.1.2 172.16.0.7]
   [certs] Generating "apiserver-kubelet-client" certificate and key
   [certs] Generating "front-proxy-client" certificate and key
   [certs] Generating "apiserver-etcd-client" certificate and key
   [certs] Generating "etcd/peer" certificate and key
   [certs] etcd/peer serving cert is signed for DNS names [devsecops-1 localhost] and IPs [172.16.1.2 127.0.0.1 ::1]
   [certs] Generating "etcd/server" certificate and key
   [certs] etcd/server serving cert is signed for DNS names [devsecops-1 localhost] and IPs [172.16.1.2 127.0.0.1 ::1]
   [certs] Generating "etcd/healthcheck-client" certificate and key
   [certs] Valid certificates and keys now exist in "/etc/kubernetes/pki"
   I0628 14:46:19.176848   23131 certs.go:69] creating new public/private key files for signing service account users
   [certs] Using the existing "sa" key
   [kubeconfig] Generating kubeconfig files
   [kubeconfig] Using kubeconfig folder "/etc/kubernetes"
   [kubeconfig] Writing "admin.conf" kubeconfig file
   [kubeconfig] Writing "controller-manager.conf" kubeconfig file
   [kubeconfig] Writing "scheduler.conf" kubeconfig file
   [control-plane] Using manifest folder "/etc/kubernetes/manifests"
   [control-plane] Creating static Pod manifest for "kube-apiserver"
   I0628 14:46:19.597637   23131 manifests.go:91] [control-plane] getting StaticPodSpecs
   W0628 14:46:19.597705   23131 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,   RBAC"
   I0628 14:46:19.597891   23131 manifests.go:104] [control-plane] adding volume "ca-certs" for component "kube-apiserver"
   I0628 14:46:19.597899   23131 manifests.go:104] [control-plane] adding volume "etc-ca-certificates" for component    "kube-apiserver"
   I0628 14:46:19.597903   23131 manifests.go:104] [control-plane] adding volume "etc-pki" for component "kube-apiserver"
   I0628 14:46:19.597920   23131 manifests.go:104] [control-plane] adding volume "k8s-certs" for component "kube-apiserver"
   I0628 14:46:19.597924   23131 manifests.go:104] [control-plane] adding volume "usr-local-share-ca-certificates" for    component "kube-apiserver"
   I0628 14:46:19.597943   23131 manifests.go:104] [control-plane] adding volume "usr-share-ca-certificates" for component    "kube-apiserver"
   I0628 14:46:19.603068   23131 manifests.go:121] [control-plane] wrote static Pod manifest for component "kube-apiserver" to    "/etc/kubernetes/manifests/kube-apiserver.yaml"
   [control-plane] Creating static Pod manifest for "kube-controller-manager"
   I0628 14:46:19.603099   23131 manifests.go:91] [control-plane] getting StaticPodSpecs
   W0628 14:46:19.603161   23131 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,   RBAC"
   I0628 14:46:19.603328   23131 manifests.go:104] [control-plane] adding volume "ca-certs" for component   "kube-controller-manager"
   I0628 14:46:19.603335   23131 manifests.go:104] [control-plane] adding volume "etc-ca-certificates" for component    "kube-controller-manager"
   I0628 14:46:19.603340   23131 manifests.go:104] [control-plane] adding volume "etc-pki" for component    "kube-controller-manager"
   I0628 14:46:19.603343   23131 manifests.go:104] [control-plane] adding volume "flexvolume-dir" for component   "kube-controller-manager"
   I0628 14:46:19.603347   23131 manifests.go:104] [control-plane] adding volume "k8s-certs" for component    "kube-controller-manager"
   I0628 14:46:19.603350   23131 manifests.go:104] [control-plane] adding volume "kubeconfig" for component   "kube-controller-manager"
   I0628 14:46:19.603354   23131 manifests.go:104] [control-plane] adding volume "usr-local-share-ca-certificates" for    component "kube-controller-manager"
   I0628 14:46:19.603358   23131 manifests.go:104] [control-plane] adding volume "usr-share-ca-certificates" for component    "kube-controller-manager"
   I0628 14:46:19.604036   23131 manifests.go:121] [control-plane] wrote static Pod manifest for component    "kube-controller-manager" to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
   [control-plane] Creating static Pod manifest for "kube-scheduler"
   I0628 14:46:19.604047   23131 manifests.go:91] [control-plane] getting StaticPodSpecs
   W0628 14:46:19.604083   23131 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,   RBAC"
   I0628 14:46:19.604211   23131 manifests.go:104] [control-plane] adding volume "kubeconfig" for component "kube-scheduler"
   I0628 14:46:19.604505   23131 manifests.go:121] [control-plane] wrote static Pod manifest for component "kube-scheduler" to    "/etc/kubernetes/manifests/kube-scheduler.yaml"
   [check-etcd] Checking that the etcd cluster is healthy
   I0628 14:46:19.605209   23131 local.go:78] [etcd] Checking etcd cluster health
   I0628 14:46:19.605217   23131 local.go:81] creating etcd client that connects to etcd pods
   I0628 14:46:19.605222   23131 etcd.go:178] retrieving etcd endpoints from "kubeadm.kubernetes.io/etcd.advertise-client-urls"   annotation in etcd Pods
   I0628 14:46:19.629766   23131 etcd.go:102] etcd endpoints read from pods: https://172.16.0.10:2379
   I0628 14:46:19.646069   23131 etcd.go:250] etcd endpoints read from etcd: https://172.16.0.10:2379
   I0628 14:46:19.646092   23131 etcd.go:120] update etcd endpoints: https://172.16.0.10:2379
   I0628 14:46:19.674660   23131 kubelet.go:111] [kubelet-start] writing bootstrap kubelet config file at /etc/kubernetes/  bootstrap-kubelet.conf
   I0628 14:46:19.675442   23131 kubelet.go:145] [kubelet-start] Checking for an existing Node in the cluster with name   "devsecops-1" and status "Ready"
   I0628 14:46:19.679680   23131 kubelet.go:159] [kubelet-start] Stopping the kubelet
   [kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system    namespace
   [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
   [kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
   [kubelet-start] Starting the kubelet
   [kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...
   I0628 14:46:24.991322   23131 cert_rotation.go:137] Starting client certificate rotation controller
   I0628 14:46:24.993292   23131 kubelet.go:194] [kubelet-start] preserving the crisocket information for the node
   I0628 14:46:24.993303   23131 patchnode.go:30] [patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock"   to the Node API object "devsecops-1" as an annotation
   I0628 14:46:27.016435   23131 local.go:130] creating etcd client that connects to etcd pods
   I0628 14:46:27.016449   23131 etcd.go:178] retrieving etcd endpoints from "kubeadm.kubernetes.io/etcd.advertise-client-urls"   annotation in etcd Pods
   I0628 14:46:27.038161   23131 etcd.go:102] etcd endpoints read from pods: https://172.16.0.10:2379
   I0628 14:46:27.053314   23131 etcd.go:250] etcd endpoints read from etcd: https://172.16.0.10:2379
   I0628 14:46:27.053340   23131 etcd.go:120] update etcd endpoints: https://172.16.0.10:2379
   I0628 14:46:27.053348   23131 local.go:139] Adding etcd member: https://172.16.1.2:2380
   [etcd] Announced new etcd member joining to the existing etcd cluster
   I0628 14:46:27.073331   23131 local.go:145] Updated etcd member list: [{storage https://172.16.0.10:2380} {devsecops-1   https://172.16.1.2:2380}]
   [etcd] Creating static Pod manifest for "etcd"
   [etcd] Waiting for the new etcd member to join the cluster. This can take up to 40s
   I0628 14:46:27.073879   23131 etcd.go:509] [etcd] attempting to see if all cluster endpoints ([https://172.16.0.10:2379    https://172.16.1.2:2379]) are available 1/8
   {"level":"warn","ts":"2020-06-28T14:46:45.350+0530","caller":"clientv3/retry_interceptor.go:61","msg":"retrying of unary   invoker failed","target":"passthrough:///https://172.16.1.2:2379","attempt":0,"error":"rpc error: code = DeadlineExceeded   desc = context deadline exceeded"}
   I0628 14:46:45.350261   23131 etcd.go:489] Failed to get etcd status for https://172.16.1.2:2379: context deadline exceeded
   [upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
   [mark-control-plane] Marking the node devsecops-1 as control-plane by adding the label "node-role.kubernetes.io/master=''"
   [mark-control-plane] Marking the node devsecops-1 as control-plane by adding the taints [node-role.kubernetes.io/  master:NoSchedule]
   
   This node has joined the cluster and a new control plane instance was created:
   
   * Certificate signing request was sent to apiserver and approval was received.
   * The Kubelet was informed of the new secure connection details.
   * Control plane (master) label and taint were applied to the new node.
   * The Kubernetes control plane instances scaled up.
   * A new etcd member was added to the local/stacked etcd cluster.
   
   To start administering your cluster from this node, you need to run the following as a regular user:
   
   	mkdir -p $HOME/.kube
   	sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   	sudo chown $(id -u):$(id -g) $HOME/.kube/config
   
   Run 'kubectl get nodes' to see this node join the cluster.
   ```
* Adding third Master(Optional)
  * Create a new token with a 5 minute TTL.
    ```
    kubeadm token create --ttl 5m --print-join-command
    ```
    Output:
    ```
    kubeadm join 172.16.0.7:6443 \
      --token 9ui8ar.j689k9f1wc43z696 \
      --discovery-token-ca-cert-hash sha256:44175bd8c5956df594f4d0a9db3fa79c59adb1fed6c0ec0459b6166de1c5eb3d
    ```
  * Run the upload-certs phase of kubeadm init
    ```
    sudo kubeadm init phase upload-certs --upload-certs
    ```  
    Output:
    ```
    [upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
    [upload-certs] Using certificate key:
    3bb3894eb962701fead1ba95fec3807dec17a2f24c3a7e8d55ca681e8ec403ec
    ```
  * Use the outputs from the previous steps to run a control-plane join command.
    ```
    kubeadm join 172.16.0.7:6443 \
      --token 9ui8ar.j689k9f1wc43z696 \
      --discovery-token-ca-cert-hash sha256:44175bd8c5956df594f4d0a9db3fa79c59adb1fed6c0ec0459b6166de1c5eb3d \
      --control-plane \
      --certificate-key 3bb3894eb962701fead1ba95fec3807dec17a2f24c3a7e8d55ca681e8ec403ec
    ```  
  * After completion, verify by running `kubectl get nodes`

* Adding Worker node:
   ```
   sudo kubeadm join 172.16.0.7:6443 --token yeu1oi.8n0iiqo84evixjld \
       --discovery-token-ca-cert-hash sha256:44175bd8c5956df594f4d0a9db3fa79c59adb1fed6c0ec0459b6166de1c5eb3d
   W0628 14:48:52.240876   13309 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when   control-plane flag is not set.
   [preflight] Running pre-flight checks
   [preflight] Reading configuration from the cluster...
   [preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
   [kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system    namespace
   [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
   [kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
   [kubelet-start] Starting the kubelet
   [kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...
   
   This node has joined the cluster:
   * Certificate signing request was sent to apiserver and a response was received.
   * The Kubelet was informed of the new secure connection details.
   
   Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
   
   ```