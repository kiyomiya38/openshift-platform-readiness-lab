# 09. OpenShift Virtualization・VM移行

## 目的

OpenShift上でVMを稼働させる設計と、Migration Toolkit for Virtualization（MTV）を用いて既存VMware VMを段階移行する計画を、クラスタ基盤、storage、network、guest OS、業務切替、切り戻しの観点から理解します。

## 実務での使用場面

- OpenShift Virtualization導入の前提とサポート条件を確認する
- VM台帳と依存関係を調査し、移行可否を評価する
- source/destination providerとnetwork/storage mappingを設計する
- Cold／Warm migration、Wave、停止時間、受入条件を決める
- Go/No-Go、切り戻し、移行後のsource保持と廃止を管理する

## 入力

- OpenShift 4.22クラスタのnode、CPU、network、storage、capacity、運用設計
- OpenShift VirtualizationとMTVの対象版・互換性
- vCenter/ESXi、VM hardware、guest OS、disk、NIC、snapshot、暗号化情報
- 業務依存、DNS、IP、証明書、監視、backup、license、停止可能時間
- VMごとのRPO/RTO、受入担当、source保持期間

## 判断

OpenShift VirtualizationはKVM/QEMUとKubeVirtを基礎に、Operator、VM API、network、storage、console、migrationなどをOpenShiftへ統合します。container PodとVMは同じクラスタ上で管理されますが、guest OS内部の運用、license、application整合性まで自動的に解決するわけではありません。

### 導入前提

- 対応するRHCOS workerとCPU virtualization extension
- VM用capacityとscheduling／eviction方針
- VMのaccess mode、performance、snapshot、clone、live migration要件を満たすstorage
- Pod network、secondary network、IPAM、VLAN、MTU、security policy
- backup、monitoring、guest agent、時刻、OS supportの運用境界

### 移行方式

| 観点 | Cold migration | Warm migration |
| --- | --- | --- |
| source停止 | 本転送前に停止 | 初回copy中は稼働し、cutover時に停止 |
| 停止時間 | data量と転送時間に影響されやすい | delta転送により短縮を狙う |
| 複雑性 | 比較的単純 | snapshot、変更率、cutover管理が増える |
| 選定条件 | 停止許容、data量、検証条件 | 対応source、RPO/RTO、変更率、互換性 |

方式は「停止時間が短そう」だけで選びません。製品の対応条件、VM特性、data整合性、業務停止、network切替、rollback deadlineを合わせて決めます。

## 成果物例の読み方

### Virtualization・MTV設計

[Virtualization・MTV設計書](../projects/enterprise-openshift-platform/docs/11-virtualization-mtv-design.md)では、Operator境界、前提条件、代表VM台帳、destination設計、互換性Gate、credential、mapping、移行方式候補、受入を確認します。[仮想化移行図](../projects/enterprise-openshift-platform/diagrams/virtualization-migration.mmd)でsourceからdestinationまでの関係を把握します。

VM台帳はCPU・memory・diskだけでは不十分です。boot mode、guest OS support、VMware tools、snapshot、NIC/VLAN、static IP、DNS、証明書、application依存、backup、監視、owner、停止許容を含めて読みます。

### 移行計画

[移行計画書](../projects/enterprise-openshift-platform/docs/18-migration-plan.md)では、代表3 VMをWaveに分け、相対schedule、Gate、Cold/Warm分岐、destination validation、traffic cutover、証跡、source保持を定義しています。`Plan`の実行成功だけでなく、guest OS、application、通信、監視、backupを業務ownerが受け入れる流れを確認します。

### 切り戻し

[切り戻し計画書](../projects/enterprise-openshift-platform/docs/19-rollback-plan.md)は、trigger、判断権限、decision deadline、source authority回復、data整合性、DNS/traffic復帰、再検証を定義します。sourceとdestinationの双方で書き込みを受け付けるsplit-brainを避ける原則が重要です。

## 他文書とのつながり

- cluster capacity、network、storage、security、backupの前提は基本設計へ戻る
- VM別パラメータとmappingは詳細設計・台帳へ展開する
- migration受入ケースは[試験仕様書](../projects/enterprise-openshift-platform/docs/16-test-specification.md)へ接続する
- 移行リスクとTBDは[課題・リスク管理台帳](../projects/enterprise-openshift-platform/docs/21-issue-risk-register.md)で追跡する
- 移行後のVM監視、backup、guest OS運用は[運用引き継ぎ](../projects/enterprise-openshift-platform/docs/20-operations-handover.md)へ渡す

## 関連製品領域の位置づけ

[Kong／Sysdig連携設計](../projects/enterprise-openshift-platform/docs/12-kong-sysdig-integration-design.md)はPhase 2以降の設計境界例です。KongはAPI公開・policy、Sysdigは可観測性・securityとの統合点を検討しますが、この架空案件では実装済みとは扱いません。ROSA／ARO、Disconnected、OpenShift AIも技術参考として比較し、オンプレミスOCP 4.22の実装済み要素と区別します。

## レビューで指摘されやすい点

- OpenShift導入完了だけでVirtualizationの前提が満たされたとする
- CSIが使用可能という理由だけでVM性能、snapshot、live migration要件を満たすとする
- VM台帳にapplication依存、license、backup、monitoring、ownerがない
- network/storage mappingの名称だけで、到達性や性能・securityを確認しない
- MTVの転送成功を業務移行成功と同一視する
- cutover後の書き込み権威、rollback deadline、source保持・廃止条件がない
- version compatibilityやguest OS supportを実施時点で再確認しない

## 公式一次資料

- [OpenShift Virtualization 4.22 - Installing](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/virtualization/installing)
- [OpenShift Virtualization 4.22 documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/virtualization/)
- [Migration Toolkit for Virtualization documentation](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/)

基礎は[OpenShift Virtualization](../reference/technical/openshift-virtualization.md)、[KVM・QEMU・KubeVirt](../reference/technical/kvm-qemu-kubevirt.md)、[MTV VM移行](../reference/technical/mtv-vm-migration.md)を参照してください。

## 次に読む章

[10. 運用・保守](10-operations-maintenance.md)へ進みます。
