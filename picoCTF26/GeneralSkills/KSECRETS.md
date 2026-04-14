Author: Darkraicg492

100pt.
#### Description
We have a kubernetes cluster setup and flag is in the secrets. You think you can get it?

Additional details will be available after launching your challenge instance.

Hints:
1. Where are secrets usually stored in Kubernetes
2. How are Kubernetes secrets stored internally? Can you decode them?
3. Please ignore TLS

---
**Solution steps**

1. **Prepare the `kubeconfig`**  
   The provided `kubeconfig` originally pointed to `https://127.0.0.1:6443`.  
   Update the `server` field inside the `kubeconfig` file to the actual challenge endpoint:

   ```yaml
   server: https://green-hill.picoctf.net:56422
   ```

   Then tell `kubectl` to use this file:

   ```bash
   export KUBECONFIG=./kubeconfig
   ```

2. **Verify connectivity**  
   Confirm we can talk to the API server (TLS verification is skipped because it's a self-signed certificate):

   ```bash
   kubectl version --insecure-skip-tls-verify
   ```

   Output shows client v1.33.4 and server v1.35.1+k3s1 — minor version skew warning is harmless in a CTF.

3. **Enumerate secrets**  
   List all secrets across all namespaces:

   ```bash
   kubectl get secrets --all-namespaces --insecure-skip-tls-verify
   ```

   Output:

   ```
   NAMESPACE     NAME                       TYPE                               DATA   AGE
   kube-system   chart-values-traefik       helmcharts.helm.cattle.io/values   1      10m
   kube-system   chart-values-traefik-crd   helmcharts.helm.cattle.io/values   0      10m
   kube-system   k3s-serving                kubernetes.io/tls                  2      10m
   picoctf       ctf-secret                 Opaque                             1      9m59s
   ```

   The interesting one is clearly `ctf-secret` in the `picoctf` namespace.

4. **Read the secret**  
   Dump the base64-encoded data:

   ```bash
   kubectl get secret ctf-secret -n picoctf -o jsonpath='{.data}' --insecure-skip-tls-verify
   ```

   Output:

   ```
   {"flag":"cGljb0NURntrczNjcjM3NV80MW43X3M0ZjNfNTJmNjAzYzR9Cg=="}
   ```

5. **Decode the flag**  
   Extract just the value and decode it from base64:

   ```bash
   echo "cGljb0NURntrczNjcjM3NV80MW43X3M0ZjNfNTJmNjAzYzR9Cg==" | base64 -d
   ```

   Result:

   ```
   picoCTF{ks3cr375_41n7_s4f3_52f603c4}
   ```

**Flag**  
`picoCTF{ks3cr375_41n7_s4f3_52f603c4}`

**Key learning points**  
- Kubernetes Secrets of type `Opaque` store data as base64-encoded values (not encrypted unless encryption-at-rest is configured).  
- With admin-level access (via client certificate), you can read any secret in the cluster.  
- Always check namespaces other than `default`,  CTF flags are frequently placed in challenge-specific namespaces like `picoctf`.

Short, sweet, and solved in under 5 minutes once you point `kubectl` at the right API server. Classic `kube` secret leak challenge.