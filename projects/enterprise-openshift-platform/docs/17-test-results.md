# 17. OpenShift基盤 試験結果報告書

## 文書管理・総合判定

| 項目 | 内容 |
| --- | --- |
| 文書ID | `TEST-RESULT-OCP-001` |
| 対象仕様 | [16-test-specification.md](16-test-specification.md) 0.1 |
| 版／成果物状態 | 0.1／Draft・未レビュー・未承認 |
| 対象OpenShift／構成 | 4.22.z未確定／commit未設定 |
| 試験環境 | なし |
| 実施期間／実施者 | ／ |
| 総合判定 | **NOT RUN（判定不可）** |

> [!IMPORTANT]
> 実機環境がなく、以下のケースは1件も実行していません。期待結果をactualへ転記せず、実績と証跡の欄を空欄にしています。

| 仕様ケース数 | Pass | Fail | Blocked | NOT RUN |
| ---: | ---: | ---: | ---: | ---: |
| 72 | 0 | 0 | 0 | 72 |

## 1. ケース別結果

| 試験ID | 試験名 | 結果 | 実績（actual） | 証跡 | 課題ID |
| --- | --- | --- | --- | --- | --- |
| TST-INS-001 | Agent ISO生成 | NOT RUN | | | |
| TST-INS-002 | install-complete | NOT RUN | | | |
| TST-INS-003 | ClusterVersion/Operator | NOT RUN | | | |
| TST-INS-004 | Console接続 | NOT RUN | | | |
| TST-REG-001 | Internal Image Registry永続化/push-pull | NOT RUN | | | |
| TST-NOD-001 | 6ノード構成 | NOT RUN | | | |
| TST-NOD-002 | RHCOS確認 | NOT RUN | | | |
| TST-DNS-001 | API正引き | NOT RUN | | | |
| TST-DNS-002 | apps wildcard | NOT RUN | | | |
| TST-DNS-003 | 逆引き | NOT RUN | | | |
| TST-DNS-004 | DNS片系障害 | NOT RUN | | | |
| TST-NTP-001 | NTP片系障害 | NOT RUN | | | |
| TST-LB-001 | API LB/readyz | NOT RUN | | | |
| TST-LB-002 | MCS到達制御 | NOT RUN | | | |
| TST-LB-003 | Ingress LB | NOT RUN | | | |
| TST-LB-004 | LBフェイルオーバー | NOT RUN | | | |
| TST-NET-001 | Cluster network設定 | NOT RUN | | | |
| TST-NET-002 | Pod/Service通信 | NOT RUN | | | |
| TST-NET-003 | NetworkPolicy否定 | NOT RUN | | | |
| TST-NET-004 | Route | NOT RUN | | | |
| TST-PRX-001 | Proxy設定 | NOT RUN | | | |
| TST-PRX-002 | 外部接続 | NOT RUN | | | |
| TST-PRX-003 | noProxy | NOT RUN | | | |
| TST-PRX-004 | Proxy障害/復旧 | NOT RUN | | | |
| TST-IAM-001 | IdP正常認証 | NOT RUN | | | |
| TST-IAM-002 | IdP異常/無効ID | NOT RUN | | | |
| TST-IAM-003 | RBAC許可 | NOT RUN | | | |
| TST-IAM-004 | RBAC拒否 | NOT RUN | | | |
| TST-SEC-001 | SCC/Pod admission | NOT RUN | | | |
| TST-SEC-002 | Secret scan | NOT RUN | | | |
| TST-SEC-003 | 証明書 | NOT RUN | | | |
| TST-SEC-004 | ServiceAccount最小権限 | NOT RUN | | | |
| TST-AUD-001 | 監査追跡 | NOT RUN | | | |
| TST-STG-001 | 動的PV | NOT RUN | | | |
| TST-STG-002 | RWO再attach | NOT RUN | | | |
| TST-STG-003 | RWX | NOT RUN | | | |
| TST-STG-004 | VolumeSnapshot復元 | NOT RUN | | | |
| TST-STG-005 | PVC拡張 | NOT RUN | | | |
| TST-STG-006 | Storage性能 | NOT RUN | | | |
| TST-MON-001 | 監視target/rule | NOT RUN | | | |
| TST-MON-002 | Alert発火/通知 | NOT RUN | | | |
| TST-MON-003 | Alert復旧 | NOT RUN | | | |
| TST-MON-004 | 通知経路障害 | NOT RUN | | | |
| TST-LOG-001 | infrastructure log | NOT RUN | | | |
| TST-LOG-002 | application log | NOT RUN | | | |
| TST-LOG-003 | audit log | NOT RUN | | | |
| TST-LOG-004 | forward障害 | NOT RUN | | | |
| TST-BKP-001 | etcd backup | NOT RUN | | | |
| TST-BKP-002 | resource backup | NOT RUN | | | |
| TST-BKP-003 | PV backup | NOT RUN | | | |
| TST-BKP-004 | external DB backup | NOT RUN | | | |
| TST-RST-001 | etcd restore | NOT RUN | | | |
| TST-RST-002 | resource restore | NOT RUN | | | |
| TST-RST-003 | PV/DB整合復元 | NOT RUN | | | |
| TST-RST-004 | RTO/RPO総合 | NOT RUN | | | |
| TST-HA-001 | worker障害 | NOT RUN | | | |
| TST-HA-002 | control plane障害 | NOT RUN | | | |
| TST-HA-003 | Ingress backend障害 | NOT RUN | | | |
| TST-SLO-001 | 月間SLI | NOT RUN | | | |
| TST-CAP-001 | node障害時容量 | NOT RUN | | | |
| TST-CAP-002 | PV容量予測 | NOT RUN | | | |
| TST-PER-001 | 性能 | NOT RUN | | | |
| TST-OPS-001 | 障害運用 | NOT RUN | | | |
| TST-OPS-002 | Runbook再現性 | NOT RUN | | | |
| TST-UPG-001 | 更新 | NOT RUN | | | |
| TST-CHG-001 | 変更管理 | NOT RUN | | | |
| REV-RACI-001 | RACI review | NOT RUN | | | |
| REV-QA-001 | 未実施表示QA | NOT RUN | | | |
| TST-VIR-001 | Virtualization前提 | NOT RUN | | | |
| TST-MTV-001 | MTV test migration | NOT RUN | | | |
| TST-MTV-002 | MTV cutover計時 | NOT RUN | | | |
| TST-MTV-003 | MTV切り戻し | NOT RUN | | | |

