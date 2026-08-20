# 静的検証記録

## 検証情報

| 項目 | 記入内容 |
| --- | --- |
| 検証ID | `<VAL-ID>` |
| 対象revision | `<commit / artifact digest>` |
| 実施環境 | `<OS / tool versions>` |
| 実施日時 | `未実施` |
| 実施者 | `<role or anonymized ID>` |
| 総合状態 | `NOT RUN` |

## 検証一覧

| ID | 対象 | 方法・tool/version | 期待結果 | 実結果 | 証跡ID | 判定 |
| --- | --- | --- | --- | --- | --- | --- |
| `<VAL-001>` | `<YAML / link / template / Mermaid>` | `<command / parser>` | `<判定条件>` | `未実施` | `—` | `NOT RUN` |

## 検証境界

静的検証で確認できるのは、構文、schemaの一部、参照整合、template rendering、差分などです。次は静的検証だけでは確認できません。

- 実clusterでのadmission、Operator reconciliation、API互換性
- 実networkのDNS、NTP、Proxy、Firewall、LB疎通
- hardware、firmware、driver、storage性能・障害動作
- serviceの実起動、冗長切替、security制御、RPO/RTO
- workloadの業務機能とend-to-end通信

## 差異・除外

| ID | 対象 | 理由 | 影響 | 後続確認 |
| --- | --- | --- | --- | --- |
| `—` | `未実施` | `—` | `—` | `<test ID>` |

## 結論

- 確認できた範囲：`未実施`
- 確認できない範囲：`実機・製品APIによる確認が必要`
- 後続試験：`<test IDs>`
- 承認：`未実施`
