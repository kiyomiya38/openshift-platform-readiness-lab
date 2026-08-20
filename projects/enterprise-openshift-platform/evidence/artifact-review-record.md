# 成果物レビュー記録

> [!IMPORTANT]
> 本書は成果物単位のreview statusを管理します。初期状態はすべて`Draft / Not Reviewed / Not Approved`です。`Present`はファイルの存在だけを示し、技術的妥当性や承認を示しません。

## 1. Review status

| Status | Meaning |
| --- | --- |
| Not Reviewed | 指定観点によるレビュー記録なし |
| In Review | reviewer、scope、開始日を記録して確認中 |
| Needs Correction | 指摘と対応ownerが登録済み |
| Reviewed | 根拠、指摘、判断、open itemを含むレビュー記録あり |
| Not Approved | 正式承認なし |
| Approved | 承認者、日付、scope、条件を記録済み |

## 2. 文書レビュー台帳

| No. | Artifact | File state | Technical review | Source check | Open correction | Approval |
| ---: | --- | --- | --- | --- | --- | --- |
| S | [SCENARIO](../SCENARIO.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 00 | [Project charter](../docs/00-project-charter.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 01 | [Requirements](../docs/01-requirements.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 02 | [Assumptions / constraints](../docs/02-assumptions-constraints.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 03 | [Scope / responsibility](../docs/03-scope-responsibility.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 04 | [Requirements traceability](../docs/04-requirements-traceability.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 05 | [Basic design](../docs/05-basic-design.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 06 | [Architecture design](../docs/06-architecture-design.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 07 | [Network / DNS / LB](../docs/07-network-dns-lb-design.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 08 | [Security / identity](../docs/08-security-identity-design.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 09 | [Storage / backup](../docs/09-storage-backup-design.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 10 | [Observability / operations](../docs/10-observability-operations-design.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 11 | [Virtualization / MTV](../docs/11-virtualization-mtv-design.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 12 | [Kong / Sysdig](../docs/12-kong-sysdig-integration-design.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 13 | [Detailed design](../docs/13-detailed-design.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 14 | [Parameter sheet](../docs/14-parameter-sheet.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 15 | [Build procedure](../docs/15-build-procedure.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 16 | [Test specification](../docs/16-test-specification.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 17 | [Test results](../docs/17-test-results.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 18 | [Migration plan](../docs/18-migration-plan.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 19 | [Rollback plan](../docs/19-rollback-plan.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 20 | [Operations handover](../docs/20-operations-handover.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 21 | [Issue / risk](../docs/21-issue-risk-register.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 22 | [Change register](../docs/22-change-register.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 23 | [Architecture decisions](../docs/23-architecture-decisions.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |
| 24 | [Deliverable usage guide](../docs/24-deliverable-usage-guide.md) | Draft | Not Reviewed | Not Reviewed | None Recorded | Not Approved |

## 3. 構築資材レビュー台帳

| Artifact group | Scope | File state | Technical review | Static validation | Runtime validation | Approval |
| --- | --- | --- | --- | --- | --- | --- |
| [`install/`](../install/) | install-config、agent-config、DNS/LB、Butane | Draft | Not Reviewed | Partial Pass（YAMLのみ） | Not Run | Not Approved |
| [`ansible/`](../ansible/) | inventory、vars、preflight、LB playbook/template | Draft | Not Reviewed | Partial Pass（YAMLのみ） | Not Run | Not Approved |
| [`manifests/`](../manifests/) | Namespace、RBAC、Quota、NetworkPolicy、workload、Route | Draft | Not Reviewed | Pass（YAML parse / Kustomize renderのみ） | Not Run | Not Approved |
| [`diagrams/`](../diagrams/) | architecture、network、build、migration、responsibility | Draft | Not Reviewed | Not Run | Not Run | Not Approved |

## 4. レビュー記録テンプレート

```text
Review ID:
Artifact and exact revision:
Review date/time/timezone:
Reviewer role:
Review scope and checklist:
Primary source title/version/URL/section:
Finding:
Severity and impact:
Decision:
Correction / action owner / due:
Open question:
Review status:
Approval status:
Related issue/change/ADR:
```

## 5. `Reviewed`の判定条件

対象revision、reviewer role、確認観点、根拠、指摘、判断、未解決事項、actionを記録した場合にだけ`Reviewed`へ変更します。通読、ファイル生成、link掲載だけでは状態を変更しません。
