# 05. OpenShift基盤 基本設計書

## 文書管理

| 項目 | 内容 |
| --- | --- |
| 文書ID | `BD-001-DOC` |
| 対応要件 | [01-requirements.md](01-requirements.md) |
| 版／状態 | 0.1／Draft（机上設計） |
| 基準日 | 2026-08-17 |
| 作成 | 生成 AI 支援ドラフト（本人レビュー前） |
| レビュー／承認 | Platform team／基盤責任者（未実施） |

> [!IMPORTANT]
> 本書は完全に架空の学習用基本設計です。実機構築・試験・製品互換性確認は未実施であり、商用OpenShift設計経験または商用経験の証明ではありません。値は [SCENARIO.md](../SCENARIO.md) に基づく暫定値です。

## 1. 設計目的

業務要件を、詳細設計・構築・試験で検証可能な技術方針へ変換します。本書では「何を、なぜその方式にするか」を定め、製品固有値やコマンドは後続文書へ委ねます。

## 2. 設計原則

1. Red Hatが対象版でサポートする構成・操作を優先する。
2. Control Plane、外部LB、DNS/NTP、ストレージなどの単一障害点を明示する。
3. 管理操作は個人ID、最小権限、変更ID、監査ログで追跡する。
4. 宣言的設定を版管理し、Secret実値は版管理しない。
5. etcd、Kubernetesリソース、PV、外部DBの保護を混同しない。
6. 期待結果と実測結果を分離し、未実施は未実施と記録する。
7. 製品版・互換性・性能は推測で確定せず、Owner付きTBDにする。

## 3. 主要設計判断

| 設計ID | 判断 | 理由 | 代替案 | Owner | 状態 |
| --- | --- | --- | --- | --- | --- |
| BD-001 | OCP 4.22.zを採用候補とする | 共通シナリオと最新機能の学習対象 | 他のサポート済み4.x | Platform | z版TBD |
| BD-002 | Agent-based Installer、`platform: none`を採用 | ベアメタルでインフラを外部管理し、宣言的なAgent設定を使う | IPI bare metal、Assisted Installer | Platform | Draft |
| BD-003 | 6ノードすべてRHCOSとする | OpenShiftノードの一貫したOperator管理 | RHEL worker | Platform | Draft |
| BD-004 | Control Plane 3台、Worker 3台とする | etcd quorumとワークロード分散の最小構成を作る | 追加worker/infra node | Platform | Draft |
| BD-005 | IPv4、OVN-Kubernetes、指定CIDR、静的IPとする | 共通シナリオとの整合、予測可能な名前・アドレス管理 | Dual-stack、DHCP | Network/Platform | Draft |
| BD-006 | APIとApplication Ingressを外部冗長L4 LBへ分離可能な論理poolとして設計 | `platform: none`の事前要件と独立スケール | 単一LB、L7終端 | Network | 製品/HA方式TBD |
| BD-007 | 組織IdP、グループRBAC、標準SCCを優先 | BR-007、最小権限、監査可能性 | ローカル共有ID、広いSCC | Security | IdP方式TBD |
| BD-008 | etcd、OADP対象、PV、外部DBを分離した多層バックアップ | 各データの整合性と復旧手段が異なる | 単一ツールに集約 | Platform/Application/Storage | 方式TBD |
| BD-009 | 更新は事前検証、変更承認、更新前etcdバックアップ、段階確認を必須とする | 24x365想定で変更リスクを制御 | 自動即時更新 | Platform | channel/TBD |
| BD-010 | 標準基盤監視を使用し、ユーザーワークロード監視を有効化候補とする | 基盤とアプリの責任・権限を分離 | 外部監視のみ | Platform/Operations | 通知先TBD |
| BD-011 | Virtualization/MTVを第2段階PoCとして分離 | ストレージ・CPU・停止時間の不確実性を本番基盤受入から分離 | 初期導入へ同梱 | Platform/Application | 未実施 |
| BD-012 | 外部サービスはインターフェース仕様とRACIで受け渡す | DNS/LB/CSI/IdPをOpenShift側だけで完結できない | 単一チーム所有 | PM | Draft |

## 4. 論理構成

