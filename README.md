# Example of using Kustomize to deploy multiple environments to a Digital Ocean Kubernetes cluster

## Cluster Setup

### STEP 1: Create a Kubernetes Cluster on Digital Ocean

https://cloud.digitalocean.com/kubernetes/clusters

### STEP 2: Install doctl

```shell
sudo snap install doctl
sudo snap connect doctl:kube-config
doctl auth init
```

### STEP 3: Add your new cluster to kubeconfig

See "Connecting and managing this cluster" on the Overview tab of your cluster in Digital Ocean.

The command will be something like this:
```shell
doctl kubernetes cluster kubeconfig save <YOUR_CLUSTER_ID_HERE>
```

### STEP 4: Install the Nginx Ingress Controller
  
```shell
helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.publishService.enabled=true
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.publishService.enabled=true
```

Reference:  
https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-on-digitalocean-kubernetes-using-helm#step-2-installing-the-kubernetes-nginx-ingress-controller

### STEP 5: Apply manifests for both the test and production environment

```shell
kubectl apply -f overlays/test
kubectl apply -f overlays/prod
```

### STEP 6: Setup networking

In Digital Ocean, go to Networking and get the IP address of the load balancer.
https://cloud.digitalocean.com/networking/load_balancers

**For local testing only:**  
Create an entry in your local `/etc/hosts` resolving both `example.cinlogic.com` and `example-test.cinlogic.com` to the load balancer's IP address.

**DNS Configuration:**  
Create an `A` record in your DNS resolving to the load balancer's IP address.

### STEP 7: Test it out

Open a browser and navigate to: 
* http://example.cinlogic.com/si
* http://example-test.cinlogic.com/si

Check the environment variables. They should have different `ENV` and `MESSAGE` for test and prod environments.

Refresh the page. You should be seeing requests are being distributed to different pods.
