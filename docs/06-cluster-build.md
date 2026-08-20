# 06. クラスタ構築

## 目的

承認済み入力を用いて、外部依存サービスの準備、Agent ISO生成、ノード起動、インストール監視、導入後設定、受入確認を安全に実施する流れを理解します。机上教材のためコマンドは実行せず、各手順の目的、確認点、証跡、停止条件を成果物から読みます。

## 実務での使用場面

- 作業責任者の統制下で構築手順を実行する
- 手順ごとの入力、期待結果、確認者、証跡を記録する
- 想定外事象で継続・停止・復旧を判断する
- 実装差異を課題・変更として管理する
- 試験担当と運用担当へ構築状態を引き渡す

## 入力

- Go判定済みの[構築手順書](../projects/enterprise-openshift-platform/docs/15-build-procedure.md)
- 承認済み[パラメータシート](../projects/enterprise-openshift-platform/docs/14-parameter-sheet.md)
- 検証済みのInstaller入力、DNS、LB、NTP、Proxy、Firewall設定
- 正規取得したOpenShift 4.22資材と安全に受け渡されたSecret
- 作業記録、証跡保存先、連絡先、停止・再開・切り戻し権限

## 判断

構築では「コマンドが終了した」ことと「手順の完了条件を満たした」ことを分けます。各stepで次を判断します。

1. 入力は承認版と一致しているか
2. コマンドや変更対象は想定どおりか
3. 出力と観測結果は期待範囲か
4. Secretや個人情報を含まない証跡を保存できたか
5. 差異がある場合、続行可能か、停止して承認が必要か
6. 次stepへ進む前提が成立したか

### 標準的な構築の流れ

| 段階 | 主な作業 | 確認の焦点 |
| --- | --- | --- |
| 1 | 版・資材・入力の最終確認 | digest、schema、承認版、秘匿値 |
| 2 | DNS/NTP/LB/Proxy/Firewall準備 | 実応答、時刻同期、backend、冗長性 |
| 3 | `install-config`と`agent-config`生成 | CIDR、host、MAC、role、`noProxy` |
| 4 | Agent ISO生成・配布 | 生成ログ、checksum、安全な保管 |
| 5 | Node起動・host割当確認 | hardware、NIC、disk、role、rendezvous |
| 6 | Install開始・監視 | bootstrap、API、Operator、timeout |
| 7 | 導入直後確認 | node、ClusterOperator、MCP、credential回収 |
| 8 | Storage/Registry/IdP/監視など設定 | 依存順、Operator状態、永続性 |
| 9 | サンプルworkload適用 | RBAC、quota、network、route、rollback |
| 10 | 構築完了判定 | 差異、既知課題、証跡、試験引渡し |

## 成果物例の読み方

[構築フロー図](../projects/enterprise-openshift-platform/diagrams/build-flow.mmd)で段階を把握し、[構築手順書](../projects/enterprise-openshift-platform/docs/15-build-procedure.md)で各段階の詳細を確認します。特に事前条件、承認checkpoint、失敗時の証跡、段階別の戻し方を読みます。

### 外部RHELサービス

[Ansible構成](../projects/enterprise-openshift-platform/ansible/README.md)は、LB用RHEL hostのpreflightとHAProxy/keepalived構成例です。inventoryと変数を本番値へ置き換える前に、対象host、差分、validator、service再起動、冗長切替への影響をレビューします。check modeは予測であり、実hostの通信・service動作確認を置き換えません。

### Installer入力

[Installer入力例](../projects/enterprise-openshift-platform/install/README.md)の各ファイルを[詳細設計書](../projects/enterprise-openshift-platform/docs/13-detailed-design.md)と照合します。AgentConfigのhost識別、rendezvous IP、network raw config、InstallConfigのCIDR、proxy、additional trust bundleが同じ設計を表現しているか確認します。

### 導入後リソース

[サンプルワークロード](../projects/enterprise-openshift-platform/manifests/README.md)は段階適用し、Namespaceとpolicyを先に確認してからworkloadと公開経路を扱います。`kubectl kustomize`やYAML parserによる静的検証と、API serverへのdry-run、実適用、実通信確認は別の証跡です。

### 内部Image Registry

bare metalまたは共有可能な既定storageがないplatformでは、導入直後のImage Registry管理状態を確認し、業務利用前に対応する永続storageと`Managed`状態を設計・試験します。クラスタインストール完了と、内部Registryの本番受入完了は別の判定です。

## 他文書とのつながり

- 作業中の実装差異は[詳細設計](../projects/enterprise-openshift-platform/docs/13-detailed-design.md)と[パラメータ](../projects/enterprise-openshift-platform/docs/14-parameter-sheet.md)へ戻す
- 許容できない差異は[課題・リスク](../projects/enterprise-openshift-platform/docs/21-issue-risk-register.md)または[変更管理](../projects/enterprise-openshift-platform/docs/22-change-register.md)へ登録する
- 完了状態と既知課題を[試験仕様書](../projects/enterprise-openshift-platform/docs/16-test-specification.md)へ引き渡す
- 実際の操作、時刻、確認者、結果は[作業記録](../evidence/work-record.md)へ残す
- 生成物の静的検証は[静的検証記録](../evidence/static-validation-record.md)へ残す

## レビューで指摘されやすい点

- 破壊的・不可逆な操作の対象確認、承認checkpoint、停止条件がない
- コマンドだけで目的、期待結果、異常時判断、証跡がない
- Installer待機のtimeoutを根拠なく延長し、根本事象を調査しない
- Secretをterminal履歴、screen capture、Git、作業記録へ残す
- Operatorが`Available`でも`Degraded`や進行中状態を確認しない
- 構築時の手修正を詳細設計や構成管理へ戻さない
- 静的構文検査をクラスタ上の受入試験と表現する
- 失敗時にログを保全せず、すぐ再実行して事象を上書きする

## 公式一次資料

- [Installing with the Agent-based Installer](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installing_an_on-premise_cluster_with_the_agent-based_installer/installing-with-agent-based-installer)
- [Postinstallation configuration](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/postinstallation_configuration/)
- [Machine configuration](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/machine_configuration/)
- [Registry Operator configuration](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/registry/configuring-registry-operator)

製品構造は[OpenShiftコア知識](../reference/technical/openshift-core-knowledge.md)、RHELとAnsibleは[RHEL基礎](../reference/technical/rhel-linux-foundation.md)と[Ansible自動化](../reference/technical/ansible-automation.md)で補足します。

## 次に読む章

[07. 試験](07-testing.md)へ進みます。
