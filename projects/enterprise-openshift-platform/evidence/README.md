# プロジェクト証跡・レビュー記録索引

> [!IMPORTANT]
> このディレクトリは、架空プロジェクトの成果物レビュー、静的検証、実行ログ、試験証跡、公開時の機密除去基準を管理するためのサンプルです。初期状態はDraft・未レビュー・未承認で、実環境の構築、試験、移行、復旧はすべて`Not Run`です。

## 1. プロジェクト状態

| 区分 | 状態 | 意味 |
| --- | --- | --- |
| 成果物セット | Draft | 架空条件に基づくサンプルが存在する |
| 技術レビュー | Not Reviewed | 指定された観点と役割によるレビュー記録がない |
| 正式承認 | Not Approved | 意思決定者の承認記録がない |
| 静的検証 | Partial Pass | link、Markdown構造、YAML、Kustomize render、試験ID、公開patternを検査。Mermaid、Ansible、全parameter整合はNot Run |
| Tabletop review | Not Run | 構築・移行・ロールバックの机上確認を実施していない |
| OpenShift構築 | Not Run | 承認済み検証環境がない |
| 試験72件 | `NOT RUN` | actual resultと試験証跡がない |
| VM移行・ロールバック | Not Run | VMware sourceとOpenShift destinationがない |

成果物の存在、期待結果、設定例、サンプル出力を`Reviewed`、`Approved`、`Pass`へ読み替えません。

## 2. 証跡ファイル

| ファイル | 用途 | 初期状態 |
| --- | --- | --- |
| [成果物レビュー記録](artifact-review-record.md) | 文書・設定・図のレビュー状態、指摘、承認を管理する | Not Reviewed |
| [設計レビュー実施記録](design-review-log.md) | review meeting、論点、判断、actionを時系列で残す | No Records |
| [検証記録](verification-record.md) | link、YAML、Ansible、Mermaid、parameter等の静的検証を記録する | Partial Pass |
| [実行ログ](execution-log.md) | 構築、変更、移行、復旧など実環境操作を記録する | Not Run |
| [試験証跡索引](test-evidence-index.md) | 72試験の結果行と保管証跡を対応付ける | `NOT RUN` |
| [公開・機密除去基準](publication-safety.md) | 一般公開時に保持できる値と除去対象を定める | Draft |

## 3. 状態定義

| 状態 | 定義 |
| --- | --- |
| Draft | 作成中または正式レビュー前 |
| Not Reviewed | 指定reviewerが内容と根拠を確認していない |
| Reviewed | 日付、対象revision、観点、指摘、判断、未解決事項を記録済み |
| Not Approved | 意思決定者による承認がない |
| Approved | 承認者、日付、scope、条件を記録済み |
| Not Run / `NOT RUN` | 操作・検証・試験を実行していない |
| Blocked | 前提不成立により判定へ進めない |
| Fail | 実行したがexpected resultを満たさない |
| Pass | 承認済み条件で実行し、actual resultと証跡がexpected resultを満たす |

## 4. 証跡登録原則

1. 日時、timezone、実施役割、対象環境、変更IDを記録する。
2. 成果物revision、製品版、tool版、実際のcommandまたはactionを記録する。
3. expected resultとactual resultを分離する。
4. failure、warning、deviation、未確認事項を省略しない。
5. 証跡の場所、ファイル名、checksum、機密除去の有無を記録する。
6. Secret、token、password、private key、kubeconfig、内部情報を保存しない。
7. 証跡が存在しない場合は`Not Run`または`Unknown`を維持する。

## 5. 成果物の場所

- 要求・設計・手順・試験・移行・運用: [`../docs/`](../docs/)
- Installer入力と周辺設定: [`../install/`](../install/)
- Ansible: [`../ansible/`](../ansible/)
- OpenShift Manifest: [`../manifests/`](../manifests/)
- 構成図: [`../diagrams/`](../diagrams/)
- 共通条件: [SCENARIO](../SCENARIO.md)

## 6. 証跡の保管境界

この公開リポジトリには、機密除去済みの索引、サンプル様式、公開可能な検証結果だけを置きます。実環境のraw log、support bundle、screenshot、設定backup、credentialを含む証跡は、アクセス制御と保持期限を設定した組織内保管先で管理します。
