# 成果物レビュー記録

> [!IMPORTANT]
> 初期状態はすべて本人レビュー前です。`Present` はファイルの存在だけを表し、内容の正確性や理解を表しません。

## 1. Review status definitions

| Status | Meaning |
| --- | --- |
| Not Started | 本人は未確認 |
| In Review | source と artifact を照合中 |
| Needs Correction | correction が必要 |
| Reviewed | source、correction、open question を記録済み |
| Approved | 指定 reviewer が承認済み |

## 2. Document review register

| Order | Artifact | File state | Learner review | Primary-source check | Correction / note | Reviewer approval |
| ---: | --- | --- | --- | --- | --- | --- |
| S | [SCENARIO](../SCENARIO.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 00 | [Project charter](../docs/00-project-charter.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 01 | [Requirements](../docs/01-requirements.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 02 | [Assumptions / constraints](../docs/02-assumptions-constraints.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 03 | [Scope / responsibility](../docs/03-scope-responsibility.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 04 | [Requirements traceability](../docs/04-requirements-traceability.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 05 | [Basic design](../docs/05-basic-design.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 06 | [Architecture design](../docs/06-architecture-design.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 07 | [Network / DNS / LB](../docs/07-network-dns-lb-design.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 08 | [Security / identity](../docs/08-security-identity-design.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 09 | [Storage / backup](../docs/09-storage-backup-design.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 10 | [Observability / operations](../docs/10-observability-operations-design.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 11 | [Virtualization / MTV](../docs/11-virtualization-mtv-design.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 12 | [Kong / Sysdig](../docs/12-kong-sysdig-integration-design.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 13 | [Detailed design](../docs/13-detailed-design.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 14 | [Parameter sheet](../docs/14-parameter-sheet.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 15 | [Build procedure](../docs/15-build-procedure.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 16 | [Test specification](../docs/16-test-specification.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 17 | [Test results](../docs/17-test-results.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 18 | [Migration plan](../docs/18-migration-plan.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 19 | [Rollback plan](../docs/19-rollback-plan.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 20 | [Operations handover](../docs/20-operations-handover.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 21 | [Issue / risk](../docs/21-issue-risk-register.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 22 | [Change register](../docs/22-change-register.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 23 | [Architecture decisions](../docs/23-architecture-decisions.md) | Present | Not Started | Not Started | None recorded | Not Approved |
| 24 | [Learning guide](../docs/24-learning-guide.md) | Present | Not Started | Not Started | None recorded | Not Approved |

## 3. Build artifact review

| Artifact group | Scope | Learner review | Static validation | Lab validation |
| --- | --- | --- | --- | --- |
| [`install/`](../install/) | install-config、agent-config、DNS/LB、Butane examples | Not Started | Not Run | Not Run |
| [`ansible/`](../ansible/) | inventory、vars、preflight、LB playbook/templates | Not Started | Not Run | Not Run |
| [`manifests/`](../manifests/) | sample namespace/RBAC/quota/network/workload/route | Not Started | Not Run | Not Run |
| [`diagrams/`](../diagrams/) | architecture/network/build/migration/responsibility | Not Started | Not Run | Not Run |

## 4. Review entry template

```text
Artifact:
Exact revision / hash:
Review date and duration:
Primary source title / version / URL:
Relevant source section:
What the artifact says:
What I verified:
Correction / disagreement:
Still unknown and impact:
Next action / owner / due:
Review status:
```

## 5. Review completion rule

`Reviewed` にできるのは、少なくとも primary source、artifact revision、本人の要約、correction または「correction なし」の根拠、open question、next action を記録した場合です。単なる通読では `Reviewed` にしません。
