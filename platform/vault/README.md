# Vault in the local development cluster

Vault is installed from the official HashiCorp Helm chart with
`values-dev.yaml`. This configuration is intentionally a single-node,
HTTP-only learning environment. It is not a production Vault architecture.

## Install or reconcile

```powershell
helm repo add hashicorp https://helm.releases.hashicorp.com --force-update
helm repo update
helm upgrade --install vault hashicorp/vault `
  --namespace intelliops-dev `
  --version 0.34.0 `
  --values platform/vault/values-dev.yaml
```

Do not use `--wait` before initialization. A sealed Vault intentionally fails
its readiness probe.

## Operator lifecycle

Initialization is performed only once for a new data volume. Never save its
unseal keys or initial root token in this repository.

After every Vault pod restart, check its state:

```powershell
kubectl exec vault-0 -n intelliops-dev -- vault status
```

If it is sealed, unseal it interactively so the key is not written to shell
history:

```powershell
kubectl exec -it vault-0 -n intelliops-dev -- vault operator unseal
```

## Application authentication

Config Server authenticates through the `config-server-role` AppRole and can
read only `secret/data/*`. Its RoleID and SecretID are stored in the runtime
Kubernetes Secret named `config-server-vault-approle`; that Secret is never
rendered from Git.

For production, replace standalone file storage and manual unsealing with an
HA design, TLS, automated KMS/HSM unsealing, audit storage, backups, and
Kubernetes-native workload authentication.
