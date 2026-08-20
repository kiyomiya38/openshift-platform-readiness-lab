# インシデント記録

## 基本情報

| 項目 | 記入内容 |
| --- | --- |
| Incident ID | `<INC-ID>` |
| 状態 | `<Open / Monitoring / Resolved / Closed>` |
| 重大度 | `<SEV1–SEV4。組織基準を参照>` |
| 検知日時 | `<YYYY-MM-DD hh:mm TZ>` |
| 復旧日時 | `未復旧／<date time>` |
| 対象環境・version | `<environment / versions>` |
| Incident commander | `<role>` |
| 技術担当／連絡担当 | `<roles>` |

## 申告と事実

- 申告された症状：`<利用者またはalertの表現>`
- 確認できた事実：`<誰が・どこから・何へ・いつ・どの結果>`
- 影響範囲：`<user/service/data/region>`
- 影響していない範囲：`<confirmed unaffected scope>`
- 直前変更：`<change IDs / none confirmed>`
- 正常時baseline：`<test/evidence IDs>`

## Timeline

| 時刻 | 事実・操作 | 実施者 | 結果・証跡ID | 判断 |
| --- | --- | --- | --- | --- |
| `<time TZ>` | `<observation/action>` | `<role>` | `<result/evidence>` | `<next step>` |

## 仮説と検証

| 仮説ID | 仮説 | 予測される観測 | 確認方法 | 結果 | 状態 |
| --- | --- | --- | --- | --- | --- |
| `<H-001>` | `<hypothesis>` | `<expected observation>` | `<safe check>` | `未確認` | `Open` |

## 影響抑制・復旧

- 暫定対応：`<approved mitigation>`
- 変更・承認ID：`<ID>`
- 停止条件：`<condition>`
- 復旧確認：`<user path / cluster / monitoring / data>`
- 残存影響：`<known impact>`

## 原因と改善

- 直接原因：`未確定`
- 寄与要因：`未確定`
- 検知・対応上のgap：`未確定`
- 根拠証跡：`<IDs>`

| 改善ID | 恒久対策 | Owner | 期限 | 対象文書・設定・試験 | 状態 |
| --- | --- | --- | --- | --- | --- |
| `<ACT-001>` | `<corrective/preventive action>` | `<role>` | `<date>` | `<paths/IDs>` | `Open` |

## Close条件

- 利用者影響の解消確認
- 監視状態の正常化と再発監視期間の完了
- 原因または未確定理由と残存リスクの承認
- 恒久対策のOwner・期限・追跡先確定
- 設計、Runbook、alert、試験、変更記録への反映
- 公開用記録からSecret・顧客固有情報を除去

Close承認：`未実施`
