# 03. スコープ・責任分界書

## 文書管理

| 項目 | 内容 |
| --- | --- |
| 文書ID | `SCOPE-RACI-001` |
| 版／状態 | 0.1／Draft（机上設計） |
| 基準日 | 2026-08-17 |
| 作成 | 生成 AI 支援ドラフト（本人レビュー前） |
| レビュー／承認 | PM・各領域責任者／基盤責任者（未実施） |

> [!IMPORTANT]
> 完全に架空の学習用責任分界です。実組織への割当、実機構築、試験、引き継ぎは未実施であり、商用経験の証明ではありません。

## 1. スコープ境界

| 区分 | 対象 | 完了条件 | 備考 |
| --- | --- | --- | --- |
| 第1段階・対象 | OCP基盤の要件、設計、構築、基盤試験、運用引き継ぎ | G1〜G6成果物と実機試験が承認される | 本演習は文書作成まで |
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
- 運用者が閲覧・演習し、受領記録を残している。
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
| 0.1 | 2026-08-17 | 初版 | 生成 AI 支援ドラフト |
