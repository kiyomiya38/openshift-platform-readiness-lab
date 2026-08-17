# 11. OpenShift Virtualization・MTV 設計書

> [!IMPORTANT]
> 本書は完全に架空の学習用設計です。OpenShift Virtualization と Migration Toolkit for Virtualization（MTV）の導入、VM 移行、試験はすべて **未実施** です。商用経験や製品互換性を証明する文書ではありません。

## 1. 文書情報

| 項目 | 値 |
| --- | --- |
| 文書 ID | DES-VIRT-001 |
| 対象案件 | Example Enterprise OpenShift 基盤導入 |
| 版 | 0.1 Draft |
| 基準日 | 2026-08-17 |
| ステータス | 本人レビュー前・未承認 |
| 対象基盤 | OpenShift Container Platform 4.22.z（暫定） |
| 実施状態 | 設計のみ。構築・移行・試験は未実施 |

前提は [共通シナリオ](../SCENARIO.md)、基盤方式は [プラットフォーム基本設計](05-basic-design.md)、詳細値は [詳細設計](13-detailed-design.md) と [パラメータシート](14-parameter-sheet.md) を参照します。構成関係は [Virtualization 移行図](../diagrams/virtualization-migration.mmd) に示します。

関連要件は `REQ-VIR-001` と `REQ-MTV-001`、中央試験は `TST-VIR-001` と `TST-MTV-001〜003` です。要件から設計・試験への対応は [要件トレーサビリティ](04-requirements-traceability.md)、試験の正本は [試験仕様書](16-test-specification.md)、実績の正本は [試験結果記録](17-test-results.md) とします。

## 2. 目的と範囲

第 2 段階として、既存 VMware VM を OpenShift Virtualization へ移す技術的成立性を 3 台の代表 VM で確認する PoC を設計します。

対象:

- OpenShift Virtualization の OLM による導入方針
- VM 用 compute、storage、network、security、backup、運用の設計
- VMware を移行元とする MTV の provider、mapping、plan、cutover の設計
- 3 台の架空 VM の cold / warm 移行候補と受入条件
- 本番移行判断に必要な課題・測定値の抽出

対象外:

- 実際の Operator 導入、provider 登録、VDDK image 作成、VM データ転送
- 製品版、subscription、license、サポート可否の確定
- 全 VM の本番移行承認、VMware 廃止、アプリケーションのコンテナ化
- GPU、SR-IOV、guest-initiated iSCSI/NFS、特殊 USB/device passthrough の検証

## 3. 製品境界と版管理

| 要素 | 本設計での扱い | 確定条件 |
| --- | --- | --- |
| OpenShift Container Platform | `4.22.z` を暫定前提 | 実施前に正確な z、更新チャネル、サポート期間を確認 |
| OpenShift Virtualization | OLM で導入する Red Hat Operator | OCP と対応する版、channel、CSV、既知問題を確認 |
| MTV | OpenShift Virtualization とは別の Operator | OCP、Virtualization、vCenter/ESXi、guest OS、VDDK の互換表を確認 |
| 参照した MTV 文書 | 2.12 文書を設計観点の参考に使用 | **2.12 採用を意味しない**。実施時の対応版を選定 |
| CSI storage | 製品・StorageClass とも未定 | Red Hat Ecosystem Catalog、CSI snapshot、RWX/Block、性能を確認 |
| VMware | source としてのみ想定 | vCenter/ESXi、hardware version、権限、契約、停止可能時間を確認 |

### 3.1 Operator の分離

1. OCP クラスタは Agent-based Installer で先に構築する。
2. クラスタ健全性と基盤試験を完了する。
3. OpenShift Virtualization Operator を **OLM** で導入する。
4. `HyperConverged` CR を作成し、Virtualization components の健全性を確認する。
5. MTV Operator は、互換性と license / support 条件を確認した後に **別途** 導入する。

Agent-based Installer で OCP を導入することと、OpenShift Virtualization Operator を OLM で導入することを混同しません。本案件では、Virtualization を Installer bundle で同時導入したとは扱いません。

### 3.2 OLM 方針

