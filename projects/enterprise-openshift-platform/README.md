# Enterprise OpenShift 4.22基盤導入 実務成果物サンプル

## 1. このコンテンツの位置づけ

本コンテンツは、インフラストラクチャの実務経験がありOpenShift案件へ初めて参加するエンジニア向けに、基盤導入プロジェクトで使用される成果物例と文書間のつながりを解説する一般公開用サンプルです。

知識問題や採点課題は収録していません。架空案件の要求、設計、設定例、手順、試験、移行、運用、管理記録を、実務工程と同じ順序で読み進められるように構成しています。

| 項目 | 現在のプロジェクト状態 |
| --- | --- |
| シナリオ・成果物 | Draft |
| 技術レビュー | Not Reviewed |
| 正式承認 | Not Approved |
| OpenShiftクラスタ構築 | Not Run |
| 試験72件 | すべて `NOT RUN` |
| Virtualization / MTV導入・移行 | Not Run |
| Kong / Sysdig導入 | 対象外（連携設計のみ） |

文書が揃っていることは、設計の承認、製品互換性、構築成功、試験合格を意味しません。共通条件と公開上の境界は[共通シナリオ](SCENARIO.md)、成果物状態と証跡の扱いは[プロジェクト証跡索引](evidence/README.md)を正とします。

## 2. 架空案件の概要

- OpenShift Container Platform 4.22.zをオンプレミスのx86_64ベアメタルへ導入する。
- Control Plane 3台、Worker 3台で構成する。
- Agent-based Installerと`platform: none`を使用する。
- 外部DNS、NTP、Load Balancer、Proxy、CSIストレージと連携する。
- 第1段階はOpenShift基盤、第2段階はOpenShift VirtualizationとMTVによる代表VM 3台の移行PoCとする。
- KongとSysdigは連携点を設計するが、製品選定・導入・接続は行わない。

詳細な前提値、架空アドレス、未確定事項は[SCENARIO.md](SCENARIO.md)に集約しています。

## 3. 技術領域と適用境界

| 技術領域 | 本サンプルでの扱い | 主な参照先 |
| --- | --- | --- |
| OpenShift Container Platform 4.22 | 第1段階の構築対象。ただし実機作業はNot Run | [基本設計](docs/05-basic-design.md)、[構築手順](docs/15-build-procedure.md) |
| RHEL / Ansible | DNS・LBなど周辺RHELサーバーの設定例 | [Ansible package](ansible/README.md) |
| OpenShift Virtualization / MTV | 第2段階のPoC設計・移行計画。導入と移行はNot Run | [Virtualization・MTV設計](docs/11-virtualization-mtv-design.md) |
| Kong / Sysdig | インターフェースと責任分界の設計のみ | [Kong・Sysdig連携設計](docs/12-kong-sysdig-integration-design.md) |
| ROSA / ARO | オンプレミス方式との比較対象。構築対象外 | [基本設計](docs/05-basic-design.md) |
| OpenShift AI | 将来検討領域。今回の要件・設計・構築対象外 | [共通シナリオ](SCENARIO.md) |
| Disconnected環境 | 将来検討領域。今回はProxy経由のconnected構成 | [共通シナリオ](SCENARIO.md)、[ネットワーク設計](docs/07-network-dns-lb-design.md) |

Ansibleは周辺RHELサーバーを対象とし、RHCOSノードのパッケージ導入や直接設定変更には使用しません。RHCOSの構成変更は対象版でサポートされるMachineConfig等の方式を前提とします。

## 4. 成果物の読み方

成果物は`00`から`24`まで番号順に並んでいます。各文書では、次の関係に注目すると、実務上の役割を追いやすくなります。

```text
要求
  → 前提・制約・責任分界
    → 基本設計・方式判断
      → 詳細値・構築資材・作業手順
        → 試験仕様・結果・証跡
          → 移行・ロールバック・運用
            → 課題・変更・ADR
```

文書ごとの目的、入力、主要な判断、後続成果物、レビュー観点は[24. 成果物利用ガイド](docs/24-deliverable-usage-guide.md)で解説します。

## 5. 文書一覧

### 5.1 プロジェクト開始・要求整理

| 番号 | 成果物 | 実務での役割 |
| ---: | --- | --- |
| 00 | [プロジェクト憲章](docs/00-project-charter.md) | 目的、範囲、体制、ゲート、完了条件を定める |
| 01 | [要件定義書](docs/01-requirements.md) | 業務・機能・非機能・移行・運用要求を識別する |
| 02 | [前提条件・制約・未確定事項](docs/02-assumptions-constraints.md) | 事実、仮定、制約、TBDと解消責任を管理する |
| 03 | [スコープ・責任分界書](docs/03-scope-responsibility.md) | 対象範囲、RACI、組織間インターフェースを定める |
| 04 | [要件トレーサビリティ](docs/04-requirements-traceability.md) | 要件から設計・実装・試験・運用までを追跡する |

### 5.2 基本設計