[プラットフォーム構成図](../diagrams/platform-architecture.mmd)を正とします。

- API利用者とノードは `api.ocp-prd.example.com` をAPI VIPへ解決します。
- Application利用者は `*.apps.ocp-prd.example.com` をIngress VIPへ解決します。
- LBはTLS内容を改変しないL4方式とし、API poolはControl Plane、Ingress poolはWorkerをbackendとします。
- Control Plane 3台がAPI、スケジューラ、controller、etcd等を担います。通常の業務ワークロードは配置しません。
- Worker 3台がIngress、監視の一部、アプリ、将来のVMを担います。専用infra nodeは本シナリオに含めません。
- DNS、NTP、LB、Proxy、IdP、CSI、オブジェクトストレージはクラスタ外サービスです。

## 5. 可用性設計

| 対象 | 方針 | 障害時の期待 | 前提・残余リスク |
| --- | --- | --- | --- |
| Control Plane/etcd | 3台、同時障害範囲を分散 | 1台故障時もquorumを維持する設計 | ラック・電源分散はTBD-003。2台喪失はquorum喪失 |
| Worker | 3台、複数replicaと分散制約をアプリへ要求 | 1台停止時に他nodeで継続 | 単一replica、local依存、容量不足は継続不可 |
| API/Ingress LB | LB node 2台、VIP引継ぎ | LB 1台故障時にVIP継続 | 製品、HA、切替時間はTBD-004 |
| DNS/NTP | 各2endpoint | 1endpoint障害時に継続 | クライアント設定と実切替試験が必要 |
| Storage | 外部CSI側の冗長方式を要求 | node/経路障害時のI/O継続 | 製品、access mode、fencing未確認 |
| アプリ | 原則2replica以上、PDB、spread、probe | node保守時の提供継続 | アプリ適合性と負荷余力が必要 |

月間99.9%は保証ではなく架空目標です。30日月なら許容停止時間の目安は約43分ですが、測定地点、計画保守除外、部分障害の扱いを `TBD-014` で確定するまで判定できません。

## 6. ネットワーク基本方針

| 項目 | 方針 |
| --- | --- |
| Machine network | `192.0.2.0/24`（文書用アドレス） |
| Pod network | `10.128.0.0/14`、hostPrefix `/23` |
| Service network | `172.30.0.0/16` |
| CNI | OVN-Kubernetes |
| API | `api.ocp-prd.example.com` → `192.0.2.10`、L4 TCP/6443 |
| Internal API | `api-int.ocp-prd.example.com` → `192.0.2.10` |
| Application | `*.apps.ocp-prd.example.com` → `192.0.2.11`、L4 TCP/80,443 |
| 外部接続 | `proxy.example.com:3128`経由。Proxy CAと許可先はTBD |
| NetworkPolicy | tenant namespaceはdefault denyを基本とし、必要通信を明示許可 |

詳細は [07-network-dns-lb-design.md](07-network-dns-lb-design.md) を参照します。

## 7. 認証・認可・セキュリティ基本方針

- 組織IdPをOpenShift OAuthへ連携し、個人IDを使用します。IdP方式・属性・MFAはTBDです。
- 権限は組織グループへRoleBinding/ClusterRoleBindingで付与し、個人への直接付与を例外とします。
- `cluster-admin`常用を禁止し、Platform運用権限を職務単位へ分割します。
- 標準SCC、特に制限された実行条件を優先し、default SCCを編集しません。
- Secret実値、pull secret、秘密鍵、tokenをGit・設計書・実施ログへ記録しません。
- API/Ingress証明書の発行・更新・期限監視とProxy CAの信頼配布をSecurityと共同設計します。
- bootstrap用`kubeadmin`はIdPと代替cluster-admin確認後に削除します。削除前の復旧不能リスクを手順でレビューします。

詳細は [08-security-identity-design.md](08-security-identity-design.md) を参照します。

## 8. ストレージ・バックアップ基本方針

