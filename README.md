Automating SealedSecrets Re-encryption in Kubernetes Submitted by: Mahmoud salah Mahmoud – Infrastructure Track

The Problem When using SealedSecrets, Kubernetes secrets are encrypted with a public key and decrypted inside the cluster with a private key. After key rotation (every 30 days), old secrets must be re-encrypted with the new public key. Manual re-encryption is: • Error-prone • Inefficient in large clusters • Risky if done improperly Goal: Automate the re-encryption process.

How It Works

Discover All SealedSecrets Use kubectl to fetch all SealedSecrets cluster-wide:
kubectl get sealedsecrets -A -o json
You can filter by:

• Namespace: --namespace=prod
• Labels: -l env=production
Fetch the Latest Public Key Get the newest encryption key from the controller:
kubeseal --fetch-cert > latest-cert.pem
Always use the most recent certificate for re-encryption.

Re-encrypt Securely Challenge: The private key must never leave the cluster! Solution: Run a Kubernetes Job inside the cluster to handle decryption and re-encryption.
Example Job YAML:

Apply Re-encrypted Secrets
kubectl apply -f reencrypted-secrets/
Preserves name and namespace — no impact to existing deployments.

Logging & Reporting Generate audit logs:

Performance Optimizations • Run in parallel:

kubeseal reencrypt --concurrency=10
• Batch large requests:

kubeseal reencrypt --batch-size=100
Security Enhancements • Test without applying changes:

kubeseal reencrypt --dry-run
• Ensure proper permissions (RBAC):

Example Workflow

Export all SealedSecrets
kubectl get sealedsecrets -A -o json > secrets.json
Re-encrypt using latest key
kubeseal reencrypt --input=secrets.json --output=updated-secrets/
Apply changes
kubectl apply -f updated-secrets/
