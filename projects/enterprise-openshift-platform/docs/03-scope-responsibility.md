# 03. スコープ・責任分界書

## 文書管理

| 項目 | 内容 |
| --- | --- |
| 文書ID | `SCOPE-RACI-001` |
| 版／状態 | 0.1／Draft（机上設計） |
| 基準日 | 2026-08-17 |
| 作成 | 文書作成チーム（サンプル） |
| レビュー／承認 | PM・各領域責任者／基盤責任者（未実施） |

> [!IMPORTANT]
> 一般公開用の架空プロジェクトにおける責任分界サンプルです。実在組織への割当、実機構築、試験、引き継ぎは未実施で、役割名はシナリオ上のものです。

## 1. スコープ境界

| 区分 | 対象 | 完了条件 | 備考 |
| --- | --- | --- | --- |
| 第1段階・対象 | OCP基盤の要件、設計、構築、基盤試験、運用引き継ぎ | G1〜G6成果物と実機試験が承認される | 本サンプルは文書・設定例まで |
| 第2段階・対象 | OpenShift Virtualization導入評価、MTV代表VM 3台PoC | 技術課題、停止時間、性能、復旧性を報告 | 本番移行承認ではない |
| 連携設計 | Kong、Sysdig | インターフェース、権限、通信、責任を定義 | 製品内部設計は対象外 |
| 将来比較 | ROSA、ARO | オンプレミス採用判断の比較材料 | 構築対象外 |

### 1.1 明示的な対象外

- 業務アプリのソースコード改修と業務機能試験
- 外部DB製品内部のHA／バックアップ詳細設計
- DNS、LB、NTP、Firewall、CSI、IdP製品そのものの構築（OpenShift接続要件は対象）
- 実調達、契約、ライセンス、ベンダーサポート契約
- 本番VM全台移行、ROSA／ARO、OpenShift AI、Disconnected

## 2. 成果物責任

<a id="role-definitions"></a>

### 2.1 役割の定義

ここでの役割名は、特定の職種や人数を表すものではなく、案件上の責任の単位です。1人が複数の役割を兼務する場合も、1つの役割を複数チームで分担する場合もあります。実案件では、組織図ではなく「誰が実行し、誰が最終承認するか」を氏名または正式チーム名へ割り当てます。

| 役割 | 主な責任 | 代表的な成果物・判断 | 責任の境界 |
| --- | --- | --- | --- |
| PM | スコープ、日程、予算、品質、課題、変更、会議体、関係者間の調整を管理する | プロジェクト憲章、工程計画、課題・変更管理、各承認Gateの進行 | OpenShiftの設定値を単独で技術承認する役割ではない |
| 基盤責任者 | 基盤全体の最終説明責任を持ち、重要方針、残存リスク、工程開始・移行可否を承認する | 要件・基本設計の承認、Go/No-Go、運用受入、リスク受容 | 実装作業の主担当とは限らない。Platformの設計を承認・受容する立場 |
| Platform | OpenShiftクラスタのアーキテクチャ、設計、構築、設定、更新、各外部サービスとの技術統合を担う | 基本・詳細設計、パラメータ、構築手順、クラスタ試験、技術判断 | 企業DNS、物理機器、ストレージ装置などの正本管理者ではない |
| Network/Infra | クラスタ外のDNS、NTP、Load Balancer、Firewall、Proxy、ルーティングを担う | DNSレコード、Firewall申請、LB設定、Proxy/noProxy、外部疎通試験 | OVN-KubernetesやIngressControllerなどクラスタ内設定はPlatformとの共同境界 |
| Hardware | Bare Metalサーバー、BMC、Firmware、NIC、ディスク、電源・ラックなど物理基盤を担う | 機器台帳、HCL確認、Firmware baseline、BMC/ISO boot、物理障害対応 | RHCOSやOpenShiftの論理設定はPlatformが管理する |
| Storage | 外部ストレージ、CSI前提、容量、性能、Snapshot、複製、保管を担う | ストレージ設計、CSI互換性、StorageClass要件、容量・性能試験、Snapshot・復元 | OpenShift側のStorageClass/PVC定義はPlatformと共同で設計する |
| Security | IdP、認証・認可方針、証明書、Secret管理、監査、脆弱性、データ取扱いを担う | Security設計、RBAC/SCC方針、証明書要件、監査要件、例外承認 | OpenShiftリソースの実装は、Security方針に従いPlatformが行う場合がある |
| Application | アプリ、DB、データ整合性、停止・再開順序、業務試験、業務受入を担う | アプリ要件、配備条件、DB整合手順、RPO/RTO受入、移行後の業務判定 | クラスタ基盤自体の正常性はPlatformが判定する |
| Operations | 監視、ログ、一次障害対応、定常作業、変更受付、エスカレーション、運用引き継ぎを担う | 運用設計、監視・通知、Runbook、Incident記録、定常作業記録、運用受入 | 一次検知者が必ずしも原因領域の修正責任者とは限らない |
| Product owner | 本シナリオではKong・Sysdigの要件、製品選定、契約・ライセンス、費用、SLA、導入受入を担う | 製品要件、選定Gate、ベンダー窓口、契約条件、接続・受入判定 | 一般的な「業務プロダクト全体のOwner」ではなく、本サンプルにおける周辺製品Ownerを指す |

