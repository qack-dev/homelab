# 自宅学習用サーバ インフラ構成管理リポジトリ

> **概要:** Linux(Debian), Docker, Python, Tailscaleを活用した自宅学習環境のインフラ構成をコードとドキュメントで管理するリポジトリです。学習と実践の場として、日々改善を重ねています。

## 📖 アーキテクチャ構成図 (Architecture)

このサーバは、Tailscale Funnelによる公開を行っております。サーバ内ではDockerを利用して各アプリケーションをコンテナとして分離・管理し、メンテナンス性と再現性を高めています。

```mermaid
graph TD
    subgraph "インターネット (Internet)"
        J[User]
        K[My Android]
    end
    subgraph "プライベートネットワーク"
        A[My Windows PC]
        B[My Debian Server on OrangePi]
    end
    subgraph "Docker(Debianサーバ内)"
        C[server monitor用コンテナ]
        D[App 1 Wiki.js]
        E[App 2 FreshRSS]
        F[App 3 Redmine]
        G[App 4 Heimdall]
    end
    A -- SSH --> B
    B -- Docker Socket --> C
    B -- Docker Socket --> D
    B -- Docker Socket --> E
    B -- Docker Socket --> F
    B -- Docker Socket --> G
    A -- HTTP --> D
    A -- HTTP --> E
    A -- HTTP --> F
    A -- HTTP --> G
    J -- Tailscale Funnel (HTTPS) --> B
    K -- Tailscale VPN --> B
    style B fill:#f9f,stroke:#333,stroke-width:2px
```

Pythonスクリプトとcronを組み合わせ、ラジオの録音を自動化しています。Tailscale VPNにより、ラジオはAndroidから聴くことができます。

```mermaid
graph TD
    subgraph "自動化処理 (Automation)"
        H[Cronジョブ]
        I[Pythonスクリプト]
    end
    H -- 実行 --> I
    I -- API Call --> RSS
    I -- ファイル作成 --> 共有サーバ
```
