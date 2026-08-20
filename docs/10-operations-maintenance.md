# 10. 運用・保守

## 目的

構築・試験済みのOpenShift基盤を、監視、定常作業、変更、更新、バックアップ・復元、障害対応、容量・証明書・権限管理によって維持する方法を理解します。運用引き継ぎは文書の受領ではなく、責任、アクセス、手順、監視、既知課題、エスカレーションの受入です。

## 実務での使用場面

- 基盤チームから運用チームへサービスを移管する
- daily/weekly/monthlyの健全性・容量・期限を確認する
- alertをIncidentへ変換し、Runbookと責任分界に沿って対応する
- 設定変更、証明書更新、cluster updateを安全に実施する
- backupとrestoreを運用し、RPO/RTOを継続評価する
- 問題管理と改善によって監視・手順・設計を更新する

## 入力

- 承認済み設計、構築差異、試験結果、移行判定
- 構成情報、credential保管場所、連絡先、RACI、保守窓
- alert catalog、SLI/SLO、log保持、Runbook、escalation条件
- backup対象、schedule、retention、暗号化、restore手順
- version lifecycle、update graph、Operator/CSI/Virtualization互換性
- 既知課題、残存リスク、例外、次回見直し期限

## 判断

### 運用状態

「稼働中」だけでは状態を表せません。cluster health、利用者影響、冗長性、capacity、backup、security、変更有無を組み合わせ、Normal／Degraded／Incident／Maintenanceなどの状態と判断権限を定義します。

### 観測と対応

| 運用領域 | 観測するもの | 判断・対応 |
| --- | --- | --- |
| Cluster | ClusterOperator、node、MCP、API | 影響、劣化、変更停止、escalation |
| Capacity | CPU、memory、disk、PV、pod、IP | trend、threshold、増強lead time |
| Network | DNS、LB、Ingress、egress、packet loss | 内外境界、依存team、冗長性 |
| Storage | CSI、PVC、latency、capacity、snapshot | workload影響、failover、vendor escalation |
| Security | auth failure、audit、certificate、vulnerability | 封じ込め、期限、例外、更新 |
| Data protection | backup job、repository、restore test | RPO/RTO、保護gap、再試験 |
| Update | available update、risk、compatibility | canary、保守窓、停止条件 |

アラートは「閾値を超えた」だけで完結しません。利用者影響、重大度、通知先、初動時間、Runbook、抑止・重複排除、close条件を設計し、false positiveや無対応alertを改善します。

### 変更と更新

OpenShift updateは通常の定常作業に見えても、cluster全体、node再起動、Operator、API互換性、workload PDB、容量、保守時間へ影響します。release notes、known issues、update path、conditional update、Operator/CSI/Virtualization互換性を確認し、事前health、backup、監視強化、停止条件、事後試験を変更計画へ含めます。

## 成果物例の読み方

[監視・ログ・運用設計書](../projects/enterprise-openshift-platform/docs/10-observability-operations-design.md)で、運用目標、監視architecture、alert catalog、SLI/SLO、log、Incident、定常作業、更新、Runbook一覧を確認します。

[運用引き継ぎ書](../projects/enterprise-openshift-platform/docs/20-operations-handover.md)では、引き渡しpackage、責任境界、access、運用状態、定常作業、保護領域、Runbook、escalation package、known gaps、受入条件を読みます。空欄の連絡先や未試験Runbookがあれば、運用受入の条件として扱います。

管理文書は運用開始後も継続使用します。

- [課題・リスク管理台帳](../projects/enterprise-openshift-platform/docs/21-issue-risk-register.md)：障害ではない懸念、既知問題、残存リスク
- [変更管理台帳](../projects/enterprise-openshift-platform/docs/22-change-register.md)：通常・緊急変更の理由、影響、承認、結果
- [ADR](../projects/enterprise-openshift-platform/docs/23-architecture-decisions.md)：将来も説明が必要な技術判断

## 他文書とのつながり

- 要求で合意した可用性、RPO/RTO、security、保守条件を運用指標へ変換する
- 試験結果を正常時baselineとし、未実施・例外はknown gapとして引き継ぐ
- Incidentから問題管理、恒久対策、設計・試験・Runbook改訂へ戻す
- 変更後は構成情報、パラメータ、証跡、監視、backup、手順を更新する
- updateで製品版が変われば一次資料確認記録と互換性情報を更新する

## 関連サービスの運用境界

- **ROSA／ARO**：managed serviceでも責任が消えるのではなく、providerと利用者の分担が変わる
- **Kong**：Gateway policy、route、certificate、plugin、API監視の運用をcluster運用と接続する
- **Sysdig**：agent/collector coverage、policy、alert、脆弱性、証跡保持の責任を接続する
- **OpenShift AI**：model、data、notebook、accelerator、pipeline、権限、監査の運用を基盤境界と分ける
- **Disconnected**：mirror registry、release/operator catalog更新、署名・digest、同期障害を追加管理する

この架空案件では、これらの一部は設計上の将来候補であり、導入・動作確認済みとは扱いません。

## レビューで指摘されやすい点

- 製品画面やCLI一覧を運用設計とし、判断条件・責任・対応時間がない
- monitoring platform自体の容量、可用性、保持、通知経路を監視しない
- backup成功だけを確認し、restore testとRPO/RTOを測定しない
- emergency changeを事後記録せず、構成baselineと乖離する
- update前の互換性、事前health、停止条件、事後試験がない
- certificate、license、capacity、EOSLを期限前に検知できない
- 未実施試験や既知課題を運用teamへ明示しない

## 公式一次資料

- [Monitoring stack for Red Hat OpenShift 4.22](https://docs.redhat.com/en/documentation/monitoring_stack_for_red_hat_openshift/4.22/)
- [OpenShift 4.22 Updating clusters](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/updating_clusters/)
- [OpenShift 4.22 Backup and restore](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/backup_and_restore/)
- [OpenShift Container Platform Life Cycle Policy](https://access.redhat.com/support/policy/updates/openshift)

補足は[監視・ログ・バックアップ](../reference/technical/monitoring-logging-backup.md)、[Kong](../reference/technical/kong-api-ai-gateway.md)、[Sysdig](../reference/technical/sysdig-container-security.md)、[AIガバナンス](../reference/technical/ai-governance.md)を参照してください。

## 次に読む章

[11. 実務模擬プロジェクト](11-reference-project.md)へ進みます。
