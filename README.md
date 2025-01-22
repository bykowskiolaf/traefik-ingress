# Tutorial: Deploying Traefik with Cloudflare and Whoami Application

## Prerequisites
1. **Kubernetes Cluster**: Ensure you have a running Kubernetes cluster (e.g., K3s).
2. **kubectl**: Installed and configured to connect to your cluster.
3. **Cloudflare API Token**: You’ll need a Cloudflare API token with DNS edit permissions.

---

## Step 1: Set Up the Namespace
1. **Create a Namespace**:
   Apply the namespace definition:
   ```bash
   kubectl apply -f 00-namespace.yaml
   ```

2. **Verify the Namespace**:
   Ensure the namespace is created:
   ```bash
   kubectl get namespaces
   ```

---

## Step 2: Configure the Cloudflare API Token Secret
The file `01-cloudflare-api-token-secret.yaml` is ignored in version control for security reasons.

1. **Create the Secret File**:
   ```bash
   cp certification/01-cloudflare-api-token-secret.yaml.template certification/01-cloudflare-api-token-secret.yaml
   ```

2. **Edit the File**:
   Replace `<YOUR_CLOUDFLARE_API_TOKEN>` with your Cloudflare API token:
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: cloudflare-api-token-secret
     namespace: cert-manager
   stringData:
     api-token: <YOUR_CLOUDFLARE_API_TOKEN>
   ```

---

## Step 3: Deploy Cert-Manager for Cloudflare Certificate

1. **Apply the Cloudflare API Secret**:
   ```bash
   kubectl apply -f certification/01-cloudflare-api-token-secret.yaml
   ```

2. **Apply the ClusterIssuer**:
   ```bash
   kubectl apply -f certification/02-cloudflare-issuer.yaml
   ```

3. **Request a Wildcard Certificate**:
   ```bash
   kubectl apply -f certification/03-cloudflare-certificate.yaml
   ```

4. **Deploy the TLSStore**:
   ```bash
   kubectl apply -f certification/04-tlsstore.yaml
   ```

5. **Verify the Certificate**:
   ```bash
   kubectl get certificates -n cert-manager
   kubectl describe certificate wildcard-bykowski.dev -n cert-manager
   ```

---

## Step 4: Set Up Traefik as the Ingress Controller

### 4.1. **Create the Service Account and Role for Traefik**

1. **Apply the Service Account**:
   ```bash
   kubectl apply -f traefik/05-account.yaml
   ```

2. **Apply the ClusterRole**:
   ```bash
   kubectl apply -f traefik/05-role.yaml
   ```

3. **Apply the RoleBinding**:
   ```bash
   kubectl apply -f traefik/06-role-binding.yaml
   ```

### 4.2. **Deploy Traefik**

1. **Deploy the Traefik Ingress Controller**:
   ```bash
   kubectl apply -f traefik/07-traefic-deployment.yaml
   ```

2. **Create the Traefik Service**:
   ```bash
   kubectl apply -f traefik/09-traefik-service.yaml
   ```

3. **Apply the IngressClass for Traefik**:
   ```bash
   kubectl apply -f traefik/08-ingress-class.yaml
   ```

4. **Verify the Traefik Deployment**:
   ```bash
   kubectl get all -n traefik
   kubectl logs -n traefik -l app=traefik
   ```

---

## Step 5: (Optional) Deploy the Whoami Application

### 5.1. **Deploy Whoami**

1. **Apply the Whoami Deployment**:
   ```bash
   kubectl apply -f whoami/10-whoami-deployment.yaml
   ```

2. **Apply the Whoami Service**:
   ```bash
   kubectl apply -f whoami/12-whoami-service.yaml
   ```

### 5.2. **Configure the Whoami Ingress**

1. **Apply the Whoami Ingress**:
   ```bash
   kubectl apply -f whoami/11-whoami-ingress.yaml
   ```

2. **Verify the Ingress**:
   ```bash
   kubectl describe ingress whoami-ingress -n traefik
   kubectl logs -n traefik -l app=traefik
   ```

---

## Step 6: Test the Setup

1. **Ensure DNS is Set Up**:
   Verify that `*.bykowski.dev` points to your VPS's public IP.

2. **Access Whoami**:
   - Test HTTP:
     ```bash
     curl -I http://whoami.bykowski.dev
     ```
   - Test HTTPS:
     ```bash
     curl -Ik https://whoami.bykowski.dev
     ```

3. **Check Logs**:
   If something doesn’t work, check the logs:
   ```bash
   kubectl logs -n traefik -l app=traefik
   ```

