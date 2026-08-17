# 21. 課題・リスク管理台帳

> [!IMPORTANT]
> 本台帳は架空案件の初期管理例です。記載した owner と期限は役割／milestone の候補であり、実在組織への割当や対応完了を示しません。初期状態はすべて未確認または Open です。

## 1. 文書情報

| 項目 | 値 |
| --- | --- |
| 文書 ID | PM-IR-001 |
| 版 | 0.1 Draft |
| 基準日 | 2026-08-17 |
| ステータス | 本人レビュー前・未承認 |
| Risk owner | 各行の role。実名未定 |
| Review cadence | weekly candidate + 各 Gate 前 |

## 2. Definitions

- **課題（Issue）**: すでに不足・未決定・阻害として存在する事項。
- **リスク（Risk）**: 将来発生する可能性があり、発生時に目的へ影響する不確実性。
- **Assumption**: 現時点で計画に使う仮定。確認できなければ issue / risk へ変換する。
- **Decision**: 選択肢と根拠を [Architecture Decision](23-architecture-decisions.md) に記録する。

## 3. Rating method

Likelihood と Impact を 1〜5 で評価し、`Score = Likelihood × Impact` とします。

| Score | Level | Required action |
| --- | --- | --- |
| 15〜25 | High | Gate 前に低減、回避、移転、または authority による明示受容 |
| 8〜14 | Medium | owner、trigger、treatment、due を設定 |
| 1〜7 | Low | monitoring と定期 review |

Impact `5` は score にかかわらず Change manager / Service owner へ報告します。数値は学習用 rule で、正式組織基準が優先します。

## 4. Issue register

| ID | Issue / current fact | Impact | Owner role | Due milestone | Next action | Status |
| --- | --- | --- | --- | --- | --- | --- |
| ISS-001 | 正確な OCP 4.22.z、channel、support lifecycle 未確定 | install/update/support 設計を確定できない | Platform | Design Gate | Red Hat release/lifecycle と subscription を確認 | Open |
| ISS-002 | Bare metal model、CPU、firmware、NIC、HCL 未確認 | install / virtualization 不成立の可能性 | Hardware + Platform | Procurement Gate | Red Hat Ecosystem Catalog と vendor support を照合 | Open |
| ISS-003 | CSI product、StorageClass、RWX/Block、snapshot、およびInternal Image Registryの`Removed`→`Managed`方式・時期が未選定 | VM/live migration/backup/RTOを設計できず、registry未構成のままでは本番受入No-Go | Platform + Storage | Post-install Acceptance Gate | candidate製品とcapacity/performanceを確認し、registry永続storageを構成して`TST-REG-001`を実施 | Open |
| ISS-004 | VMware/vCenter/ESXi version、権限、VDDK 条件未確認 | MTV provider / transfer 可否不明 | VMware owner | PoC Build Gate | inventory と MTV compatibility を確認 | Open |
| ISS-005 | guest OS/device/application support 未確認 | 3 VM の conversion/operation risk 不明 | Application + Platform | PoC Go/No-Go | VM ごとに support matrix と vendor condition を記録 | Open |
| ISS-006 | IdP、MFA、group、break-glass 手順未確定 | secure administration 不可 | Security | Security Gate | identity design review を開催 | Open |
| ISS-007 | LB product、redundancy、health check 未確定 | API/Ingress availability と test 不明 | Network | Build Gate | external LB design と failure test を確定 | Open |
| ISS-008 | OADP/object storage/CSI backup 組合せ未確定 | RPO/RTO を検証できない | Platform + Storage | Test Gate | supported version と restore plan を選定 | Open |
| ISS-009 | Kong product/version/topology/license 未選定 | API Gateway integration を構築できない | Product owner | Optional Product Gate | KIC/Operator/Konnect と plugin edition を比較 | Open |
| ISS-010 | Sysdig offering/version/module/license 未選定 | agent/data/security control を確定できない | Product owner + Security | Optional Product Gate | SaaS/on-prem、Agent/Shield、data residency を比較 | Open |
| ISS-011 | 実際の lab / cluster がない | build/test/migration evidence を取得できない | Learning owner | Before execution | approved lab と budget を用意する | Open |
| ISS-012 | 実運用担当、on-call、vendor contacts 未定 | handover/escalation 不可 | Service owner | Handover Gate | role に実名と連絡手段を割り当てる | Open |
| ISS-013 | certificate issuer、Proxy CA、rotation owner 未確定 | install、TLS、renewal の failure risk | Security | Build Gate | certificate inventory と責任者を確定 | Open |
| ISS-014 | source baseline と performance acceptance 未取得 | PoC 性能を判定できない | Application | D-15 | CPU/memory/IO/network/app baseline を測定 | Open |

## 5. Risk register

`Residual` は treatment 後に再評価する欄です。現在は未実施のため `TBD` とします。

