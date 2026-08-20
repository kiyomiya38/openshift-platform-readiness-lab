# 20. 運用引き継ぎ書

> [!IMPORTANT]
> 本書は一般公開用の架空プロジェクトにおける運用引き継ぎサンプルです。クラスタ、VM、Kong、Sysdigは構築されておらず、runbookも未実施です。運用受け入れとSLA達成はいずれも未判定です。

## 1. 文書情報

| 項目 | 値 |
| --- | --- |
| 文書 ID | OPS-HO-001 |
| 対象案件 | Example Enterprise OpenShift 基盤導入 |
| 版 | 0.1 Draft |
| 基準日 | 2026-08-17 |
| ステータス | Draft・未レビュー・未承認 |
| Service owner | 未定 |
| Operations acceptance | 未実施 |

## 2. Service summary

| 項目 | 内容 |
| --- | --- |
| Service | On-premises OpenShift shared platform（架空） |
| OCP | 4.22.z（暫定、正確な z は要確認） |
| Topology | Control Plane 3、Worker 3、platform `none` |
| Platform services | external DNS / LB / NTP / Proxy / CSI / IdP |
| Workload | sample container workload、OpenShift Virtualization PoC VM 3 台 |
| Optional integration | Kong / Sysdig は設計のみ。運用対象外 |
| Availability target | monthly 99.9%（架空の目標、保証・実測なし） |
| Data target | application RPO 1 hour / RTO 4 hours（未検証） |
| Service hours | 24x365 想定。planned maintenance は事前承認 |

Architecture は [基本設計](05-basic-design.md)、network は [Network・DNS・LB 設計](07-network-dns-lb-design.md)、storage/backup は [Storage・Backup 設計](09-storage-backup-design.md)、Virtualization は [Virtualization・MTV 設計](11-virtualization-mtv-design.md) を参照します。

## 3. Handover package

| Category | Artifact | Acceptance condition |
| --- | --- | --- |
| Requirements | [Requirements](01-requirements.md)、[Traceability](04-requirements-traceability.md) | approved requirement IDs |
| Architecture | [Basic design](05-basic-design.md)、[Architecture](06-architecture-design.md) | diagram and values consistent |
| Detailed design | [Detailed design](13-detailed-design.md)、[Parameter sheet](14-parameter-sheet.md) | final as-built revision |
| Build | [Build procedure](15-build-procedure.md)、`../install/`、`../ansible/`、`../manifests/` | executed revision and deviations recorded |
| Internal Image Registry | [Storage・Backup設計](09-storage-backup-design.md)、`TST-REG-001` | production向け永続storageで`Managed`、Operator/Pod/PVC正常、push/pull・永続性がPass |
| Test | [Test specification](16-test-specification.md)、[Test results](17-test-results.md) | mandatory tests がすべて Pass。Fail / Blocked / Not Run は No-Go |
| Migration | [Migration plan](18-migration-plan.md)、[Rollback plan](19-rollback-plan.md) | approved wave result and rollback measure |
| Management | [Issue/Risk](21-issue-risk-register.md)、[Change](22-change-register.md)、[ADR](23-architecture-decisions.md) | open items have owner/due date |
| Evidence | [Project evidence index](../evidence/README.md) | sanitized, immutable location and access |

現時点では as-built、test evidence、actual contacts がなく、handover package は未完成です。

## 4. Responsibility boundary

[責任分界図](../diagrams/responsibility-boundary.mmd)を併読します。

| Domain | Accountable / Responsible | Consulted | Boundary |
| --- | --- | --- | --- |
| OCP API / Operator / cluster config | Platform team | Security、Network、Storage | physical / external service は各 team |
| Bare metal / BMC / firmware | Hardware team | Platform | guest/app 内は対象外 |
| DNS / LB / Firewall / NTP / Proxy | Network/Infrastructure team | Platform、Security | Route / Service は Platform と共同 |
| CSI / capacity / snapshot | Storage team | Platform、Application | DB consistency は Application |
| IdP / certificate / audit / vulnerability | Security team | Platform | namespace RBAC 操作は Platform |
| Application / DB / business test | Application team | Platform、Network、Storage | cluster control plane は対象外 |
| Backup / restore | Platform + Storage + Application | Security | object/PV/app consistency を分離 |
| Kong / Sysdig | Product owner + Platform + Security | Network、Application | 未選定・未導入のため現運用対象外 |
| Change / incident decision | Service owner / Change manager | all teams | 正式組織規程を優先 |

## 5. Access and security operations

