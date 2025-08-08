# 自宅学習用サーバ インフラ構成管理リポジトリ

> **概要:** Linux(Debian), Docker, Python, Tailscaleを活用した自宅学習環境のインフラ構成をコードとドキュメントで管理するリポジトリです。学習と実践の場として、日々改善を重ねています。

## 📖 アーキテクチャ構成図 (Architecture)

このサーバは、Tailscale Funnelによる公開を行っております。サーバ内ではDockerを利用して各アプリケーションをコンテナとして分離・管理し、メンテナンス性と再現性を高めています。

```mermaid
graph TD
    subgraph "インターネット (Internet)"
        J[ユーザ]
    end
    subgraph "プライベートネットワーク"
        A[開発者PC]
        B[Debian Server on OrangePi]
    end
    subgraph "Docker 環境 (Debian Server内)"
        C[server monitor]
        D[Portainer]
        E[App 1 Wiki.js]
        F[App 2 FreshRSS]
        G[App 3 その他アプリ]
    end
    A -- SSH --> B
    B -- Docker Socket --> C
    B -- Docker Socket --> D
    B -- Docker Socket --> E
    B -- Docker Socket --> F
    B -- Docker Socket --> G
    A -- Portainer UI (HTTPS) --> D
    A -- Wiki (HTTPS) --> E
    J -- Tailscale Funnel (HTTPS) --> B
    style B fill:#f9f,stroke:#333,stroke-width:2px
```

Pythonスクリプトとcronを組み合わせ、定期的なタスクを自動化しています。

```mermaid
graph TD
    subgraph "自動化処理 (Automation)"
        H[Cronジョブ]
        I[Pythonスクリプト]
    end
    H -- 実行 --> I
    I -- API Call --> AWS_Boto3
    I -- Local Backup --> Volume
```
