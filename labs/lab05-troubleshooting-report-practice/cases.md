# Lab 05 ケース索引

> [!IMPORTANT]
> **状態:** 机上演習資料: 準備済み / 本人実施: 未実施 / 本人作成報告書: なし

| ケース | ケース資料 | 主な切り分け層 |
| --- | --- | --- |
| Pod 起動失敗 | [01-pod-startup-failure.md](cases/01-pod-startup-failure.md) | Pod state、ConfigMap、Application log、変更履歴 |
| Route アクセス不可 | [02-route-unreachable.md](cases/02-route-unreachable.md) | DNS、Router、Route、Service、EndpointSlice、Pod |
| PVC 未割当 | [03-pvc-pending.md](cases/03-pvc-pending.md) | PVC Event、StorageClass、CSI、backend、topology |
| Operator 異常 | [04-operator-degraded.md](cases/04-operator-degraded.md) | ClusterOperator、Subscription、CSV、Operator、Operand |
| DNS 不通 | [05-dns-failure.md](cases/05-dns-failure.md) | Pod、Namespace、NetworkPolicy、DNS Service、upstream DNS |

[README](README.md) に従い、まず本人が `templates/troubleshooting-report-template.md` へ事実、仮説、追加確認、原因、対応、復旧条件を記録します。ケース資料の架空ログは実機証跡ではありません。
