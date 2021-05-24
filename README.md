# Go, Gokit, Gorilla Toolkit REST Service K8s Resources (K8s YAML, K8s Helm, K8s Kustomize)
K8s GitOps Resource Styles (K8s YAML, K8s Helm and K8s Kustomize) for Go Project for Web Service go-gokit-gorilla-restsvc at Gitub repo `https://github.com/isgo-golgo13/go-gokit-gorilla-restsvc.git`. This is the Git source repo for ArgoCD (isgo-golgo13/isgogolgo13-argocd-apps) and FluxCD (isgo-golgo13/isgogolgo13-fluxcd-apps).


### Development Workflow (Create the Cluster)

This creates a 3 Server Node/3 Worker/Agent Node Cluster.
```
k3d cluster create go-gokit-gorilla-restsvc-cluster --api-port 127.0.0.1:6443 --k3s-server-arg "--disable=traefik" -p 80:80@loadbalancer -p 443:443@loadbalancer --agents 3 --servers 3 --verbose
```
**or** 

For a 1 Server Node/3 Worker/Agent Node Cluster.
```
k3d cluster create go-gokit-gorilla-restsvc-cluster --api-port 127.0.0.1:6443 --k3s-server-arg "--no-deploy=traefik" -p 80:80@loadbalancer -p 443:443@loadbalancer --agents 3 --servers 1 --verbose
```

Both `k3d cluster create` commands will automatically enable the K8s metrics server that is required for using the VPA (Vertical Pod Autoscaler) or the HPA (Horizontal Pod Autoscaler). The HPA requirement is that CPU and/or Memory resources configs on the Deployment are defined.

### Running the K8s (K8s YAML Approach, K8s Helm Approach)

To run the `go-gokit-gorilla-restsvc` on K8s cluster we will use Rancher `k3d` using Traefik 2.2 LoadBalancer/Ingress Controller (k3d originally included Traefik v1 and lacked production features Traefik v2 includes) and to use v2 the step to include v2 is to provide arguments to the `k3d cluster create` command.

#### 1. Create the K3D cluster (default 3 server node, 3 agent node) - Disable default Traefik v1 Load Balancer/Ingress (See top of page as this is again depicted)
```
k3d cluster create go-gokit-gorilla-restsvc-cluster --api-port 127.0.0.1:6443 --k3s-server-arg "--disable=traefik" -p 80:80@loadbalancer -p 443:443@loadbalancer --agents 3 --servers 3 --verbose
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



#### 2. Install Traefik Load Balancer/Ingress Controller v2

**Check if Traefik v2 Helm Repository is Listed Locally as Cached/Downloaded**
```
helm repo list 
```

**If Check DOES NOT Include Locally Cached/Downloaded, Download the Chart Repository to Cache**
```
helm repo add traefik https://containous.github.io/traefik-helm-chart  
```

**Download/Install to Running K8s Cluster (Check K8s Context Config to Ensure Install to Correct Cluster)**

```
kubectx #kubectx checks the current cluster context prior to any deployments

helm install traefik traefik/traefik
```

**If Executed Correctly, Deployment Status SHOULD Show:**

```
NAME: traefik
LAST DEPLOYED: Mon Jan 25 14:52:09 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

#### 3. Check Traefik Dashboard 

```
kubectl port-forward $(kubectl get pods --selector "app.kubernetes.io/name=traefik" --output=name) 9000:9000
```

Now go to `http://localhost:9000/dashboard` to check if Traefik Dashboard is serving.


#### 4. Deploy K8s Resources to k3d Cluster

The following will show how to deploy the K8s Resources **(first using K8s YAML, second K8s Helm Chart)**

#### Using the K8s Autoscaler (HPA - Horizontal Pod Autoscaler)

In the `k8s-yaml, k8s-helm, k8s-istio-helm, k8s-istio-yaml, k8s-istio-kustomize and k8s-istio-kustomize` directories, all the K8s resources have defined a HorizontalPodAutoscaler to auto-scale on CPU or Memory the microservice pods on a conditional threshold of CPU or Memory.

To use the HPA, the K8s `metrics-server' MUST be actively installed and running on the K8s cluster. This Git repo uses K3D K8s cluster which has the metrics-server installed and running as a default. To check the metrics-server is running do the following at the shell (after you have created the K3D K8s cluster):

```
kubectl top nodes
```

This will show CPU and Memory for the running nodes if and only if the metrics-server is actively running.


If the K8s resources are deployed to the cluster using either the provided K8s raw YAML directories, the K8s Helm directories or the K8s Kustomize directories, the HPA will require an HTTP traffic load generator to trigger the auto-scaling as the CPU or Memory is throttled upward to the threshold levels (defined in the HPA). The Linux `siege` toolchain is this HTTP traffic load generator. 

To trigger the throttling of concurrent requests to escalate the CPU or Memory do the following from a seperate shell:

```
Syntax:

siege -q -c <# of concurrent requests> -t <requested length of time of active requests> <url of K8s service exposed from ingress load balancer>

Actual use: 

This calls the HTTP REST /engines/<ID> API concurrently 100x for 5 min at the ingress controller exposed endpoint of `http://localhost/engines/00001`. The `/00001` ID is assuming that an HTTP POST REST API `http://localhost/engines/` with a JSON POST payload was sent to the server and stored in its DB.

siege -q -c 100 -t 5m http://localhost/engines/00001

```

To watch the installed HPA at work, run the   `kubectl get hpa` and `kubectl describe hpa <hpa-name>` as it will
take several minutes for the HPA to take effect and as the `siege` command is applied to simulate the HTTP traffic load into the cluster service, this will spike up the CPU or the Memory (depending on the config for CPU or Memory in the HPA) and as the threshold limit is eventually hit, the HPA will scale the `REPLICAS` to the `MAX PODS` configured in the HPA.

After the 5 min request window expires, the cool down period will occur and the scale down of CPU or Memory will start. The final status of the REPLICAS will go from MAX defined in the HPA to the MIN of 1.




#### K8s YAML Approach 

For K8s YAML deployment issue in the exact order as follows:

From the `k8s` directory (at the shell):

```
kubectl apply -f ./k8s-yaml
```

This will create all the K8s resources into the K8s K3D cluster from the `k8s/k8s-yaml` directory.





#### K8s Helm Approach (Helm 3)

For K8s Helm deployment issue as follows:

From the `k8s/k8s-helm/gokit-gorilla-restsvc-chart` directory (at the shell):

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
