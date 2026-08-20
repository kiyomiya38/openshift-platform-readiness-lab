# 24. 実務成果物の読み方・利用ガイド

> [!IMPORTANT]
> 本ガイドは、架空のOpenShift 4.22基盤導入プロジェクトを題材に、実務成果物の役割と工程間の受け渡しを解説します。収録された文書はDraft・未レビュー・未承認で、構築、試験、移行、復旧はすべて未実施です。実案件へ適用する場合は、対象環境と組織標準に合わせて再設計してください。

## 1. 対象と目的

対象は、サーバー、ネットワーク、ストレージ、運用などのインフラ実務経験があり、OpenShift基盤案件へ初めて参加するエンジニアです。

本ガイドでは製品用語を単独で説明するのではなく、用語や設定値が次の流れのどこで決まり、どの成果物へ引き継がれるかを示します。

```text
要求
→ 前提・制約・責任分界
→ 基本設計・方式判断
→ 詳細設計・パラメータ
→ 構築資材・作業手順
→ 試験仕様・試験結果
→ 移行・ロールバック・運用
→ 課題・変更・アーキテクチャ判断
```

## 2. 成果物を読む共通観点

実務文書は、次の六つの観点で確認します。

| 観点 | 確認内容 |
| --- | --- |
| Purpose | この文書が解決する問題と、工程上必要になる理由 |
| Input | 上位要件、前提、既存環境、製品仕様、組織標準 |
| Decision | 採用方式、値、責任、例外、代替案と根拠 |
| Output | 後続工程へ渡す設計、設定値、手順、判定基準 |
| Status | Draft、Reviewed、Approved、Not Run、Passなどの状態 |
| Evidence | 判断根拠、レビュー記録、actual result、ログ、承認記録 |

設定値だけでなく、owner、期限、確認方法、失敗時の扱いまで記載されているかが重要です。`TBD`は欠陥とは限りませんが、解消責任と期限がなければ管理できません。

## 3. 文書番号と実務工程

### 3.1 プロジェクト開始・要求整理（00〜04）

| 文書 | 主な入力 | 文書で決めること | 後続への出力 |
| --- | --- | --- | --- |
| [00 プロジェクト憲章](00-project-charter.md) | 企画背景、期待成果、制約 | 目的、範囲、体制、ゲート、完了条件 | プロジェクト全体の判断基準 |
| [01 要件定義書](01-requirements.md) | 業務要求、運用要求、セキュリティ要求 | 要件ID、優先度、受入基準 | 設計・試験の起点 |
| [02 前提条件・制約](02-assumptions-constraints.md) | 未確認情報、契約、設備条件 | 前提、制約、TBD、owner、期限 | 設計可能範囲と停止条件 |
| [03 スコープ・責任分界](03-scope-responsibility.md) | 対象システム、関係チーム | in/out、RACI、引き渡し条件 | 作業分担と組織間依存 |
| [04 要件トレーサビリティ](04-requirements-traceability.md) | 要件、設計、手順、試験ID | 対応関係と欠落 | 変更影響と受入根拠 |

要件定義では「高可用」「十分な性能」のような曖昧語をそのまま残さず、受入条件、測定方法、適用範囲を明確にします。未確定の製品名や数値は推測で埋めず、`TBD`として責任者と期限を付けます。

### 3.2 基本設計（05〜12）

| 文書 | 主な設計対象 | 後続で使用する箇所 |
| --- | --- | --- |
| [05 基本設計](05-basic-design.md) | 導入方式、可用性、ネットワーク、ストレージ、運用方針 | 詳細設計、ADR、試験方針 |
| [06 アーキテクチャ設計](06-architecture-design.md) | node役割、外部依存、failure domain、処理経路 | node設計、構成図、障害試験 |
| [07 Network / DNS / LB](07-network-dns-lb-design.md) | CIDR、FQDN、VIP、port、health check | DNS/LB設定、Firewall申請、接続試験 |
| [08 Security / Identity](08-security-identity-design.md) | IdP、RBAC、SCC、Secret、証明書、監査 | 権限設定、Secret運用、セキュリティ試験 |
| [09 Storage / Backup](09-storage-backup-design.md) | CSI、StorageClass、Registry、etcd、OADP、RPO/RTO | 永続化、バックアップ、復元試験 |
| [10 Observability / Operations](10-observability-operations-design.md) | metrics、logs、alerts、通知、保守 | 監視設定、runbook、運用引き継ぎ |
| [11 Virtualization / MTV](11-virtualization-mtv-design.md) | HCO、VM要件、provider、mapping、wave | Operator導入、PoC試験、移行計画 |
| [12 Kong / Sysdig](12-kong-sysdig-integration-design.md) | interface、通信、権限、データ、選定Gate | 将来PoC、製品選定、連携試験 |

