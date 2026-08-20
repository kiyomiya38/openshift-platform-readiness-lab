# 23. Architecture Decision Record（ADR）

> [!IMPORTANT]
> 本書は一般公開用の架空プロジェクトにおけるADRサンプルです。`シナリオ固定`はサンプル内の前提を表し、実案件の承認、構築、検証を意味しません。`Proposed` / `Deferred`は未決定です。

## 1. 文書情報

| 項目 | 値 |
| --- | --- |
| 文書 ID | ARC-ADR-001 |
| 版 | 0.1 Draft |
| 基準日 | 2026-08-17 |
| ステータス | Draft・未レビュー・未承認 |
| Decision authority | 案件責任者 / Architecture review board（未定） |

## 2. Status meanings

| Status | Meaning |
| --- | --- |
| シナリオ固定 | 架空プロジェクトの一貫性を保つために与えられた前提。実案件承認ではない |
| Proposed | review / evidence / approval 前の候補 |
| Accepted | authority が decision と consequences を承認 |
| Deferred | dependency 不成立または current scope 外で先送り |
| Superseded | newer ADR に置換。履歴は保持 |
| Rejected | 採用しない理由を記録して保持 |

## 3. ADR index

| ID | Decision | Status | Main consequence |
| --- | --- | --- | --- |
| ADR-001 | on-premises OCP を target platform とする | シナリオ固定 | infra responsibility を組織が持つ |
| ADR-002 | Agent-based Installer + platform `none` | シナリオ固定 | DNS/LB/host provisioning を外部で用意 |
| ADR-003 | OCP node は RHCOS、3 Control Plane + 3 Worker | シナリオ固定 | node OS lifecycle は MachineConfig/OCP に従う |
| ADR-004 | OpenShift Virtualization を OLM で後導入 | シナリオ固定 | OCP health 完了後、対応版を別 lifecycle で管理 |
| ADR-005 | VM storage は RWX + Block を第一候補 | Proposed | live migration に有利だが CSI/性能/費用を要検証 |
| ADR-006 | 代表VM 3台で技術課題を抽出するPoC | シナリオ固定 | 本番移行承認やbulk migrationとは分離 |
| ADR-013 | 架空VM名・wave順序、Cold default / Warm conditional | Proposed | 方式と順序はcompatibility・停止条件・実測で承認 |
| ADR-007 | source VM を rollback 期間保持し single-writer を強制 | Proposed | capacity/license を消費するが安全な rollback path を維持 |
| ADR-008 | Kong は design-only、KIC/Operator/topology 選定を延期 | Deferred | API Gateway を current build scope に入れない |
| ADR-009 | Sysdig は design-only、offering/agent/data 選定を延期 | Deferred | security/observability integration を current build scope に入れない |
| ADR-010 | connected + organization Proxy、Disconnected は対象外 | シナリオ固定 | external registry/catalog dependency を持つ |
| ADR-011 | OpenShift AI と production VM bulk migration は対象外 | シナリオ固定 | platform core と representative PoC に集中 |
| ADR-012 | evidenceはNot Run / Blocked / Fail / Passを明示 | シナリオ固定 | 未実施を完了・合格として記録しない |

## 4. ADR-001: Target platform

**Context:** 社内Web/APIと代表VMを共通基盤へ段階収容する架空シナリオです。ROSA / AROは比較対象ですがbuild対象ではありません。

**Decision:** on-premises x86_64 bare metal 上の OpenShift Container Platform 4.22.z（暫定）を target とします。

**Alternatives:** ROSA、ARO、managed Kubernetes、既存 VMware 継続。

**Consequences:** hardware、DNS、LB、NTP、Proxy、storage、IdP、backup の設計・運用責任が組織に残ります。OCP exact z、subscription、hardware compatibility は未確定です。

**Validation needed:** requirements fit、TCO、skill、support、RPO/RTO、managed service comparison。

## 5. ADR-002: Installation method

**Context:** Static IP、external DNS/LB、platform `none` の bare-metal environment を想定します。

**Decision:** Agent-based Installer を使い、platform `none` とします。

**Alternatives:** installer-provisioned infrastructure、Assisted Installer、manual UPI-style provisioning。

**Consequences:** host inventory、rendezvous、DNS/LB、port、ISO、BMC/manual boot、day-2 lifecycle を事前設計します。Agent-based Installer の current documented requirement を実施時に再確認します。

**Validation needed:** 4.22.z docs、host reachability、TCP/8090 discovery/bootstrap、API/MCS/Ingress health。

## 6. ADR-003: Node OS and topology

**Decision:** Control Plane 3、Worker 3、OCP node OS は RHCOS とします。RHEL worker を採用しません。

**Consequences:** quorum と worker failure domain を持ちますが、3 worker だけで VM + container + failure spare capacity が十分とは限りません。RHCOS は SSH/manual package management を通常運用にしません。

**Validation needed:** sizing、failure simulation、capacity、hardware compatibility、MachineConfig update。

## 7. ADR-004: OpenShift Virtualization installation

**Context:** OCP installation と VM platform enablement を段階分離します。

**Decision:** OCP 基盤試験後、OpenShift Virtualization Operator を OLM で `openshift-cnv` へ導入する方針です。

**Alternatives:** Agent-based / Assisted virtualization bundle、Virtualization を導入しない。

**Consequences:** OCP と対応する OpenShift Virtualization minor、stable channel、approval strategy、HyperConverged CR を管理します。MTV は別 Operator、別 compatibility check です。

**Validation needed:** exact CSV/channel、release notes、CPU/storage/network、Operator health、update procedure。

