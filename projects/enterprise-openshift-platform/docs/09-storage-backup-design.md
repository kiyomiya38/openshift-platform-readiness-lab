# 09. ストレージ・バックアップ・復旧設計書

## 文書管理

| 項目 | 内容 |
| --- | --- |
| 文書ID | `STG-BKP-DES-001` |
| 上位要件 | `REQ-STG-001`、`REQ-BKP-001`、`REQ-DR-001`、`REQ-DR-002` |
| 版／状態 | 0.1／Draft（机上設計） |
| 基準日 | 2026-08-17 |
| 作成 | 生成 AI 支援ドラフト（本人レビュー前） |
| レビュー／承認 | Storage/Platform/Application／基盤責任者（未実施） |

> [!IMPORTANT]
> 本書は架空の学習用机上設計です。CSI、OADP、Snapshot、DBバックアップ、復元性能は未選定・未実施であり、RPO/RTOを達成した証拠や商用経験の証明ではありません。

## 1. RPO/RTOの適用範囲

- RPO 1時間: アプリケーションデータを、障害直前から最大1時間以内の整合点へ戻す目標です。
- RTO 4時間: 障害宣言から、対象アプリの復元・技術確認・業務再開承認までの目標です。
- このRTO案は「クラスタと復元先ストレージが利用可能なアプリ障害」を基準とします。
- サイト全損・クラスタ全損からの4時間復旧は、待機クラスタや代替機材がない現設計では保証できません。別DR要件 `TBD-BKP-001` とします。
- etcd backupの時点とアプリデータの整合点は同じとは限りません。クラスタ状態復旧と業務データ復旧を別手順で調整します。

## 2. ストレージ設計判断

| ID | 判断 | 根拠 | Owner | 状態 |
| --- | --- | --- | --- | --- |
| STG-001 | OpenShift 4.22.z対応の外部CSIを採用 | 動的provisioningとサポート性 | Storage | 製品TBD |
| STG-002 | workload用途別にRWO/RWX/性能tierを分ける | 性能・障害・費用の分離 | Storage/Application | class名TBD |
| STG-003 | default StorageClassは一般アプリ用RWO候補1つに限定 | 意図しないtier利用防止 | Storage/Platform | TBD |
| STG-004 | 使用率・増加率・増設lead timeを容量監視 | 枯渇予防 | Storage/Operations | 実値TBD |
| STG-005 | IOPS/throughput/latencyをworkload別に受入 | 容量だけでは性能を保証できない | Application/Storage | 目標TBD |
| STG-006 | Snapshot、clone、expansion、reclaim、fencingを明示的に検証 | backup/VM/削除事故への影響 | Storage | 未実施 |

## 3. 論理StorageClass設計

名称は設計上の論理名です。CSI製品の実StorageClass名へ置き換えます。

| 論理class | 用途 | AccessMode | VolumeMode | 性能 | reclaimPolicy案 | expansion | Snapshot | 状態 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `sc-rwo-general` | 一般アプリ、監視 | RWO | Filesystem | 標準 | Delete（データ分類でRetain選択） | 必須候補 | 必須候補 | 製品TBD |
| `sc-rwx-shared` | 共有データ、Registry候補 | RWX | Filesystem | 標準 | Retain候補 | 必須候補 | 要確認 | 製品TBD |
| `sc-rwo-fast` | DB補助領域、高I/Oアプリ | RWO | Filesystem/Block | 高IOPS | Retain候補 | 必須候補 | 必須候補 | 製品TBD |
| `sc-vm` | 第2段階VM disk | RWO/RWX（方式依存） | Block候補 | latency重視 | Retain | 必須候補 | 必須候補 | Virtualization適合TBD |

`volumeBindingMode`、topology、multipath、encryption at rest、online expansion、snapshot classはベンダー/Red Hat対応表と実機試験で確定します。

### 3.1 Image Registryのstorageと状態遷移

本シナリオの`platform: none`ベアメタル構成は、Image Registry用の共有object storageをinstallerが自動提供しません。この場合、Image Registry Operatorは導入時に`managementState: Removed`でbootstrapされます。

