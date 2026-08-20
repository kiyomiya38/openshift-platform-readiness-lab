# 10. 監視・ログ・運用設計書

## 文書管理

| 項目 | 内容 |
| --- | --- |
| 文書ID | `OBS-OPS-DES-001` |
| 上位要件 | `REQ-MON-001`、`REQ-LOG-001`、`REQ-OPS-001`、`REQ-MNT-001`、`REQ-AVL-001` |
| 版／状態 | 0.1／Draft（机上設計） |
| 基準日 | 2026-08-17 |
| 作成 | 文書作成チーム（サンプル） |
| レビュー／承認 | Operations/Platform/Security／運用責任者（未実施） |

> [!IMPORTANT]
> 本書は一般公開用の架空プロジェクトにおける机上設計サンプルです。監視設定、通知、ログ収集、24x365運用、障害対応、更新は未実施で、SLOは未承認・未測定です。

## 1. 運用目標

- 基盤・アプリの異常を利用者申告より前に検知できる状態を目指します。
- アラートは通知だけで終わらせず、Owner、重要度、一次確認、Runbook、終了条件を持たせます。
- 月間可用性99.9%を測るため、SLI地点・計画保守除外・部分障害をG2で定義します。
- 変更、障害、容量、backup、certificate、権限、更新を定期作業として管理します。
- 標準監視、Logging、Sysdigの責務を重複させたままにせず、system of recordを決めます。

## 2. 監視設計判断

| ID | 判断 | 根拠 | Owner | 状態 |
| --- | --- | --- | --- | --- |
| MON-001 | OpenShift標準の基盤監視を主監視にする | preinstalled/self-updating stackとサポート性 | Platform | Draft |
| MON-002 | user workload monitoringを有効化候補とする | Project単位のアプリmetrics/alert | Platform/Application | 容量TBD |
| MON-003 | Alertmanagerから外部通知/incident受付へ連携 | 24x365想定の初動 | Operations | endpoint/TBD |
| MON-004 | 可用性SLIは外形、API、Route、業務応答を階層化 | 原因箇所と業務影響を分離 | Operations/Application | 測定地点TBD |
| MON-005 | backup、certificate、capacity、更新も監視対象 | 予防保守 | Platform | Draft |
| MON-006 | Sysdigは標準監視の代替ではなく追加統制候補 | 責任・サポート境界を明確化 | Product owner/Security | 版/TBD |

## 3. 監視アーキテクチャ

OpenShift標準監視はcore platformを監視します。user-defined projectの監視は任意機能として分離し、`openshift-user-workload-monitoring`側の資源・retention・権限を見積もってから有効化します。

| Layer | 対象 | 収集/判定 | 主Owner | 例 |
| --- | --- | --- | --- | --- |
| 外形 | API、Console、主要Route、業務API | クラスタ外probe候補 | Operations/Application | TLS、HTTP code、latency |
| Platform | ClusterOperator、node、etcd、API、Ingress、MCP | 標準monitoring | Platform | Available/Degraded、NodeReady |
| Resource | CPU、memory、filesystem、PV、Pod | Prometheus metrics | Platform/Application | saturation、OOM、restart |
| Application | request、error、duration、queue、business KPI | user workload monitoring | Application | RED metrics、業務件数 |
| Dependency | DNS、LB、NTP、Proxy、CSI、IdP、backup endpoint | 外部監視 + synthetic check | 各Owner | availability、latency、expiry |
| Security | audit、image、runtime、policy | Logging/Sysdig候補 | Security | suspicious action、vulnerability |

## 4. アラートカタログ

しきい値は設計案です。標準アラートを無効化・変更する前に対象版のサポート範囲を確認します。

