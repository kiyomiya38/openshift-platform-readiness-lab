# 24. 架空案件・実務文書 学習ガイド

> [!IMPORTANT]
> 本ガイドは、設計・構築・試験・移行・運用文書を順番に学ぶための手順です。成果物は AI 支援を含む draft であり、本人レビュー、実機実行、商用経験を示しません。理解・訂正・実施した事実だけを [学習ログ](../evidence/learning-log.md) に記録します。

## 1. 学習のゴール

この架空案件で身につける対象は、製品用語の暗記ではなく、次の連鎖です。

```text
要求
→ 前提・制約・責任分界
→ 基本設計と設計判断
→ 詳細値・構築手順
→ 判定可能な試験
→ 移行・切り戻し
→ 運用・課題・変更・証跡
```

完了時には、各値について「どの要求から来たか」「誰が決めるか」「何で確認するか」「失敗時にどう戻すか」を文書間で追跡できる状態を目指します。

## 2. Current start state

| Item | State |
| --- | --- |
| Existing repository `docs/00〜27` initial reading | User reported one pass completed |
| This fictional project `docs/00〜24` review | Not started / 本人レビュー前 |
| Official source verification by learner | Not started |
| Static validation by learner | Not started |
| OpenShift / Kubernetes lab | No environment; Not Run |
| Cluster build / test / migration | Not Run |
| Commercial experience | None claimed |

既存技術章の通読と、この project artifact の本人レビューは別に記録します。

## 3. Reading order

分かりやすさを優先し、**`docs/00` から `docs/24` まで番号順**に進めます。

| Order | Artifact | 主に学ぶこと | 作業成果 |
| ---: | --- | --- | --- |
| 00 | [Project charter](00-project-charter.md) | 案件目的、success、stage | 目的・非目的を 5 行に整理 |
| 01 | [Requirements](01-requirements.md) | business / functional requirement | requirement の曖昧語を抽出 |
| 02 | [Assumptions / constraints](02-assumptions-constraints.md) | fact、assumption、constraint、TBD | 確認先と影響を追記 |
| 03 | [Scope / responsibility](03-scope-responsibility.md) | in/out、team boundary、RACI | 抜け・重複責任を整理 |
| 04 | [Requirements traceability](04-requirements-traceability.md) | requirement → design → build → test | 3 要件を end-to-end 追跡 |
| 05 | [Basic design](05-basic-design.md) | architecture 方針と理由 | 選択肢と採否理由を整理 |
| 06 | [Architecture design](06-architecture-design.md) | component / dependency / failure domain | 図と本文の整合確認 |
| 07 | [Network / DNS / LB design](07-network-dns-lb-design.md) | address、name、port、traffic path | client-to-workload path を trace |
| 08 | [Security / identity design](08-security-identity-design.md) | IdP、RBAC、SCC、Secret、certificate | normal/denied/admin path を整理 |
| 09 | [Storage / backup design](09-storage-backup-design.md) | CSI、PV、RPO/RTO、restore | protection domain ごとに責任分離 |
| 10 | [Observability / operations design](10-observability-operations-design.md) | signal、alert、owner、response | 重大事象の detection-to-action を整理 |
| 11 | [Virtualization / MTV design](11-virtualization-mtv-design.md) | OLM、VM、MTV、compatibility、PoC | 3 VM の mapping と Gate を確認 |
| 12 | [Kong / Sysdig integration](12-kong-sysdig-integration-design.md) | API path、agent/data path、product boundary | design-only / version-license TBD を説明 |
| 13 | [Detailed design](13-detailed-design.md) | resource-level configuration | basic design との矛盾を抽出 |
| 14 | [Parameter sheet](14-parameter-sheet.md) | exact value、unit、source、setting target | scenario/diagram/procedure の値を照合 |
| 15 | [Build procedure](15-build-procedure.md) | pre-check、step、expected、stop、rollback | tabletop で手順を追跡 |
| 16 | [Test specification](16-test-specification.md) | requirement、precondition、operation、expected | 判定不能な表現を定量化 |
| 17 | [Test results](17-test-results.md) | Not Run / Blocked / Fail / Pass と evidence | 未実施が Pass でないことを確認 |
| 18 | [Migration plan](18-migration-plan.md) | wave、Go/No-Go、cutover、communication | Wave 1 を時系列で読み上げ |
| 19 | [Rollback plan](19-rollback-plan.md) | fencing、single-writer、deadline、recovery | trigger から復旧まで tabletop |
| 20 | [Operations handover](20-operations-handover.md) | service ownership、routine、runbook、escalation | 1 alert を end-to-end trace |
| 21 | [Issue / risk register](21-issue-risk-register.md) | fact vs uncertainty、score、treatment | current Open item を 3 件更新 |
| 22 | [Change register](22-change-register.md) | approval、implementation、validation、closure | CHG-005 を tabletop review |
| 23 | [Architecture decisions](23-architecture-decisions.md) | context、decision、alternative、consequence | Proposed と scenario-fixed を区別 |
| 24 | 本ガイド | 全成果物の結合 | evidence と次の学習計画を更新 |

