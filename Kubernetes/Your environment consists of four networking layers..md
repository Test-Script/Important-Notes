
# 1. Environment Architecture

Your environment consists of **four networking layers**.

Windows Host Machine
        │
        │  (VirtualBox NAT + Port Forward)
        ▼
Linux VM (Ubuntu)
        │
        │  kubectl port-forward
        ▼
Minikube Kubernetes Cluster
        │
        ▼
Istio Service Mesh
        │
        ▼
Bookinfo Microservices

# 2. Components Running in the Environment

### Host Machine

Windows

Used for:

* Running VirtualBox
* Accessing application via browser / curl

### Virtualization Layer

VirtualBox

Runs a **Linux VM** where Minikube is installed.

Network Mode:

NAT

VM interface:

10.0.2.15

### Kubernetes Environment

Inside the Linux VM:

Minikube

Cluster network:

192.168.58.0/24

Minikube Node IP:

192.168.58.2

### Service Mesh

Installed using:

Istio 1.29

Key components running:

istiod
istio-ingressgateway
istio-proxy (sidecars)

Namespace:

istio-system

### Application Deployed

Istio sample application:

Bookinfo

Services:

productpage
reviews
ratings
details

Call chain:

productpage
     │
     ├── details
     │
     └── reviews
            │
            └── ratings

# 3. Istio Ingress Gateway Configuration

Service:

istio-ingressgateway

Type:

LoadBalancer

Ports:

80:31723/TCP
443:31189/TCP

NodePort used:

31723

# 4. Problem Faced

Direct access from Windows failed because of **multi-layer networking**.

Original traffic path attempted:

Windows
   │
   ▼
VirtualBox NAT
   │
   ▼
Linux VM
   │
   ▼
Minikube internal network (192.168.58.x)
   │
   ▼
Istio NodePort

Issue:

VirtualBox NAT cannot route traffic to Minikube internal network.

Result:

Connection reset
Connection refused

# 5. Solution Implemented

Used **kubectl port-forward** to expose the Istio ingress gateway on the VM interface.

Command executed inside VM:

kubectl port-forward -n istio-system svc/istio-ingressgateway 8080:80 --address 0.0.0.0

This creates a tunnel:

VM:8080  →  Istio IngressGateway:80

Listening interface:

0.0.0.0:8080

Verified using:

sudo ss -tulnp | grep 8080

# 6. VirtualBox Port Forwarding Configuration

Configured rule:

| Name       | Protocol | Host IP   | Host Port | Guest IP  | Guest Port |
| ---------- | -------- | --------- | --------- | --------- | ---------- |
| istio-http | TCP      | 127.0.0.1 | 8080      | 10.0.2.15 | 8080       |

Traffic mapping:

Windows localhost:8080
        │
        ▼
VirtualBox NAT
        │
        ▼
Linux VM 10.0.2.15:8080
        │
        ▼
kubectl port-forward
        │
        ▼
Istio Ingress Gateway

# 7. Final Working Request Flow

End-to-end traffic path:

Windows Browser / Curl
        │
        ▼
http://localhost:8080/productpage
        │
        ▼
VirtualBox NAT Forwarding
        │
        ▼
Linux VM (10.0.2.15:8080)
        │
        ▼
kubectl port-forward
        │
        ▼
Istio IngressGateway
        │
        ▼
Istio Gateway
        │
        ▼
VirtualService Routing
        │
        ▼
productpage service
        │
        ├── details service
        │
        └── reviews service
               │
               └── ratings service

# 8. Verification Steps

Inside Linux VM:

curl http://localhost:8080/productpage
Inside Windows:

curl http://localhost:8080/productpage

Browser:

http://localhost:8080/productpage

Successful response:

Bookinfo Sample Application HTML

# 9. Istio Observability (Optional)

You can visualize service mesh traffic using **Kiali**.

Start dashboard:

istioctl dashboard kiali

Generate traffic:

while true; do curl http://localhost:8080/productpage; sleep 1; done

Kiali graph shows:

productpage
     │
     ├── details
     │
     └── reviews
           │
           └── ratings

Metrics available:

Request rate
Latency
Error rate
mTLS status

# 10. Key Learning From This Setup

Important networking lesson:

VirtualBox NAT cannot reach Minikube internal network directly.

Therefore:

kubectl port-forward acts as a bridge between VM network and Kubernetes network.

# 11. Recommended Production-like Local Setup

Current architecture:

Windows
   │
VirtualBox
   │
Linux
   │
Minikube
   │
Istio

Better development architecture:

Windows
   │
Docker Desktop
   │
Minikube (docker driver)
   │
Istio

Advantages:

No VirtualBox networking issues
Direct localhost access
Simpler networking
Faster startup
