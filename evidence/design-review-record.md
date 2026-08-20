# 設計レビュー記録

## レビュー情報

| 項目 | 記入内容 |
| --- | --- |
| レビューID | `<REV-ID>` |
| 対象文書 | `<path / document ID>` |
| 対象revision | `<revision or commit>` |
| レビュー種別 | `<self / peer / architecture / security / operations>` |
| 予定日時 | `<YYYY-MM-DD hh:mm TZ>` |
| 実施日時 | `未実施` |
| 作成者／レビュー者／承認者 | `<roles>` |
| 状態 | `DRAFT` |

## レビュー範囲

- 対象章・設定：`<scope>`
- 対象要求・ADR：`<IDs>`
- 対象外：`<out of scope>`
- 使用した公式根拠：`<source verification IDs>`

## 観点

| 観点 | 確認内容 | 結果 |
| --- | --- | --- |
| 完全性 | 要求、前提、例外、TBD、受入条件があるか | `未確認` |
| 整合性 | 図、本文、パラメータ、実装、試験が一致するか | `未確認` |
| 追跡性 | 要求ID、設計、試験、変更が接続するか | `未確認` |
| 実装可能性 | 値、依存、順序、権限が具体的か | `未確認` |
| 障害・復旧 | 障害mode、停止、復旧、rollbackを扱うか | `未確認` |
| 運用性 | 監視、backup、更新、責任、Runbookがあるか | `未確認` |
| Security | 最小権限、Secret、証明書、auditを扱うか | `未確認` |
| Version | 対象versionとsupport条件を確認したか | `未確認` |

## 指摘一覧

| 指摘ID | 重要度 | 箇所 | 指摘事実 | 影響 | 対応案 | Owner | 期限 | 状態 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `<REV-001>` | `<Critical/Major/Minor/Note>` | `<section>` | `<指摘>` | `<影響>` | `<対応>` | `<role>` | `<date>` | `Open` |

## 判断記録

| 判断ID | 選択肢 | 決定 | 根拠 | 影響文書 | 承認 |
| --- | --- | --- | --- | --- | --- |
| `<DEC-001>` | `<alternatives>` | `<decision>` | `<source/ADR>` | `<paths>` | `未承認` |

## 完了条件と結論

- Critical/Major指摘：`未集計`
- 未解決TBD：`未集計`
- 後続工程を停止する指摘：`未判定`
- 結論：`未レビュー`
- 承認者・日時：`未実施`