- `openshift-install ... wait-for install-complete`の完了と、production向けinternal registryの利用可能性を別の状態として判定します。
- インストール後、CSIのRWX capability、容量、性能、backupを確認し、`sc-rwx-shared`相当のサポートされる永続storageをRegistryへ割り当てます。実StorageClass名とaccess modeはCSI選定後に確定します。
- `configs.imageregistry.operator.openshift.io/cluster`へ承認済みstorage設定を反映し、`managementState: Managed`へ変更します。
- Image Registry Operator/Pod、PVC `Bound`、image push/pull、Pod再作成後のimage永続性を確認します。
- registryが`Managed`かつ正常でなく、永続性試験が不合格なら、installerが完了していても基盤の本番受入は`No-Go`です。
- 一時storageで本番受入を代替しません。外部object storage等へ方式変更する場合も、OCP 4.22.zの対応方式を再確認します。

## 4. 容量設計

| 対象 | 初期容量入力 | 成長率 | 予約/余力 | 警告案 | 重大案 | Owner | 状態 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Registry | TBD | TBD | image GCと再生成性を考慮 | 70% | 85% | Platform | 未算定 |
| Application PV | TBD | TBD | RPO対象とsnapshot増分を含む | 70% | 85% | Application/Storage | 未算定 |
| Monitoring | TBD | retention/series数依存 | compaction/一時領域を含む | 70% | 85% | Platform | 未算定 |
| Logging | TBD | log rate/保持依存 | object store lifecycle含む | 70% | 85% | Platform/Security | 未算定 |
| VM | PoC 3台分TBD | 本番移行対象外 | live migration/clone余力 | 70% | 85% | Platform/Storage | 未算定 |
| Backup store | TBD | hourly/daily保持依存 | immutability/複製を含む | 70% | 85% | Storage | 未算定 |

容量確定式は `実データ + 成長期間 + Snapshot/backup + 障害/再配置余力 + 運用buffer` とし、重複排除・圧縮率は実測前に保証しません。

## 5. バックアップ設計判断

| ID | 判断 | 理由 | Owner | 状態 |
| --- | --- | --- | --- | --- |
| BKP-001 | データ区分ごとに方式と復元Ownerを分離 | 単一方式で全整合性を保護できない | Platform/Application/Storage | Draft |
| BKP-002 | etcdは公式手順で単一Control Planeから取得しcluster外保管 | 全master個別取得は不要、cluster状態保護 | Platform | 未実施 |
| BKP-003 | OADPをnamespace/resource/PV保護候補にする | アプリ単位の保護・復元 | Platform | 版/plugin TBD |
| BKP-004 | OADPをetcdや完全cluster DRの代替にしない | 公式サポート境界 | Platform | Draft |
| BKP-005 | PVはCSI Snapshotまたはdata mover方式を適合性で選ぶ | storage能力と整合性が異なる | Storage/Platform | TBD |
| BKP-006 | 外部DBはDB native backup/PITRとアプリquiesceを使う | transaction整合性 | Application/DB | DB製品TBD |
| BKP-007 | 復旧手順へ4時間のtime budgetと業務判定を設定 | RTOを作業時間へ分解 | Application/Platform | 未検証 |
| BKP-008 | 定期復元試験で実効性を確認 | backup成功だけでは復旧を証明しない | Operations | 未実施 |

## 6. バックアップ対象・頻度・保持

以下はRPOを検証するための設計案で、承認済み運用値ではありません。

| 対象 | 方式案 | 頻度案 | 保持案 | 保存先 | 暗号化 | Owner | 状態 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| etcd + static pod resources | `cluster-backup.sh`相当の公式手順 | 日次、更新/重大変更前 | 日次14、週次8世代案 | secure off-cluster | transit/at rest必須 | Platform | 未実施 |
| Git管理可能resource | Git repository + release tag | 変更ごと | repository policy | Git service | TLS/権限制御 | Platform/Application | 未実施 |
| Namespace resource | OADP | 1時間ごと案 | hourly 48、daily 30案 | 192.0.2.15 object endpoint | TLS + at rest必須 | Platform | OADP TBD |
| PV | CSI Snapshot/data mover | 1時間ごと案 | 上記と整合 | storage/object endpoint | 必須 | Storage | 方式TBD |
| 外部DB | native full + log/PITR | full日次、log 15分以下案 | DB規程TBD | DB backup store | 必須 | Application/DB | 製品TBD |
| Registry/internal image | Git/CI再生成 + OADP対象評価 | releaseごと/1時間案 | image policy | registry/object store | 必須 | Application/Platform | TBD |