| ID | Risk event and consequence | L | I | Score | Level | Preventive treatment | Trigger / contingency | Owner | Residual |
| --- | --- | ---: | ---: | ---: | --- | --- | --- | --- | --- |
| RSK-001 | node sizing 不足により workload/VM が Pending、障害退避不能 | 3 | 5 | 15 | High | baseline、capacity model、N+1 margin、load test | scheduling/usage threshold 超過で scale/resize change | Platform | TBD |
| RSK-002 | worker CPU 差異または virtualization extension 無効で VM/live migration 失敗 | 3 | 5 | 15 | High | hardware validation、CPU compatibility、preflight | migration fail 時は cold/placement/rollback | Hardware + Platform | TBD |
| RSK-003 | CSIがRWX/Block/snapshotを満たさない、またはRegistry永続storageを構成できず、VM運用・backup・image提供が不成立 | 4 | 5 | 20 | High | compatibility proof、vendor test、alternate storage、`TST-REG-001` | storage/registry test Failなら本番受入を止めproduct selectionへ戻す | Platform + Storage | TBD |
| RSK-004 | VM disk transfer が window/RTO を超過 | 4 | 4 | 16 | High | data size/changed rate/bandwidth baseline、wave 分割 | forecast 超過で cancel/rollback | Migration manager | TBD |
| RSK-005 | source/destination 二重起動で IP/DB split-brain | 3 | 5 | 15 | High | write fencing、two-person state check、isolated validation | duplicate 検知で write 停止・incident | Platform + Application | TBD |
| RSK-006 | guest OS/device/agent 非互換で boot/application failure | 3 | 5 | 15 | High | inventory、support matrix、representative PoC | unsupported なら exclude/retain on source | Application | TBD |
| RSK-007 | DNS/LB/Firewall 誤設定で API/Ingress/VM unavailable | 3 | 5 | 15 | High | peer review、staged test、before/after export | health Fail で previous config へ rollback | Network | TBD |
| RSK-008 | backup は成功表示でも restore/DB consistency 不成立 | 3 | 5 | 15 | High | isolated restore、DB-native test、checksum | RPO/RTO breach を宣言し recovery plan | App + Platform + Storage | TBD |
| RSK-009 | privileged Operator/Agent/SCC が node/cluster attack surface を拡大 | 3 | 5 | 15 | High | vendor manifest review、dedicated SA、least privilege | policy violation で deployment halt / isolate | Security | TBD |
| RSK-010 | Sysdig SaaS へ機密 data が許可なく送信される | 3 | 5 | 15 | High | data inventory、region/legal review、capture default off | egress stop、key rotate、incident | Security | TBD |
| RSK-011 | Kong outage/config error が全 API へ波及 | 3 | 5 | 15 | High | HA、declarative review、canary、config rollback | proxy health Fail で prior revision rollback | Product owner + Platform | TBD |
| RSK-012 | Operator automatic update が未試験時点で入る | 3 | 4 | 12 | Medium | maintenance policy、release monitoring、compatibility review | update alert で impact/rollback/vendor escalation | Platform | TBD |
| RSK-013 | Proxy/CA/certificate renewal failure で registry/operator/API 接続不能 | 3 | 4 | 12 | Medium | expiry monitoring、overlap rotation、trust inventory | failure 時は approved prior cert / issuer escalation | Security + Network | TBD |
| RSK-014 | monitoring noise / false positive により重大 alert を見逃す | 4 | 3 | 12 | Medium | owner、severity、tuning、synthetic alert test | SLO breach で rule/route review | Operations + Security | TBD |
| RSK-015 | documentation と as-built drift で誤操作 | 4 | 4 | 16 | High | Git revision、post-change update、drift review | mismatch 発見時は change freeze | Change manager | TBD |
| RSK-016 | 学習資料を実機経験として誤って表現する | 3 | 4 | 12 | Medium | status banner、evidence gate、本人説明確認 | external submission 前に claim audit | Learning owner | TBD |
| RSK-017 | AI 生成内容に誤り・古い API が含まれる | 4 | 4 | 16 | High | primary source、version pin、human review、lab validation | discrepancy なら correction と source log 更新 | Learning owner | TBD |

## 6. Risk response planning

| Response | Meaning | Example in this project |
| --- | --- | --- |
| Avoid | risky scope / method を選ばない | unsupported device VM を PoC から除外 |
| Reduce | likelihood / impact を下げる | one-VM wave、preflight、rollback rehearsal |
| Transfer | contract / vendor support へ一部移す | supported hardware/CSI/product subscription |
| Accept | authority が残余 risk と期限を承認 | low-impact known limitation を observation 付きで受容 |

「担当に確認する」だけでは treatment になりません。確認事項、判断者、deadline、decision criterion を書きます。

## 7. Review workflow

1. 新規 item を Issue または Risk として分類する。
2. fact、impact、affected requirement/design/change を記載する。
3. owner role と due milestone を決める。
4. Risk は L/I と treatment、trigger、contingency を定義する。
5. weekly / Gate review で status と evidence を更新する。
6. decision が必要なら ADR、implementation が必要なら Change を起票する。
7. Close は evidence と approver を記録した場合のみ行う。

## 8. Status rules

| Status | Definition |
| --- | --- |
| Open | owner / action を設定し未解決 |
| Investigating | evidence を収集中 |
| Waiting | external decision/dependency 待ち。期限と escalation あり |
| Mitigating | approved treatment を実施中 |
| Accepted | authority、理由、期限、residual risk を記録 |
| Closed | resolution evidence と closure approver あり |

期限超過を自動的に Closed にしません。

## 9. Update log template

| Date | Item ID | New fact / evidence | Rating/status change | Action / owner | Approver |
| --- | --- | --- | --- | --- | --- |
| YYYY-MM-DD | RSK-nnn | `<fact>` | `<before → after>` | `<action / role / due>` | `<role>` |

現時点で実対応の update log はありません。
