# 自宅学習用サーバ インフラ構成管理リポジトリ (My Homelab)

> **概要:** Linux(Debian)を搭載したSBC (Orange Pi)上で、Docker, Python, Tailscale等を活用して構築した自宅学習環境です。Webアプリケーションの運用、日々の作業自動化の実践の場として改善を重ねています。

## ✨ 設計思想

この環境は、以下の3つの原則に基づいて設計・構築しています。

1.  **セキュリティの確保:**
    *   ルーターのポートを直接開放するのではなく、**Tailscale**を全面的に採用。`Tailscale Funnel`機能で特定のサービスのみを安全にインターネットへ公開し、セキュリティを確保しています。

2.  **分離と安定的運用:**
    *   **Docker**によりアプリケーションをコンテナ化し管理しています。依存関係の衝突を回避し、コンテナに利用メモリの上限を設定することで、SBCのメモリ圧迫を阻止しています。

3.  **生活に根差した自動化:**
    *   **Pythonとcron**を使い、「実生活で役立つ自動化」を追求しています（後述のラジオ録音システムなど）。

## 📖 アーキテクチャ構成図

### 1. 物理・論理ネットワーク構成

サーバー内部に**仮想ネットワーク基盤**を構築しました。物理的な構成はシンプルですが、その上で論理的に分割されたネットワークが動作しています。

```mermaid
graph LR
    subgraph "インターネット"
        direction TB
        A[一般ユーザ]
        B[My Android]
    end

    subgraph "自宅LAN (192.168.0.0/24)"
        C[開発用PC Windows]
        D[SBC Orange Pi 03 Debian]
    end

    subgraph "Debianサーバ内部"
        direction TB
        subgraph "仮想ネットワーク基盤"
            E["<b>仮想スイッチ (Linux Bridge)</b><br>br0 (管理IP: 192.168.0.104)"]
            F["<b>仮想ルーター (FRRouting)</b>"]
            G["VLAN10 (192.168.10.0/24)"]
            H["VLAN20 (192.168.20.0/24)"]
        end

        I["Dockerサービス"]

        E <-- 物理NIC(end0)を束ねる --> F
        F -- VLAN間ルーティング --> G & H
    end

    C -- SSH --> D
    A -- Tailscale Funnel --> D
    B -- Tailscale VPN --> D
    D -- Docker Socket --> I
```

### 2. アプリケーション（コンテナ）構成

各コンテナは、現状ではDockerが管理するデフォルトのブリッジネットワーク上で動作しています。

```mermaid
graph TD
    subgraph "SBC Orange Pi 03 Debian"
        A[ホストOS]
    end

    subgraph "Dockerコンテナ群"
        direction LR
        D[App Wiki.js]
        F[App Redmine]
        G[App Heimdall]
        subgraph "Dockerネットワーク rss-net"
            E[App FreshRSS]
            H[App RSSHub]
        end
    end

    A -- "Dockerブリッジネットワーク<br>(docker0, br-xxxxなど)" --- D & F & G & E & H
```

### ネットワーク特記事項と今後の展望

*   **現状:**
    *   サーバーホスト内に、Linux Bridge, VLAN, FRRoutingを用いた**仮想ネットワーク基盤**の構築が完了しました。
    *   これにより、`192.168.10.0/24` (VLAN10) と `192.168.20.0/24` (VLAN20) という2つの論理ネットワークセグメントが利用可能になっています。
    *   既存のDockerコンテナ群は、従来通りDockerのブリッジネットワーク上で稼働しています。

*   **今後の展望 (Next Actions):**
    *   Dockerの`macvlan`ネットワークドライバを利用して、各コンテナをその役割に応じたVLAN（VLAN10またはVLAN20）に直接所属させる予定です。
    *   これにより、コンテナレベルでのIPアドレス管理と、`iptables`によるより厳格なセキュリティポリシーの適用を目指します。

## 🛠️ 主なアプリケーションと役割

| コンテナ名 | 役割 | なぜ導入したか |
| :--- | :--- | :--- |
| **Heimdall** | アプリケーションポータル | 各サービスへの入り口。 |
| **Wiki.js** | ナレッジベース | サーバの構築手順や学習メモなどを集約。 |
| **FreshRSS** | RSSリーダー | 情報の効率的な収集。 |
| **RSSHub** | RSSフィード生成ツール | 公式RSSを提供しないサイト(Twitter等)の情報を収集。 |
| **Redmine** | プロジェクト管理 | 個人の学習タスクを管理。 |

## 📖 自動化構成図

```mermaid
graph TD
    subgraph "自動化処理"
        H[cronジョブ]
        I[Pythonスクリプト]
    end
    H -- 実行 --> I
    I -- API Call --> RSS
    I -- ファイル保存 --> 共有サーバ
```

## 🤖 自動化 : ラジオ録音システム

日々の情報収集の一環として、ラジオ番組の自動録音・配信システムを構築しています。
例） [らじる★らじる 聴き逃し番組ダウンローダー](https://github.com/qack-dev/rec_rajiru)

*   **実行トリガー:** `cron`により、毎週指定した時刻にPythonスクriptを実行。
*   **録音処理:** Pythonスクリプトが`Podcast`などのRSSにアクセスし、対象の番組のURLを取得・ダウンロード。
*   **ファイル共有:** ダウンロードされたファイルは、Sambaで共有されたストレージ領域に保存。
*   **聴取:** **Tailscale VPN**経由で、Androidスマートフォンから自宅ネットワークの共有ストレージに安全にアクセスし、いつでもどこでも録音を聴くことが可能。

`#Linux` `#Docker` `#Python` `#Automation` `#Networking` `#Security` `#VLAN` `#FRRouting`