| 項目 | 暫定設計 |
| --- | --- |
| Catalog source | Red Hat 提供 source。connected + 組織 Proxy |
| Virtualization namespace | `openshift-cnv`（公式文書で必須とされる namespace） |
| Channel | `stable` を候補。OCP 4.22 対応版を実施直前に再確認 |
| InstallPlan approval | `Automatic` を候補。更新統制との整合を変更審査で確認 |
| MTV namespace/channel | 対象版の Operator 推奨値を確認後に確定 |
| 更新順序 | OCP と Virtualization の対応関係、MTV release note を確認して計画 |

Red Hat は OpenShift Virtualization について、対応する OCP minor version と組み合わせ、`stable` channel と自動承認を推奨しています。ただし、組織の変更統制、保守時間、更新前試験と矛盾しない運用手順を別途承認します。

## 4. 前提条件

### 4.1 クラスタ

- Worker 3 台は RHCOS、x86_64、CPU virtualization extension 有効とする。
- CPU model / feature が worker 間で live migration 可能か確認する。
- VM 用 resource request と、1 台障害・maintenance 時の退避余力を capacity 計画で確認する。
- Virtualization components と VM workload の node placement、taint、toleration は実測後に確定する。
- `cluster-admin` 相当の作業は承認された個人 ID と時間限定手順で行う。

### 4.2 Storage

- VM disk の第一候補は CSI 対応の `ReadWriteMany` + `Block` volume mode とする。
- live migration を必要とする VM は shared RWX storage を前提とする。
- `VolumeSnapshotClass`、CSI snapshot、clone、expansion、OADP 連携を確認する。
- performance は平均値だけでなく latency p95/p99、IOPS、throughput、queue を source baseline と比較する。
- product、StorageClass 名、replication、failure domain は未確定であり、例示名を本番値にしない。

### 4.3 Network

- 既存 VMware Port Group と、destination の Pod network または `NetworkAttachmentDefinition`（NAD）を一対一で mapping 表にする。
- guest IP を保持する場合は L2 continuity、VLAN、gateway、DHCP/IPAM、重複検知を確認する。
- Pod network の masquerade を使う場合は、guest IP 保持と同じ要件を満たさないため DNS / access path を再設計する。
- migration traffic は tenant traffic への影響を避けるため dedicated Multus network を候補とする。
- MTU、bandwidth、firewall、return path、DNS TTL を移行前に測定する。

## 5. 代表 VM 台帳

すべて架空値です。guest OS が MTV / virt-v2v / OpenShift Virtualization で対応するかは **要確認** です。

| VM ID | 役割・特性 | 暫定仕様 | Source | Destination candidate | 移行方式候補 |
| --- | --- | --- | --- | --- | --- |
| VM-P01 `poc-web-01` | 静的 Web、local state なし | RHEL 8.x、2 vCPU、4 GiB、60 GiB、`198.51.100.51` | `PG-WEB` / `ds-general` | `poc-web` / `nad-web` / `sc-vm` | Cold。最初の小規模 wave |
| VM-P02 `poc-app-01` | Java API、外部 DB 接続、systemd service | RHEL 8.x、4 vCPU、8 GiB、40+80 GiB、`198.51.100.61` | `PG-APP` / `ds-general` | `poc-app` / `nad-app` / `sc-vm` | Warm 候補。CBT・VDDK・互換性不成立時は Cold |
| VM-P03 `poc-db-01` | PostgreSQL、書込データあり | RHEL 8.x、8 vCPU、32 GiB、80+500 GiB、`198.51.100.71` | `PG-DB` / `ds-db` | `poc-db` / `nad-db` / `sc-vm`（高 I/O 適合要確認） | Cold。DB 停止・整合 backup 後に実施 |

追加棚卸し項目:

- VM UUID、firmware（BIOS/UEFI）、secure boot、virtual hardware version
- disk bus、snapshot、暗号化、RDM、共有 disk、guest-initiated mount
- NIC 数、MAC/IP 固定条件、VLAN、DNS、Firewall、load balancer
- guest OS subscription、kernel、driver、VMware Tools、QEMU guest agent
- CPU instruction、NUMA、huge pages、latency、license の socket/core 条件
- 起動順、停止順、依存 API/DB、batch、backup agent、監視 agent
- 所有者、業務受入者、停止可能時間、RPO/RTO、切り戻し期限