## 8. ADR-005: VM storage candidate

**Context:** node maintenance と live migration を候補要件とし、VM disk performance も必要です。

**Proposed decision:** CSI の RWX access mode + Block volume mode を第一候補とします。High-I/O DB 用 class は別候補とします。

**Alternatives:** RWO、Filesystem、local storage、VM ごとの live migration 不使用。

**Consequences:** shared storage product、cost、latency、snapshot、backup、failure domain の要件が増えます。RWX/Block だけで性能・可用性は保証されません。

**Validation needed:** Red Hat ecosystem support、CSI features、failure test、IO baseline、OADP/MTV compatibility。

## 9. ADR-006: MTV PoC boundary

**Context:** [SCENARIO](../SCENARIO.md)の`BR-009`は、VM移行を本番承認ではなく代表VM 3台のPoCに限定しています。VM名、役割、wave順序、Cold/Warm方式まではシナリオ固定値ではありません。

**Decision:** PoC対象を代表VM 3台に限定し、技術課題と実測値を収集します。PoC完了だけをproduction bulk migrationやVMware廃止の承認条件にはしません。

**Consequences:** 3台以外のguest、device、性能、運用条件を代表できるとは限らず、追加対象は別のscope/change/acceptanceが必要です。

**Validation needed:** `REQ-MTV-001`、`TST-MTV-001〜003`、PoC結果と未解決riskのreview。

## 10. ADR-013: MTV PoC wave and method candidate

**Context:** 3台の架空inventoryを用いて、障害影響を分離しながらCold/Warmの成立条件を学ぶための具体的な机上案が必要です。

**Proposed decision:** `poc-web-01`、`poc-app-01`、`poc-db-01`を一台ずつWave 1〜3で扱います。Coldをdefault、App VMだけをwarm candidateとします。compatibility、CBT、VDDK、snapshot、停止条件が成立しない場合は承認済みchangeでColdへ変更します。

**Alternatives:** 異なる代表VM・wave順序、一括plan、全VM Cold、全VM Warm、manual disk conversion。

**Consequences:** durationは長くなりますが、waveごとにissueを隔離し、Go/No-Goとrollbackを判断できます。この方式・順序はDraft・未レビュー・未承認で、実施済みdecisionではありません。

**Validation needed:** [PoC acceptance criteria](11-virtualization-mtv-design.md#8-poc-受入基準)、[Migration plan](18-migration-plan.md)、採用版compatibilityとsource baseline。

## 11. ADR-007: Rollback authority and source retention

**Proposed decision:** Source VM を D+7 minimum candidate の rollback period 中、停止・保護・network-controlled 状態で保持します。Destination を fence してから source を authoritative に戻します。

**Alternatives:** cutover 直後に source delete、bidirectional operation、backup restore のみ。

**Consequences:** temporary duplicate capacity/license cost が生じます。起動権限を制限しないと split-brain risk が増えます。

**Validation needed:** measured rollback duration、license、data delta handling、source security、retirement approval。

## 12. ADR-008: Kong deferral

**Context:** Kongは連携候補ですが、確定したproduct requirementとcontractがありません。

**Decision:** integration point だけ設計し、KIC / Kong Operator、DB-less / hybrid / Konnect、plugin、version、license の選定を延期します。

**Consequences:** current sample workload は default OpenShift Route で検証し、Kong availability/security を達成済みとしません。

**Revisit trigger:** API policy requirement、product owner、budget、data residency、OCP compatibility が確定した時。

## 13. ADR-009: Sysdig deferral

**Context:** Sysdig Secure/Monitor、SaaS/on-prem、Agent/Shield、module、data region が未定です。

**Decision:** egress、SCC/RBAC、Secret、event/metadata、SIEM の integration point だけ設計し、導入を延期します。

**Consequences:** built-in OpenShift monitoring/logging を含む actual observability design を別途成立させる必要があります。Sysdig coverage を達成済みとしません。

**Revisit trigger:** security use case、contract、data approval、agent compatibility、overhead test plan が確定した時。

## 14. ADR-010 and ADR-011: Scope limits

**Decision:** organization Proxy 経由の connected architecture とし、Disconnected、OpenShift AI、本番 VM bulk migration は対象外です。

**Consequences:** external catalog/registry endpoint の availability と Proxy CA を管理します。対象外項目は将来 issue として評価し、現設計へ暗黙に含めません。

## 15. ADR-012: Evidence semantics

**Context:** [SCENARIO](../SCENARIO.md)の`BR-010`で、実施していない確認を成功扱いせず、未実施と要確認を明示することが固定されています。

**Decision:** test / review / validationは`Not Run`、`Blocked`、`Fail`、`Pass`を区別し、計画上のexpected resultをactual resultとして扱いません。

**Consequences:** document completeness、technical review、approval、execution evidenceを分けて追跡できます。公開前に[公開・機密除去基準](../evidence/publication-safety.md)をレビューします。

## 16. ADR template

```text
ADR-ID / Title:
Status:
Date / Deciders:
Context and constraints:
Decision:
Alternatives considered:
Positive consequences:
Negative consequences / risks:
Validation and evidence:
Revisit trigger:
Related requirements / changes / risks:
```

## 16. Review rule

- ADR は設計を説明しますが implementation/test evidence の代替ではありません。
- Proposed を実施する前に Change approval が必要です。
- 前提が変わった場合は過去 ADR を書き換えて履歴を消さず、Superseding ADR を追加します。
- Product/version に依存する decision は official release/support information を添付します。