`KONG-T01〜07` と `SYSDIG-T01〜07` は [連携設計](12-kong-sysdig-integration-design.md)内のlocal planned-check IDであり、現行72試験のresult rowではありません。製品選定とbaseline改訂前に結果を追記せず、行がないことを `Pass` または対象外判定と解釈しません。

## 2. 基盤健全性

| 確認 | コマンド／方法 | 実績 | 判定 |
| --- | --- | --- | --- |
| API target | `oc whoami --show-server` | | NOT RUN |
| Version | `oc get clusterversion` | | NOT RUN |
| ClusterOperator | `oc get clusteroperators` | | NOT RUN |
| Node | `oc get nodes` | | NOT RUN |
| Event | `oc get events -A --sort-by=.lastTimestamp` | | NOT RUN |

## 3. 性能・復旧実績

| 指標 | 目標 | 測定条件 | 実績 | 判定 |
| --- | --- | --- | --- | --- |
| Application RPO | 1時間以内 | 復元データの最終整合時点 | | NOT RUN |
| Application RTO | 4時間以内 | 障害宣言から業務再開承認 | | NOT RUN |
| 月間可用性 | 99.9% | SLI地点・除外条件TBD | | NOT RUN |
| Storage/業務性能 | 目標TBD | load model/TBD | | NOT RUN |

## 4. 差異・不具合

実行していないため、観測された不具合はありません。「不具合なし」という合格判定ではありません。

| ID | 期待 | 実績 | 影響 | 暫定対応 | 恒久対応／期限 | 状態 |
| --- | --- | --- | --- | --- | --- | --- |
| | | | | | | |

## 5. 証跡・承認

| 項目 | 内容 |
| --- | --- |
| 証跡保管先／権限 | |
| Secret/token/個人情報除去 | 未実施 |
| 保持・廃棄日 | |

| 役割 | 氏名 | 判定 | 日付 | 条件／コメント |
| --- | --- | --- | --- | --- |
| 試験責任者 | | 未承認 | | |
| Platform責任者 | | 未承認 | | |
| 業務責任者 | | 未承認 | | |

## 6. 次回更新条件

実環境を確保し、試験仕様をレビュー・承認した後だけ結果を更新します。各行は実施時刻、観測事実、原本証跡、課題IDを用いて個別に変更し、まとめてPassへ変更しません。
