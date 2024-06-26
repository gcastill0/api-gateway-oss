```bash
cat <<EOF | kind create cluster --name gloo-gateway --image kindest/node:v1.29.2 --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 31500
    hostPort: 31500
    protocol: TCP
  - containerPort: 32500
    hostPort: 32500
    protocol: TCP
EOF
```

```
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.29.2) 🖼 
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-gloo-gateway

Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂
```

```bash
kubectl cluster-info --context kind-gloo-gateway
```

```
Kubernetes control plane is running at https://127.0.0.1:52684
CoreDNS is running at https://127.0.0.1:52684/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```