基本設計は「何を採用するか」と「なぜ採用するか」を扱います。具体的なhostnameやIPだけを並べる文書ではありません。障害時の挙動、単一障害点、責任境界、未選定製品の決定Gateも設計対象です。

ROSAとAROはオンプレミスOCPとの比較対象、OpenShift AIとDisconnected構成は将来検討領域です。このサンプルでは構築対象に含めず、[共通シナリオ](../SCENARIO.md)で境界を明示しています。

### 3.3 詳細設計・構築（13〜15）

| 文書・資材 | 実務での役割 |
| --- | --- |
| [13 詳細設計](13-detailed-design.md) | 基本設計を実装可能なcomponent、resource、port、parameterへ分解する |
| [14 パラメータシート](14-parameter-sheet.md) | hostname、IP、MAC、CIDR、FQDN、設定値の正本を一元管理する |
| [15 構築手順](15-build-procedure.md) | pre-check、操作、期待結果、stop condition、rollback、証跡を定める |
| [Installer関連資材](../install/README.md) | `install-config`、`agent-config`、DNS、HAProxy、keepalived、Butaneの例を示す |
| [Ansible関連資材](../ansible/README.md) | 周辺RHELサーバーのpreflightとLB構成を自動化する例を示す |
| [OpenShift Manifest](../manifests/README.md) | 導入後のNamespace、RBAC、Quota、NetworkPolicy、workload、Routeを示す |

パラメータは複数文書へコピーするほど不整合が起きやすくなります。正本を決め、変更IDを付け、`SCENARIO → parameter sheet → install/Ansible/Manifest → test`の順に反映します。

構築手順はコマンド集ではありません。各stepに、実行条件、対象、権限、期待結果、停止条件、失敗時処置、証跡が必要です。破壊的操作や切替操作には承認とロールバックを関連付けます。

### 3.4 試験（16〜17）

| 文書 | 内容 |
| --- | --- |
| [16 試験仕様書](16-test-specification.md) | 72件の試験について目的、前提、手順、期待結果、証跡、優先度を定義する |
| [17 試験結果書](17-test-results.md) | 同じ72件のactual result、判定、証跡、課題IDを記録する |

試験仕様と試験結果は分離します。仕様書に期待結果が書かれていても、実行しなければ結果は`NOT RUN`です。サンプル出力、静的なYAML parse、server-side dry-run、実クラスタ試験は証明できる範囲が異なります。

正常系だけでなく、異常検知、冗長系への切替、復旧後の整合性、権限拒否、証跡取得を含めます。RPO/RTO、性能、停止時間は、条件と計測開始・終了点が揃わなければ判定できません。

### 3.5 移行・ロールバック（18〜19）

| 文書 | 内容 |
| --- | --- |
| [18 移行計画](18-migration-plan.md) | readiness、rehearsal、wave、cutover、Go/No-Go、連絡、証跡を定める |
| [19 ロールバック計画](19-rollback-plan.md) | 発動条件、判断期限、source保持、write-fence、復旧順序を定める |

VM移行では、sourceとdestinationのどちらがauthoritativeかを時系列で一意にします。特にDB VMは二重書き込みを避けるため、停止、最終同期、write-fence、業務確認、source保持期限を明示します。

本サンプルのMTVは3台の代表VMによるPoC設計であり、本番移行の承認や互換性確認を表しません。Cold/Warm方式、storage/network mapping、guest OS、VDDK、OpenShift Virtualizationとの互換性は採用版で確定します。

### 3.6 運用・プロジェクト管理（20〜23）

| 文書 | 管理対象 |
| --- | --- |
| [20 運用引き継ぎ](20-operations-handover.md) | 定常作業、障害対応、監視、連絡、権限、SLA/SLO、受入条件 |
| [21 課題・リスク管理](21-issue-risk-register.md) | 発生済み課題と将来リスク、影響、対策、owner、期限 |
| [22 変更管理](22-change-register.md) | 変更理由、影響、承認、実施、確認、rollback、closure |
| [23 ADR](23-architecture-decisions.md) | 選択肢、判断、根拠、trade-off、再検討条件 |

障害調査では、事象、影響、時刻、変更履歴、観測情報、仮説、確認結果、暫定対応、恒久対応を分けます。調査中の推測を原因として確定せず、runbookの操作には影響と停止条件を付けます。

Issue、Risk、Change、ADRは用途が異なります。発生済みの問題はIssue、将来起こり得る事象はRisk、状態を変える作業はChange、長期的な技術判断はADRへ記録し、必要に応じて相互参照します。

## 4. 要件から試験までの追跡例

### 4.1 RPO 1時間・RTO 4時間

