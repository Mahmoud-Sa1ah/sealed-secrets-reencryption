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
