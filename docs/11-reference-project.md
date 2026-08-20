# 11. 実務模擬プロジェクト

## 目的

これまで章ごとに扱った内容を、[架空エンタープライズOpenShift基盤導入プロジェクト](../projects/enterprise-openshift-platform/README.md)の一つの成果物体系として読み直します。各成果物が実務のどの時点で作られ、何を判断し、どこへ接続するかを案内する索引です。

## 実務での使用場面

- 新規案件の成果物一覧・レビュー計画を考える際の参考にする
- 新規参画時にプロジェクト文書を工程順に把握する
- 一つの要求や設定値を上流から下流まで追跡する
- 文書、設定、試験、移行、運用の不整合をレビューする
- 自組織のtemplateへ必要項目を取り込む際の比較材料にする

## 入力

- [共通シナリオ](../projects/enterprise-openshift-platform/SCENARIO.md)
- [プロジェクトのREADME](../projects/enterprise-openshift-platform/README.md)
- プロジェクト文書`00`〜`24`
- `install/`、`ansible/`、`manifests/`、`diagrams/`、プロジェクト固有`evidence/`

## 判断

この成果物群は一般公開可能な架空例であり、実際の導入結果ではありません。次の境界を保って読みます。

| 記述 | 扱い |
| --- | --- |
| 架空の値 | 文書間の整合性を学ぶための例 |
| 設計案 | 対象環境で再評価・承認が必要 |
| `.example` | Secret投入、値確定、版検証前の入力例 |
| `NOT RUN` | 実機試験なし。合格・確認済みではない |
| TBD／Open risk | 後工程開始条件または残存リスクとして管理 |
| 公式URL | 閲覧時点と対象versionを再確認する入口 |

## 成果物例の読み方

### 1. 案件を開始する

1. [プロジェクト憲章](../projects/enterprise-openshift-platform/docs/00-project-charter.md)で目的、成功条件、scope、stakeholder、Gateを確認します。
2. [要件定義](../projects/enterprise-openshift-platform/docs/01-requirements.md)、[前提・制約・TBD](../projects/enterprise-openshift-platform/docs/02-assumptions-constraints.md)、[責任分界](../projects/enterprise-openshift-platform/docs/03-scope-responsibility.md)で設計入力を確認します。
3. [トレーサビリティ](../projects/enterprise-openshift-platform/docs/04-requirements-traceability.md)を索引にします。

この段階の読みどころは、要望を実装方式へ飛ばさず、要求、制約、依存、責任、停止条件に分解している点です。

### 2. 基本設計を合意する

1. [基本設計](../projects/enterprise-openshift-platform/docs/05-basic-design.md)で全体方針を確認します。
2. [アーキテクチャ](../projects/enterprise-openshift-platform/docs/06-architecture-design.md)、[ネットワーク](../projects/enterprise-openshift-platform/docs/07-network-dns-lb-design.md)、[セキュリティ](../projects/enterprise-openshift-platform/docs/08-security-identity-design.md)、[ストレージ](../projects/enterprise-openshift-platform/docs/09-storage-backup-design.md)、[監視・運用](../projects/enterprise-openshift-platform/docs/10-observability-operations-design.md)へ分解します。
3. [構成図](../projects/enterprise-openshift-platform/diagrams/platform-architecture.mmd)と[通信図](../projects/enterprise-openshift-platform/diagrams/network-flow.mmd)を本文・tableと照合します。

全体設計と分野別設計の重複は誤りではありません。ただし、正本と詳細化先を定め、値や状態が矛盾しないことが必要です。

### 3. 第2段階と将来統合を境界化する

[Virtualization・MTV設計](../projects/enterprise-openshift-platform/docs/11-virtualization-mtv-design.md)と[Kong・Sysdig連携設計](../projects/enterprise-openshift-platform/docs/12-kong-sysdig-integration-design.md)を読みます。Phase 1のクラスタ導入前提と、Phase 2または将来候補を混ぜず、導入済み／未導入、検証済み／未検証を明示しているか確認します。

