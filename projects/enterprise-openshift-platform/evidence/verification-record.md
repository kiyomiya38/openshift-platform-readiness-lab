# 検証記録

> [!IMPORTANT]
> 初期状態ではすべて **Not Run** です。本書の expected result や command 例は actual test result ではありません。

## 1. Environment

| Item | Value |
| --- | --- |
| Approved lab | None |
| OpenShift cluster | None |
| Kubernetes cluster | None |
| VMware source | None |
| Kong / Sysdig account | None |
| Test data | None |
| Executed by learner | No |

## 2. Planned verification register

| ID | Category | Scope | Tool / environment | Expected evidence | Status | Actual result |
| --- | --- | --- | --- | --- | --- | --- |
| VER-001 | Markdown | relative links / headings | local parser candidate | broken-link list | Not Run | None |
| VER-002 | Mermaid | `.mmd` and embedded diagrams | Mermaid CLI candidate | render output / error | Not Run | None |
| VER-003 | YAML syntax | install/manifests/Ansible vars | YAML parser candidate | file-by-file parse result | Not Run | None |
| VER-004 | Kubernetes schema | manifests against target APIs | target `oc` / server-side dry-run | admission result | Not Run | None |
| VER-005 | Butane | `.bu.example` rendering | target-supported Butane | generated MachineConfig diff | Not Run | None |
| VER-006 | Ansible syntax | playbooks/inventory/templates | pinned ansible-core / collections | syntax/lint result | Not Run | None |
| VER-007 | Parameter consistency | SCENARIO/docs/install/ansible | desk review / script | mismatch list | Not Run | None |
| VER-008 | Build tabletop | build procedure | cross-team desk review | timing/stop/rollback notes | Not Run | None |
| VER-009 | Test review | test spec/result | desk review | ambiguity/gap list | Not Run | None |
| VER-010 | Migration tabletop | 3 VM waves | migration bridge simulation | Go/No-Go and timing notes | Not Run | None |
| VER-011 | Rollback rehearsal | source/destination fencing | approved lab | measured recovery and data result | Not Run | None |
| VER-012 | OCP install | Agent-based Installer | approved six-node lab | install/health actual evidence | Not Run | None |
| VER-013 | Backup/restore | etcd/app/PV/VM/DB | approved lab + storage | actual RPO/RTO | Not Run | None |
| VER-014 | OpenShift Virtualization | OLM/HCO/VM/live migration | compatible lab | exact version and test output | Not Run | None |
| VER-015 | MTV | 3 fictitious VM profiles | compatible VMware lab | preflight/migration/acceptance | Not Run | None |
| VER-016 | Kong | design candidate | selected licensed product lab | KONG-T01〜T07 | Not Run | None |
| VER-017 | Sysdig | design candidate | selected licensed product lab | SYSDIG-T01〜T07 | Not Run | None |

## 3. Verification entry template

```text
Verification ID:
Date/time/timezone:
Executor / checker:
Approved change / lab:
Artifact revision:
Environment and exact product/tool versions:
Preconditions:
Actual command/action:
Expected result:
Actual result:
Status: Not Run / Blocked / Fail / Pass
Evidence location and checksum:
Sanitization performed:
Deviation / issue / risk / change:
Cleanup and final state:
```

## 4. Rules

- local syntax success は OpenShift 4.22 API admission、runtime、hardware compatibility の証明ではありません。
- server-side dry-run も network、storage、performance、failure recovery の証明ではありません。
- AI または別担当者の実行結果を learner execution として記録しません。
- result を後から想像で補完しません。evidence がなければ `Not Run` または `Unknown` です。
- credential、Secret、private key、customer/internal data を record へ保存しません。

## 5. Current conclusion

検証環境と actual execution がないため、build、test、migration、rollback、RPO/RTO、Kong、Sysdig に関する合格判定はありません。

