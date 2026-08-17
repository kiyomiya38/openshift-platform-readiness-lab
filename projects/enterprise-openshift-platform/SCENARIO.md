# 共通シナリオ定義

## 文書状態

| 項目 | 値 |
| --- | --- |
| 案件名 | Example Enterprise OpenShift 基盤導入 |
| 文書版 | 0.1 Draft |
| 基準日 | 2026-08-17 |
| 性質 | 完全に架空の学習用案件 |
| 実機構築 | 未実施 |
| 試験 | 未実施 |
| 商用経験の証明 | 該当しない |

本フォルダは、要件から設計、構築手順、試験、移行、運用引き継ぎまでを学ぶための机上演習です。記載した値は架空の設計入力または暫定値であり、実環境へそのまま適用しません。秘密情報、実在する顧客情報、実IPアドレスは含めません。

## 案件目的

既存の社内Web/APIワークロードと一部のVMを、共通のOpenShift基盤へ段階的に収容します。第1段階でOpenShift Container Platformを導入し、第2段階でOpenShift VirtualizationとMTVによる代表VMの移行PoCを行います。KongとSysdigは周辺製品として連携点を設計します。

## 技術方式

| 項目 | シナリオ上の決定 |
| --- | --- |
| OpenShift | OpenShift Container Platform 4.22.z。正確なzリリースは実施前に要確認 |
| 導入方式 | Agent-based Installer |
| platform | none |
| 基盤 | x86_64ベアメタル |
| ノード | Control Plane 3台、Worker 3台 |
| OS | OpenShiftノードはRHCOS。RHELを直接導入しない |
| ネットワーク | IPv4、OVN-Kubernetes、静的IP |
| 外部接続 | 組織Proxy経由のconnected構成 |
| DNS/LB/NTP | クラスタ外の冗長サービスを利用 |
| ストレージ | OpenShift対応の外部CSIストレージ。製品とStorageClassは要確認 |
| バックアップ | etcd、アプリケーション、PV、外部DBを分けて設計 |
| Virtualization | OpenShift VirtualizationをOLMで導入する計画。導入は未実施 |
| VM移行 | VMwareを移行元とするMTV PoC。製品版と互換性は要確認 |
| Kong | API Gateway連携の設計のみ。製品版・方式・ライセンスは要確認 |
| Sysdig | 監視・コンテナセキュリティ連携の設計のみ。製品版・方式・ライセンスは要確認 |
| ROSA/ARO | 方式比較の対象。今回の構築対象外 |
| OpenShift AI | 対象外 |
| Disconnected | 対象外。将来課題 |

platform noneでは、DNSとAPI/Application Ingressの外部Load Balancerを事前に用意します。APIはLayer 4でTCP/6443、Machine Config ServerはTCP/22623、Application IngressはTCP/80と443を扱います。Agent-based Installerのdiscovery/bootstrap中は、全ホストからrendezvous hostへのTCP/8090も確認対象です。正確な要件は対象リリースの公式資料で再確認します。

## 架空の業務要件

| ID | 要件 |
| --- | --- |
| BR-001 | 社内Web/APIと代表VMを共通基盤へ段階的に収容する |
| BR-002 | 利用者100名、開発・運用関係者20名を想定する |
| BR-003 | サービスは24時間365日を想定し、計画保守は事前承認する |
| BR-004 | 可用性目標は月間99.9%とする。保証値ではなく架空要件 |
| BR-005 | アプリケーションデータのRPOは1時間、RTOは4時間を目標とする |
| BR-006 | プラットフォーム、アプリ、ネットワーク、ストレージ、セキュリティの責任を分ける |
| BR-007 | 管理操作は最小権限、組織IdP、監査可能な個人IDを前提とする |
| BR-008 | 変更は申請、レビュー、承認、実施、結果確認、切り戻し判定を記録する |
| BR-009 | VM移行は本番承認ではなく、3台の代表VMで技術課題を抽出するPoCとする |
| BR-010 | 実施していない確認を成功扱いにせず、未実施と要確認を明示する |

