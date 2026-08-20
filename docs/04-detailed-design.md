# 04. 詳細設計

## 目的

基本設計で合意した方式を、構築担当が曖昧さなく実装し、試験担当が期待状態を判定できる粒度へ具体化します。ホスト、インターフェース、CIDR、DNS名、ポート、パラメータ、設定ファイル、適用順序、確認方法を一貫させます。

## 実務での使用場面

- 設計方針を具体的な設定値へ展開する
- 複数チームへDNS、Firewall、Storageなどの作業を依頼する
- 設定ファイルとパラメータシートをレビューする
- 構築手順、試験条件、運用Runbookの入力を固定する
- 変更時に影響する設定と文書を特定する

## 入力

- 承認済み基本設計とADR
- IP、VLAN、DNS、NTP、Proxy、Firewall、証明書、IdP、Storageの払い出し情報
- ノードごとのNIC、MAC、ディスク、CPU、メモリー、BMC情報
- InstallerおよびOperatorの対象バージョン・スキーマ
- 組織の命名、時刻、ログ、Secret、構成管理標準

## 判断

詳細設計では、値の出所と管理主体を明示します。

| 分類 | 例 | 確認すべきこと |
| --- | --- | --- |
| Installer入力 | cluster/base domain、pull secret、SSH key、network CIDR | スキーマ、重複、秘匿、再生成条件 |
| Host入力 | hostname、role、MAC、IP、gateway、DNS | 物理NICとの対応、正逆引き、予約 |
| LB | VIP、frontend/backend、port、health check | API/MCS/Ingressの分離、永続性、切替 |
| Proxy | HTTP(S) proxy、`noProxy`、追加CA | Cluster/Service/Machine CIDRと内部domainの除外 |
| Storage | CSI、StorageClass、access mode、容量 | 対応表、default、拡張、障害動作 |
| Security | IdP claim、group、RBAC、証明書 | 最小権限、期限、更新、break-glass |
| Workload | namespace、quota、limit、network policy、PDB | tenant境界、既定拒否、可用性の限界 |

Secret値、pull secret、秘密鍵、実パスワードは詳細設計書やGitへ記録しません。参照名、保管先、投入者、利用時点、ローテーション、検証状態のみを管理します。

## 成果物例の読み方

### 詳細設計書とパラメータシート

[詳細設計書](../projects/enterprise-openshift-platform/docs/13-detailed-design.md)で、実装対象、設定単位、依存関係、検証方法を読みます。[パラメータシート](../projects/enterprise-openshift-platform/docs/14-parameter-sheet.md)では値だけでなく、状態、出所、適用先、TBDを確認します。

同じ値を複数ファイルへ複写する場合、正本を決めます。たとえばノードIPをパラメータシート、AgentConfig、DNS、HAProxyで使うなら、変更時に全箇所を更新・再レビューする関係を明示します。

### Installer入力

- [`install-config.yaml.example`](../projects/enterprise-openshift-platform/install/install-config.yaml.example)：クラスタ全体、network、proxy、platform方式
- [`agent-config.yaml.example`](../projects/enterprise-openshift-platform/install/agent-config.yaml.example)：ホスト識別、role、rendezvous、静的network
- [DNSレコード例](../projects/enterprise-openshift-platform/install/dns/forward-records.example)：API、API-int、wildcard、各node
- [HAProxy設定例](../projects/enterprise-openshift-platform/install/haproxy/haproxy.cfg.example)：frontend/backend、port、health check
- [keepalived設定例](../projects/enterprise-openshift-platform/install/haproxy/keepalived-primary.conf.example)：VIP、priority、監視script
- [Butane例](../projects/enterprise-openshift-platform/install/openshift/99-worker-chrony.bu.example)：RHCOSのNTP設定をMachineConfigへ変換する入力

ファイルは拡張子`.example`の学習用入力です。実行前には使用する`openshift-install`と`butane`の版でスキーマを検証し、Secretを外部から安全に投入します。

### RHEL外部サービスとワークロード

[Ansible README](../projects/enterprise-openshift-platform/ansible/README.md)からinventory、変数、preflight、LB構成の関係を確認します。Ansibleの対象はRHEL上の外部サービスであり、RHCOSノードの恒久変更はMachineConfigなどOpenShiftの管理機構を用います。

[Manifest README](../projects/enterprise-openshift-platform/manifests/README.md)では、Namespace、ServiceAccount/RBAC、Quota、LimitRange、NetworkPolicy、Deployment、Service、Route、PDBを`kustomization.yaml`でまとめる例を確認します。リソース間のselector、label、service port、container portの一致を追います。

## 他文書とのつながり

- 値の根拠は[基本設計](../projects/enterprise-openshift-platform/docs/05-basic-design.md)と分野別設計へ戻る
- 未確定値は[前提・制約・TBD](../projects/enterprise-openshift-platform/docs/02-assumptions-constraints.md)へ登録する
- 適用順序と確認コマンドは[構築手順書](../projects/enterprise-openshift-platform/docs/15-build-procedure.md)へ渡す
- 期待状態は[試験仕様書](../projects/enterprise-openshift-platform/docs/16-test-specification.md)へ渡す
- 値の変更は[変更管理台帳](../projects/enterprise-openshift-platform/docs/22-change-register.md)と影響文書へ反映する

## レビューで指摘されやすい点

- 同じIP、host名、CIDR、portが文書・設定間で不一致
- `noProxy`に内部domain、Cluster/Service network、API/Ingress宛先が不足
- DNSの正引きだけで、nodeおよびAPIの逆引き要件を確認していない
- API、Machine Config Server、Ingressを同じbackend・health checkで扱う
- パラメータの出所、所有者、確定状態がない
- Secretや証明書秘密鍵をサンプル値としてコミットする
- サポート対象のCSI、Operator channel、更新互換性を確認していない
- PDBをノード障害時のPod稼働保証と誤解する

## 公式一次資料

- [Agent-based Installer configuration](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installing_an_on-premise_cluster_with_the_agent-based_installer/installing-with-agent-based-installer)
- [OpenShift 4.22 Machine configuration](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/machine_configuration/)
- [OpenShift 4.22 Configuring the registry](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/registry/configuring-registry-operator)
- [OpenShift 4.22 Authentication and authorization](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/authentication_and_authorization/)

補足は[RHEL基礎](../reference/technical/rhel-linux-foundation.md)、[Ansible自動化](../reference/technical/ansible-automation.md)、[設計文書ガイド](../reference/technical/design-document-guide.md)を参照してください。

## 次に読む章

[05. 構築準備](05-build-preparation.md)へ進みます。
