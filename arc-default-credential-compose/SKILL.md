---
name: arc-default-credential-compose
description: "Azure Arc の Managed Identity (DefaultAzureCredential) をコンテナから使うための docker-compose.yaml 設定を作成・更新する。Use when: Edge/Arc 接続済みホスト上のコンテナで Arc HIMDS を経由した認証を通したい、docker-compose で `extra_hosts`, `/var/opt/azcmagent` マウント, `IMDS_ENDPOINT` / `IDENTITY_ENDPOINT` を整えたい場合。"
---

# Arc Default Credential Compose

## Overview

Azure Arc の Managed Identity を Docker コンテナ内の `DefaultAzureCredential` から使えるようにするため、docker-compose.yaml 側の設定を整理・追加する。ホスト側の設定（HIMDS 中継や firewall 等）は扱わない。

## Quick Checklist

次を満たす compose 設定を作る。

- `extra_hosts` に `host.docker.internal:host-gateway` を追加する
- `volumes` で `/var/opt/azcmagent:/var/opt/azcmagent:ro` をマウントする
- `environment` で `IMDS_ENDPOINT` と `IDENTITY_ENDPOINT` を `http://host.docker.internal:40442` に向ける
- `DefaultAzureCredential` を使うため、必要に応じて接続文字列を空にする
- ストレージ接続先のいずれかを設定する（`AZURE_BLOB_ACCOUNT_URL` または `AZURE_STORAGE_ACCOUNT_NAME`）

## Workflow

1. 対象サービスの docker-compose.yaml を開く。
2. サービスに `extra_hosts` と `/var/opt/azcmagent` マウントを追加する。
3. `IMDS_ENDPOINT` と `IDENTITY_ENDPOINT` を設定する。
4. `DefaultAzureCredential` を使う前提で、接続文字列やアカウント情報を整理する。
5. 変更点を確認し、同一構成が必要な他サービスにも反映する。

## Implementation Notes

以下を必ず確認する。

- このスキルは **コンテナ側のみ** を扱う。ただし、ユーザーに聞かれた場合はホスト側（HIMDS 中継、iptables/ufw、Arc 接続確認など）の手順を説明できるようにしておく。実作業や変更指示はユーザー判断に委ねる。
- 既存の compose で `IMDS_ENDPOINT` / `IDENTITY_ENDPOINT` を別値にしている場合は、Arc の中継先（通常 `http://host.docker.internal:40442`）へ揃える。
- `host.docker.internal` が効かない Docker では `extra_hosts` が必須。
- `DefaultAzureCredential` で接続文字列を優先させたくない場合は `AZURE_BLOB_CONNECTION_STRING` を空文字で上書きする。

## References

Compose の具体例は `arc-default-credential-compose/references/compose-snippets.md` を参照する。ホスト側の手順やつまずきポイントを聞かれた場合は `arc-default-credential-compose/references/host-setup.md` を参照する。