### 2.2 混同しやすい役割の境界

| 役割の組み合わせ | 分け方 |
| --- | --- |
| PM / 基盤責任者 | PMはプロジェクトの進め方を管理し、基盤責任者は基盤として受け入れられるかを最終判断する |
| 基盤責任者 / Platform | 基盤責任者は承認・リスク受容を担い、Platformは設計・実装・技術説明を担う |
| Platform / Network/Infra | Platformは主にクラスタ内とOCP設定、Network/Infraは主にクラスタ外の共通通信基盤を担う。API/Ingress、Proxy、DNSは共同境界 |
| Platform / Operations | Platformは設計・構築・高度な原因調査、Operationsは定常監視・一次対応・手順化された作業を主に担う |
| Application / Product owner | Applicationは業務アプリとデータの成否、Product ownerは本サンプルではKong・Sysdigの採否と製品条件を担う |

### 2.3 RACI対応表

凡例: `A` 最終説明責任、`R` 実行責任、`C` 協議、`I` 共有。1行につきAは1役割とします。

| 成果物／活動 | PM | 基盤責任者 | Platform | Network/Infra | Hardware | Storage | Security | Application | Operations | Product owner |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 要件定義・変更管理 | R | A | C | C | C | C | C | C | I | C |
| OCP基本・詳細設計 | I | I | A/R | C | C | C | C | C | C | I |
| DNS/NTP/LB/Firewall/Proxy | I | I | C | A/R | I | I | C | I | C | I |
| Bare metal/BMC/Firmware | I | I | C | C | A/R | I | C | I | C | I |
| CSI/StorageClass/Snapshot | I | I | C | I | I | A/R | C | C | C | I |
| IdP/RBAC/SCC/証明書/監査 | I | I | R | C | I | I | A | C | C | I |
| アプリ/DB整合性/業務受入 | I | I | C | I | I | C | C | A/R | C | I |
| etcdバックアップ | I | I | A/R | I | I | C | C | I | C | I |
| アプリ/PV/DB復元試験 | I | I | R | I | I | R | C | A | C | I |
| 監視・ログ・一次運用 | I | I | R | C | I | C | C | C | A | C |
| Kong/Sysdig連携 | I | I | R | C | I | I | C | C | C | A |
| Virtualization/MTV PoC | I | I | A/R | C | C | C | C | R | C | I |
| Go/No-Go判定 | R | A | C | C | C | C | C | C | C | I |

## 3. サービス境界と引き渡し条件

