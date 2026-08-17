# 架空案件の証跡索引

> [!IMPORTANT]
> このフォルダの初期状態は **本人レビュー前・実機未実施・試験未実施** です。文書や設定例が存在することは、本人の理解、構築成功、商用経験の証明ではありません。

## 1. Overall status

| Category | State | Meaning |
| --- | --- | --- |
| AI-supported draft generation | Prepared | 架空条件から draft を作成した状態 |
| Learner artifact review | Not Started | 本人による source 照合・訂正なし |
| Learner official-source verification | Not Started | link 掲載と本人確認を区別 |
| Desk / tabletop review | Not Run | procedure、test、migration、rollback 未演習 |
| Static validation by learner | Not Run | Markdown/YAML/Ansible/Mermaid 未検証 |
| OpenShift cluster build | Not Run | 検証環境なし |
| Test / migration / rollback | Not Run | actual result なし |
| Commercial experience | Not applicable | 架空案件であり商用経験ではない |
| External submission readiness | Not Ready | 本人レビューと claim audit が必要 |

## 2. Evidence files

| File | Purpose | Initial state |
| --- | --- | --- |
| [Artifact review record](artifact-review-record.md) | 文書ごとの本人 review / correction | Not Started |
| [Verification record](verification-record.md) | static、tabletop、lab の actual result | Not Run |
| [Learning log](learning-log.md) | 本人が学んだ内容、source、疑問、次 action | Not Started |
| [Submission boundary](submission-boundary.md) | 外部説明できる事実と禁止表現 | Review Required |

## 3. Evidence level

| Level | Label | Minimum evidence |
| --- | --- | --- |
| E0 | Generated draft | artifact の存在。competence evidence ではない |
| E1 | Learner reviewed | date、exact revision、primary source、correction |
| E2 | Learner explained | 本人作成 summary / walkthrough record |
| E3 | Static / tabletop verified | tool/version/command、actual output、limitation |
| E4 | Lab verified | environment、approved change、actual Pass/Fail、sanitized evidence |
| E5 | Repeated | 別 run で再現し差異を説明 |

Project 全体の初期値は E0 です。E4/E5 でも商用経験とは別です。

## 4. Result semantics

| Result | Definition |
| --- | --- |
| Not Run | 実行していない |
| Blocked | 前提不成立で判定まで進めない |
| Fail | 実行したが expected result を満たさない |
| Pass | approved condition で actual result が expected result を満たす |
| Needs Review | 内容または source の確認が必要 |

`Blocked`、draft の期待結果、sample output を `Pass` にしません。

## 5. Evidence registration rule

1. 実施者が本人であることを明示する。
2. date/time、environment、artifact revision、tool/product version を記録する。
3. planned command ではなく actual command / action と result を記録する。
4. failure、warning、deviation、未確認を省略しない。
5. token、Secret、private key、内部/customer information を redaction する。
6. screenshot だけでなく machine-readable output または log を可能な範囲で残す。
7. AI の説明や生成 artifact を本人実施の evidence として登録しない。

## 6. Artifact locations

- Requirements / design / procedure / test / migration / operation: [`../docs/`](../docs/)
- Installer examples: [`../install/`](../install/)
- Ansible examples: [`../ansible/`](../ansible/)
- OpenShift manifests: [`../manifests/`](../manifests/)
- Architecture diagrams: [`../diagrams/`](../diagrams/)
- Scenario boundary: [SCENARIO](../SCENARIO.md)

## 7. Current allowed statement

現時点で説明できるのは、次の範囲です。

> OpenShift 基盤導入の工程を学ぶため、完全に架空の要件から設計・構築手順・試験・移行・運用文書の draft を AI 支援で準備しています。本人レビュー、公式資料との照合、実機構築と試験はまだ実施していません。この成果物は商用経験を示すものではありません。

External submission 前に [Submission boundary](submission-boundary.md) を本人と reviewer が確認します。

