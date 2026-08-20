# 05. 構築準備

## 目的

構築当日に判断を持ち越さないよう、設計、外部依存、資材、権限、証跡、作業体制、停止・切り戻し条件を事前に確認します。準備完了とはファイルが存在することではなく、入力の承認と依存サービスの実確認が終わり、安全に開始できる状態です。

## 実務での使用場面

- 構築計画と作業申請をレビューする
- DNS、NTP、LB、Firewall、Proxy、Storageなどの前提を確認する
- Installer、ISO、CLI、Ansible collection、container imageの版を固定する
- 作業者、確認者、承認者、連絡経路を確定する
- Go/No-Go会議で開始可否を判定する

## 入力

- 承認済みの基本設計、詳細設計、パラメータシート
- [構築手順書](../projects/enterprise-openshift-platform/docs/15-build-procedure.md)
- 対象z-streamのrelease image、`openshift-install`、`oc`、Butane
- pull secret、SSH公開鍵、追加CAなどの安全な受け渡し手順
- サーバー、ネットワーク、DNS、NTP、LB、Proxy、Firewall、Storageの準備結果
- 作業申請、変更番号、保守時間、連絡・エスカレーション先

## 判断

### 設計凍結とTBD

すべてのTBDを消す必要はありません。ただし、構築入力または安全性に影響するTBDは開始前に解消します。[前提・制約・未確定事項](../projects/enterprise-openshift-platform/docs/02-assumptions-constraints.md)の「導入開始前の停止条件」を基準に、未解消でも進められる項目とNo-Go項目を分けます。

### バージョンと供給元

4.22というminorだけでなく、採用z-stream、release image digest、取得元、取得日時、checksum、互換性確認結果を記録します。Installerとrelease imageは整合する組み合わせを使い、Operator、CSI、Virtualization、MTVも対象版のサポート情報を別途確認します。

### 実行経路と権限

踏み台から必要な宛先へ到達できるか、Proxyと追加CAが機能するか、DNS正逆引き、NTP同期、LB VIP、BMC/console、証跡保管先を確認します。権限は事前に発行し、共有IDや平文Secretで回避しません。

## 成果物例の読み方

[パラメータシート](../projects/enterprise-openshift-platform/docs/14-parameter-sheet.md)の「実装ファイルと検証状態」から、各値の確定・未確定を確認します。次に[構築手順書](../projects/enterprise-openshift-platform/docs/15-build-procedure.md)の事前条件、Go/No-Go、作業記録、証跡命名を読みます。

[`install/README.md`](../projects/enterprise-openshift-platform/install/README.md)では、入力例が未実行であり、Secret未投入、スキーマ再確認が必要である境界を確認します。[Ansible README](../projects/enterprise-openshift-platform/ansible/README.md)では、clean RHELでvalidatorが未導入の場合のcheck modeの限界と、承認を要するpackage導入を分けて読みます。

### 代表的な準備確認表

| 分類 | 開始前に確認する事実 | 不備時の扱い |
| --- | --- | --- |
| 設計 | 承認版、TBD、変更差分、要求追跡 | 影響項目はNo-Go |
| Node | HW互換性、firmware、CPU virtualization、disk、NIC、時刻 | 再設計または是正 |
| Network | CIDR重複、MTU、gateway、必要port、Proxy | 疎通確立までNo-Go |
| DNS | API/API-int/wildcard/nodeの正引き、必要なPTR | 修正・再確認 |
| LB | VIP所有、frontend/backend、health check、冗長切替 | 修正・再確認 |
| Storage | 対応CSI、StorageClass、Registry用永続領域 | 利用機能に応じNo-Go |
| Security | pull secret、SSH key、CA、権限、保管・廃棄 | 安全な受渡しまでNo-Go |
| Tool | version、digest、checksum、供給元 | 正規資材へ交換 |
| Operation | 作業窓、連絡先、停止判断者、証跡領域 | 体制確定までNo-Go |

## 他文書とのつながり

- 準備中に発見した設計差異は[課題・リスク管理台帳](../projects/enterprise-openshift-platform/docs/21-issue-risk-register.md)へ登録する
- 基準化後の値変更は[変更管理台帳](../projects/enterprise-openshift-platform/docs/22-change-register.md)で承認する
- Go判定後は[構築手順書](../projects/enterprise-openshift-platform/docs/15-build-procedure.md)の順序と停止条件に従う
- 事前確認結果は[作業記録](../evidence/work-record.md)またはプロジェクト固有の記録へ残す
- 静的検証は[静的検証記録](../evidence/static-validation-record.md)へ残し、実機試験と区別する

## レビューで指摘されやすい点

- `oc version`などの表示だけで、release image digestと互換性を確認しない
- DNSやFirewallの申請済みを、反映確認済みとして扱う
- check modeを実適用結果やサービス動作確認とみなす
- 証明書、pull secret、SSH鍵の投入・回収・廃棄責任がない
- 失敗時のログ採取先、停止判断者、再開条件がない
- 構築当日に未確定値を口頭で決め、文書へ反映しない
- 作業前バックアップや既存設定の退避方法がない

## 公式一次資料

- [Agent-based Installer prerequisites](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installing_an_on-premise_cluster_with_the_agent-based_installer/preparing-to-install-with-agent-based-installer)
- [OpenShift 4.22 Installation overview](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installation_overview/)
- [Red Hat Ecosystem Catalog](https://catalog.redhat.com/)
- [OpenShift Container Platform Life Cycle Policy](https://access.redhat.com/support/policy/updates/openshift)

Disconnected環境での資材準備は[Disconnected導入](../reference/technical/disconnected-install.md)を参照します。

## 次に読む章

[06. クラスタ構築](06-cluster-build.md)へ進みます。
