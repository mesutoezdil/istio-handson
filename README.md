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
<img width="479" alt="Screenshot 2024-11-15 at 14 50 46" src="https://github.com/user-attachments/assets/7d67be1e-8d44-48d5-a61d-5e89f4e5c46e">

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
---