- 外部CSIを採用しますが、製品・StorageClass・access mode・Snapshot・暗号化・性能は未確定です。
- Registry、一般アプリ、共有領域、VM、監視、ログで性能・access modeを分けて評価します。
- OADPはアプリ関連リソースとPV保護に利用する候補ですが、etcdや完全なクラスタDRの代替にしません。
- etcdは1台のControl Planeから公式スクリプトで取得し、安全なクラスタ外へ保管します。
- 外部DBはDBネイティブ方式で整合点を管理し、アプリ復元順序と関連付けます。
- RPO 1時間、RTO 4時間は実データ量での復元試験に合格するまで未検証です。

詳細は [09-storage-backup-design.md](09-storage-backup-design.md) を参照します。

## 9. 監視・ログ・運用基本方針

- OpenShift標準の基盤監視をサポートされた設定方法で利用します。
- user workload monitoringは有効化候補とし、アプリチームがProject内のServiceMonitor/PrometheusRuleを管理できる役割を設計します。
- Alertmanagerから外部通知先への連携、営業時間外の受付、重要度別エスカレーションはTBDです。
- infrastructure/application/auditログを区分し、短期調査用と長期監査用の保存先を分けます。
- LoggingおよびLokiの正確な対応版・容量・保持はOCP 4.22.z確定後に互換性を確認します。
- Sysdigは標準監視を置換せず、セキュリティ・ランタイム検知等の責任を追加する候補です。

詳細は [10-observability-operations-design.md](10-observability-operations-design.md) を参照します。

## 10. 更新・変更・構成管理

1. 本番と同等条件の検証クラスタで対象更新を先行確認します（本演習環境には存在しないためTBD）。
2. リリースノート、既知問題、Operator/CSI/Kong/Sysdig互換性をレビューします。
3. etcdと対象アプリの最新バックアップ、および復元可能性を確認します。
4. 変更申請に手順、影響、中断基準、切り戻し方針、連絡先を添付します。
5. 承認後に一段階ずつ実施し、ClusterOperator、node、業務外形を確認します。
6. OpenShift更新の安易なダウングレードを切り戻し策とせず、停止・サポート相談・復旧の判断点を明示します。

## 11. 要件対応要約

| 要件 | 設計対応 | 確認 |
| --- | --- | --- |
| REQ-PLT-001〜003 | BD-001〜004 | TST-INS/NOD系列（未実施） |
| REQ-NET-001〜004 | BD-005/006 | TST-DNS/LB/NET/PRX系列（未実施） |
| REQ-IDM/IAM/SEC/AUD | BD-007 | TST-IAM/SEC/AUD系列（未実施） |
| REQ-STG/BKP/DR | BD-008 | TST-STG/BKP/RST系列（未実施） |
| REQ-MON/LOG/OPS/MNT | BD-009/010 | TST-MON/LOG/OPS/UPG系列（未実施） |
| REQ-VIR/MTV | BD-011 | TST-VIR/MTV系列（未実施） |

## 12. 基本設計レビューの完了条件

- 主要設計判断に要件ID、根拠、Ownerがある。
- `TBD-003/004/006/007/008/010/013/014`の解消またはリスク受容が記録される。
- 外部チームがインターフェースとRACIを受領する。
- 可用性、RPO/RTO、セキュリティ、更新の試験可能な受入条件が定まる。
- 対象z版とサポート範囲を詳細設計開始前に固定する。

## 13. 公式根拠

- [OpenShift 4.22 Agent-based Installer（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html-single/installing_an_on-premise_cluster_with_the_agent-based_installer/index)
- [OpenShift 4.22 Networking overview（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html-single/networking_overview/index)
- [OpenShift 4.22 Authentication and authorization（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/authentication_and_authorization/)
- [OpenShift 4.22 Backup and restore（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html-single/backup_and_restore/index)
- [Monitoring stack for Red Hat OpenShift 4.22（Red Hat公式）](https://docs.redhat.com/en/documentation/monitoring_stack_for_red_hat_openshift/4.22/)

## 14. 承認・変更履歴

| 役割 | 判定 | 日付 | コメント |
| --- | --- | --- | --- |
| Platform lead | 未レビュー | - | 実機・製品確認前 |
| 基盤責任者 | 未承認 | - | 架空案件 |

| 版 | 日付 | 内容 | 作成者 |
| --- | --- | --- | --- |
| 0.1 | 2026-08-17 | 初版 | 生成 AI 支援ドラフト |