| Alert ID | 条件案 | 継続 | Severity | 一次対応 | Runbook | 状態 |
| --- | --- | ---: | --- | --- | --- | --- |
| ALT-001 | API外形失敗または重大なAPI可用性alert | 1分 | P1 | Operations→Platform/Network | RB-API-001 | 未実装 |
| ALT-002 | ClusterOperator Degraded=True | 10分（重大componentは即時） | P2/P1 | Operations→Platform | RB-CO-001 | 未実装 |
| ALT-003 | Node NotReady | 5分 | P2、複数nodeはP1 | Operations→Platform/Hardware | RB-NODE-001 | 未実装 |
| ALT-004 | 主要Route外形失敗 | 3回連続 | P1/P2 | Operations→Application/Platform | RB-APP-001 | 未実装 |
| ALT-005 | Pod CrashLoop/OOM/restart急増 | 10分 | P2/P3 | Application | RB-POD-001 | 未実装 |
| ALT-006 | Worker CPU/memory request 70%超 | 30分 | P3 | Platform | RB-CAP-001 | 未実装 |
| ALT-007 | PV 70%/85%超 | 15分 | P3/P2 | Storage/Application | RB-STG-001 | 未実装 |
| ALT-008 | backup失敗または最終成功から1時間超 | 即時 | P2 | Platform/Storage | RB-BKP-001 | 未実装 |
| ALT-009 | certificate期限60/30/14日 | 日次 | P3/P2/P1 | Security/Platform | RB-CERT-001 | 未実装 |
| ALT-010 | log collector/forwarder停止 | 10分 | P2 | Platform/Security | RB-LOG-001 | 未実装 |
| ALT-011 | IdP/OAuth login失敗率急増 | 5分 | P2 | Security/Platform | RB-IDP-001 | 未実装 |
| ALT-012 | etcd容量/leader/quorum関連標準alert | 標準rule | P1/P2 | Platform | RB-ETCD-001 | 未実装 |

### 4.1 アラート品質

- 各alertにsummary、impact、Owner、dashboard、Runbook、変更/抑止条件を付けます。
- P1/P2は人へ通知し、P3は業務時間内queue、P4は傾向分析とします。実受付時間・応答OLAはTBDです。
- 保守silenceには変更ID、対象、開始・終了、承認者を必須とし、広範囲・長期間を避けます。
- false positive、重複、未対応、通知失敗を月次レビューします。

## 5. SLI/SLO設計

| SLI ID | 測定 | 算定案 | 目標 | 除外 | 状態 |
| --- | --- | --- | --- | --- | --- |
| SLI-001 | 主要業務Routeの成功率 | 成功probe / 全probe | 月間99.9%候補 | 計画保守の扱いTBD | 未実測 |
| SLI-002 | API可用性 | API ready/外形成功時間率 | 内部運用目標TBD | なしを起点 | 未実測 |
| SLI-003 | 業務応答時間 | p95/p99 latency | Applicationが定義 | maintenance/TBD | 目標TBD |
| SLI-004 | backup freshness | 現在時刻 − 最終正常restore point | 1時間以内 | 承認停止のみ | 未実測 |
| SLI-005 | incident recovery | 障害宣言から業務再開 | 4時間以内（対象scenario） | cluster/site全損は別要件 | 未実測 |

99.9%は外形監視だけで原因を示しません。SLI-001を業務判定、SLI-002やcomponent metricsを原因分析に使います。

## 6. ログ設計判断

| ID | 判断 | 根拠 | Owner | 状態 |
| --- | --- | --- | --- | --- |
| LOG-001 | application/infrastructure/auditを区分して収集 | アクセス権・保持・用途が異なる | Security/Platform | Draft |
| LOG-002 | OpenShift Logging対応版をOCP z版確定後に固定 | Loggingは独立release cycle | Platform | 版TBD |
| LOG-003 | Lokiは短期トラブルシュート用途候補とし、長期監査は外部保管 | 公式の用途境界と容量管理 | Security | 保存先TBD |
| LOG-004 | audit logは既定で安全に長期保存されると仮定しない | 欠損・権限・保持risk | Security | Draft |
| LOG-005 | log転送停止・drop・buffer枯渇を監視 | sink障害時のsilent loss防止 | Platform | 未実装 |
| LOG-006 | token/Secret/個人データをmaskし、閲覧をrole分離 | 情報漏えい防止 | Security/Application | mask rule TBD |

## 7. ログ分類・保持案

| Log | Source | 短期保存案 | 長期保存案 | 閲覧 | Owner | 状態 |
| --- | --- | ---: | ---: | --- | --- | --- |
| Application | workload stdout/stderr | 30日 | 必要な業務logのみ外部90日案 | Project team | Application | 要件TBD |
| Infrastructure | node/container/platform | 30日 | 障害・規程に応じ90日案 | Platform | Platform | 容量TBD |
| Audit | API audit | 短期検索領域TBD | 外部365日案 | Security限定 | Security | 規程TBD |
| Security/Sysdig | runtime/vulnerability events | 製品側TBD | 規程TBD | Security | Product owner | 製品TBD |

