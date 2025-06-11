# Automating SealedSecrets Re-encryption in Kubernetes

**Submitted by:** Mahmoud Salah Mahmoud – Infrastructure Track

## 📌 Problem Statement

When using [SealedSecrets](https://github.com/Mahmoud-Sa1ah/sealed-secrets-reencryption/), Kubernetes secrets are encrypted using a **public key** and decrypted inside the cluster using a **private key**. After each key rotation (e.g., every 30 days), all old secrets must be **re-encrypted** using the **new public key**.

**Manual re-encryption is:**
- ❌ Error-prone  
- 🐌 Inefficient in large clusters  
- ⚠️ Risky if done improperly  

---

## 🎯 Goal

**Automate** the re-encryption process of SealedSecrets to:
- Increase reliability
- Save time
- Reduce operational risk

---

## ⚙️ How It Works

### 1. 🔍 Discover All SealedSecrets

Use `kubectl` to fetch all SealedSecrets in the cluster:

```bash
kubectl get sealedsecrets -A -o json



Namespace:

```
kubectl get sealedsecrets --namespace=prod -o json
```

```
kubectl get sealedsecrets -A -l env=production -o json
```
🔑 Fetch the Latest Public Key
Get the newest encryption key from the controller:

```
kubeseal --fetch-cert > latest-cert.pem
```
📝 Always use the most recent certificate for re-encryption.

🔁 Re-encrypt Securely
Challenge: The private key must never leave the cluster!
Solution: Run a Kubernetes Job inside the cluster to handle decryption and re-encryption.

🧩 Example Job YAML:
```
yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sealedsecrets-reencrypt
spec:
  template:
    spec:
      containers:
      - name: reencrypt
        image: your-custom-image-with-kubeseal
        command: ["kubeseal", "reencrypt", "--input=/data/secrets.json", "--output=/data/updated/"]
        volumeMounts:
        - name: data
          mountPath: /data
      restartPolicy: Never
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: your-pvc
          ```
📥 Apply Re-encrypted Secrets
```
kubectl apply -f reencrypted-secrets/
```
✅ Preserves name and namespace — no impact to existing deployments.

📄 Logging & Reporting
Generate audit logs for transparency and compliance.

🚀 Performance Optimizations
Run in parallel:

```
kubeseal reencrypt --concurrency=10
```
Batch large requests:

bash
Copy
Edit
kubeseal reencrypt --batch-size=100
🔒 Security Enhancements
Test without applying changes:

```
kubeseal reencrypt --dry-run
```
Ensure proper permissions using RBAC.

🔄 Example Workflow
Export all SealedSecrets:

```
kubectl get sealedsecrets -A -o json > secrets.json
```
Re-encrypt using latest key:

```
kubeseal reencrypt --input=secrets.json --output=updated-secrets/
```

Apply changes:
```
kubectl apply -f updated-secrets/
```