## ネットワーク値

RFC 5737の文書用IPv4アドレスを使用します。これらは実環境で使用する値ではありません。

| 用途 | 架空値 |
| --- | --- |
| Cluster name | ocp-prd |
| Base domain | example.com |
| Cluster domain | ocp-prd.example.com |
| Machine network | 192.0.2.0/24 |
| Default gateway | 192.0.2.1 |
| DNS | 192.0.2.2、192.0.2.3 |
| NTP | 192.0.2.4、192.0.2.5 |
| Proxy | proxy.example.com:3128 |
| API VIP | 192.0.2.10 |
| Ingress VIP | 192.0.2.11 |
| LB nodes | 192.0.2.12、192.0.2.13 |
| Bastion | 192.0.2.14 |
| Backup object storage endpoint | 192.0.2.15 |
| Control Plane | 192.0.2.21〜23 |
| Worker | 192.0.2.31〜33 |
| Pod network | 10.128.0.0/14、hostPrefix /23 |
| Service network | 172.30.0.0/16 |
| API FQDN | api.ocp-prd.example.com |
| Internal API FQDN | api-int.ocp-prd.example.com |
| Application wildcard | *.apps.ocp-prd.example.com |

## ノード

| Host | Role | IPv4 | 架空MAC | 暫定サイジング |
| --- | --- | --- | --- | --- |
| cp01.ocp-prd.example.com | master | 192.0.2.21 | 02:00:00:00:00:21 | 8 vCPU、32 GiB、250 GiB boot |
| cp02.ocp-prd.example.com | master | 192.0.2.22 | 02:00:00:00:00:22 | 8 vCPU、32 GiB、250 GiB boot |
| cp03.ocp-prd.example.com | master | 192.0.2.23 | 02:00:00:00:00:23 | 8 vCPU、32 GiB、250 GiB boot |
| wk01.ocp-prd.example.com | worker | 192.0.2.31 | 02:00:00:00:00:31 | 32 vCPU、256 GiB、250 GiB boot |
| wk02.ocp-prd.example.com | worker | 192.0.2.32 | 02:00:00:00:00:32 | 32 vCPU、256 GiB、250 GiB boot |
| wk03.ocp-prd.example.com | worker | 192.0.2.33 | 02:00:00:00:00:33 | 32 vCPU、256 GiB、250 GiB boot |

サイジングは学習用の入力値です。Red Hatの最小要件、ハードウェア互換性、VM収容計画、障害時退避余力、ストレージ性能を確認して確定します。

## 責任分界

| 領域 | 主担当 |
| --- | --- |
| OpenShift設計・構築 | Platform team |
| DNS、NTP、LB、Firewall | Network/Infrastructure team |
| Bare metal、BMC、Firmware | Hardware team |
| CSI、容量、Snapshot | Storage team |
| IdP、証明書、脆弱性、監査 | Security team |
| アプリ、DB整合性、業務受入 | Application team |
| バックアップ保管、復旧試験調整 | Platform + Application + Storage |
| Kong、Sysdig | Product owner + Platform + Security |

## 未確定事項

- 正確なOpenShift 4.22.z、更新チャネル、サポート期間
- ハードウェア互換性、Firmware、BIOS、CPU仮想化支援
- CSI製品、StorageClass、Snapshot、RWX/RWO、Live Migration適合性
- 内部Image Registryの永続ストレージ、`Removed`から`Managed`へ移す時期と受入条件
- 組織IdP、MFA、グループ名、break-glass手順
- 証明書発行元、更新責任、Proxy CA
- Load Balancerの製品、冗長化方式、health check実装
- OADP、MTV、Kong、Sysdigの対応版・ライセンス・サポート条件
- VMware側の版、権限、ネットワーク、データストア、停止可能時間

## 公式資料

- [OpenShift 4.22 Agent-based Installer](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installing_an_on-premise_cluster_with_the_agent-based_installer/)
- [OpenShift 4.22 Virtualization](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/virtualization/)
- [OpenShift 4.22 Backup and restore](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/backup_and_restore/)