| 番号 | 成果物 | 実務での役割 |
| ---: | --- | --- |
| 05 | [OpenShift基盤 基本設計書](docs/05-basic-design.md) | 技術方式と採用理由を定める |
| 06 | [クラスタ構成・アーキテクチャ設計書](docs/06-architecture-design.md) | 物理・論理構成、依存関係、障害領域を定める |
| 07 | [ネットワーク・DNS・LB設計書](docs/07-network-dns-lb-design.md) | アドレス、名前解決、VIP、通信、負荷分散を定める |
| 08 | [セキュリティ・認証認可設計書](docs/08-security-identity-design.md) | IdP、RBAC、SCC、Secret、証明書、監査を定める |
| 09 | [ストレージ・バックアップ・復旧設計書](docs/09-storage-backup-design.md) | CSI、Registry、保護対象、RPO/RTO、復旧方式を定める |
| 10 | [監視・ログ・運用設計書](docs/10-observability-operations-design.md) | signal、alert、通知、保守、runbookの方針を定める |
| 11 | [Virtualization・MTV設計書](docs/11-virtualization-mtv-design.md) | VM実行基盤、移行方式、PoC判定、責任分界を定める |
| 12 | [Kong・Sysdig連携設計書](docs/12-kong-sysdig-integration-design.md) | 周辺製品との接続点、権限、通信、選定Gateを定める |

### 5.3 詳細設計・構築・試験

| 番号 | 成果物 | 実務での役割 |
| ---: | --- | --- |
| 13 | [詳細設計書](docs/13-detailed-design.md) | 基本設計を実装可能な単位へ具体化する |
| 14 | [パラメータシート](docs/14-parameter-sheet.md) | ホスト、IP、CIDR、ポート、設定値の正本を管理する |
| 15 | [構築手順書](docs/15-build-procedure.md) | 事前確認、変更、確認、停止条件、戻し方を定める |
| 16 | [試験仕様書](docs/16-test-specification.md) | 正常・異常・復旧の条件、手順、期待値、証跡を定める |
| 17 | [試験結果書](docs/17-test-results.md) | 72試験のactual resultと証跡を記録する。現在は全件`NOT RUN` |

### 5.4 移行・運用・プロジェクト管理

| 番号 | 成果物 | 実務での役割 |
| ---: | --- | --- |
| 18 | [移行計画書](docs/18-migration-plan.md) | リハーサル、wave、切替、Go/No-Go、証跡を定める |
| 19 | [ロールバック計画書](docs/19-rollback-plan.md) | 発動条件、write authority、復旧順序、判断期限を定める |
| 20 | [運用引き継ぎ書](docs/20-operations-handover.md) | 定常・非定常作業、監視、連絡、受入条件を定める |
| 21 | [課題・リスク管理表](docs/21-issue-risk-register.md) | 課題、リスク、対策、owner、期限を追跡する |
| 22 | [変更管理表](docs/22-change-register.md) | 変更理由、影響、承認、実施状態、戻し方を追跡する |
| 23 | [アーキテクチャ判断記録](docs/23-architecture-decisions.md) | 選択肢、判断、根拠、影響、再検討条件を残す |
| 24 | [成果物利用ガイド](docs/24-deliverable-usage-guide.md) | 各成果物の役割、受け渡し、レビュー観点を解説する |

## 6. 構築資材と証跡様式

| ディレクトリ | 内容 | 現在の状態 |
| --- | --- | --- |
| [install/](install/) | Installer入力、DNS、HAProxy、keepalived、Butaneの例 | サンプル／実環境適用Not Run |
| [ansible/](ansible/) | 周辺RHELサーバー向けinventory、variables、playbook、template | サンプル／実行Not Run |
| [manifests/](manifests/) | Namespace、RBAC、Quota、NetworkPolicy、Deployment、Service、Route、PDB | サンプル／適用Not Run |
| [diagrams/](diagrams/) | 構成、通信、構築フロー、VM移行、責任分界 | Draft |
| [evidence/](evidence/) | 成果物レビュー、静的検証、実行ログ、試験証跡、公開時の機密除去基準 | 様式のみ／レビュー・実行Not Run |

## 7. 実務で区別する状態

| 状態 | 意味 |
| --- | --- |
| Draft | 内容が作成途中または正式レビュー前 |
| Not Reviewed | 技術的な妥当性を指定レビュアーが確認していない |
| Not Approved | 意思決定者の正式承認を得ていない |
| Not Run / `NOT RUN` | 構築、試験、移行、復旧などを実行していない |
| Blocked | 前提不成立により判定まで進めない |
| Fail | 実行したが期待結果を満たさない |
| Pass | 承認済み条件で実行し、actual resultと証跡が期待結果を満たす |

期待結果、設定例、サンプル出力をactual resultや`Pass`として扱いません。

## 8. 使用上の注意

- 例示値を実環境へそのまま適用しないでください。
- OCPの正確な4.22.z、Installer、Operator、CSI、MTV、Kong、Sysdigの互換性とサポート条件は実施直前に確認してください。
- pull secret、password、token、SSH秘密鍵、kubeconfig、証明書秘密鍵をリポジトリへ保存しないでください。
- 実在する組織名、担当者名、内部FQDN/IP/MAC、ticket、連絡先、ログ、support bundleは公開用に除去・置換してください。
- 変更・停止・削除・切替操作は、対象、影響、承認、停止条件、ロールバックを確認してから実施してください。
- 公開時の検査項目は[公開・機密除去基準](evidence/publication-safety.md)を参照してください。

## 9. 最初に参照するファイル

1. [共通シナリオ](SCENARIO.md)で架空条件と適用境界を確認する。
2. [00. プロジェクト憲章](docs/00-project-charter.md)から番号順に成果物を確認する。
3. 文書の役割が不明な場合は[24. 成果物利用ガイド](docs/24-deliverable-usage-guide.md)の対応節を参照する。
4. 実行済みと計画を区別する場合は[プロジェクト証跡索引](evidence/README.md)を参照する。
