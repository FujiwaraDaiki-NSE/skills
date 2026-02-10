# Host Setup Knowledge (Arc HIMDS)

## Purpose

This repository documents reproducible steps for connecting on-prem Linux to Azure Arc and enabling System-assigned Managed Identity (SMI) access to Azure resources. The goal here is validating that containers (Docker/IoT Edge modules) can access Azure Blob Storage via Arc Managed Identity, including common HIMDS pitfalls.

## High-Level Flow

1. Confirm Arc SMI issuance on Azure side.
2. Grant RBAC to the SMI for target Azure resources.
3. Prepare host-side HIMDS proxy and firewall to allow container access.
4. Run container-side smoke tests against Blob Storage.

## Key Premise

Arc Managed Identity does not use the VM IMDS endpoint (169.254.169.254). Instead, Arc provides a local HIMDS endpoint (typically `127.0.0.1:40342`) that is loopback-only and requires Basic auth using `/var/opt/azcmagent/tokens/*.key`.

Therefore, containers need:

- Host-side port proxy (e.g., 40442 -> 127.0.0.1:40342)
- Read-only bind mount of `/var/opt/azcmagent` into containers

## 1. Verify Arc SMI Exists (Azure Side)

Check the Arc machine identity on Azure ARM.

```bash
az connectedmachine show -g <arc_rg> -n <arc_machine> --query identity -o json
```

## 2. Grant RBAC to the SMI

Assign RBAC role to the SMI `principalId` (object id) against the target scope.

```bash
az role assignment create \
  --assignee-object-id <principalId> \
  --assignee-principal-type ServicePrincipal \
  --role "Storage Blob Data Contributor" \
  --scope "/subscriptions/<subId>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<account>"
```

If validating Arc MI, avoid using connection strings. Rotate any leaked keys and move to RBAC + MI.

## 3. Host HIMDS Proxy and Firewall

Use the repository script to configure proxy + firewall and validate realm/TOKEN_OK.

```bash
sudo bash 05_arc_mi_container_access_setup.sh
```

For debug logging:

```bash
sudo DEBUG=1 bash 05_arc_mi_container_access_setup.sh
```

## 4. Container Smoke Test (FastAPI)

Bring up the test compose (includes `extra_hosts` and `/var/opt/azcmagent:ro`).

```bash
sudo docker compose -f docker-compose.mi-blob.yml up --build
```

Expected API:

- `GET /blobs` list blobs (fails without RBAC)
- `GET /blobs/{blob_name}/head` metadata
- `GET /blobs/{blob_name}` read first N bytes

## 5. Common Pitfalls

- Missing `Metadata:true` header: 400 (Required metadata header not specified)
- Wrong `api-version`: 400 (Supported are ...)
- 401 + `Www-Authenticate: Basic realm=.../tokens/*.key`: reachability OK; bind mount tokens next
- TCP timeout from container: likely firewall (INPUT DROP) blocking docker bridge
- Not mounting `/var/opt/azcmagent`: SDK cannot read realm-indicated `.key`, stays 401

## Files Referenced (Example)

- `05_arc_mi_container_access_setup.sh`: HIMDS proxy + firewall + validation
- `docker-compose.mi-blob.yml`: FastAPI test compose
- `mi_blob_fastapi/`: FastAPI app (Blob list/head/read)
- `env.example`: .env example (no dot)

## UAMI vs SAMI Note

You cannot set a user-assigned managed identity (UAMI) used by Azure Container Apps as the same identity as an Arc machine SAMI. Treat them as separate identities.

Mitigations:

- (A) Assign the same RBAC roles to the Arc device SAMI.
- (B) Use an Entra security group containing both identities, then assign RBAC once at the group level.
