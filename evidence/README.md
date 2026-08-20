# 実務記録様式

このディレクトリは、OpenShift基盤案件で発生する作業、レビュー、技術根拠確認、静的検証、試験証跡、Incidentを記録するための汎用様式です。案件ごとの事実、判断、証跡を再現可能に残すために使用します。

## 記録様式

| ファイル | 記録対象 | 実務上の目的 |
| --- | --- | --- |
| [work-record.md](work-record.md) | 構築・変更・確認作業 | 実施事実、承認、結果、差異を再現可能にする |
| [design-review-record.md](design-review-record.md) | 設計・手順・試験文書のレビュー | 指摘、判断、修正、承認を追跡する |
| [source-verification-record.md](source-verification-record.md) | 公式一次資料とsupport条件の確認 | 技術判断の根拠、version、確認日を残す |
| [static-validation-record.md](static-validation-record.md) | YAML、link、template、syntax等の静的検証 | 実機試験と区別して検証範囲を示す |
| [test-evidence-index.md](test-evidence-index.md) | 試験IDと証跡 | 結果、証跡、環境、不具合を追跡する |
| [incident-record.md](incident-record.md) | 障害・劣化・誤検知 | 事実、timeline、仮説、操作、復旧、改善を分離する |

架空案件固有の記録例は[`projects/enterprise-openshift-platform/evidence/`](../projects/enterprise-openshift-platform/evidence/README.md)にあります。本ディレクトリの様式と案件固有記録を混在させないでください。

## 共通ルール

1. 予定、期待値、静的検証、実行結果を区別する。
2. 実施していない項目は`未実施`または`NOT RUN`とする。
3. 日時にはtimezoneを含め、対象cluster、version、revisionを記録する。
4. 操作・確認・承認の担当を分け、代理承認を推測で記録しない。
5. 失敗や差異を消さず、課題・変更・不具合IDへ接続する。
6. Secret、token、password、private key、個人情報、顧客固有情報を記録しない。
7. 公開前にhost、IP、URL、account、logを再確認し、必要な箇所を匿名化する。
8. 証跡原本はアクセス制御された保管先に置き、この索引には識別子と保管場所だけを書く。

## 状態の定義

| 状態 | 意味 |
| --- | --- |
| `DRAFT` | 作成中で未レビュー |
| `IN REVIEW` | レビュー中 |
| `APPROVED` | 指定された承認者が承認済み |
| `NOT RUN` | 実行していない |
| `PASS` | 定義済み期待結果を満たし、証跡あり |
| `FAIL` | 期待結果を満たさない |
| `BLOCKED` | 前提不足により実行・判定不能 |
| `N/A` | 承認された理由により対象外 |

`APPROVED`や`PASS`は、署名欄や証跡が空のまま使用しません。
