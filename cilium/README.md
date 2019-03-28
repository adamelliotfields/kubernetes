# Cilium

Cilium is a CNI plugin using BPF instead of IP Tables.

This example uses the ETCD node and certificates deployed when creating a Kubeadm cluster.

### Create `cilium-etcd-secrets`

```bash
USER=adam

sudo kubectl create secret generic cilium-etcd-secrets \
-n kube-system \
--kubeconfig="/home/${USER}/.kube/config" \
--from-file=/etc/kubernetes/pki/etcd/ca.crt \
--from-file=/etc/kubernetes/pki/etcd/server.key \
--from-file=/etc/kubernetes/pki/etcd/server.crt
```

### Download `cilium-external-etcd`

```bash
wget https://raw.githubusercontent.com/cilium/cilium/v1.4/examples/kubernetes/1.13/cilium-external-etcd.yaml
```

### Edit the ConfigMap in the manifest

Run `kubectl get pod etcd-kubernetes -o=jsonpath='{.spec.containers[0].command}'` to get the
`--advertise-client-urls` address from ETCD.

```yaml
...
data:
  etcd-config: |-
    ---
    endpoints:
      - https://<ETCD_ADVERTISE_CLIENT_URL>:2379
    ca-file: '/var/lib/etcd-secrets/ca.crt'
    key-file: '/var/lib/etcd-secrets/server.key'
    cert-file: '/var/lib/etcd-secrets/server.crt'
...
```

### Deploy Cilium

```bash
kubectl apply -f ./cilium-external-etcd.yaml
```