# Go, Gokit, Gorilla Toolkit REST Service (isgo-golgo13/go-gokit-gorilla-restsvc gitrepo app) K8s Resources (K8s YAML, K8s Helm, K8s Kustomize)
K8s GitOps Resource Styles (K8s YAML, K8s Helm and K8s Kustomize) for Go Project for Web Service go-gokit-gorilla-restsvc at Gitub repo `https://github.com/isgo-golgo13/go-gokit-gorilla-restsvc.git`. This is the Git source repo for ArgoCD (isgo-golgo13/isgogolgo13-argocd-apps) and FluxCD (isgo-golgo13/isgogolgo13-fluxcd-apps).


### Development Workflow (Create the Cluster)

```
# Create a local k3d cluster with the appropriate port forwards (1 server node/1 agent node default with load balancer)

k3d cluster create go-gokit-gorilla-restsvc-cluster --api-port 127.0.0.1:6443 --k3s-server-arg "--disable=traefik" --k3s-server-arg "--disable=metrics-server" -p 80:80@loadbalancer -p 443:443@loadbalancer 

or

k3d cluster create go-gokit-gorilla-restsvc-cluster --api-port 127.0.0.1:6443 --k3s-server-arg "--no-deploy=traefik" -p 80:80@loadbalancer -p 443:443@loadbalancer

```

### K8s YAML Workflow

### K8s Helm Workflow

### K8S Kustomize Workflow