### 4. 実装へ具体化する

1. [詳細設計](../projects/enterprise-openshift-platform/docs/13-detailed-design.md)で実装単位と設定方法を確認します。
2. [パラメータシート](../projects/enterprise-openshift-platform/docs/14-parameter-sheet.md)で値の出所・状態・反映先を確認します。
3. [構築手順](../projects/enterprise-openshift-platform/docs/15-build-procedure.md)で依存順、checkpoint、停止、証跡を確認します。
4. [`install/`](../projects/enterprise-openshift-platform/install/README.md)、[`ansible/`](../projects/enterprise-openshift-platform/ansible/README.md)、[`manifests/`](../projects/enterprise-openshift-platform/manifests/README.md)で文書が機械可読入力へどう変換されたか追います。

### 5. 検証し、受け入れる

[試験仕様](../projects/enterprise-openshift-platform/docs/16-test-specification.md)と[試験結果](../projects/enterprise-openshift-platform/docs/17-test-results.md)の試験IDが一致するか確認します。現在は机上資料のため全結果`NOT RUN`です。この表現により、仕様の網羅性と実環境での実績を分離しています。

### 6. 移行し、運用へ渡す

[移行計画](../projects/enterprise-openshift-platform/docs/18-migration-plan.md)、[切り戻し計画](../projects/enterprise-openshift-platform/docs/19-rollback-plan.md)、[運用引き継ぎ](../projects/enterprise-openshift-platform/docs/20-operations-handover.md)を続けて読みます。技術手順だけでなく、判断時刻、権限、連絡、業務受入、source保持、known gapを確認します。

### 7. 全工程を統制する

[課題・リスク](../projects/enterprise-openshift-platform/docs/21-issue-risk-register.md)、[変更管理](../projects/enterprise-openshift-platform/docs/22-change-register.md)、[ADR](../projects/enterprise-openshift-platform/docs/23-architecture-decisions.md)は工程横断文書です。[成果物利用ガイド](../projects/enterprise-openshift-platform/docs/24-deliverable-usage-guide.md)は、この架空成果物だけを読む際の詳細索引として利用できます。

## 一つの判断を追跡する例

内部Image Registryの永続化を例にすると、要求のdata保護、Storage基本設計、CSI/StorageClass詳細、導入後設定、Registry試験、backup/restore、運用Runbookへ接続します。未確定storageがあればTBDとNo-Go、採用方式が決まればADR・変更、試験前なら`NOT RUN`という状態も合わせて追います。

## 他文書とのつながり

- 各工程の解説は[00〜10](00-introduction.md)へ戻って確認する
- 製品仕様で止まった場合は[12. 参考資料](12-references.md)から該当技術を開く
- 実務記録の構造は[`evidence/`](../evidence/README.md)を参照する
- 文書のひな型だけが必要な場合は[`templates/`](../templates/)を参照する

## レビューで指摘されやすい点

- 成果物例をそのまま別案件へ流用し、要求・version・責任境界を置き換えない
- 文書の存在を完成条件とし、承認状態、TBD、差異、証跡を確認しない
- 一つの変更を設定ファイルだけへ反映し、要求・設計・試験・運用へ戻さない
- 図、本文、パラメータ、実装ファイルで同じ値が不一致
- 将来候補やPhase 2を導入済みの機能として扱う
- 机上の期待結果を実行結果として扱う

## 公式一次資料

成果物中の技術判断は[OpenShift Container Platform 4.22 documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/)を起点に確認します。Virtualization、MTV、Kong、Sysdig、ROSA、ARO、OpenShift AIは、それぞれの製品公式資料と互換性情報を実施時点で確認します。

## 次に読む章

[12. 参考資料](12-references.md)で技術資料と公式一次資料の使い分けを確認します。