バックアップ時刻は同時I/O集中を避け、application resource、PV、DBの整合順序をアプリごとのbackup profileで決めます。

## 7. etcd保護

- 最初のcertificate rotation完了前の取得を避け、対象4.22.zの公式注意事項に従います。
- I/O影響を考慮し非peak時間に取得します。
- 1回の取得につきControl Plane 1台だけでbackup scriptを実行します。
- snapshotとstatic pod resourcesの組を同じbackup IDで保管します。
- OpenShift更新前に取得し、復元時は同じz-streamのbackupが必要という制約を台帳へ記録します。
- etcd restoreは稼働clusterの状態を巻き戻す破壊的な最終手段です。APIで通常修復できる事象に使用しません。

## 8. OADP・アプリ保護

- OADPはcustomer workload namespaceと関連cluster-scope resourceの範囲を設計します。
- Operator自体をアプリbackupへ無条件に含めず、対象版の公式除外要件を確認します。
- backup selectionはnamespace/label/resource typeで明示し、platform namespaceを一括対象にしません。
- CSI Snapshotの整合性はstorage snapshotだけではDB transaction整合を保証しないため、pre/post hookまたはDB native方式を組み合わせます。
- object storage credentialは専用ServiceAccount/Secret参照とし、restore環境以外からの削除権限を制限します。
- Backup CRの`Completed`だけで合格とせず、item error、warning、PV object、保管先objectを確認します。

## 9. 復元シナリオ

| Scenario | 復旧対象 | 主手段 | 判断者 | 完了条件 | 状態 |
| --- | --- | --- | --- | --- | --- |
| RST-01 誤削除 | namespace/resource | GitまたはOADP restore | Application owner | resource、data、外形、業務整合OK | 未実施 |
| RST-02 PV破損 | PVC/PV data | Snapshot/data mover restore | Storage/Application | 最終整合時点≤1h、checksum/業務確認 | 未実施 |
| RST-03 DB障害 | 外部DB | native restore/PITR | DB/Application | transaction整合、アプリ接続、業務確認 | 未実施 |
| RST-04 control plane状態破損 | etcd/cluster state | etcd restore（最終手段） | Platform lead | API/Operator/node正常、data不整合評価 | 未実施 |
| RST-05 cluster/site全損 | cluster + all workload | 再構築 + all data restore | 基盤責任者 | DR受入基準 | 未設計。4h保証外 |

## 10. RTO 4時間のtime budget案

対象はRST-01〜03で、復元可能なclusterがある場合です。

| 区間 | 目標 | 主担当 | Exit criteria |
| --- | ---: | --- | --- |
| 検知・障害宣言 | 15分 | Operations/Application | incident ID、影響、開始時刻確定 |
| 復旧判断・整合点選択 | 15分 | Application/Platform/Storage | restore pointと手段承認 |
| 復旧先準備 | 60分 | Platform/Storage | namespace/PVC/DB target準備完了 |
| resource/data restore | 90分 | Platform/Storage/DB | restore job成功、error確認 |
| 技術・業務検証 | 45分 | Application | data時点、外形、主要業務OK |
| buffer・再開承認 | 15分 | 業務責任者 | 再開記録・監視強化開始 |
| 合計 | 240分 | 複数 | RTO 4時間以内 |

各区間を実データで計時し、超過した場合は並列化、pre-provision、backup頻度、回線/性能、手順を改善します。

## 11. 復元試験