## 6. Destination 設計

### 6.1 Namespace と権限

| Namespace 候補 | 用途 | 変更権限 |
| --- | --- | --- |
| `openshift-cnv` | OpenShift Virtualization Operator | Platform administrator のみ |
| `<mtv-recommended-namespace>` | MTV Operator と controller | Platform migration administrator のみ |
| `openshift-mtv` または製品推奨値 | MTV plan/provider 用候補 | 対象版の手順確認後に確定 |
| `poc-web` / `poc-app` / `poc-db` | 移行先 VM | 各 application owner は namespace scoped role |

VM live migration 操作は、対象版で提供される `kubevirt.io:migrate` role を信頼された運用者へ namespace 単位で付与する方針です。不要な cluster-wide binding を避けます。

### 6.2 VM 実行方針

- `VirtualMachine` を desired state、`VirtualMachineInstance` を実行状態として監視する。
- CPU / memory は source の割当値を機械的に移さず、実測使用量と peak から request/limit を決める。
- 重要 VM には affinity / anti-affinity、topology spread、priority、disruption の要件を定義する。
- non-migratable VM に `LiveMigrate` を設定して node drain を阻害しないよう、storage/device 条件と eviction strategy を組でレビューする。
- QEMU guest agent を対応 guest に導入し、snapshot consistency、IP 情報、graceful shutdown を確認する。

### 6.3 Backup

- VM object と disk data は OADP + 対応 CSI を候補とする。
- database consistency は OADP のみで保証せず、Application team の quiesce / dump / transaction recovery 手順と組み合わせる。
- online snapshot は QEMU guest agent の稼働と、hot-plug disk の包含範囲を確認する。
- etcd backup は workload backup と別管理であり、OADP を full-cluster recovery とみなさない。
- restore は別 namespace / isolation network で行い、source と同時に production IP で起動しない。

## 7. MTV PoC 設計

### 7.1 互換性 Gate

次のすべてが確認できるまで provider credential や migration plan を作成しません。

| Gate | 確認内容 | 状態 |
| --- | --- | --- |
| COMP-01 | OCP 4.22.z と OpenShift Virtualization の対応版 | 要確認 |
| COMP-02 | その組合せに対応する MTV version/channel | 要確認 |
| COMP-03 | vCenter/ESXi version と権限 | 要確認 |
| COMP-04 | 3 guest OS、filesystem、firmware、device の対応 | 要確認 |
| COMP-05 | VDDK 使用条件、image build/repository/license | 要確認 |
| COMP-06 | CSI、snapshot、RWX/Block、DataMover の対応 | 要確認 |
| COMP-07 | source/destination network reachability と MTU | 要確認 |
| COMP-08 | warm migration の CBT、snapshot、cutover 条件 | 要確認 |

### 7.2 Credential

- VMware account は公式資料の minimum privileges を基に専用個人追跡可能 ID として発行する。
- Password、token、VDDK download credential は approved vault へ保存し、Git、YAML 例、ログへ記載しない。
- vCenter certificate は組織 CA trust を原則とし、証明書検証 skip を恒久運用にしない。
- Credential の発行、使用、失効を Security team が監査できるようにする。

### 7.3 Mapping

| Source | Destination candidate | 確認事項 |
| --- | --- | --- |
| `ds-general` | 論理 class `sc-vm` | 容量、RWX、Block、clone/snapshot、性能、暗号化 |
| `ds-db` | 論理 class `sc-vm`（高 I/O 適合を別測定） | latency/IOPS、500 GiB 転送時間、整合性、拡張 |
| `PG-WEB` | `nad-web` | VLAN、MTU、IP 重複、DNS/LB、Firewall |
| `PG-APP` | `nad-app` | DB/API への route、TLS、source IP 条件 |
| `PG-DB` | `nad-db` | client 制限、backup、管理経路、split-brain 防止 |

名称は例示です。StorageClass と NAD は作成・試験前に [変更台帳](22-change-register.md) で承認します。

### 7.4 Migration Plan