保持期間は仮値です。法務・監査要件、1日当たり量、圧縮率、検索性能、削除・legal holdを確認して確定します。Loki/object storageに保持が未設定のまま容量無制限となる状態を避けます。

## 8. Incident管理

| Severity | 例 | 初回応答案 | 更新間隔案 | 判断者 |
| --- | --- | ---: | ---: | --- |
| P1 | 全面停止、データ損失、重大侵害 | 15分 | 30分 | Incident commander/基盤責任者 |
| P2 | 冗長性喪失、主要機能劣化、backup RPO逸脱 | 30分 | 60分 | Platform/領域責任者 |
| P3 | 回避可能な部分異常、容量予兆 | 4営業時間 | 日次 | 各Owner |
| P4 | 問合せ、改善候補 | 2営業日 | 合意時 | Service owner |

### 8.1 標準フロー

1. 検知時刻、影響、対象、最終正常、直前変更を記録します。
2. incident IDとcommanderを決め、推測と事実を分けます。
3. DNS→LB→API/Operator→node→workload→dependencyの順で観測し、証拠を時系列化します。
4. 安全な暫定対処と中断基準を承認し、同時に復旧/切り戻しを準備します。
5. 技術正常だけで終了せず、Applicationが業務再開を承認します。
6. P1/P2は原因、寄与要因、再発防止、Owner、期限を事後レビューします。

## 9. 定常運用

| 周期 | 作業 | 証跡 | Owner |
| --- | --- | --- | --- |
| 日次 | P1/P2、ClusterOperator/node、backup freshness、log pipeline | 日次check記録 | Operations |
| 週次 | capacity trend、失敗job、certificate、未解決incident/変更 | 週次report | Platform |
| 月次 | SLO、alert品質、権限差分、backup世代、脆弱性、費用/容量 | 月次service report | Service owner |
| 四半期 | RBAC棚卸し、restore drill、failover、Runbook訓練 | review/test evidence | 複数Owner |
| 半期 | end-to-end RTO試験、DR gap、architecture review | resilience report | 基盤責任者 |
| 更新前 | compatibility、release note、backup、手順、通知、No-Go | change record | Platform |

## 10. 更新・保守運用

| ID | 方針 | 確認 | Owner |
| --- | --- | --- | --- |
| OPS-001 | 24x365一次監視と領域別escalation | 連絡・受付試験 | Operations |
| OPS-002 | P1〜P4分類とincident commander | tabletop drill | Operations |
| OPS-003 | Runbookをalertから直接参照 | link/permission check | Platform |
| OPS-004 | 月間SLOを報告しerror budgetをレビュー | monthly report | Service owner |
| OPS-005 | backup/log/certificate監視を業務monitoringと同格管理 | simulated alert | Platform |
| OPS-006 | capacity予測と増設lead timeを月次確認 | capacity report | Platform/Storage |
| OPS-007 | 更新は非本番先行、互換性review、段階実施 | update plan/test | Platform |
| OPS-008 | 計画保守は申請、影響、通知、承認、結果を記録 | change record | PM/Operations |
| OPS-009 | 構成差分と監査logを変更IDへ関連付け | audit sample | PM/Security |
| OPS-010 | 運用引き継ぎは教育・訓練・受領署名まで含む | handover record | Operations |

更新時は正確な4.22.z、channel、次version、Operator/CSI/Kong/Sysdig互換性、既知問題を確認します。OpenShiftのdowngradeを通常のrollbackと見なさず、失敗時は作業停止、健全性確認、Red Hatサポート連携、必要時の復旧を個別判断します。

## 11. Runbook一覧

| Runbook ID | 対象 | 最低限の内容 | Owner | 状態 |
| --- | --- | --- | --- | --- |
| RB-API-001 | API/Console不可 | DNS/LB/readyz/Operator/証明書/直前変更 | Platform/Network | 未作成 |
| RB-CO-001 | ClusterOperator Degraded | condition、related object、event、support data | Platform | 未作成 |
| RB-NODE-001 | Node NotReady | condition、kubelet相当、network、BMC、drain可否 | Platform/Hardware | 未作成 |
| RB-POD-001 | Pod異常 | event、log、probe、quota、SCC、image、dependency | Application | 未作成 |
| RB-STG-001 | PVC/storage異常 | PVC/PV/CSI event、path、backend、data risk | Storage | 未作成 |
| RB-BKP-001 | backup/RPO異常 | 最終成功、対象、storage、credential、再実行判断 | Platform/Storage | 未作成 |
| RB-CERT-001 | certificate期限/失効 | owner、SAN、chain、更新、rollback | Security | 未作成 |
| RB-LOG-001 | log欠損 | collector、buffer、sink、drop、復旧不能範囲 | Platform/Security | 未作成 |
| RB-IDP-001 | 認証障害 | IdP/OAuth/DNS/CA/token、break-glass | Security/Platform | 未作成 |
| RB-ETCD-001 | etcd異常 | quorum、member、I/O、公式手順、restore禁止条件 | Platform | 未作成 |