| 試験ID | 内容 | 頻度案 | 合格条件 | 証跡 | 状態 |
| --- | --- | --- | --- | --- | --- |
| TST-STG-001 | 動的PVC作成/削除 | 導入・変更時 | class、PV、reclaimが設計どおり | YAML/event | 未実施 |
| TST-REG-001 | Image Registry永続化 | 導入・storage変更時 | `Managed`、Operator/Pod/PVC正常、push/pullと永続性OK | config/PVC/image digest | 未実施 |
| TST-STG-002 | RWO node移動 | 導入・変更時 | detach/attach後もdata整合、二重mountなし | event/checksum | 未実施 |
| TST-STG-003 | RWX同時mount | 導入・変更時 | 複数nodeのread/writeとdata整合 | Pod/node/checksum | 未実施 |
| TST-STG-004 | VolumeSnapshot復元 | 導入・四半期 | 別PVCへ復元しdata一致 | snapshot/PVC/checksum | 未実施 |
| TST-STG-005 | PVC online expansion | 導入時 | filesystem/volume拡張成功、data保持 | before/after/checksum | 未実施 |
| TST-STG-006 | Storage性能 | 導入・変更時 | 合意済みIOPS/latency目標を満たす | profile/raw result | 未実施 |
| TST-BKP-001 | etcd backup取得・検査 | 月次案 | fileset、size、hash、off-cluster copy確認 | backup log/hash | 未実施 |
| TST-BKP-002 | OADP backup | 月次案 | errorなし、対象resource/PV存在 | Backup CR/object list | 未実施 |
| TST-RST-001 | etcd restore演習 | 半期案 | 隔離環境で対象z版の公式手順によりAPI/resource状態を復旧 | timeline/log | 未実施 |
| TST-RST-002 | namespace resource restore | 四半期案 | Deployment/Service/Route/RBACが期待版へ復元 | diff/`oc`出力 | 未実施 |
| TST-RST-003 | PV/DB整合復元 | 四半期案 | 同一整合点、data loss≤1h、業務整合OK | time/checksum/業務判定 | 未実施 |
| TST-RST-004 | end-to-end RPO/RTO | 半期案 | 障害宣言から業務承認≤4h | timeline/approval | 未実施 |

復元試験はproductionを上書きしない隔離環境で行い、復元先、network、credential、後片付けを事前承認します。

## 12. 保護・監視・アクセス制御

- backupはcluster外へ保管し、production管理者だけで全世代を削除できない権限分離を検討します。
- TLS、at-rest encryption、key Owner、rotation、法定保持、immutability/air-gap copyはSecurityと決定します。
- job失敗、最終成功時刻、RPO逸脱、保存先容量、Snapshot失敗、credential期限を監視します。
- 日次で失敗、週次で世代・容量、月次でrestore sample、四半期でend-to-endを確認する案です。
- 削除・期限切れ・legal holdの処理を変更記録へ残します。

## 13. TBDと中断条件

| ID | 内容 | Owner | 期限 | 中断条件 |
| --- | --- | --- | --- | --- |
| TBD-BKP-001 | cluster/site全損のRPO/RTOと代替基盤 | 基盤責任者 | G2前 | 4hを全損にも要求するなら現方式を再設計 |
| TBD-BKP-002 | CSI製品・互換性・Snapshot/VM適合 | Storage | G2前 | compatibility未確認ならstateful workload受入不可 |
| TBD-BKP-003 | OADP版/plugin/data mover | Platform/Storage | G3前 | restore PoC不合格なら本番化不可 |
| TBD-BKP-004 | DB製品・quiesce/PITR手順 | Application/DB | G3前 | 整合性確認不能ならRPO受入不可 |
| TBD-BKP-005 | 保持、暗号化、immutability、削除権限 | Security/Storage | G3前 | backup改ざん/消失risk未受容なら開始不可 |

## 14. 公式根拠

- [OpenShift 4.22 Backup and restore（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html-single/backup_and_restore/index)
- [Control plane backup and restore（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/backup_and_restore/control-plane-backup-and-restore)
- [Configuring the Image Registry Operator（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/registry/configuring-registry-operator)

公式資料上、OADPはetcdやOperatorのDR、完全cluster backup/restoreの代替ではありません。本設計も同じ境界を採用します。

## 15. 承認・変更履歴

| 役割 | 判定 | 日付 | コメント |
| --- | --- | --- | --- |
| Storage lead | 未レビュー | - | CSI未選定 |
| Application/DB owner | 未レビュー | - | data/整合手順未定 |
| Platform lead | 未レビュー | - | backup/restore未実施 |
| 基盤責任者 | 未承認 | - | 架空案件 |

| 版 | 日付 | 内容 | 作成者 |
| --- | --- | --- | --- |
| 0.1 | 2026-08-17 | 初版 | 生成 AI 支援ドラフト |