## 4. 一つの文書を学ぶ手順

理解度テスト形式ではなく、各文書で同じ 6 作業を行います。

1. **Status を確認する**: Draft、Not Run、要確認、対象外を先に読む。
2. **入力を確認する**: requirement、前工程、SCENARIO、official source を特定する。
3. **判断を抽出する**: decision と単なる例示値を分ける。
4. **下流を追う**: parameter、procedure、test、rollback、runbook のどこへ反映されるか確認する。
5. **誤り・疑問を記録する**: 推測で直さず、source、影響、owner を付ける。
6. **自分の言葉で要約する**: 200〜400 字程度で学習ログへ記録する。

文書を開いただけでは `Reviewed` や `Understood` にしません。

## 5. Stage-by-stage learning plan

### Stage A: Requirement and project control（00〜04）

**作業:**

- BR-005（RPO 1 hour / RTO 4 hours）を requirement → storage design → test → operation まで追う。
- BR-008（change record）を change register → build/migration → evidence まで追う。
- BR-009（3 VM PoC）を scope → Virtualization design → migration → acceptance まで追う。
- assumption と confirmed fact が混ざっていれば issue を起票する。

**完了物:** requirement trace 3 本、open question list、責任分界 correction。

### Stage B: Platform basic design（05〜10）

**作業:**

- platform `none` で外部 DNS/LB が必要な理由を architecture / network 図で確認する。
- API 6443、MCS 22623、Ingress 80/443、discovery/bootstrap 8090 の phase と path を分ける。
- RHCOS、3+3 node、OVN-Kubernetes、Proxy、CSI、IdP の dependency を整理する。
- etcd、application object/PV、DB backup を同じものとして扱っていないか確認する。

**完了物:** one-page architecture、traffic matrix correction、failure-domain note。

### Stage C: Virtualization and optional integrations（11〜12）

**作業:**

- OCP installation、OpenShift Virtualization の OLM 導入、MTV の別 Operator 導入を時系列で分ける。
- `poc-web-01`、`poc-app-01`、`poc-db-01` の CPU/memory/disk/network/application dependency を比較する。
- MTV 2.12 文書は reference であり、採用版ではないことを確認する。
- Kong と Sysdig は product/version/license 未選定、design-only であることを integration point と分けて記録する。

**完了物:** 3 VM comparison、compatibility Gate list、Kong/Sysdig selection Gate list。

### Stage D: Detailed design and build package（13〜15）

**作業:**

- [SCENARIO](../SCENARIO.md)、parameter sheet、install example、Ansible inventory の hostname/IP/MAC/CIDR を横断照合する。
- placeholder、Secret、actual value の区別を確認する。
- [Install package](../install/README.md)、[Ansible package](../ansible/README.md)、[Manifest package](../manifests/README.md) の適用順と ownership を読む。
- procedure の各 change step に pre-check、expected result、stop condition、rollback があるか確認する。

**完了物:** value consistency sheet、tabletop execution record、要修正一覧。

> [!WARNING]
> 実在環境、production、共有 cluster へ example file を適用しません。Agent ISO 作成、DNS/LB 変更、Ansible、`oc apply` は approved disposable lab と change plan ができるまで Not Run のままにします。

### Stage E: Test design（16〜17）

**作業:**

