### Step-by-Step Guide for Installing Istio 1.24.0 on Kubernetes

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

For the following manifest.yaml file, we will use only the relevant file in the repository belonging to Google: https://github.com/GoogleCloudPlatform/microservices-demo/blob/main/release/kubernetes-manifests.yaml

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
![Screenshot 2024-11-15 at 16 56 24](https://github.com/user-attachments/assets/21487ca8-a41d-4e76-b771-ce4c86e4a005)

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

---






