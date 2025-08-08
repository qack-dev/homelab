# 自宅学習用サーバ インフラ構成管理リポジトリ

> **概要:** Linux(Debian), Docker, Python, Tailscaleを活用した自宅学習環境のインフラ構成をコードとドキュメントで管理するリポジトリです。学習と実践の場として、日々改善を重ねています。

## 📖 アーキテクチャ構成図 (Architecture)

このサーバは、セキュリティを確保しつつ外部から安全にアクセスするため、TailscaleによるVPNを基盤としています。サーバ内ではDockerを利用して各アプリケーションをコンテナとして分離・管理し、メンテナンス性と再現性を高めています。

```mermaid
graph TD
    subgraph "インターネット (Internet)"
        A[開発者PC]
    end
    subgraph "Tailscale VPN (仮想プライベートネットワーク)"
        B[Debian Server on OrangePi/VM]
    end
    subgraph "Docker 環境 (Debian Server内)"
        C[Nginx Proxy Manager]
        D[Portainer]
        E[App 1 Wiki.js]
        F[App 2 Uptime Kuma]
        G[App 3 その他アプリ]
    end
    subgraph "自動化処理 (Automation)"
        H[Cronジョブ]
        I[Pythonスクリプト]
    end
    A -- HTTPS/SSH --> B
    B -- Docker Socket --> C
    B -- Docker Socket --> D
    B -- Docker Socket --> E
    B -- Docker Socket --> F
    B -- Docker Socket --> G
    A -- Portainer UI (HTTPS) --> D
    A -- Wiki (HTTPS) --> E
    H -- 実行 --> I
    I -- API Call --> AWS_Boto3
    I -- Local Backup --> Volume
    style B fill:#f9f,stroke:#333,stroke-width:2px
```
