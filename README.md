# Go, Gokit, Gorilla Toolkit REST Service K8s Resources (K8s YAML, K8s Helm, K8s Kustomize)
K8s GitOps Resource Styles (K8s YAML, K8s Helm and K8s Kustomize) for Go Project for Web Service go-gokit-gorilla-restsvc at Gitub repo `https://github.com/isgo-golgo13/go-gokit-gorilla-restsvc.git`. This is the Git source repo for ArgoCD (isgo-golgo13/isgogolgo13-argocd-apps) and FluxCD (isgo-golgo13/isgogolgo13-fluxcd-apps).


### Development Workflow (Create the Cluster)

```
Creates a 3 Server Node/3 Worker/Agent Node Cluster

k3d cluster create go-gokit-gorilla-restsvc-cluster --api-port 127.0.0.1:6443 --k3s-server-arg "--disable=traefik" --k3s-server-arg "--disable=metrics-server" -p 80:80@loadbalancer -p 443:443@loadbalancer --agents 3 --servers 3 --verbose
```
**OR** 

```
Creates a 3 Server Node/3 Worker/Agent Node Cluster

k3d cluster create go-gokit-gorilla-restsvc-cluster --api-port 127.0.0.1:6443 --k3s-server-arg "--no-deploy=traefik" -p 80:80@loadbalancer -p 443:443@loadbalancer --agents 3 --servers 3 --verbose

```

### Running the K8s (K8s YAML Approach, K8s Helm Approach)

To run the `sgogolgo13/go-gokit-gorilla-restsvc` on K8s cluster we will use Rancher `k3d` using Traefik 2.2 LoadBalancer/Ingress Controller as of 1-25-2020
k3d included Traefik uses v1 and to use v2 the step to include v2 is to provide arguments to the `k3d cluster create` command.

**1.** Create the K3D cluster (default 3 server node, 3 agent node) - Disable default Traefik v1 Load Balancer/Ingress (See top of page as this is again depicted)
```
k3d cluster create go-gokit-gorilla-restsvc-cluster --api-port 127.0.0.1:6443 --k3s-server-arg "--disable=traefik" --k3s-server-arg "--disable=metrics-server" -p 80:80@loadbalancer -p 443:443@loadbalancer --agents 3 --servers 3 --verbose
```

The following flags and flag arguments are as follows:

| Flag                    |                        | Result                                                                | 
|:-----------------------:|:----------------------:|:--------------------------------------------------------------------- | 
| --api-port              |  127.0.0.1:6443        | k3d cluster will serve at IP: 127.0.0.1 (localhost) on port 6443      | 
| -p                      | 80:80@loadbalancer     | tie localhost port 80 to port 80 to k3d virtual load balancer         | 
| -p                      | 443:443@loadbalancer   | tie localhost port 443 to port 443 to k3d virtual load balancer       | 
| --k3s-server-arg        | "--no-deploy=traefik"  | **disable Traefik load balancer/ingress v1 to override with v2**      |
| - agents                |       3                | create cluster with 3 agent/worker nodes (HA if server nodes 3)       | 
| --servers               |       3                | create cluster with 3 server nodes (HA if agent nodes 3)              |
| --verbose               |       (no arg)         | turns on full verbose mode on during creation of cluster              |



**2.** Install Traefik Load Balancer/Ingress Controller v2

**Check if Traefik v2 Helm Repository is Listed Locally as Cached/Downloaded**
```
helm repo list 
```

If Check DOES NOT Include Locally Cached/Downloaded, Download the Chart Repository to Cache
```
helm repo add traefik https://containous.github.io/traefik-helm-chart  
```

Download/Install to Running K8s Cluister (Check K8s Context Config to Ensure Install to Correct Cluster)

```
kubectx 

helm install traefik traefik/traefik
```

The result should show :

```
NAME: traefik
LAST DEPLOYED: Mon Jan 25 14:52:09 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

**3.** Check Traefik Dashboard 

```
kubectl port-forward $(kubectl get pods --selector "app.kubernetes.io/name=traefik" --output=name) 9000:9000
```

Now go to `http://localhost:9000/dashboard` to check if Traefik Dashboard is serving.


**4.** Deploy K8s Resources to k3d Cluster

The following will show how to deploy the K8s Resources **(first using K8s YAML, second K8s Helm Chart)**

#### K8s YAML Approach 

For K8s YAML deployment issue in the exact order as follows:

From the `k8s/k8s-yaml` diretory (at the shell):

`1.` Deploy the `deployment.yaml` file
```
kubectl create -f deployment.yaml
```

`2.` Deploy the `service.yaml` file
```
kubectl create -f  service.yaml
```

`3.` Deploy the `ingress.yaml` file (annotated ingress using Traefik)
```
kubectl create -f ingress.yaml
```



#### K8s Helm Approach (Helm 3)

For K8s Helm deployment issue as follows:

From the `k8s/k8s-helm/gokit-gorilla-restsvc-chart` diretory (at the shell):

`1.` Render the Helm Chart (Local Render)
```
helm template gokit-gorilla-restsvc-chart .
```

`2.` Deploy the Helm Chart
```
Syntax:
helm upgrade -i <release-name> <chart-dir>

Actual Use:
helm upgrade -i gokit-gorilla-restsvc-chart .
```
Helm handles the correct deployment order of the underlying K8s resources (first deployment, second the service, third and final the ingress)

`3.` Uninstall the Chart
```
Syntax:
helm uninstall <release-name>

Actual Use:
helm uninstall gokit-gorilla-restsvc-chart
```



### Execute the K8s Deployed Application 

#### Create/Register an Engine:

```bash
$ curl -d '{"id":"00001","factory_id":"utc_pw_10-0001", "engine_config" : "Radial", "engine_capacity": 660.10, "fuel_capacity": 400.00, "fuel_range": 240.60}' -H "Content-Type: application/json" -X POST http://localhost/engines/
{}
```

Here the `http://localhost/engines/` does not need to explicitly reference the port (port 80 defined in the ingress.yaml) or run `kubectl get ingress` as the `ingress.yaml` as detailed earlier defined the `servicePort` which is port 80 that in-turn is tied to the `service` port of 80 and in turn the service is tied to the `containerPort` defined in the `deployment.yaml` as 8080. **The Go REST app is running on 8080.**

#### Retrieve an Engine
 
```bash
$ curl http://localhost/engines/00001
{"engine":{"id":"00001","factory_id":"utc_pw_10-0001", "engine_config" : "Radial", "engine_capacity": 660.10, "fuel_capacity": 400.00, "fuel_range": 240.60}}
```


###
