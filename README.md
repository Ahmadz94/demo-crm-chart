# Demo CRM v4 Helm Chart

This project packages the Demo CRM application as a production-ready Helm chart. It includes MongoDB, cert-manager, and the F5 NGINX Ingress Controller as dependencies, with externalized secrets via Google Secret Manager + External Secrets Operator (ESO).

## Prerequisites
- Kubernetes cluster (GKE recommended)
- kubectl and Helm v3
- DNS hostname pointing to the Ingress LoadBalancer IP
- Google Secret Manager secrets created
- External Secrets Operator installed (if using external secrets)

## Architecture Overview
The chart deploys:
- Demo CRM app (Deployment + Service + Ingress)
- MongoDB replicaset (StatefulSet + headless Service + PVCs)
- cert-manager for TLS certificates
- F5 NGINX Ingress Controller for traffic routing
- External Secrets Operator to sync secrets from Google Secret Manager

Overview:
- ![Architecture Diagram](docs/arc.bmp)

## Installation

### Quick Start
1) Update `demo-crm/values.yaml` with your domain and project settings.
2) Pull dependencies:
   ```bash
   helm dependency update demo-crm
   ```
3) Install:
   ```bash
   helm install demo-crm demo-crm
   ```

### Custom Install
```bash
helm install demo-crm demo-crm -f my-values.yaml
```

## CRD Installation
cert-manager and External Secrets Operator require CRDs. This chart installs them via the dependency charts:
- cert-manager: `installCRDs: true`
- external-secrets: `installCRDs: true` (set in the external-secrets release if you install it separately)

cert-manager CRDs:
- `certificaterequests.cert-manager.io`
- `certificates.cert-manager.io`
- `challenges.acme.cert-manager.io`
- `clusterissuers.cert-manager.io`
- `issuers.cert-manager.io`
- `orders.acme.cert-manager.io`

external-secrets CRDs:
- `externalsecrets.external-secrets.io`
- `secretstores.external-secrets.io`
- `clustersecretstores.external-secrets.io`
- `clusterexternalsecrets.external-secrets.io`
- `clusterpushsecrets.external-secrets.io`
- `pushsecrets.external-secrets.io`
- `acraccesstokens.generators.external-secrets.io`
- `cloudsmithaccesstokens.generators.external-secrets.io`
- `ecrauthorizationtokens.generators.external-secrets.io`
- `gcraccesstokens.generators.external-secrets.io`
- `githubaccesstokens.generators.external-secrets.io`
- `grafanas.generators.external-secrets.io`
- `mfas.generators.external-secrets.io`
- `passwords.generators.external-secrets.io`
- `quayaccesstokens.generators.external-secrets.io`
- `sshkeys.generators.external-secrets.io`
- `stssessiontokens.generators.external-secrets.io`
- `uuids.generators.external-secrets.io`
- `vaultdynamicsecrets.generators.external-secrets.io`
- `webhooks.generators.external-secrets.io`
- `clustergenerators.generators.external-secrets.io`
- `generatorstates.generators.external-secrets.io`
- `fakes.generators.external-secrets.io`

## Configuration Parameters
Key values in `demo-crm/values.yaml`:

| Key            |    Description    |        Example         |
| -------------- | ----------------- | -----------------------|
| `replicaCount` | Demo CRM replicas | `1` |
| `image.repository` | App image repo | `europe-north1-docker.pkg.dev/.../demo-crm` |
| `image.tag` | App image tag | `latest` |
| `service.type` | Service type | `ClusterIP` |
| `service.port` | Service port | `80` |
| `service.targetPort` | Container port | `3000` |
| `ingress.enabled` | Enable Ingress | `true` |
| `ingress.className` | Ingress class | `nginx` |
| `ingress.hosts` | Ingress host | `ademocrm.ddns.net` |
| `ingress.tls` | TLS secret + hosts | `demo-crm-tls` |
| `resources` | CPU/memory requests/limits | see values.yaml |
| `mongodb.enabled` | Install MongoDB | `true` |
| `mongodb.auth.existingSecret` | MongoDB creds secret | `mongodb-creds` |
| `externalSecrets.enabled` | Enable ESO resources | `true` |
| `externalSecrets.secretStore.projectId` | GSM project ID | `pure-hue-483512-h6` |
| `externalSecrets.secretStore.serviceAccountRef` | KSA for WI | `eso-gsm` |

## Testing
Use these checks after install:
```bash
kubectl get pods
kubectl get statefulset mongodb
kubectl get ingress
kubectl get externalsecret
curl -I https://ademocrm.ddns.net
```

## Troubleshooting
- **ExternalSecrets not Ready**: Verify Workload Identity is enabled and the KSA exists.
- **Ingress has no external IP**: Wait for LoadBalancer provisioning and recheck the Service.
- **TLS not Ready**: Check cert-manager logs and ClusterIssuer status.
- **MongoDB pods pending**: Verify PVC provisioning and storage class.

## Dependencies
Defined in `demo-crm/Chart.yaml`:
- `bitnami/mongodb`
- `jetstack/cert-manager`
- `nginx-ingress` (F5 NGINX)

## ArgoCD Multi-App Setup
Best practice is to deploy platform components in their own namespaces as separate ArgoCD apps:
- cert-manager (namespace: cert-manager)
- nginx-ingress (namespace: ingress-nginx)
- external-secrets (namespace: external-secrets)
- demo-crm (namespace: default)

This chart disables cert-manager and nginx-ingress by default. If you want a single app deployment, install with:
```bash
helm install demo-crm demo-crm -f values-all-in-one.yaml
```