- 「正常であること」のような表現を condition、value、duration、allowed difference へ変える。
- normal、error、recovery の 3 種類を requirement ごとに確認する。
- evidence に command output、timestamp、version、actor、target、log location を定義する。
- actual value がない result row は必ず `Not Run` にする。

**完了物:** test case correction、expected evidence list、execution prerequisites。

### Stage F: Migration and rollback tabletop（18〜19）

**作業:**

- D-30 から D+7 までの role と deliverable を読み上げる。
- 各 Gate で誰が Go/No-Go を決めるか RACI と照合する。
- Wave 1 → 2 → 3 を一度に進めない理由を risk register と結びつける。
- source/destination の power、network、write authority を時系列で一意にする。
- rollback deadline を仮の実測値で計算するが、actual result として保存しない。

**完了物:** tabletop minutes、decision timeline、rollback gap list。

### Stage G: Operations and governance（20〜23）

**作業:**

- one alert を trigger → evidence → triage → owner → approved action → validation → closure まで追う。
- issue、risk、change、ADR のどれに書くべきかを実例で分類する。
- CHG-005（Virtualization OLM）、CHG-009（DB VM）、CHG-010/011（Kong/Sysdig）を review する。
- handover checklist の未成立項目を production blocker と operational backlog に分類する。

**完了物:** runbook correction、3 management-record updates、handover gap report。

### Stage H: Evidence and claim boundary

**作業:**

- [Evidence index](../evidence/README.md) で本人 review / static validation / lab execution を分ける。
- [Artifact review record](../evidence/artifact-review-record.md)へ文書ごとの correction と review date を記録する。
- [Verification record](../evidence/verification-record.md)へ command、environment、actual result を実施後だけ記録する。
- [Submission boundary](../evidence/submission-boundary.md)で外部説明可能な範囲を確認する。

**完了物:** evidence status update。実施していない row は空欄にせず `Not Run` を維持する。

## 6. Artifact-to-source map

Primary official source を優先し、repository の既存章は日本語の導入／navigation として使います。