```text
BR-005 / REQ-DR-001・002
→ 09 Storage / Backup設計
→ 13 詳細設計・14 パラメータ
→ 15 backup準備・復旧手順
→ 16 TST-BKP-001〜004 / TST-RST-001〜004
→ 17 actual resultと測定証跡
→ 20 復旧runbook
```

RPO/RTOは目標値であり、バックアップファイルの作成成功だけでは達成を判定できません。業務データ整合性、復旧開始・終了点、依存サービス、承認まで含めて測定します。

### 4.2 変更管理

```text
BR-008 / REQ-MNT-001・REQ-AUD-001
→ 03 RACI
→ 15 change step・stop condition・rollback
→ 16 TST-CHG-001
→ 22 change register
→ 21 deviation / risk
```

### 4.3 VM移行PoC

```text
BR-009 / REQ-MTV-001
→ 11 Virtualization / MTV設計
→ 13 VM・network・storage詳細
→ 16 TST-MTV-001〜003
→ 18 wave移行計画
→ 19 rollback計画
→ 20 運用受入
```

## 5. 状態と証跡の読み分け

| 状態 | 解釈 |
| --- | --- |
| Draft | 作成中または正式レビュー前 |
| Not Reviewed | 指定された観点・役割で技術確認していない |
| Not Approved | 意思決定者が正式承認していない |
| Not Run / `NOT RUN` | 操作や試験を実行していない |
| Blocked | 前提が成立せず判定へ進めない |
| Fail | 実行したが期待結果を満たさない |
| Pass | 承認済み条件で実行し、actual resultと証跡が期待結果を満たす |

[プロジェクト証跡索引](../evidence/README.md)では、成果物レビュー、静的検証、実行ログ、試験証跡を分離します。ファイルの存在や期待結果の記載だけで、`Reviewed`、`Approved`、`Pass`へ変更しません。

## 6. OpenShift 4.22固有情報の確認先

| 対象 | 公式資料 |
| --- | --- |
| OpenShift Container Platform 4.22全般 | [OCP 4.22 documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/) |
| Agent-based Installer / platform none | [Installing an on-premise cluster with the Agent-based Installer](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installing_an_on-premise_cluster_with_the_agent-based_installer/) |
| 認証・認可 | [Authentication and authorization](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/authentication_and_authorization/) |
| MachineConfig | [Machine configuration](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/machine_configuration/) |
| Image Registry | [Registry](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/registry/) |
| Backup / OADP | [Backup and restore](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/backup_and_restore/) |
| Monitoring | [Monitoring stack for Red Hat OpenShift 4.22](https://docs.redhat.com/en/documentation/monitoring_stack_for_red_hat_openshift/4.22/) |
| OpenShift Virtualization | [Virtualization](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/virtualization/) |
| MTV | [Migration Toolkit for Virtualization](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/) |
| Kong | [Kong Kubernetes Ingress Controller](https://developer.konghq.com/kubernetes-ingress-controller/)、[Kong Operator](https://developer.konghq.com/operator/) |
| Sysdig | [Sysdig Docs](https://docs.sysdig.com/) |

実施前には正確な`4.22.z`、release image、Installer/client、update channel、Operator channel/CSV、CSI、MTV、source vSphere、Kong、Sysdigの対応組み合わせを同一時点で確認します。検索結果や二次情報だけで互換性を確定しません。

## 7. 実機がない状態で確認できる範囲

| 確認 | 確認できること | 確認できないこと |
| --- | --- | --- |
| Markdown link / heading | 文書参照の欠落、構造上の不整合 | 技術内容の妥当性 |
| YAML parse | 基本文法 | OCP API admission、runtime behavior |
| Ansible syntax / lint | 構文、変数参照の一部 | 対象hostへの変更結果 |
| Mermaid render | 図の構文と描画可否 | 実環境構成との一致 |
| Parameter consistency | hostname、IP、CIDR、portの文書間一致 | 到達性、性能、冗長動作 |
| Tabletop review | 手順の抜け、役割、停止条件、rollback gap | 実停止時間、データ整合性、RPO/RTO |

静的確認の結果は[検証記録](../evidence/verification-record.md)、実環境の操作は[実行ログ](../evidence/execution-log.md)、試験の証跡は[試験証跡索引](../evidence/test-evidence-index.md)へ分けて記録します。

## 8. 公開サンプルとしての安全境界

本サンプルは`example.com`、RFC 5737の文書用IPv4アドレス、RFC 1918のクラスタ内部CIDR、placeholderを使用します。実在する組織名、担当者名、内部FQDN/IP/MAC、credential、ログ、ticket、連絡先を含めません。

公開時の確認項目と置換基準は[公開・機密除去基準](../evidence/publication-safety.md)を参照してください。