- IdP group と namespace role を使い、個人 ID で操作する。
- `cluster-admin` は routine task に使わず、time-bound approval と session evidence を要求する。
- break-glass account は sealed procedure、dual approval、use alert、post-use rotation を設計する。
- token、pull secret、certificate private key、BMC / vCenter credential を Git や ticket 本文へ置かない。
- service account token は workload identity の要件に合わせ、自動生成 long-lived token を前提にしない。
- quarterly candidate で group membership、ClusterRoleBinding、SCC use、expired exception をレビューする。

## 6. Operational status model

| Severity | Example | Initial response target candidate | Escalation |
| --- | --- | --- | --- |
| Sev1 | API/Ingress 全停止、data loss、split-brain | 即時（正式値は要承認） | Incident commander + all domain owners + vendor |
| Sev2 | redundancy loss、重要 VM/API unavailable | 30 分候補 | Platform lead + affected domain |
| Sev3 | limited namespace impact、backup failure with valid prior backup | business hours or on-call policy | owning team |
| Sev4 | inquiry、planned improvement、low-risk warning | backlog/SLA candidate | service owner |

Severity と response target は架空候補です。組織標準、契約、on-call 体制で確定します。

### Health evidence baseline

以下は planned diagnostic commands で、実行済みではありません。

```bash
oc whoami
oc version
oc get clusterversion
oc get clusteroperators
oc get nodes -o wide
oc get pods -A --field-selector=status.phase!=Running,status.phase!=Succeeded
oc get events -A --sort-by=.metadata.creationTimestamp
oc get subscription,csv,installplan -A
oc get pvc,pv,storageclass -A
oc get route,service,endpointslice -A
```

巨大出力や Secret を ticket へ直接貼らず、approved evidence storage へ保存します。

## 7. Routine operations

| Frequency candidate | Task | Owner | Evidence |
| --- | --- | --- | --- |
| Daily | ClusterOperator、node、critical alert、failed backup、certificate alert | Platform | daily health record |
| Daily | Image Registry Operator、registry Pod/PVC、storage容量・失敗alert | Platform + Storage | config/Pod/PVC/alert record |
| Daily | DNS/LB/NTP/Proxy、CSI backend critical status | each infrastructure team | monitoring event / ticket |
| Weekly | capacity trend、pending pod/PVC、top alert、failed job、VM status | Platform + Storage | trend report |
| Weekly | security finding / exception / audit ingestion | Security | finding review |
| Monthly | patch/update advisory、operator channel、known issue、support lifecycle | Platform | update assessment |
| Monthly | restore sample / schedule review candidate | Platform + App + Storage | restore test record |
| Quarterly | RBAC/SCC/secret/certificate/access review | Security + Platform | access review approval |
| Quarterly | capacity/failure-domain/RPO/RTO drill | Service owner + all teams | resilience report |

Frequency は対象 workload、contract、cost、RPO/RTO に基づき承認します。

## 8. Backup and recovery operations

### 8.1 Protection domains

| Domain | Mechanism candidate | Owner | Important boundary |
| --- | --- | --- | --- |
| etcd | OCP documented etcd backup + off-node protected copy | Platform | application/PV backup の代替ではない |
| Kubernetes app objects / PV | OADP + compatible CSI / object storage | Platform + Storage | full cluster restore ではない |
| Internal Image Registry | production向け永続storage + image再生成/OADP適用評価 | Platform + Storage + Application | `Removed`を正常運用とせず、image source・digest・再生成性を別管理 |
| VM object / disk | OADP with OpenShift Virtualization supported options | Platform + Storage | guest/app consistency を別確認 |
| Application DB | DB-native backup/log + storage protection | Application + Storage | crash-consistent snapshot だけに依存しない |
| External systems | system owner procedure | respective owner | OpenShift scope 外 |

### 8.2 Schedule candidate

- application RPO 1 hour に対し、DB log / backup / OADP schedule の effective recovery point が 1 時間以内か測定する。
- etcd backup は daily と high-risk change 前を候補にする。official support procedure と retention を確認する。
- backup success alert だけでなく、object existence、age、encryption、retention、restore completion を監視する。
- monthly candidate で isolated restore、quarterly candidate で cross-team recovery drill を行う。

実際の schedule、retention、OADP/CSI version は未確定です。

## 9. Change and update operations

1. Advisory、release notes、support lifecycle、Operator compatibility を確認する。
2. Scope、risk、dependency、backup、rollback、test を [Change register](22-change-register.md) に記録する。
3. non-production / canary で検証し、observability と workload impact を測る。
4. OCP、Image Registry、OpenShift Virtualization、MTV、CSI、Kong、Sysdig の supported sequence を確認する。
5. maintenance window と decision deadline を承認する。
6. before/after health、version、events、test evidence を保存する。
7. defect と drift を as-built / parameter / ADR へ反映する。

OpenShift Virtualization は対応する OCP minor と組み合わせる必要があります。MTV、Kong、Sysdig はそれぞれ別 lifecycle であり、OCP update だけから互換性を推定しません。