- 3 VM を一つの plan で同時移行せず、wave ごとに独立 plan とする。
- Wave 1 は low-risk の Web VM を cold migration し、transfer、conversion、network、boot の基本を確認する。
- Wave 2 は App VM の warm migration を候補とする。CBT、VDDK、snapshot、precopy、cutover window が満たせない場合は cold へ変更する。
- Wave 3 は DB VM を cold migration し、application-consistent backup、write fencing、recovery test を優先する。
- preflight / plan validation の error、warning、conversion risk は owner と disposition を記録する。
- plan の `Archive` / `Delete` は不可逆な面があるため、証跡保全完了まで実施しない。

実行順序と Go / No-Go は [移行計画](18-migration-plan.md)、復帰は [切り戻し計画](19-rollback-plan.md) に定義します。

## 8. PoC 受入基準

PoC は本番移行承認ではありません。次の基準をすべて満たし、未解決の重大リスクを判断材料として提示できたときに「PoC 技術評価完了」とします。

| ID | 判定可能な基準 | 現在状態 |
| --- | --- | --- |
| PAC-01 | 対象版の compatibility matrix と release notes が記録されている | Not Run |
| PAC-02 | 3 VM の preflight に未処置の Error がなく、Warning に owner と判断がある | Not Run |
| PAC-03 | 各 wave で source disk と destination disk の数・容量が一致する | Not Run |
| PAC-04 | destination VM が起動し、guest OS、service、QEMU guest agent の状態を確認できる | Not Run |
| PAC-05 | DNS、route、required port、外部依存先の正常・拒否経路を確認できる | Not Run |
| PAC-06 | App team の smoke test と DB integrity check が合格する | Not Run |
| PAC-07 | source baseline に対し、合意した latency / throughput 許容差を満たす | Not Run |
| PAC-08 | node maintenance または live migration 可否を VM ごとに判定できる | Not Run |
| PAC-09 | backup と隔離 restore の手順が試験され、RPO 1 時間 / RTO 4 時間との差を測定できる | Not Run |
| PAC-10 | rollback rehearsal で二重起動を防ぎ、source を再開できる | Not Run |
| PAC-11 | migration log、VM manifest、event、test evidence から第三者が追跡できる | Not Run |
| PAC-12 | 高リスク、制約、費用、工数を本番移行判断者へ提示できる | Not Run |

性能許容差は source baseline 取得後に定量化します。仮の数値を合格結果にしません。

### 8.1 PAC と既存試験 ID の対応

`PAC-*` は本書内の受入観点であり、中央試験 ID を追加するものではありません。実行結果は次の既存試験 ID へ登録し、対応する中央試験が `Pass` になる前に PAC を合格扱いしません。

| PAC ID | 対応する既存試験・管理記録 | 対応内容 |
| --- | --- | --- |
| PAC-01 | `TST-VIR-001` | OCP・Virtualization・MTV・source・guest の compatibility |
| PAC-02 | `TST-VIR-001` | PoC 前提、preflight Error、Warning disposition |
| PAC-03 | `TST-MTV-001` | source / destination disk の変換・照合 |
| PAC-04 | `TST-MTV-001` | destination boot、guest、service、guest agent |
| PAC-05 | `TST-MTV-001` | 移行後 network、DNS、Route、dependency |
| PAC-06 | `TST-MTV-001`、DB整合性は `TST-RST-003` | application smoke と DB consistency |
| PAC-07 | `TST-PER-001` | source baseline と合意済み性能目標の比較 |
| PAC-08 | `TST-VIR-001` | VM ごとの node maintenance / live migration 可否判定 |
| PAC-09 | `TST-BKP-003`、`TST-BKP-004`、`TST-RST-003`、`TST-RST-004` | PV・DB backup、隔離restore、RPO/RTO計測 |
| PAC-10 | `TST-MTV-003` | source保持、二重起動防止、切り戻し |
| PAC-11 | `TST-MTV-001〜003` | plan、timeline、event、rollback evidence の追跡 |
| PAC-12 | `TST-MTV-001〜003` と [課題・リスク管理台帳](21-issue-risk-register.md) | 技術結果と残余リスクを判断者へ提示 |

複数の試験 ID を示す行は、該当するすべての結果と証跡を確認します。`PAC-*` の状態は初期値 `Not Run` のままであり、この対応表自体は実施証跡ではありません。

