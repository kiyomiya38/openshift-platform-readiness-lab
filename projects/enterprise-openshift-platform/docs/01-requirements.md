# 01. 要件定義書

## 文書管理

| 項目 | 内容 |
| --- | --- |
| 文書ID | `REQ-DEF-001` |
| 対応案件 | Example Enterprise OpenShift 基盤導入 |
| 版／状態 | 0.1／Draft（机上設計） |
| 基準日 | 2026-08-17 |
| 作成 | 生成 AI 支援ドラフト（本人レビュー前） |
| レビュー／承認 | 各領域責任者／基盤責任者（いずれも未実施） |

> [!IMPORTANT]
> 本書は架空条件を用いた学習用の要件定義です。ヒアリング、実機確認、性能測定、契約確認は未実施であり、商用経験の証明ではありません。記載の「必須」は本シナリオ内の優先度であり、実顧客の合意ではありません。

## 1. 記述規則

- 要件IDは変更しても再利用しません。
- 優先度は `Must`（受入に必須）、`Should`（原則必要）、`Could`（条件付き）とします。
- 受入基準が実測を要する項目は、現時点では「未実施」です。
- 要件の設計・試験対応は [04-requirements-traceability.md](04-requirements-traceability.md) で管理します。

## 2. 業務要件

| ID | 要件 | 受入基準 | Owner | 優先度 | 状態 |
| --- | --- | --- | --- | --- | --- |
| BR-001 | 社内Web/APIと代表VMを共通基盤へ段階的に収容する | 第1段階のコンテナ基盤と第2段階のVM PoCの境界・完了条件が文書化される | 基盤責任者 | Must | 未確認 |
| BR-002 | 利用者100名、開発・運用関係者20名を想定する | ID・権限・運用窓口が想定人数を扱える設計である | 業務責任者 | Must | 未確認 |
| BR-003 | 24時間365日のサービスを想定し、計画保守を事前承認する | 保守申請、通知、実施、復旧、切り戻しの流れが定義される | 運用責任者 | Must | 未確認 |
| BR-004 | 月間可用性目標を99.9%とする | SLI、測定範囲、除外条件、月次報告方法が定義される | 業務責任者 | Must | 未確認 |
| BR-005 | アプリケーションデータのRPOを1時間、RTOを4時間とする | 対象データ、バックアップ方式、復元試験、時間配分が定義される | Application | Must | 未確認 |
| BR-006 | 領域別に責任を分ける | RACIと障害・変更時の引き継ぎ点が合意される | PM | Must | 未確認 |
| BR-007 | 最小権限、組織IdP、監査可能な個人IDを使う | 共有管理IDを常用せず、グループRBACと監査ログを設計する | Security | Must | 未確認 |
| BR-008 | 変更ライフサイクルを記録する | 申請、レビュー、承認、実施、結果、切り戻し判定を変更IDで追跡できる | PM | Must | 未確認 |
| BR-009 | 代表VM 3台のMTV PoCで技術課題を抽出する | 本番承認と分離したPoC計画、評価軸、終了条件がある | Platform/Application | Should | 未確認 |
| BR-010 | 未実施確認を成功扱いにしない | 結果欄が「未実施」「要確認」「実測済み」を区別する | 品質担当 | Must | 適用中 |

## 3. 機能要件

| ID | 要件 | 受入基準 | Owner | 優先度 | 状態 |
| --- | --- | --- | --- | --- | --- |
| REQ-PLT-001 | OCP 4.22.zをAgent-based Installer、`platform: none`で導入できること | 対象z版を固定し、公式要件と整合した設定・手順・前提確認が揃う | Platform | Must | z版TBD |
| REQ-PLT-002 | Control Plane 3台、Worker 3台で構成すること | 6台の役割、FQDN、IP、暫定資源がパラメータ化される | Platform/Hardware | Must | 机上定義済み |
| REQ-PLT-003 | OpenShiftノードはRHCOSを使用すること | RHELをノードOSとして導入しない手順になっている | Platform | Must | 設計予定 |
| REQ-NET-001 | API、内部API、Application wildcardを名前解決できること | 必須A/CNAME/PTRの事前確認手順がある | Network | Must | 未実施 |
| REQ-NET-002 | API、Machine Config Server、Ingressを冗長LBで公開すること | L4、対象ポート、backend、health check、非session persistenceが定義される | Network | Must | 製品TBD |
| REQ-NET-003 | OVN-Kubernetes、指定CIDR、静的IPを利用すること | Machine/Pod/Service CIDRの非重複確認と設定値がある | Network/Platform | Must | 未実施 |
| REQ-NET-004 | 組織Proxy経由で必要な外部宛先へ接続すること | proxy/noProxy、CA、許可先、疎通試験を定義する | Network/Security | Must | CA・許可先TBD |
| REQ-IDM-001 | 組織IdPとOpenShift OAuthを連携すること | 対応方式、属性、グループ同期、MFA責任、障害時挙動が定義される | Security | Must | IdP方式TBD |
| REQ-IAM-001 | 個人IDとグループRBACで最小権限を付与すること | 管理、運用、監査、Project利用者の権限表とレビュー手順がある | Security/Platform | Must | 未実施 |
| REQ-STG-001 | OpenShift対応の外部CSIで永続領域を提供すること | 互換性、RWO/RWX、Snapshot、性能、容量、暗号化の合格基準がある | Storage | Must | 製品TBD |
| REQ-BKP-001 | etcd、アプリ、PV、外部DBを分離して保護すること | 各対象の方式、頻度、保管、暗号化、Owner、復元手順がある | Platform/Application/Storage | Must | 未実施 |
| REQ-MON-001 | 基盤メトリクスとアラートを監視すること | 標準監視の状態、通知先、優先度、Runbookを定義する | Platform | Must | 未実施 |
| REQ-LOG-001 | infrastructure/application/auditログを収集・転送できること | 対象、保存先、保持、アクセス制御、欠損時アラートを定義する | Platform/Security | Must | 製品版TBD |
| REQ-VIR-001 | OpenShift Virtualization導入可否を第2段階で評価すること | CPU、Firmware、CSI、ネットワーク、容量、対応版をPoC前に確認する | Platform | Should | 未実施 |
| REQ-MTV-001 | VMwareから代表VM 3台をMTV PoC対象にできること | source/provider、権限、変換、停止時間、検証、切り戻しを計画する | Platform/Application | Should | VMware情報TBD |
| REQ-INT-001 | KongのAPI Gateway連携点を設計すること | north-south通信、TLS、認証、監視、責任分界を定義する | Product owner | Could | 製品版TBD |
| REQ-INT-002 | Sysdigの監視・セキュリティ連携点を設計すること | Agent方式、権限、通信、データ保管、アラート分担を定義する | Product owner/Security | Could | 製品版TBD |

