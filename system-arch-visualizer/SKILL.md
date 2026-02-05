---
name: system-arch-visualizer
description: ソフトウェア構成図や論理アーキテクチャ図を作成・更新するためのスキル。Mermaid形式（flowchart）を用いて、機器とサービスの関係やデータの流れを可視化します。ユーザーからシステム構成の可視化や、インフラ構成の説明を求められた際に使用します。
---

# System Architecture Visualizer

このスキルは、プロジェクトのシステム構成を「どの機器で」「どのサービスが動き」「どこにデータを送るか」という観点で可視化するための手順とパターンを提供します。

## ワークフロー

1.  **現状の整理**:
    *   登場する「機器」をリストアップする。
    *   各機器で動作する「サービス（プロセス）」を特定する。
    *   サービス間の「通信方向」と「プロトコル」を確認する。
2.  **図の作成**:
    *   下記のテンプレートを参考にMermaidコードを記述する。
    *   ラベルに改行 `<br>` を入れて、読みやすさを調整する。
3.  **ドキュメントへの配置**:
    *   `README.md` 等に Mermaid ブロックとして挿入する。
    *   図の下に各コンポーネントの役割や、設計のポイントを解説として加える。

## Mermaid デザインパターン

### 1. 標準的な構成図テンプレート

```mermaid
flowchart TB
  subgraph Cloud["クラウド / 外部"]
    Dest["外部ストレージ/API"]
  end

  subgraph Server["メインサーバ / NAS"]
    DB[("データベース")]
  end

  subgraph Edge["エッジデバイス / PC"]
    direction TB
    Svc1["サービスA (収集)"]
    Svc2["サービスB (転送)"]
  end

  subgraph Source["データソース / PLC"]
    Data["現場データ"]
  end

  %% 通信関係
  Data -->|プロトコル| Svc1
  Svc1 --> Svc2
  Svc2 -->|書き込み| DB
  DB -->|エクスポート| Dest
```

### 2. コンポーネント命名規則

*   **機器名**: 大文字または判別しやすい固有名詞（例: `Jetson`, `PLC`, `NAS`）
*   **サービス名**: 機能がわかる名称（例: `OPC UA Server`, `InfluxDB`, `Data Exporter`）
*   **通信ラベル**: 通信方式や内容を明記（例: `|OPC UA|`, `|HTTPS/JSON|`, `|1時間毎CSV出力|`）

## 追加リソース

*   より詳細な設計パターンについては [references/design-patterns.md](references/design-patterns.md) を参照してください。