| Artifact | Repository learning source | Primary official source |
| --- | --- | --- |
| 00〜04 project control | [基盤案件の進め方](../../../docs/05-infra-project-process.md)、[設計文書ガイド](../../../docs/24-design-document-guide.md) | 組織の project/change standard（未提供） |
| 05〜06 platform | [OpenShift core](../../../docs/06-openshift-core-knowledge.md)、[基本設計](../../../docs/07-openshift-basic-design.md) | [OCP 4.22 documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/) |
| 07 network | [Network/DNS/LB](../../../docs/17-network-dns-lb-firewall.md) | [Agent-based Installer for on-premise OCP 4.22](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installing_an_on-premise_cluster_with_the_agent-based_installer/) |
| 08 security | [OpenShift core](../../../docs/06-openshift-core-knowledge.md) | [OCP 4.22 Authentication and authorization](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/authentication_and_authorization/) |
| 09 backup/storage | [Monitoring/log/backup](../../../docs/15-monitoring-logging-backup.md)、[Storage](../../../docs/16-storage-csi-odf.md) | [OCP 4.22 Backup and restore](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/backup_and_restore/) |
| 10 observability | [Monitoring/log/backup](../../../docs/15-monitoring-logging-backup.md) | [Monitoring stack for Red Hat OpenShift 4.22](https://docs.redhat.com/en/documentation/monitoring_stack_for_red_hat_openshift/4.22/) |
| 11 Virtualization/MTV | [OpenShift Virtualization](../../../docs/08-openshift-virtualization.md)、[MTV](../../../docs/10-mtv-vm-migration.md) | [OCP 4.22 Virtualization](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/virtualization/)、[MTV documentation](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/) |
| 12 Kong/Sysdig | [Kong](../../../docs/20-kong-api-ai-gateway.md)、[Sysdig](../../../docs/21-sysdig-container-security.md) | [Kong KIC](https://developer.konghq.com/kubernetes-ingress-controller/)、[Kong Operator](https://developer.konghq.com/operator/)、[Sysdig Docs](https://docs.sysdig.com/) |
| 13〜15 detail/build | [設計文書ガイド](../../../docs/24-design-document-guide.md)、[Ansible](../../../docs/12-ansible-automation.md) | Agent-based Installer docs + exact product docs selected before execution |
| 16〜17 test | [試験文書ガイド](../../../docs/25-test-document-guide.md) | OCP / product operation docs for each test target |
| 18〜19 migration/rollback | [MTV](../../../docs/10-mtv-vm-migration.md) | [MTV 2.12 planning](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.12/html/planning_your_migration_to_red_hat_openshift_virtualization/)（version selection pending） |
| 20 operations | [OpenShift troubleshooting](../../../docs/14-openshift-troubleshooting.md) | [OCP 4.22 Support](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/support/) |
| 21〜23 governance | [基盤案件の進め方](../../../docs/05-infra-project-process.md)、[設計文書ガイド](../../../docs/24-design-document-guide.md) | 組織標準（未提供）+ product primary sources |

## 7. Version verification procedure

1. OCP Product Documentation で `4.22` を選び、release notes と lifecycle を確認する。
2. deployment 直前に exact `4.22.z`、installer/client release image、channel を記録する。
3. OpenShift Virtualization は対応する OCP minor と exact CSV/channel を確認する。
4. MTV は OCP、Virtualization、source vSphere、guest OS、VDDK の compatibility を同一日付で確認する。
5. CSI/OADP は Red Hat Ecosystem Catalog、Operator matrix、storage vendor support を確認する。
6. Kong/Sysdig は contract された edition、exact version/chart/agent、OCP/RHCOS compatibility、deprecation を確認する。
7. source URL、title、product version、確認日、relevant section、design impact を source review log に記録する。

Search result や生成 AI の説明だけを compatibility 根拠にしません。

## 8. Environment-free work now possible

検証環境がなくても、次は実施できます。

- Markdown 内部リンクと図・表の名称整合確認
- YAML parse / schema-oriented static review（cluster admission の代替ではない）
- Ansible syntax / lint（対象 tool が安全に使える場合）
- CIDR overlap、IP/MAC/hostname、port、parameter consistency review
- procedure / test / rollback の tabletop walkthrough
- official source と設計 statement の照合
- issue/risk/change/ADR の更新

これらは static / desk review であり、cluster behavior の検証ではありません。

## 9. Lab becomes available later

実機フェーズは次の順にし、各段階を別 change とします。

1. disposable network / DNS / LB / six-node capacity を準備
2. preflight のみ
3. Agent-based OCP install
4. cluster baseline + sample workload
5. backup / isolated restore
6. OpenShift Virtualization OLM install
7. one test VM、storage/network/live-migration checks
8. MTV compatibility and provider setup
9. VM PoC Wave 1 → rollback rehearsal → Wave 2 → Wave 3
10. optional Kong/Sysdig product PoC（selection Gate 後のみ）

各段階の Fail / Blocked を解決または受容するまで次へ進みません。

## 10. Learning log format

```text
Date / duration:
Artifact and exact revision:
Primary source and version:
What I verified in my own words:
Correction / disagreement:
Still unknown:
Desk/static/lab activity actually performed:
Evidence location:
Result: Reviewed / Needs correction / Not Run
Next action and due date:
```

「読了」だけでなく、source と correction、次 action を残します。

## 11. Completion levels

| Level | Meaning | Required evidence |
| --- | --- | --- |
| L0 Generated | artifact が存在 | AI-supported draft。能力証跡ではない |
| L1 Reviewed | 本人が source と照合し correction を記録 | dated artifact review |
| L2 Explained | requirement-to-test / rollback を自分の言葉で説明 | learner-authored summary / walkthrough note |
| L3 Static validated | syntax/link/schema/tabletop checks | command/tool/version/actual output |
| L4 Lab validated | approved lab で実行・復旧 | environment、actual result、sanitized evidence |
| L5 Repeatable | 別条件でも再実施し差異を説明 | second run and change analysis |

本 project の初期状態は L0 です。L4/L5 に達しても商用経験とは区別します。

## 12. Recommended next action

まず `00`〜`04` を順に本人レビューし、BR-005、BR-008、BR-009 の 3 本を traceability 表で追います。その correction を [Artifact review record](../evidence/artifact-review-record.md) と [Learning log](../evidence/learning-log.md)へ記録した後、`05` へ進みます。
