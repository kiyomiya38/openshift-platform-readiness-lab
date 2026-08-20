# 07. 試験

## 目的

要件と設計が実装されたことを、再現可能な条件、操作、観測点、期待結果、証跡で判定します。試験仕様と実行結果を分け、未実施、合格、不合格、対象外、保留を事実に基づいて管理します。

## 実務での使用場面

- 構築前に要件・設計から試験ケースを設計する
- 構築後の基盤受入、異常系、復元、運用試験を実施する
- 不合格、差異、再試験、残存リスクを管理する
- 移行Go/No-Goと運用引き継ぎの判断材料を提示する
- 実行結果と証跡を第三者が追跡できる状態にする

## 入力

- 要件IDと受入基準
- 基本設計、詳細設計、パラメータ、構築差異
- 対象環境、実施期間、役割、変更・障害試験の承認
- 正常時baseline、監視・ログ・バックアップ仕様
- 証跡の保管、マスキング、命名、保持ルール

## 判断

試験ケースには、少なくとも次を定義します。

| 項目 | 内容 |
| --- | --- |
| 試験ID | 要求・設計・結果・不具合をつなぐ一意ID |
| 目的 | 何を検証するか |
| 前提条件 | 構成、データ、権限、依存サービス、承認 |
| 操作 | 再現可能な手順。変更対象と復旧操作を含む |
| 観測点 | CLI、API、UI、監視、ログ、外部サービス |
| 期待結果 | 判定可能な値・状態・時間・通信可否 |
| 証跡 | ファイル名、取得時刻、取得者、保存先 |
| 復旧 | 事後確認、設定復帰、データ清掃 |

成功系だけでは、可用性、deny制御、復元性、運用性を検証できません。異常系試験は影響範囲、停止条件、実施権限、復旧手順をレビューし、安全な環境と時間帯でのみ行います。

## 成果物例の読み方

### 試験仕様書

[試験仕様書](../projects/enterprise-openshift-platform/docs/16-test-specification.md)は、開始・中止基準、共通事前事後確認、分野別ケース、障害試験の安全手順、証跡管理、完了条件から構成されています。ケース数だけでなく、次を読みます。

- 各試験IDが[要件トレーサビリティ](../projects/enterprise-openshift-platform/docs/04-requirements-traceability.md)へ接続されているか
- DNS、NTP、Proxy、LB、Storageなど外部依存の正常系・異常系があるか
- 権限は「許可される操作」と「拒否される操作」の両方を確認するか
- backup成功だけでなくrestoreとRPO/RTOの観測を扱うか
- 破壊的試験に承認、影響確認、復旧、停止条件があるか

### 試験結果

[試験結果報告書](../projects/enterprise-openshift-platform/docs/17-test-results.md)は、実機がないため全ケース`NOT RUN`です。期待値が記載されていても合格ではありません。実務では、実施日時、担当者、対象version、実測、判定、証跡、差異、不具合ID、再試験を追記します。

総合判定はケースの単純な合格率だけで決めません。未実施Must要件、重大不具合、未達RPO/RTO、復旧不能、運用受入拒否があれば、明示的な例外承認なしに完了扱いにしません。

### 証跡

[試験証跡索引](../evidence/test-evidence-index.md)は、試験IDと証跡ファイルの関係を記録する汎用様式です。CLI出力には取得時刻、対象cluster、command、exit codeを付け、Secret、token、個人情報、内部URLを公開前に除去します。

## 他文書とのつながり

- 要件と設計から期待結果を作り、[トレーサビリティ](../projects/enterprise-openshift-platform/docs/04-requirements-traceability.md)を更新する
- 構築差異は試験条件へ反映し、無断で期待結果を緩和しない
- 不合格は[課題・リスク](../projects/enterprise-openshift-platform/docs/21-issue-risk-register.md)へ登録する
- 修正を要する場合は[変更管理](../projects/enterprise-openshift-platform/docs/22-change-register.md)を経て再試験する
- 受入結果と既知課題を[移行計画](../projects/enterprise-openshift-platform/docs/18-migration-plan.md)と[運用引き継ぎ](../projects/enterprise-openshift-platform/docs/20-operations-handover.md)へ渡す

## レビューで指摘されやすい点

- 試験名だけで前提、操作、観測点、定量的期待結果がない
- 仕様作成前に実施し、結果に合わせて期待値を書く
- `oc get`の一時点だけで可用性や性能を判定する
- backup job成功をrestore可能性やRPO/RTO達成と同一視する
- 外部依存サービスを試験対象外にし、end-to-end経路を確認しない
- 証跡に対象version、時刻、試験IDがない
- `NOT RUN`を空欄、合格、確認済みへ置き換える
- 不合格を根拠なく「軽微」とし、例外承認や残存リスクを残さない

## 公式一次資料

- [OpenShift 4.22 Support - gathering cluster data](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/support/gathering-cluster-data)
- [OpenShift 4.22 Backup and restore](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/backup_and_restore/)
- [OpenShift 4.22 Monitoring](https://docs.redhat.com/en/documentation/monitoring_stack_for_red_hat_openshift/4.22/)

試験文書の構造は[試験文書ガイド](../reference/technical/test-document-guide.md)、監視・バックアップの観点は[監視・ログ・バックアップ](../reference/technical/monitoring-logging-backup.md)を参照してください。

## 次に読む章

[08. 障害調査](08-troubleshooting.md)へ進みます。
