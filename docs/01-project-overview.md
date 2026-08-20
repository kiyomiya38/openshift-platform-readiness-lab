# 01. 実務案件の全体像

## 目的

OpenShift基盤導入を、製品インストールだけでなく、要求整理から運用引き継ぎまでのプロジェクトとして理解します。各工程の開始条件、主要判断、成果物、承認ゲートを先に把握すると、個々の文書が必要な理由を説明できます。

## 実務での使用場面

- 新規参画時に案件全体と自分の担当境界を把握する
- 工程計画、成果物一覧、レビュー計画を作成する
- 設計変更が後工程のどこへ波及するか判断する
- 構築・試験開始前のGo/No-Goを判断する
- 運用担当へ何を引き渡すか合意する

## 入力

- [共通シナリオ](../projects/enterprise-openshift-platform/SCENARIO.md)
- [プロジェクト憲章](../projects/enterprise-openshift-platform/docs/00-project-charter.md)
- 契約上のスコープ、組織標準、サービスレベル、既存環境情報
- 製品サポート条件、ハードウェア・ネットワーク・ストレージ制約

## 判断

| 工程 | 主な判断 | 架空成果物例 | 次工程への受け渡し |
| --- | --- | --- | --- |
| 立ち上げ | 目的、成功条件、対象、意思決定者 | [プロジェクト憲章](../projects/enterprise-openshift-platform/docs/00-project-charter.md) | 承認された目的と範囲 |
| 要求整理 | 必要機能、非機能目標、制約、責任 | [要件定義書](../projects/enterprise-openshift-platform/docs/01-requirements.md)、[前提・制約](../projects/enterprise-openshift-platform/docs/02-assumptions-constraints.md)、[責任分界](../projects/enterprise-openshift-platform/docs/03-scope-responsibility.md) | 識別子付き要求とTBD |
| 基本設計 | トポロジー、可用性、セキュリティ、運用方式 | [基本設計書](../projects/enterprise-openshift-platform/docs/05-basic-design.md)と分野別設計 | 採用方式と設計根拠 |
| 詳細設計 | 実装値、設定単位、依存関係 | [詳細設計書](../projects/enterprise-openshift-platform/docs/13-detailed-design.md)、[パラメータシート](../projects/enterprise-openshift-platform/docs/14-parameter-sheet.md) | 承認された入力値 |
| 構築 | 手順、確認点、停止・復旧方法 | [構築手順書](../projects/enterprise-openshift-platform/docs/15-build-procedure.md)、[`install/`](../projects/enterprise-openshift-platform/install/README.md)、[`ansible/`](../projects/enterprise-openshift-platform/ansible/README.md) | 構成、作業記録、差異 |
| 試験 | 要件をどの条件・証跡で検証するか | [試験仕様書](../projects/enterprise-openshift-platform/docs/16-test-specification.md)、[試験結果報告書](../projects/enterprise-openshift-platform/docs/17-test-results.md) | 判定、未解決不具合、証跡 |
| 移行 | Wave、停止時間、Go/No-Go、切り戻し | [移行計画書](../projects/enterprise-openshift-platform/docs/18-migration-plan.md)、[切り戻し計画書](../projects/enterprise-openshift-platform/docs/19-rollback-plan.md) | 移行判定と残課題 |
| 運用移管 | 監視、定常作業、変更、障害対応 | [運用引き継ぎ書](../projects/enterprise-openshift-platform/docs/20-operations-handover.md) | 運用受入と既知の制約 |

工程は一方向にしか進まないわけではありません。試験で設計不備が判明すれば、要求、設計、実装、試験ケースを変更管理の下で更新します。

## 成果物例の読み方

最初に[プロジェクトREADME](../projects/enterprise-openshift-platform/README.md)の文書一覧を見てから、[要件トレーサビリティ](../projects/enterprise-openshift-platform/docs/04-requirements-traceability.md)を開きます。一つの要件IDについて、設計文書、試験ID、現在状態が横につながっていることを確認してください。

次に管理文書を確認します。

- [課題・リスク管理台帳](../projects/enterprise-openshift-platform/docs/21-issue-risk-register.md)：現在の不確実性と対応責任
- [変更管理台帳](../projects/enterprise-openshift-platform/docs/22-change-register.md)：基準化後の変更理由、影響、承認
- [ADR](../projects/enterprise-openshift-platform/docs/23-architecture-decisions.md)：重要な技術判断と代替案

これらは最後に作る付録ではありません。全工程で更新し、設計・構築・試験の判断を再現できるようにする文書です。

## 役割と承認

まず[役割定義表](../projects/enterprise-openshift-platform/docs/03-scope-responsibility.md#role-definitions)で、PM、基盤責任者、Platform、Network/Infra、Hardware、Storage、Security、Application、Operations、Product ownerの責任範囲を確認します。役割名は職種や人数ではなく、案件上の責任の単位です。

その上で、RACIの`R`（実行責任）と`A`（最終説明・承認責任）を混同しないことが重要です。たとえば基盤エンジニアがDNSレコード案を作っても、企業DNSへの登録承認や実作業は別チームである場合があります。[責任分界図](../projects/enterprise-openshift-platform/diagrams/responsibility-boundary.mmd)と[スコープ・責任分界書](../projects/enterprise-openshift-platform/docs/03-scope-responsibility.md)を対で確認します。

## 他文書とのつながり

- 要件IDは基本設計の判断と試験IDへつながる
- 詳細設計の値はパラメータシートと実装ファイルへつながる
- 構築中の差異は課題または変更として管理される
- 試験不合格は不具合、リスク、変更、再試験へつながる
- 移行後の既知課題は運用引き継ぎへ残る
- 重要な方式変更はADRに理由と影響を残す

## レビューで指摘されやすい点

- 工程名だけがあり、開始条件・完了条件・承認者がない
- 成果物の作成者はいるが、レビュー責任と承認責任が不明
- 設計、構築、試験で同じ項目の識別子がつながらない
- 「後で決める」項目がTBD台帳やリスク台帳に登録されない
- 技術的な完了と、業務・運用上の受入完了を混同する
- 変更された設定だけを直し、上流要求や試験ケースを更新しない

## 公式一次資料

- [OpenShift 4.22 Installing an on-premise cluster](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installing_on-premise/)
- [OpenShift 4.22 Updating clusters](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/updating_clusters/)
- [OpenShift 4.22 Support](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/support/)

一般的なインフラ案件の工程は[インフラ案件の進め方](../reference/technical/infra-project-process.md)、成果物体系は[設計文書ガイド](../reference/technical/design-document-guide.md)で補足します。

## 次に読む章

[02. 要求整理](02-requirements.md)へ進みます。
