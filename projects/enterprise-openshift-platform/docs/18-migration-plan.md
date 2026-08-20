# 18. 移行計画書

> [!IMPORTANT]
> 本書は一般公開用の架空プロジェクトにおける移行計画サンプルです。移行は **未実施** で、日付、担当者、製品版、停止時間は未承認です。3 VMの移行は本番承認ではなく、技術課題を抽出するPoCとして扱います。

## 1. 文書情報

| 項目 | 値 |
| --- | --- |
| 文書 ID | MIG-PLAN-001 |
| 対象案件 | Example Enterprise OpenShift 基盤導入 |
| 版 | 0.1 Draft |
| 基準日 | 2026-08-17 |
| ステータス | Draft・未レビュー・未承認 |
| Migration manager | 未定（役割のみ定義） |
| Change ID | 未採番 |
| 実施状態 | 全 wave Not Run |

基盤条件は [SCENARIO](../SCENARIO.md)、VM 設計は [Virtualization・MTV 設計](11-virtualization-mtv-design.md)、試験は [試験仕様書](16-test-specification.md)、復帰は [切り戻し計画](19-rollback-plan.md) に従います。

## 2. 目的と成功条件

目的は、構築済みと仮定した OpenShift 基盤へ sample container workload を配置し、続いて 3 台の代表 VM を一台ずつ移行する机上計画を作り、技術・運用・業務課題を抽出することです。

完了条件:

