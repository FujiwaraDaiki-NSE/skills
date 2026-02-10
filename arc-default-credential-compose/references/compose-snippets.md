# docker-compose Snippets (Azure Arc DefaultAzureCredential)

## Minimal Service Patch

```yaml
services:
  your-service:
    extra_hosts:
      - "host.docker.internal:host-gateway"
    volumes:
      - /var/opt/azcmagent:/var/opt/azcmagent:ro
    environment:
      - IMDS_ENDPOINT=http://host.docker.internal:40442
      - IDENTITY_ENDPOINT=http://host.docker.internal:40442/metadata/identity/oauth2/token
```

## Example With Blob Settings

```yaml
services:
  influx-blob-exporter:
    extra_hosts:
      - "host.docker.internal:host-gateway"
    volumes:
      - /var/opt/azcmagent:/var/opt/azcmagent:ro
    environment:
      - AZURE_BLOB_CONNECTION_STRING=
      - AZURE_BLOB_ACCOUNT_URL=${AZURE_BLOB_ACCOUNT_URL}
      - AZURE_STORAGE_ACCOUNT_NAME=${AZURE_STORAGE_ACCOUNT_NAME}
      - AZURE_BLOB_ENDPOINT_SUFFIX=${AZURE_BLOB_ENDPOINT_SUFFIX:-core.windows.net}
      - IMDS_ENDPOINT=http://host.docker.internal:40442
      - IDENTITY_ENDPOINT=http://host.docker.internal:40442/metadata/identity/oauth2/token
      - AZURE_CLIENT_ID=${AZURE_CLIENT_ID:-}
      - AZURE_AUTHORITY_HOST=${AZURE_AUTHORITY_HOST:-}
```

## Notes

- Arc HIMDS のデフォルトは `127.0.0.1:40342`。ホスト側で 40442 に中継している前提。
- `host.docker.internal` を使うため、Linux では `extra_hosts` が必要。
- `DefaultAzureCredential` を使う場合は接続文字列を空にするか未設定にする。