## 10. Runbooks

各 runbook は、観測事実と仮説を分け、変更操作の前に impact と approval を確認します。

### RUN-OCP-01: ClusterOperator Degraded

**Trigger:** required ClusterOperator が monitoring window を超えて `Degraded=True` または `Available=False`。

1. `oc get clusteroperators` で operator、condition、時刻を記録する。
2. `oc describe clusteroperator <name>` と関連 namespace の pod/event を確認する。
3. recent change、node、DNS、Proxy、certificate、storage dependency を相関する。
4. unsupported manual edit / pod delete は行わず、operator-specific official docs を確認する。
5. multiple operators、API/Ingress impact、upgrade block の場合は Sev を上げる。
6. Red Hat case が必要なら sanitized `oc adm must-gather`、Cluster ID、時刻、impact を準備する。
7. recovery 後、condition duration、cause、action、recurrence test を記録する。

### RUN-OCP-02: Node NotReady / hardware failure

**Trigger:** node `Ready=False/Unknown`、heartbeat loss、hardware alert。

1. workload / VM / control-plane impact と failure domain を確認する。
2. node condition、event、MachineConfigPool、BMC / switch / storage alert を収集する。
3. VM の power state、migration / eviction state、double-start risk を確認する。
4. cordon/drain/remediation/fencing は別の approved procedure とし、診断だけで実行しない。
5. Control Plane quorum または複数 node 影響なら Sev1 とする。
6. Hardware / Network / Storage team へ同一 timestamp で escalation する。

### RUN-NET-01: API / Ingress unavailable

**Trigger:** `api.ocp-prd.example.com` または `*.apps.ocp-prd.example.com` の health failure。

1. client DNS answer、TCP、TLS、HTTP を分けて確認する。
2. external LB member / health、API server / router pod、Service / EndpointSlice を確認する。
3. API VIP `6443`、MCS `22623`、Ingress `80/443` の path を責任分界に沿って確認する。
4. DNS/LB change、certificate、firewall session、router rollout を相関する。
5. bypass や certificate validation disable を恒久 workaround にしない。
6. recovery 後、external と internal test point の両方で確認する。

### RUN-STG-01: PVC Pending / storage latency

**Trigger:** PVC Pending、attach/mount failure、VM/application I/O alert。

1. PVC/PV/StorageClass/VolumeSnapshot と event を収集する。
2. CSI controller/node pods、backend capacity/health、access/volume mode を確認する。
3. VM の live migration、node drain、backup、snapshot job との競合を確認する。
4. PVC/PV finalizer 削除、force detach、manual attach を無承認で行わない。
5. latency と availability impact により Storage team と vendor へ escalation する。
6. recovery 後、data integrity と application test を行う。

### RUN-REG-01: Internal Image Registry unavailable

**Trigger:** Image Registry Operatorが`Available=False`、`managementState`が承認状態と不一致、registry Pod/PVC異常、push/pull失敗、またはstorage容量alert。

1. `configs.imageregistry.operator.openshift.io/cluster`の`managementState`とstorage設定を記録する。
2. Image Registry Operator、deployment/pod、PVC/PV/StorageClass、event、storage backendを照合する。
3. Proxy/CA/DNS、internal route、credential、容量、recent changeを切り分ける。Secretやpull secretはticketへ貼らない。
4. 障害回避として`Removed`へ戻すことをproduction復旧または受入成功と扱わない。変更・影響・再構成期限を承認する。
5. recovery後は `TST-REG-001` のOperator/Pod/PVC、承認済みtest imageのpush/pull、再作成後の永続性を再確認する。

### RUN-VIRT-01: VM not running / migration failed

**Trigger:** VM expected running に対し VMI 不在、VMI Failed、live/MTV migration stalled。

1. `VirtualMachine` desired state、VMI、launcher pod、DataVolume/PVC、event を照合する。
2. scheduler capacity、CPU compatibility、affinity、storage mode、network attachment を確認する。
3. source/destination power と write authority を先に確認し、二重起動を防ぐ。
4. MTV なら per-VM pipeline と controller log、preflight warning、mapping を確認する。
5. cutover window 内なら [切り戻し計画](19-rollback-plan.md)の trigger と deadline を優先する。
6. recovery 後、guest agent、disk/network、service、business acceptance を確認する。

### RUN-BKP-01: Backup failed / restore request

**Trigger:** schedule failure、backup age が RPO threshold 超過、approved restore request。

1. 対象 domain（etcd、object、PV、VM、DB）と latest valid recovery point を特定する。
2. OADP/CSI/object storage/DB-native job の status と event を確認する。
3. backup object を削除・上書きせず、retention / immutability を確認する。
4. RPO breach 見込みを Service owner へ通知する。
5. restore は isolation namespace/network と alternate name を用い、production endpoint と重複させない。
6. resource existence だけでなく、application/DB consistency と recovery time を記録する。

