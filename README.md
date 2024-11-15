### Step-by-Step Guide for Installing Istio 1.24.0 on Kubernetes

### Pre-requisites
- A Kubernetes cluster running and accessible (`kubectl get nodes` should return nodes)
- Kubernetes CLI (`kubectl`) installed and configured
- Sufficient resources on the cluster (2 CPUs, 4GB RAM recommended for Istio + Add-ons)

It is possible to perform this setup in different environments. I tried it in the [K3s playground environment by iximiuz](https://labs.iximiuz.com/playgrounds/k3s). Below, you can find all the steps and code step by step.

![Screenshot 2024-11-15 at 14 52 50](https://github.com/user-attachments/assets/c73c424b-222b-4138-b707-d7c024baef6a)

---

### 1. **Create a Directory for Istio Installation**
```bash
mkdir istio-installation
```
This step creates a dedicated directory to organize Istio installation files.

---

### 2. **Download Istio**
```bash
curl -L https://istio.io/downloadIstio | sh -
```
This command fetches the Istio installation script, downloads the latest version (1.24.0), and extracts it into a folder named `istio-1.24.0`.

---

### 3. **Move the Downloaded Istio Folder into the Created Directory**
```bash
mv istio-1.24.0/ istio-installation/
```
The downloaded `istio-1.24.0` directory is moved under `istio-installation` for better project structure.

---

### 4. **Navigate to the Istio Directory**
```bash
cd istio-installation/
```
Change into the newly created directory where Istio files are stored.

---

### 5. **Verify the Istio Files are Downloaded**
```bash
ls
```
This confirms that the `istio-1.24.0` folder is present in the `istio-installation` directory.

---

### 6. **Navigate to the Istio Binary Directory**
```bash
cd istio-1.24.0/bin
```
Navigate to the `bin` directory where the `istioctl` binary is located.

---

### 7. **Verify the Presence of `istioctl`**
```bash
ls
```
Ensure that the `istioctl` binary is present in the directory.

---

### 8. **Add `istioctl` to the PATH Environment Variable**
```bash
export PATH=$PATH:/home/laborant/istio-installation/istio-1.24.0/bin
```
This step temporarily adds the directory containing `istioctl` to the `PATH` environment variable, enabling global access to the binary.

---

### 9. **Verify `istioctl` Command**
```bash
istioctl
```
Test the `istioctl` command. If the `PATH` was correctly set, the command should list available options and commands.

---

### 10. **Make the PATH Change Permanent**
```bash
echo 'export PATH=$PATH:/home/laborant/istio-installation/istio-1.24.0/bin' >> ~/.bashrc
source ~/.bashrc
```
Appending the `export PATH` command to `~/.bashrc` ensures that the `istioctl` binary remains globally accessible even after a terminal restart.

---

### 11. **Verify Kubernetes Cluster Accessibility**
```bash
kubectl get ns
kubectl get pods
```
Verify that the Kubernetes cluster is accessible and operational. The namespace (`ns`) and pods list should display correctly.

---

### 12. **Install Istio on the Kubernetes Cluster**
```bash
istioctl install
```
The installation process applies Istio's default profile to the cluster. Confirm the prompt with `y`.

---

### Expected Installation Output:
```plaintext
This will install the Istio 1.24.0 profile "default" into the cluster. Proceed? (y/N) y
✔ Istio core installed ⛵️
✔ Istiod installed 🧠
✔ Ingress gateways installed 🛬
✔ Installation complete
```
<img width="852" alt="Screenshot 2024-11-15 at 15 55 36" src="https://github.com/user-attachments/assets/c2d8e25f-47d0-4d45-8990-acca8d1539da">

The Istio control plane and ingress gateway are successfully installed in the cluster.

### 13. **Verify Istio Installation Namespaces**
```bash
kubectl get ns
```
This command lists all namespaces in the Kubernetes cluster. You should see the `istio-system` namespace, which is created as part of the Istio installation.

---

### 14. **Check Istio Pods in the `istio-system` Namespace**
```bash
kubectl get pod -n istio-system
```
This verifies that the Istio control plane (`istiod`) and ingress gateway pods are running successfully.  

**Expected Output:**
```plaintext
NAME                                   READY   STATUS    RESTARTS   AGE
istio-ingressgateway-8cd544cbd-wnvlg   1/1     Running   0          8m8s
istiod-6b5fb7b484-7lbjw                1/1     Running   0          8m16s
```

Download the file from [Google's Microservices Demo Repository](https://github.com/GoogleCloudPlatform/microservices-demo/blob/main/release/kubernetes-manifests.yaml).
Save it as `kubernetes-manifests.yaml` in the `istio-installation` directory.

### 15. **Apply Kubernetes Manifest File**

```bash
kubectl apply -f kubernetes-manifests.yaml
```

This command deploys all services, deployments, and service accounts defined in the provided `kubernetes-manifests.yaml` file to the Kubernetes cluster.

**Expected Output:**
```plaintext
deployment.apps/emailservice created
service/emailservice created
serviceaccount/emailservice created
...
deployment.apps/adservice created
service/adservice created
serviceaccount/adservice created
```

---

### 16. **Verify Pods in the Default Namespace**

```bash
kubectl get pod
```

This command lists all the pods deployed in the default namespace.

**Example Initial Output:**
```plaintext
NAME                                     READY   STATUS              RESTARTS   AGE
adservice-997b6fc95-4fhmn                0/1     Running             0          20s
cartservice-59d7459964-br6q5             0/1     Running             0          21s
...
recommendationservice-7d59447ccc-bsbfp   0/1     ContainerCreating   0          22s
redis-cart-558f8d8d44-5k8g8              1/1     Running             0          21s
```

Pods might initially show as `ContainerCreating` or `Init:0/1` because Kubernetes is pulling images and starting containers.

---

### 17. **Check Pods Again After Initialization**

```bash
kubectl get pod
```

**Expected Final Output:**
```plaintext
NAME                                     READY   STATUS    RESTARTS   AGE
adservice-997b6fc95-4fhmn                1/1     Running   0          80s
cartservice-59d7459964-br6q5             1/1     Running   0          81s
checkoutservice-855ff8d99-fwwnw          1/1     Running   0          82s
currencyservice-b7dcd96f5-9zqpl          1/1     Running   0          81s
emailservice-69cfd9755c-dl2vs            1/1     Running   0          82s
frontend-c48bb8c56-bjf7b                 1/1     Running   0          82s
loadgenerator-6fd8676bfc-dn998           1/1     Running   0          81s
paymentservice-7c56f54965-7lzck          1/1     Running   0          82s
productcatalogservice-69878b7d5c-b2c46   1/1     Running   0          82s
recommendationservice-7d59447ccc-bsbfp   1/1     Running   0          82s
redis-cart-558f8d8d44-5k8g8              1/1     Running   0          81s
shippingservice-fb69f4985-7z69r          1/1     Running   0          81s
```

At this point, all pods should be in the `Running` state.

---
<img width="821" alt="Screenshot 2024-11-15 at 16 01 29" src="https://github.com/user-attachments/assets/01e277fd-377a-408e-98dd-1f97bdb140b9">

### 19. **Check Namespace Labels**

```bash
kubectl get ns default --show-labels
```

This command lists the labels of the `default` namespace, where the services are currently deployed.

**Example Output:**
```plaintext
NAME      STATUS   AGE    LABELS
default   Active   102m   kubernetes.io/metadata.name=default
```

---

### 20. **Enable Istio Sidecar Injection in the `default` Namespace**

This enables automatic sidecar injection, meaning Istio will add a sidecar proxy (`istio-proxy`) to each pod in the namespace. This proxy intercepts traffic for service mesh features like traffic control, monitoring, and security.


```bash
kubectl label namespace default istio-injection=enabled
```

This command enables automatic sidecar injection for all pods in the `default` namespace by adding the label `istio-injection=enabled`.

**Example Output:**
```plaintext
namespace/default labeled
```

Verify the label:
```bash
kubectl get ns default --show-labels
```

**Expected Output:**
```plaintext
NAME      STATUS   AGE    LABELS
default   Active   103m   istio-injection=enabled,kubernetes.io/metadata.name=default
```

---

### 21. **Delete Existing Resources**

Delete all existing resources to allow redeployment with sidecar injection:

```bash
kubectl delete -f kubernetes-manifests.yaml
```

**Example Output:**
```plaintext
deployment.apps "emailservice" deleted
service "emailservice" deleted
serviceaccount "emailservice" deleted
...
deployment.apps "adservice" deleted
service "adservice" deleted
serviceaccount "adservice" deleted
```

Verify that no pods are running in the `default` namespace:

```bash
kubectl get pod
```

**Expected Output:**
```plaintext
No resources found in default namespace.
```

---

### 22. **Redeploy the Services**

Apply the manifest file again to deploy all services with sidecar injection:

```bash
kubectl apply -f kubernetes-manifests.yaml
```

**Example Output:**
```plaintext
deployment.apps/emailservice created
service/emailservice created
serviceaccount/emailservice created
...
deployment.apps/adservice created
service/adservice created
serviceaccount/adservice created
```

---

### 23. **Verify Pods**

Check the status of the pods in the `default` namespace:

```bash
kubectl get pod
```

All pods should now have the Istio sidecar container (`istio-proxy`) injected and should be in the `Running` state.
<img width="786" alt="Screenshot 2024-11-15 at 17 28 56" src="https://github.com/user-attachments/assets/d159f89e-3fee-4831-9671-5f7b846ef9d2">

### 24. **Verify Pods with Istio Sidecar Containers**

Run the following command to list all pods in the default namespace and check if the Istio sidecar containers are injected:

```bash
kubectl get pod
```

**Expected Output:**
```plaintext
NAME                                     READY   STATUS    RESTARTS   AGE
adservice-997b6fc95-fvggf                2/2     Running   0          94s
cartservice-59d7459964-ln7nh             2/2     Running   0          95s
checkoutservice-855ff8d99-m6tjq          2/2     Running   0          95s
currencyservice-b7dcd96f5-869xr          2/2     Running   0          94s
emailservice-69cfd9755c-j8f64            2/2     Running   0          95s
frontend-c48bb8c56-smp4v                 2/2     Running   0          95s
loadgenerator-6fd8676bfc-zc92n           2/2     Running   0          95s
paymentservice-7c56f54965-28xgp          2/2     Running   0          95s
productcatalogservice-69878b7d5c-64bzq   2/2     Running   0          95s
recommendationservice-7d59447ccc-h9pkf   2/2     Running   0          95s
redis-cart-558f8d8d44-4thd4              2/2     Running   0          95s
shippingservice-fb69f4985-47m9p          2/2     Running   0          94s
```

Each pod should now show `2/2` under the `READY` column, indicating both the application container and the Istio sidecar container (`istio-proxy`) are running.

---

### 25. **Describe a Pod to Inspect Sidecar Configuration**

Use the `kubectl describe pod` command to examine a specific pod's configuration and ensure the Istio sidecar (`istio-proxy`) is properly injected:

```bash
kubectl describe pod <pod-name>
```

For example:

```bash
kubectl describe pod adservice-997b6fc95-fvggf
```

**Key Points to Look For:**
1. **Annotations:**  
   The pod should have `istio.io/rev: default` and `sidecar.istio.io/status` annotations, confirming sidecar injection.

2. **Containers:**  
   There should be two containers:
   - `server`: The primary application container.
   - `istio-proxy`: The Istio sidecar container.

3. **Init Container:**  
   The `istio-init` init container should appear under `Init Containers`. This sets up iptables rules for redirecting traffic through the sidecar proxy.

4. **Resource Requests and Limits:**  
   Verify that resource limits and requests are set correctly for both the application and sidecar containers.

---

### 26. **Verify Istio Sidecar Status**

The following fields in the `kubectl describe pod` output confirm that the Istio sidecar is functioning correctly:
- **State of `istio-proxy`:** `Running`
- **Readiness and Liveness Probes:** Ensure HTTP probes for Istio proxy and application containers are configured.
- **Annotations and Labels:** Check Istio-specific annotations such as `sidecar.istio.io/status`.

**Example Output:**
```plaintext
Annotations:
  istio.io/rev: default
  sidecar.istio.io/status:
    {"initContainers":["istio-init"],"containers":["istio-proxy"],"volumes":["workload-socket","credential-socket","workload-certs","istio-envoy","istio-data"]}

Containers:
  - server (Application)
  - istio-proxy (Sidecar)
Init Containers:
  - istio-init
```

### 27. **Navigate to the Add-ons Directory**
<img width="846" alt="Screenshot 2024-11-15 at 17 36 38" src="https://github.com/user-attachments/assets/1e0f92cb-f714-45b1-a7e6-d5e3bd94e1c5">

Navigate to the directory containing the add-ons YAML files:
```bash
cd istio-1.24.0/samples/addons/
```

---

### 28. **List Available Add-ons**

Check the available YAML files for Istio add-ons:
```bash
ls
```

**Expected Output:**
```plaintext
README.md  extras  grafana.yaml  jaeger.yaml  kiali.yaml  loki.yaml  prometheus.yaml
```

---

### 29. **Deploy the Kiali Dashboard**

Apply the `kiali.yaml` configuration file to set up the Kiali dashboard:
```bash
kubectl apply -f kiali.yaml
```

**Expected Output:**
```plaintext
serviceaccount/kiali created
configmap/kiali created
clusterrole.rbac.authorization.k8s.io/kiali created
clusterrolebinding.rbac.authorization.k8s.io/kiali created
service/kiali created
deployment.apps/kiali created
```

---

### 30. **Deploy All Add-ons**

Deploy all the add-ons (Grafana, Jaeger, Loki, Prometheus, etc.) by applying the entire directory:
```bash
kubectl apply -f .
```

**Expected Output:**
```plaintext
serviceaccount/grafana created
configmap/grafana created
service/grafana created
deployment.apps/grafana created
...
deployment.apps/prometheus created
```

---

### 31. **Verify Pod Status**

Check the status of the deployed add-ons in the `istio-system` namespace:
```bash
kubectl get pod -n istio-system
```

**Example Output:**
```plaintext
NAME                                   READY   STATUS    RESTARTS   AGE
grafana-6b45c49476-hcmfl               1/1     Running   0          59s
istio-ingressgateway-8cd544cbd-vglsc   1/1     Running   0          116m
istiod-6b5fb7b484-sqbh7                1/1     Running   0          116m
jaeger-54c44d879f-s6krp                1/1     Running   0          59s
kiali-79b6d98d5d-cr48x                 1/1     Running   0          74s
loki-0                                 1/2     Running   0          59s
prometheus-6dd9fd5446-kqqkx            2/2     Running   0          59s
```

---

### 32. **Verify Services**

Check the services exposed by the add-ons:
```bash
kubectl get svc -n istio-system
```

**Example Output:**
```plaintext
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                          AGE
grafana                ClusterIP      10.43.243.141   <none>        3000/TCP                                         86s
istio-ingressgateway   LoadBalancer   10.43.56.56     <pending>     15021:31574/TCP,80:30360/TCP,443:31875/TCP       116m
istiod                 ClusterIP      10.43.193.247   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP            117m
jaeger-collector       ClusterIP      10.43.83.203    <none>        14268/TCP,14250/TCP,9411/TCP,4317/TCP,4318/TCP   86s
kiali                  ClusterIP      10.43.28.80     <none>        20001/TCP,9090/TCP                               101s
loki                   ClusterIP      10.43.231.28    <none>        3100/TCP,9095/TCP                                86s
prometheus             ClusterIP      10.43.209.227   <none>        9090/TCP                                         86s
```

Each service (Grafana, Kiali, Prometheus, Jaeger, etc.) should be listed and accessible on their respective ports.

---

### 33. **Access Dashboards**

Use `kubectl port-forward` to access the dashboards locally:

- **Kiali:**
  ```bash
  kubectl port-forward -n istio-system svc/kiali 20001:20001
  ```

- **Grafana:**
  ```bash
  kubectl port-forward -n istio-system svc/grafana 3000:3000
  ```

- **Prometheus:**
  ```bash
  kubectl port-forward -n istio-system svc/prometheus 9090:9090
  ```

- **Jaeger:**
  ```bash
  kubectl port-forward -n istio-system svc/jaeger-collector 16686:16686
  ```

Open the respective dashboards in your browser using `localhost:<PORT>`.

### Dashboard Overview
- **Kiali Dashboard**: View and manage Istio's service mesh topology
- **Grafana Dashboard**: Pre-configured dashboards for Istio metrics (CPU, memory, latency, etc.)
- **Prometheus Dashboard**: Query Istio and Kubernetes metrics directly
- **Jaeger Dashboard**: Trace requests across microservices for debugging and performance analysis

