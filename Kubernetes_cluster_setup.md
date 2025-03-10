# Step 1: Prerequisites

1. **Install Kubernetes components on all nodes:**
   - `kubectl`
   - `kubeadm`
   - Container runtime (e.g., Docker or containerd)

2. **Ensure minimum system requirements:**
   - **Master Node:** 2 CPUs, 2GB RAM, 20GB storage
   - **Worker Nodes:** 1 CPU, 2GB RAM, 10GB storage

3. **Prepare the hosts:**
   ```
   sudo swapoff -a
   sudo systemctl stop firewalld
   sudo systemctl disable firewalld
   sudo modprobe br_netfilter
   sudo sysctl net.bridge.bridge-nf-call-iptables=1
   sudo sysctl net.ipv4.ip_forward=1
   sudo systemctl enable kubelet
   sudo systemctl start kubelet
```


### Step 2: Initialize the Kubernetes Control Plane (Master Node)
1.	Run kubeadm init:

```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```
2.	Save the kubeadm join command output for worker nodes

### Step 3: Set Up kubectl on the Master Node
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get nodes
```
### Step 4: Install a Network Plugin (CNI)

1.	Install Calico:
```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
2.	Wait for network pods to be ready:
```
kubectl wait --for=condition=Ready pods --all -n kube-system --timeout=300s
```
### Step 5: Join Worker Nodes to the Cluster

1.	Run the saved kubeadm join command on each worker node.
2.	Verify nodes on the master:
```
kubectl get nodes
```
### Step 6: Validate the Cluster

1.	Check node status:
```
kubectl get nodes
```
2.	Verify control plane components:
```
kubectl get pods -n kube-system
```
3.	Run a test deployment:
```
kubectl create deployment nginx --image=nginx
```
4.	Expose the deployment
```
kubectl expose deployment nginx --port=80 --type=LoadBalancer
```
5.	Access the service:
```
kubectl get svc nginx
Access using http://<master-node-ip>:<port>
```
6.	Verify pod scheduling across nodes:
```
kubectl create deployment nginx-test --image=nginx --replicas=3
kubectl get pods -o wide
```
7.	Test rolling updates:
```
kubectl set image deployment/nginx-test nginx=nginx:1.16.1
kubectl rollout status deployment/nginx-test
```

### Step 7: Perform Final Health Checks
1.	Get cluster info:
```
kubectl cluster-info
```
2.	Check API server health:
```
kubectl get --raw='/healthz?verbose'
```