## 4. 非機能要件

| ID | 分類 | 要件・目標 | 測定／受入方法 | Owner | 状態 |
| --- | --- | --- | --- | --- | --- |
| REQ-AVL-001 | 可用性 | 月間99.9%。計画保守除外の可否は要確認 | 外形監視の月間成功時間率。起点・終点・除外はG2で固定 | 業務/運用 | 定義TBD |
| REQ-AVL-002 | 冗長性 | 単一ノード障害でAPIと既存アプリ提供を継続する設計 | ノード停止を含む障害試験計画 | Platform | 未実施 |
| REQ-DR-001 | データ保護 | アプリデータRPO 1時間 | 復旧後の最終整合時点を記録 | Application | 未実施 |
| REQ-DR-002 | 復旧 | アプリサービスRTO 4時間 | 障害宣言から業務再開承認までを計時 | 業務/Platform | 未実施 |
| REQ-CAP-001 | 容量 | 平常時に障害退避余力を確保する | CPU、メモリ、PV容量の閾値・予測・増設条件を定義 | Platform/Storage | 実測TBD |
| REQ-PER-001 | 性能 | API・基盤・業務応答の基準を定める | 性能目標値と負荷モデルをアプリ側と合意 | Application | 目標TBD |
| REQ-SEC-001 | セキュリティ | 標準SCC優先、Secret非平文、証明書期限管理、監査可能性 | 設定レビューと否定試験 | Security | 未実施 |
| REQ-OPS-001 | 運用 | 24x365想定の監視・連絡・一次切り分け | アラートから起票・エスカレーションまでの運用試験 | Operations | 未実施 |
| REQ-MNT-001 | 保守 | 更新は事前検証、etcdバックアップ、承認後に実施 | 変更記録と更新前後チェック | Platform | 未実施 |
| REQ-AUD-001 | 監査 | 管理変更を個人ID・変更IDと関連付ける | API監査ログと変更記録のサンプル照合 | Security | 未実施 |

## 5. データ保護の対象定義

RPO/RTOの対象混同を避けるため、次を別々に管理します。

| データ区分 | 例 | 主な保護方式 | RPO/RTO適用 | Owner |
| --- | --- | --- | --- | --- |
| クラスタ状態 | etcd内のKubernetes/OpenShiftリソース状態 | etcdバックアップ | アプリRPOとは別。復旧目標は要確認 | Platform |
| Kubernetesリソース | Deployment、Service、Route等 | Git + OADP | REQ-DR-001/002の構成要素 | Platform/Application |
| PVデータ | アップロードファイル等 | CSI Snapshotまたはデータムーバー（要確認） | REQ-DR-001/002 | Storage/Application |
| 外部DB | 業務DB | DBネイティブバックアップ／PITR（要確認） | REQ-DR-001/002 | Application/DB team |
| イメージ | 内部Registryの業務イメージ | Git/CI再生成またはOADP（要確認） | 復旧優先度を要確認 | Application |

## 6. 制約・要確認事項

- 製品の正確な版、サポート期間、互換性は実施前に公式情報で固定します。
- 99.9%の測定地点、除外時間、違反時の扱いは `TBD-REQ-001`（Owner: 業務責任者、期限: G2）です。
- 性能目標と負荷モデルは `TBD-REQ-002`（Owner: Application、期限: G2）です。
- IdP、CSI、LB、ログ、Kong、Sysdig、OADP、MTVの製品版は各設計書のTBDで管理します。

## 7. 要件レビュー記録

| 日付 | 参加役割 | 対象 | 結果 | 未解決事項 |
| --- | --- | --- | --- | --- |
| - | - | - | 未実施 | 架空案件のため未レビュー |

## 8. 参照資料

- [OpenShift 4.22 Agent-based Installer（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installing_an_on-premise_cluster_with_the_agent-based_installer/)
- [OpenShift 4.22 Authentication and authorization（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/authentication_and_authorization/)
- [OpenShift 4.22 Backup and restore（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/backup_and_restore/)

## 9. 承認

| 役割 | 判定 | 日付 | コメント |
| --- | --- | --- | --- |
| 業務責任者 | 未承認 | - | 実ヒアリング未実施 |
| 基盤責任者 | 未承認 | - | 机上要件 |
| セキュリティ責任者 | 未承認 | - | 要レビュー |

## 10. 変更履歴

| 版 | 日付 | 内容 | 作成者 |
| --- | --- | --- | --- |
| 0.1 | 2026-08-17 | 初版 | 生成 AI 支援ドラフト |
