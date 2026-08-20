# 00. はじめに

## 目的

この章では、架空の実務成果物を使ってOpenShift基盤案件を学ぶ方法と、成果物に書かれた「事実・判断・予定・未確定」の区別を確認します。対象製品はOpenShift Container Platform 4.22ですが、実案件では採用するz-streamと周辺製品の互換性を着手時点で再確認します。

## 実務での使用場面

新規参画者は、コマンドより先に案件の目的、対象範囲、責任分界、承認状態を把握します。構築担当であっても、要求や停止条件を知らなければ「設定できるが、採用してよいか判断できない」状態になるためです。

本教材では次の三層を分けて読みます。

| 層 | 役割 | 主な場所 |
| --- | --- | --- |
| 学習ガイド | 工程と成果物の関係を解説する | この`docs/` |
| 架空成果物 | 一つの案件で使う文書・設定例 | [`projects/enterprise-openshift-platform/`](../projects/enterprise-openshift-platform/README.md) |
| 技術参考 | 製品・基盤技術を補足する | [`reference/technical/`](../reference/technical/) |

## 入力

- 学習者の前提：インフラ経験あり、OpenShift実務は未経験
- [共通シナリオ](../projects/enterprise-openshift-platform/SCENARIO.md)
- [プロジェクト憲章](../projects/enterprise-openshift-platform/docs/00-project-charter.md)
- OpenShift 4.22の公式ドキュメント

## 判断

成果物の記述を、次の状態に分類して読んでください。

| 状態 | 意味 | 読むときの扱い |
| --- | --- | --- |
| 架空の確定値 | シナリオ内では承認済みとみなす例示 | 文書間で一致するか確認する |
| 方針案 | レビュー対象の設計判断 | 根拠、代替案、影響を確認する |
| TBD | 情報または承認が不足 | 解決責任者、期限、停止条件を確認する |
| 期待結果 | 試験前に定義した合格条件 | 実測結果と混同しない |
| `NOT RUN` | 実行していない | 合格や動作確認済みと表現しない |

## 成果物例の読み方

最初に[共通シナリオ](../projects/enterprise-openshift-platform/SCENARIO.md)を開き、案件目的、技術方式、ネットワーク値、ノード、未確定事項を確認します。次に[プロジェクト憲章](../projects/enterprise-openshift-platform/docs/00-project-charter.md)で、なぜ案件を開始し、何を成功とし、誰が判断するかを読みます。

各成果物は、次の順で読むと判断の流れを追いやすくなります。

1. 文書管理情報と状態
2. 目的と対象範囲
3. 入力となる要求・前提
4. 採用した判断と根拠
5. TBD、リスク、停止条件
6. 次工程への引き渡し先
7. 承認・変更履歴

設定例に値が書かれていても、値だけを覚えないでください。「どの要求または設計から来た値か」「誰が払い出す値か」「変更時にどの文書へ影響するか」を追います。

## 他文書とのつながり

```text
シナリオ／憲章
  → 要件・制約・責任分界
  → 基本設計
  → 詳細設計・パラメータ
  → 構築手順・実装資料
  → 試験仕様・試験結果
  → 移行・運用・変更管理
```

この流れをつなぐ索引が[要件トレーサビリティマトリクス](../projects/enterprise-openshift-platform/docs/04-requirements-traceability.md)です。工程が変わるたびに戻って確認します。

## レビューで指摘されやすい点

- 架空の前提を実在環境の事実のように書く
- 製品の一般仕様と、その案件で採用した設計を区別しない
- 方針案、承認済み、実施済みを同じ表現で扱う
- TBDに所有者、期限、解消条件、工程への影響がない
- 試験の期待結果を実測結果として扱う
- バージョンを`4.x`だけで済ませ、確認した資料の版を残さない

## 公式一次資料

- [OpenShift Container Platform 4.22 documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/)
- [OpenShift Container Platform Life Cycle Policy](https://access.redhat.com/support/policy/updates/openshift)
- [OpenShift Container Platform Tested Integrations](https://access.redhat.com/articles/4763741)

製品仕様は公式資料を優先し、本リポジトリの例と矛盾するときは差異を記録して成果物を修正します。記録様式は[一次資料確認記録](../evidence/source-verification-record.md)を参照してください。

## 次に読む章

[01. 実務案件の全体像](01-project-overview.md)へ進みます。