### RUN-SEC-01: Certificate expiry / authentication failure

**Trigger:** expiry threshold alert、TLS handshake failure、IdP login failure。

1. certificate subject/SAN/issuer/validity/chain と endpoint を記録する。private key は収集しない。
2. IdP、OAuth、Proxy CA、router default certificate、application certificate を分ける。
3. time sync、DNS、revocation、recent rotation、trust store を確認する。
4. self-signed replacement、verification disable、shared admin account を emergency default にしない。
5. Security owner と certificate issuer へ escalation し、approved overlap rotation を行う。

### RUN-INT-01: Kong unavailable（将来用・未適用）

Kong は未導入のため現運用対象外です。採用後は router → Kong proxy → upstream を request ID で分離し、data-plane replica、configuration sync、IdP、certificate、rate-limit backend を確認する runbook を version-specific に作ります。

### RUN-INT-02: Sysdig coverage loss（将来用・未適用）

Sysdig は未導入のため現運用対象外です。採用後は expected node count、last-seen、Agent/Shield version、driver、collector egress、buffer/drop、SCC/RBAC を確認し、application traffic を不用意に止めない runbook を作ります。

## 11. Escalation package

| Recipient | Minimum information | Exclude |
| --- | --- | --- |
| Red Hat Support | subscription/Cluster ID、OCP exact version、impact、timeline、must-gather | pull secret、未マスクの組織データ |
| Hardware vendor | model/serial per approved channel、firmware、BMC event、node impact | platform token |
| Storage vendor | CSI/backend version、volume ID per secure channel、latency/error/time | Kubernetes Secret、DB content |
| Kong / Sysdig support | exact product/chart/agent version、topology、sanitized log、reproduction | access key、payload、private key |
| Internal incident | service/namespace/workload、severity、time、known fact/action | speculation presented as fact |

Support contract と case creation procedure は未確定です。

## 12. Known gaps at handover

| ID | Gap | Operational impact | Owner |
| --- | --- | --- | --- |
| OPS-G01 | exact OCP/Operator versions 未確定 | runbook/API/support 不確定 | Platform |
| OPS-G02 | monitoring alert/threshold/contact 未設定 | detection/response 不可 | Platform + Service owner |
| OPS-G03 | CSI/OADP/restore 未選定・未試験 | RPO/RTO 未証明 | Storage + Platform + App |
| OPS-G04 | IdP/certificate/break-glass 未設計確定 | secure operation 不可 | Security |
| OPS-G05 | Virtualization/MTV PoC 未実施 | VM 運用不能 | Platform |
| OPS-G06 | Kong/Sysdig product/license 未選定 | integration runbook 未適用 | Product owner |
| OPS-G07 | actual on-call / vendor contacts なし | escalation 不可 | Service owner |
| OPS-G08 | Internal Image Registryのproduction storageと`TST-REG-001`未完了 | installer完了後もbuild/image提供を本番受入できない | Platform + Storage |

これらが残る現在状態では production handover を受領しません。

## 13. Handover acceptance checklist

- [ ] Service owner、operations owner、on-call、vendor support が実名で登録済み
- [ ] as-built と parameter sheet が actual environment と一致
- [ ] mandatory tests がすべて Pass で、Fail / Blocked / Not Run がない
- [ ] Internal Image Registryがproduction向け永続storageで`Managed`となり、`TST-REG-001`がPass
- [ ] non-mandatory の差異・既知制約には承認済み disposition、owner、期限がある
- [ ] backup と isolated restore の実測 RPO/RTO が記録済み
- [ ] monitoring / alert / ticket / escalation が end-to-end で試験済み
- [ ] access、break-glass、Secret、certificate rotation が試験済み
- [ ] node / operator / storage / network / VM runbook rehearsal 済み
- [ ] open issues / risks / changes に owner と期限あり
- [ ] evidence が sanitized and access-controlled
- [ ] Operations team が walkthrough を受講し、質問と correction を記録済み

すべて未チェックです。

## 14. Primary references

- [OpenShift Container Platform 4.22: Support and gathering cluster data](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/support/)
- [OpenShift Container Platform 4.22: Backup and restore](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/backup_and_restore/)
- [OpenShift Virtualization 4.22: Nodes and maintenance](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/virtualization/nodes)
- [OpenShift Virtualization 4.22: Backup and restore](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/virtualization/backup-and-restore)
- [Kong Gateway documentation](https://developer.konghq.com/gateway/)
- [Sysdig Secure documentation](https://docs.sysdig.com/en/docs/sysdig-secure/)