1. 要件、設計、build、test、migration、rollback の ID が追跡できる。
2. 各 wave に事前条件、実行責任、定量的な受入条件、中断条件がある。
3. Source と destination の同時書込みを防止する。
4. 3 VM の [PoC 受入基準](11-virtualization-mtv-design.md#8-poc-受入基準)を評価できる証跡項目がある。
5. Pass / Fail / Blocked / Not Run を区別し、未実施を成功扱いにしない。
6. 本番移行へ進めるかではなく、次の設計判断に必要な制約と実測値を提示する。

## 3. Scope

### In scope

- Wave 0: sample container workload の deployment と route 確認
- Wave 1: `poc-web-01` cold migration
- Wave 2: `poc-app-01` warm migration candidate
- Wave 3: `poc-db-01` cold migration
- application smoke、network、storage、performance、backup/restore、rollback rehearsal
- source VM retention、evidence preservation、PoC review

### Out of scope

- production VM の一括移行、VMware retirement
- Kong / Sysdig の production integration（本計画では設計のみ）
- unsupported guest/device の workaround 実装
- DNS、IP、certificate の実在値または credential

## 4. Migration strategy

### 4.1 Principle

- 1 wave 1 VM とし、前 wave の exit criteria を満たすまで次へ進まない。
- Cold migration を default とし、warm migration は対応条件が確認できた `poc-app-01` だけの候補とする。
- 移行元 VM は rollback 保持期限まで削除、rename、disk consolidation、snapshot cleanup しない。
- Destination の業務公開前に source を停止または write-fence し、一つだけを authoritative instance にする。
- MTV plan の自動変換結果を完成設計とみなさず、NIC、disk、firmware、service、guest tools を照合する。

### 4.2 Wave overview

| Wave | Workload | Method | Planned outage | Primary validation objective | Current status |
| --- | --- | --- | --- | --- | --- |
| 0 | sample container workload | manifests / build procedure | 非本番のため N/A | Route、Service、RBAC、NetworkPolicy の基本経路 | Not Run |
| 1 | `poc-web-01` | MTV cold | 実測して記録 | 基本 transfer / conversion / boot / network | Not Run |
| 2 | `poc-app-01` | MTV warm candidate。条件不成立時 cold | cutover 時間を実測 | CBT/precopy、外部 DB/API、systemd service | Not Run |
| 3 | `poc-db-01` | MTV cold | DB 停止から受入まで実測 | write fencing、data consistency、large disk、restore | Not Run |

停止目標を架空に固定せず、source baseline と PoC 測定値から本番計画用の値を算出します。全体の RPO 1 時間 / RTO 4 時間は要件であり、MTV の成功だけで達成を証明しません。

## 5. Roles and RACI

凡例: `A` 最終責任、`R` 実行、`C` 協議、`I` 報告。

| Activity | Change manager | Migration manager | Platform | VMware | Network | Storage | Security | Application |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Plan approval / Go-No-Go | A | R | C | C | C | C | C | C |
| OCP / Virtualization readiness | I | C | A/R | I | C | C | C | I |
| Source inventory / stop / start | I | C | C | A/R | C | I | C | C |
| MTV provider / mapping / plan | I | A | R | R | C | C | C | C |
| DNS / LB / Firewall cutover | I | C | C | I | A/R | I | C | C |
| Storage capacity / snapshot | I | C | C | C | I | A/R | I | C |
| Credential / audit | I | C | R | R | I | I | A | I |
| Application freeze / validation | I | C | C | C | C | C | I | A/R |
| Rollback declaration | A | R | C | C | C | C | C | C |
| Evidence and closure | A | R | R | C | C | C | C | R |

`VMware` は source platform owner role を表します。実在担当者は未定です。

## 6. Relative schedule

実日付は change approval 後に設定します。

| Relative time | Milestone | Required output |
| --- | --- | --- |
| D-30 | Design Gate | version/compatibility、inventory、mapping、risk、license review |
| D-20 | Build Gate | OCP / Virtualization / MTV / CSI / network readiness evidence |
| D-15 | Baseline | source CPU/memory/disk/network、application test、backup duration |
| D-10 | Rehearsal | non-production rollback、communication、timing、evidence path |
| D-5 | Change review | approved procedure、rollback、contacts、decision rights |
| D-2 | Final preflight | plan validation、capacity、credential、certificate、alert status |
| D-1 | Freeze | source change freeze、config export、backup、DNS TTL confirmation |
| H-1 | Bridge open | attendance、clock、ticket、dashboard、stop authority |
| H0 | Wave start | timestamp と start authorization を記録 |
| H+n | Technical validation | VM / network / storage / platform tests |
| H+n+1 | Business acceptance | application owner acceptance or rollback request |
| H+n+2 | Observation | logs、alerts、performance、backup schedule |
| D+1 | Wave review | result、defects、lessons、next-wave decision |
| D+7 minimum | Source retention review | rollback period extension / source retirement decision |

## 7. Go / No-Go gates

### Gate G1: design readiness（D-30）

Go only if:

- exact OCP, Virtualization, MTV, vSphere, guest OS compatibility is recorded.
- CSI / network mapping and capacity are reviewed.
- all high risks have treatment and owner; unresolved risk acceptance is signed.
- application owner defines smoke, integrity, performance acceptance.

### Gate G2: environment readiness（D-2）

Go only if:

- OCP ClusterOperators and Virtualization components satisfy test criteria.
- no unrelated Severity 1/2 incident or change is active.
- MTV preflight has no unhandled Error; every Warning has disposition.
- backup, source start procedure, rollback route, communication bridge are ready.
- destination capacity includes current wave plus node-failure margin.

### Gate G3: cutover authorization（H0）

Go only if:

- Change manager states the approved Change ID and window.
- Application team confirms freeze / write fence.
- source health and last backup timestamp are recorded.
- no person required for rollback is absent.
- rollback decision deadline still leaves enough time to restore source service.

### Gate G4: accept destination

Go only if all mandatory technical and business tests Pass. `Blocked` is not Pass. If the remaining window cannot complete validation and rollback, declare No-Go before the deadline.

### Gate G5: close wave

Go only if monitoring, backup schedule, known defects, source retention, evidence location, and next action are handed over. G5 does not authorize source deletion.

## 8. Common runbook

すべて未実施の planned steps です。UI/API 名は選定 MTV version の公式手順へ合わせます。

### 8.1 Pre-cutover

1. Change ID、target VM、source/destination identifiers、operator versions を読み合わせる。
2. OCP、Virtualization、CSI、network、MTV provider / plan の health evidence を保存する。
3. Source VM configuration、disk、NIC、firmware、service、dependency、current snapshot を export する。
4. Application test baseline と performance baseline を取得する。
5. Backup completion と restore feasibility を Application / Storage が確認する。
6. DNS TTL、LB member、Firewall rule、monitoring maintenance を確認する。
7. Rollback手順を読み上げ、decision owner と deadline を確認する。
8. Gate G3 の各項目へ Pass / Fail / Blocked を記録する。

### 8.2 Cold migration branch

Cold migration では source VM を**全データ転送の前に停止し、転送中を通して停止状態に保ちます**。Warm migration の precopy 手順と混在させません。

1. Migration manager が approved cold plan と mapping の checksum / revision を照合する。
2. Application team が new request / batch / scheduler を停止し、write freeze を宣言する。
3. Application-consistent action と final backup / checkpoint を記録する。
4. Source VM を graceful shutdown し、power state が Off、書き込み先が一意であること、停止時刻を二者確認する。
5. 停止確認後にだけ MTV の cold migration を開始し、開始時刻を記録する。
6. VM ごとの conversion、transfer、DataVolume/PVC、events、controller logs を監視する。
7. Warning / retry / stall を記録し、threshold 超過または deadline 到達時は新規操作を止めて [切り戻し計画](19-rollback-plan.md)へ移る。

### 8.3 Warm migration branch

Warm migration では source VM を稼働させたまま precopy し、承認された cutover で停止して残差転送と conversion を完了します。採用版で VMware vSphere、各 VM / disk の CBT、snapshot 上限、VDDK 等の前提を満たす場合だけ使用します。

1. Migration manager が approved warm plan、mapping、CBT、cutover 条件の checksum / revision を照合する。
2. Source VM が稼働中であることを確認して MTV migration を開始し、precopy の開始時刻を記録する。
3. Snapshot / incremental transfer、changed data rate、DataVolume/PVC、events、controller logs を監視する。
4. Precopy の完了条件を確認し、承認された cutover time を MTV へ設定する。未承認の plan edit や方式変更は行わない。
5. Cutover 直前に Application team が new request / batch / scheduler を停止し、write freeze と final backup / checkpoint を記録する。
6. 承認時刻に MTV cutover を開始する。Source VM が停止された事実と時刻を確認し、remaining data transfer と conversion の完了を監視する。
7. Warning / retry / stall を記録し、threshold 超過または deadline 到達時は新規操作を止めて [切り戻し計画](19-rollback-plan.md)へ移る。

### 8.4 Destination validation and traffic cutover

1. Source VM が停止し、source と destination が同時に writeable でないことを二者確認する。
2. Destination VM を isolated validation network で起動する。
3. Disk、filesystem、NIC、route、time、service を確認する。
4. Network team が approved DNS/LB/Firewall cutover を実施する。
5. Application test を実施し、acceptance owner が Pass / Fail を記録する。

### 8.5 Post-cutover

1. Source VM が停止し、destination だけが authoritative であることを二者確認する。
2. Platform、VM、guest、application、network、storage の alert と log を確認する。
3. CPU/memory/disk/network/application latency を baseline と比較する。
4. backup schedule と next restore drill を登録する。
5. 未解決 defect に severity、owner、期限、workaround を付ける。
6. Gate G4 / G5 を判定し、次 wave の Go/No-Go を別途承認する。

## 9. Wave-specific procedure

### Wave 0: sample container workload

- [構築手順書](15-build-procedure.md) と `../manifests/` の exact revision を使用する。
- namespace、ServiceAccount、RBAC、Quota、LimitRange、NetworkPolicy、Deployment、Service、Route、PDB を順に確認する。
- これは VM migration 成功を示さない。OCP application path の基本試験として分離する。

### Wave 1: `poc-web-01`

- Cold migration で source shutdown から transfer / conversion を行う。
- static content checksum、HTTP health、TLS、DNS/LB、access log を確認する。
- rollback rehearsal を実施し、source restart と destination fencing の所要時間を測る。
- Wave 1 の mandatory test と rollback が完了するまで Wave 2 を開始しない。

### Wave 2: `poc-app-01`

- warm migration candidate とし、CBT、VDDK、source snapshot、precopy が対応する場合だけ選択する。
- cutover 前の precopy duration / changed data rate を記録する。
- source stop 後、systemd service、external DB/API、certificate trust、scheduled job、time sync を確認する。
- warm 条件が一つでも不成立なら change review を行い cold migration に切り替える。現場判断で方式変更しない。

### Wave 3: `poc-db-01`

- Cold migration とし、database owner が write fence と graceful shutdown を実施する。
- final backup、transaction / recovery log、database consistency check の基準を事前承認する。
- destination DB を production client から隔離して起動し、consistency と application read/write test 後に接続を切り替える。
- source と destination の DB を同時に writeable にしない。
- 500 GiB は架空値であり、転送時間と RTO は実測まで未達・未判定とする。

## 10. Stop and rollback triggers

一つでも該当すれば新しい操作を止め、Migration manager が Change manager へ rollback 判定を要求します。

- source backup / checkpoint が確認できない。
- preflight Error または新しい unsupported condition が発生した。
- source / destination 両方が writeable になる恐れがある。
- disk count/size、filesystem、database integrity が一致しない。
- destination boot loop、kernel panic、critical service failure が deadline までに解消しない。
- required network path、TLS、DNS が回復せず、rollback に必要な残時間へ達した。
- transfer rate から maintenance window 超過が予測される。
- OCP / storage / network に Severity 1/2 影響が発生した。
- audit trail、operator、application owner のいずれかが失われた。

詳細手順は [切り戻し計画](19-rollback-plan.md) を使用します。

## 11. Communications

| Timing | Sender | Audience | Minimum content |
| --- | --- | --- | --- |
| Start | Migration manager | all roles | Change ID、wave、start time、decision deadline |
| Every 30 min candidate | Technical lead | bridge | stage、progress、error、forecast、risk |
| Gate decision | Change manager | all roles | Go/No-Go、basis、approvers、timestamp |
| Service validation | Application owner | bridge | test IDs、result、known defect |
| Rollback | Change manager | users/operations | trigger、impact、expected recovery、next update |
| Closure | Migration manager | stakeholders | final state、evidence、issues、next wave |

実際の連絡先、chat、ticket、phone bridge は運用標準に従い、公開版へは含めません。

## 12. Evidence checklist

- [ ] Approved change and attendance record
- [ ] Source inventory and power state
- [ ] Exact product / operator / chart / VM versions
- [ ] Preflight and migration plan revision
- [ ] Start/end/cutover timestamps
- [ ] MTV per-VM status and sanitized logs
- [ ] PVC/DataVolume/disk mapping
- [ ] Network/DNS/LB before-after records
- [ ] Technical and business test results
- [ ] Performance comparison
- [ ] Backup and restore records
- [ ] Go/No-Go / rollback decisions
- [ ] Issue, risk, change, ADR updates

未チェックは未実施を意味します。

## 13. Source retention and closure

- Source VM は少なくとも D+7 の架空候補期間、停止・保護状態で保持する。正式期間は data、license、capacity、security 方針から承認する。
- Source snapshot を唯一の backup とみなさない。
- Source retirement は business acceptance、backup/restore、observation、audit、rollback-period close の別 change とする。
- MAC/IP/hostname duplicate を避けるため、retained source の起動権限と network isolation を制限する。
- PoC 完了後も VMware 廃止を推定しない。

## 14. Primary references

- [MTV 2.12: Planning your migration](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.12/html/planning_your_migration_to_red_hat_openshift_virtualization/)
- [MTV 2.12: Cold and warm migration](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.12/html/planning_your_migration_to_red_hat_openshift_virtualization/assembly_cold-warm-migration_mtv)
- [MTV 2.12: Migrating VMs from VMware vSphere](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.12/html/migrating_your_virtual_machines_to_red_hat_openshift_virtualization/assembly_migrating-from-vmware_mtv)
- [OpenShift Virtualization 4.22: Live migration](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/virtualization/live-migration)