## 9. 設計検証項目

以下は実施予定の確認であり、実行済みコマンドではありません。

```bash
oc get clusterversion,clusteroperator
oc get subscription,csv,installplan -n openshift-cnv
oc get hyperconverged -n openshift-cnv
oc get kubevirt -n openshift-cnv
oc get nodes -o wide
oc get storageclass,volumesnapshotclass
oc get vm,vmi,dv,pvc -A
oc get network-attachment-definitions -A
oc get events -A --sort-by=.metadata.creationTimestamp
```

収集時は timestamp、cluster ID、実行者、対象 namespace、command version を記録し、Secret、token、内部識別子を外部資料から除外します。

## 10. 障害・運用観点

| 事象 | 一次確認 | 設計上の備え |
| --- | --- | --- |
| VM Pending | scheduler event、request、affinity、PVC | spare capacity、placement rule、runbook |
| DataVolume stalled | importer pod、PVC、route、storage event | transfer network、capacity、timeout、再開条件 |
| live migration pending/fail | storage mode、CPU、memory、bandwidth | RWX、CPU compatibility、dedicated network |
| guest network unavailable | NAD、Multus、VLAN、MTU、guest NIC | mapping review、test IP、rollback path |
| node failure | node/VM state、fencing、storage attach | remediation 方針と double-start 防止 |
| backup fail | Backup/PodVolume/CSI event、object storage | alert、retry、restore drill、retention |

詳細な runbook と責任分界は [運用引き継ぎ](20-operations-handover.md) に記載します。

## 11. 未確定事項

| ID | 未確定事項 | 影響 | 確認先 | 期限 |
| --- | --- | --- | --- | --- |
| Q-VIRT-01 | 正確な OCP 4.22.z と Virtualization CSV | support と update | Platform / Red Hat | 設計 Gate 1 前 |
| Q-VIRT-02 | MTV 対応版・channel | 移行 plan 作成可否 | Platform / Red Hat | PoC 構築前 |
| Q-VIRT-03 | vSphere version、権限、VDDK | source 接続と転送 | VMware owner | PoC 構築前 |
| Q-VIRT-04 | CSI product と RWX/Block 性能 | live migration と RTO | Storage team | 詳細設計前 |
| Q-VIRT-05 | VLAN、MTU、IP 保持方式 | network cutover | Network team | plan 作成前 |
| Q-VIRT-06 | guest OS と application vendor support | 業務継続・license | Application team | Go/No-Go 前 |
| Q-VIRT-07 | backup/restore product compatibility | RPO/RTO | Platform + Storage | PoC 試験前 |

## 12. 公式資料

基準日現在に参照した一次資料です。製品変更があるため、実施時点で再確認します。

- [Red Hat OpenShift Container Platform 4.22: Installing OpenShift Virtualization](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/virtualization/installing)
- [Red Hat OpenShift Container Platform 4.22: Live migration](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/virtualization/live-migration)
- [Red Hat OpenShift Container Platform 4.22: Nodes and maintenance](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/virtualization/nodes)
- [Red Hat OpenShift Container Platform 4.22: Backup and restore for VMs](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/virtualization/backup-and-restore)
- [Migration Toolkit for Virtualization 2.12 documentation](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.12)
- [MTV 2.12: Planning your migration](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.12/html/planning_your_migration_to_red_hat_openshift_virtualization/)
- [MTV 2.12: Cold and warm migration](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.12/html/planning_your_migration_to_red_hat_openshift_virtualization/assembly_cold-warm-migration_mtv)
- [MTV 2.12: Migrating from VMware vSphere](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.12/html/migrating_your_virtual_machines_to_red_hat_openshift_virtualization/assembly_migrating-from-vmware_mtv)

## 13. レビュー記録

| 観点 | 状態 | 備考 |
| --- | --- | --- |
| 本人による内容理解 | 未確認 | 学習ガイドに沿って確認する |
| Platform review | 未実施 | 担当未定 |
| Network / Storage review | 未実施 | 製品・方式未確定 |
| Application acceptance review | 未実施 | 代表 VM owner は架空 |
| Red Hat compatibility check | 未実施 | 実施時点の資料で確認 |