## 12. 運用試験

| 試験ID | 内容 | 期待結果 | 証跡 | 状態 |
| --- | --- | --- | --- | --- |
| TST-MON-001 | 標準monitoring target/rule | 必須target Up、rule errorなし | Prometheus/API出力 | 未実施 |
| TST-MON-002 | sample alert発火・通知 | severity、label、通知、起票が設計どおり | alert/notification/ticket | 未実施 |
| TST-MON-003 | alert復旧 | 原因解消後にresolve通知とissue更新 | alert timeline | 未実施 |
| TST-MON-004 | 通知先障害 | 欠損検知または代替通知が設計どおり | notification log | 未実施 |
| TST-LOG-001 | infrastructure log | node/pod/namespace/timeで検索可能 | query/result | 未実施 |
| TST-LOG-002 | application marker log | markerが欠損せず権限内で閲覧可能 | query/result | 未実施 |
| TST-LOG-003 | audit marker operation | 主体、verb、resourceを追跡可能 | query/result | 未実施 |
| TST-LOG-004 | forward先停止 | buffer/欠損alert/復旧後転送が設計どおり | collector/receiver log | 未実施 |
| TST-OPS-001 | P1 tabletop | 15分以内に受付・指揮・連絡開始 | timeline | 未実施 |
| TST-UPG-001 | 更新rehearsal | 事前・実施・事後・停止判断を完遂 | change/test result | 未実施 |
| TST-SLO-001 | 月次算定 | 同一raw dataから再計算可能 | query/report | 未実施 |

## 13. TBDと中断条件

| ID | 内容 | Owner | 期限 | 中断条件 |
| --- | --- | --- | --- | --- |
| TBD-OPS-001 | 24x365受付、連絡先、OLA | Operations | G2前 | P1受付不能なら本番化不可 |
| TBD-OPS-002 | SLI地点・除外・月次算定 | 業務/Operations | G2前 | 99.9%判定定義なしなら受入不可 |
| TBD-OPS-003 | Monitoring保持・PV・通知先 | Platform | G3前 | alert通知試験不合格なら本番化不可 |
| TBD-OPS-004 | Logging版、保存先、保持、SIEM | Security/Platform | G2前 | audit保存不明なら利用開始不可 |
| TBD-OPS-005 | Sysdig責任・権限・連携 | Product owner/Security | 第2段階前 | 過剰権限/通信未承認なら導入しない |

## 14. 公式根拠

- [Monitoring stack for Red Hat OpenShift 4.22（Red Hat公式）](https://docs.redhat.com/en/documentation/monitoring_stack_for_red_hat_openshift/4.22/)
- [About OpenShift monitoring（Red Hat公式）](https://docs.redhat.com/en/documentation/monitoring_stack_for_red_hat_openshift/4.22/html-single/about_monitoring/index)
- [Configuring user workload monitoring（Red Hat公式）](https://docs.redhat.com/en/documentation/monitoring_stack_for_red_hat_openshift/4.22/html-single/configuring_user_workload_monitoring/index)
- [OpenShift 4.22 Logging（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/logging/index)
- [Red Hat OpenShift Logging documentation（Red Hat公式）](https://docs.redhat.com/en/documentation/red_hat_openshift_logging/)

## 15. 承認・変更履歴

| 役割 | 判定 | 日付 | コメント |
| --- | --- | --- | --- |
| Operations lead | 未レビュー | - | 受付/OLA未確定 |
| Platform/Security lead | 未レビュー | - | 監視・ログ未実装 |
| 基盤責任者 | 未承認 | - | 架空案件 |

| 版 | 日付 | 内容 | 作成者 |
| --- | --- | --- | --- |
| 0.1 | 2026-08-17 | 初版 | 文書作成チーム（サンプル） |
