# 03. 基本設計

## 目的

承認された要求を、クラスタ構成、可用性、ネットワーク、セキュリティ、ストレージ、監視、バックアップ、保守の方式へ変換します。基本設計では製品の既定値を転記するのではなく、案件で採用する方式、採用理由、代替案、制約、受入条件を示します。

## 実務での使用場面

- 要件定義後にプラットフォーム全体の方式を合意する
- ネットワーク、セキュリティ、ストレージ、運用チームと境界を合わせる
- 詳細設計へ渡す設計原則と構成を固定する
- 製品・構成選定の判断をレビューし、ADRへ残す
- 構築・試験で確認すべき設計意図を定義する

## 入力

- 承認済みの要求ID、前提、制約、TBD、責任分界
- 既存インフラのDNS、NTP、Load Balancer、Proxy、Firewall、IdP、Storage仕様
- ノード候補のCPU、メモリー、ディスク、NIC、ファームウェア情報
- OpenShift 4.22のインストール方式、ネットワーク、ストレージ、セキュリティ要件
- 運用組織の監視、バックアップ、変更、保守プロセス

## 判断

基本設計では、少なくとも次の判断を相互に整合させます。

| 領域 | 主な判断 | 判断後に残すもの |
| --- | --- | --- |
| 導入方式 | オンプレミス、Agent-based Installer、`platform: none` | 採用理由、責任範囲、前提 |
| トポロジー | 3 control plane＋3 worker、外部サービス冗長化 | 障害モード、容量方針 |
| ネットワーク | Machine／Cluster／Service network、MTU、OVN-Kubernetes | 重複確認、通信方向、CIDR |
| 名前解決・公開 | API／API-int／wildcard DNS、API／Ingress VIP | レコード所有者、LBヘルスチェック |
| 認証・権限 | IdP、RBAC、break-glass、Secret管理 | 認証フロー、最小権限、監査 |
| Storage | 対応CSI、StorageClass、アクセスモード、Registry用PV | ワークロード別要件と失敗時影響 |
| データ保護 | etcdとアプリ／PV／VMの保護領域分離 | RPO/RTOとの対応、復元責任 |
| 可観測性 | クラスタ監視、ユーザーワークロード監視、ログ転送 | SLI/SLO、通知、保持、Runbook |
| 更新 | Update path、保守窓、事前確認、ロールバック境界 | 承認と停止条件 |

採用方式は単独で決まりません。たとえば`platform: none`では、DNS、Load Balancer、ノードライフサイクルなどを利用者側で準備・運用する設計が必要です。ストレージ未確定のままでは、内部Image Registryの永続化やVirtualizationの受入条件を確定できません。

## 成果物例の読み方

### 全体方針

[基本設計書](../projects/enterprise-openshift-platform/docs/05-basic-design.md)の「主要設計判断」を読み、各判断がどの要求に応えるものか確認します。[プラットフォーム構成図](../projects/enterprise-openshift-platform/diagrams/platform-architecture.mmd)は配置関係を示しますが、通信条件、責任、障害時挙動は文章と表で補完します。

### アーキテクチャ

[クラスタ構成・アーキテクチャ設計書](../projects/enterprise-openshift-platform/docs/06-architecture-design.md)では、ノード役割、コンポーネント配置、制御・データフロー、障害モード、容量方針を確認します。「3台だから高可用」ではなく、どの障害まで継続し、何が劣化し、どこを監視するかを読みます。

### 分野別設計

- [ネットワーク・DNS・Load Balancer設計](../projects/enterprise-openshift-platform/docs/07-network-dns-lb-design.md)：CIDR、DNS正逆引き、VIP、ポート、Proxy、NetworkPolicy
- [セキュリティ・認証認可設計](../projects/enterprise-openshift-platform/docs/08-security-identity-design.md)：IdP、RBAC、SCC、Secret、証明書、監査
- [ストレージ・バックアップ設計](../projects/enterprise-openshift-platform/docs/09-storage-backup-design.md)：CSI、StorageClass、Registry、etcd、OADP、VM保護
- [監視・ログ・運用設計](../projects/enterprise-openshift-platform/docs/10-observability-operations-design.md)：アラート、SLI/SLO、ログ、Incident、保守

各分野のTBDが、[前提・制約・未確定事項](../projects/enterprise-openshift-platform/docs/02-assumptions-constraints.md)および[課題・リスク管理台帳](../projects/enterprise-openshift-platform/docs/21-issue-risk-register.md)と一致するか確認します。

## 要求から設計への読み替え例

| 要求の意図 | 基本設計で決めること | 詳細設計・試験への展開 |
| --- | --- | --- |
| APIの単一障害回避 | 外部LB冗長化、API VIP、Backend構成 | HAProxy/keepalived値、LB切替試験 |
| クラスタ内名前解決の安定 | 外部DNSとcluster DNSの境界 | A/PTR/wildcard、正逆引き試験 |
| 最小権限 | IdP groupとClusterRole/RoleBinding方針 | グループ名、権限正負試験 |
| RPO/RTO達成 | 保護対象別バックアップ・復元方式 | スケジュール、復元手順、時間測定 |
| VM移行 | OpenShift VirtualizationとMTVの段階導入 | 互換性Gate、mapping、Wave、rollback |

## 他文書とのつながり

- [要件トレーサビリティ](../projects/enterprise-openshift-platform/docs/04-requirements-traceability.md)に設計の対応先を記録する
- 実装可能な値は[詳細設計書](../projects/enterprise-openshift-platform/docs/13-detailed-design.md)と[パラメータシート](../projects/enterprise-openshift-platform/docs/14-parameter-sheet.md)へ展開する
- 重要な選択理由は[ADR](../projects/enterprise-openshift-platform/docs/23-architecture-decisions.md)へ残す
- 受入条件は[試験仕様書](../projects/enterprise-openshift-platform/docs/16-test-specification.md)へ展開する
- 未決事項と実装リスクは[課題・リスク管理台帳](../projects/enterprise-openshift-platform/docs/21-issue-risk-register.md)で追跡する

## レビューで指摘されやすい点

- 要求IDとの対応や設計根拠がなく、製品機能の一覧になっている
- 構成図と本文、台数表、パラメータシートの値が一致しない
- 正常系だけで、単一障害、依存サービス停止、容量超過を扱わない
- 外部DNS、Load Balancer、Storage、IdPを「別チーム担当」だけで済ませる
- 内部Image Registry、etcd、アプリ、PV、VMの保護範囲を混同する
- 検証環境の結果がないのに、性能や復旧時間を保証する
- `latest`や自動更新を無条件で採用し、変更統制と再現性を考慮しない

## 公式一次資料

- [OpenShift 4.22 Architecture](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/architecture/)
- [Agent-based Installerによるオンプレミスクラスタ導入](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installing_an_on-premise_cluster_with_the_agent-based_installer/)
- [OpenShift 4.22 Networking](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/networking_overview/)
- [OpenShift 4.22 Storage](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/storage/)

補足は[OpenShift基本設計](../reference/technical/openshift-basic-design.md)、[ネットワーク・DNS・LB・Firewall](../reference/technical/network-dns-lb-firewall.md)、[Storage・CSI・ODF](../reference/technical/storage-csi-odf.md)を参照してください。

## 次に読む章

[04. 詳細設計](04-detailed-design.md)へ進みます。