| 境界ID | 提供側 | 受領側 | 提供物 | 受入確認 | エスカレーション |
| --- | --- | --- | --- | --- | --- |
| IF-001 | Hardware | Platform | 6台、BMC、Firmware、NIC、ディスク | HCL・資源・時刻・リンク確認 | Hardware責任者 |
| IF-002 | Network | Platform | DNS A/PTR、NTP、Proxy、Firewall | 正逆引き、NTP、HTTP(S)疎通 | Network責任者 |
| IF-003 | Network | Platform | API/Ingress LB、VIP | backend health、フェイルオーバー、L4疎通 | Network責任者 |
| IF-004 | Storage | Platform | CSI、StorageClass、Snapshot | 動的PV、RWO/RWX、Snapshot、性能 | Storage責任者 |
| IF-005 | Security | Platform | IdP属性、グループ、CA、証明書 | login、RBAC、失効、監査 | Security責任者 |
| IF-006 | Platform | Application | Project、Quota、Route、監視入口 | 配備、疎通、権限、アラート | Platform責任者 |
| IF-007 | Application | Platform/Storage | DB整合手順、停止順、復元合格条件 | 復元後の業務整合性判定 | Application責任者 |
| IF-008 | Product owner | Platform/Security | Kong/Sysdig仕様・ライセンス | 対応版、権限、通信、データ所在 | Product owner |

## 4. 障害対応の責任分界

| 事象 | 一次対応 | 二次対応 | 判断責任 | 必須記録 |
| --- | --- | --- | --- | --- |
| API/Console到達不可 | Operations | Platform + Network | Platform責任者 | 発生時刻、外形、DNS/LB/API状態、変更有無 |
| Node NotReady | Operations | Platform + Hardware/Network | Platform責任者 | node condition、event、BMC、リンク、直前変更 |
| PVC attach/mount失敗 | Operations | Platform + Storage | Storage責任者 | PVC/PV/CSI event、対象ノード、ストレージ側event |
| アプリ異常 | Application | Platform | Application責任者 | Pod/Route/DB状態、業務影響、再現条件 |
| 認証不可・権限過多 | Operations | Security + Platform | Security責任者 | 個人ID、IdP/OAuth、binding変更、監査ログ |
| バックアップ失敗 | Operations | Platform + Storage + Application | 対象データOwner | backup ID、対象、失敗段階、最終成功時刻 |
| Sysdig/Kong異常 | Operations | Product owner + Platform | Product owner | 製品alert、OCP影響、サポートcase |

## 5. 変更・承認分界

| 変更分類 | 例 | 技術レビュー | 承認 | 実施 | 結果確認 |
| --- | --- | --- | --- | --- | --- |
| Cluster-wide | 更新、Proxy、Ingress、OAuth | Platform + 影響領域 | 変更責任者 | Platform | Platform + Operations |
| Network | DNS、VIP、LB pool、Firewall | Network + Platform | Network責任者 | Network | 双方 |
| Storage | StorageClass、CSI、Snapshot | Storage + Platform + Application | Storage責任者 | Storage/Platform | Application含む |
| Security | IdP、RBAC、SCC、証明書 | Security + Platform | Security責任者 | 承認済み実施者 | Security |
| Workload | Namespace内設定 | Application + Platform | Application責任者 | Application | Application |

## 6. 引き継ぎ時のDefinition of Done

- 構成・パラメータ・資格情報参照先が最新である。
- 監視、ログ、バックアップ、更新、障害、証明書、容量のRunbookがある。
- 連絡先、受付時間、重要度、SLA/OLA（TBD）が合意されている。
- 既知課題、TBD、リスク受容、次回期限が移管されている。
- 運用者が閲覧・訓練し、受領記録を残している。
- Secret実値を文書・Gitへ含めていない。

## 7. 未確定事項

| ID | 内容 | Owner | 期限 | 影響 |
| --- | --- | --- | --- | --- |
| TBD-RACI-001 | 24x365一次受付の実チームとOLA | Operations | G2前 | 障害初動遅延 |
| TBD-RACI-002 | 外部DB teamを独立役割にするか | Application | G1前 | RPO責任不明確 |
| TBD-RACI-003 | ベンダーサポート起票権限と契約窓口 | PM | G4前 | 障害解決遅延 |

## 8. 承認・変更履歴

| 役割 | 判定 | 日付 | コメント |
| --- | --- | --- | --- |
| PM | 未承認 | - | 実組織なし |
| 各領域責任者 | 未レビュー | - | 架空案件 |

| 版 | 日付 | 内容 | 作成者 |
| --- | --- | --- | --- |
| 0.1 | 2026-08-17 | 初版 | 文書作成チーム（サンプル） |
