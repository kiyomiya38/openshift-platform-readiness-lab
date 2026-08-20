# 静的検証記録

> [!IMPORTANT]
> 本書は、リポジトリ上で実行できる構文・参照・整合性検査を記録します。2026-08-17に一部のローカル静的検証を実施しました。`Pass`は各行に記載したscopeだけを示し、OpenShift 4.22 API admission、runtime、hardware compatibility、network、storage、performance、failure recoveryは確認できません。

## 1. 検証環境

| Item | Value |
| --- | --- |
| Workspace revision | 作業ツリー状態（commit未記録） |
| Operating system | Windows / PowerShell 5.1.26100.9168 |
| Tool versions | Node.js 24.12.0、js-yaml 5.2.1、kubectl 1.31.4、Kustomize 5.4.2、Mermaid CLI 11.16.0、ripgrep 15.2.0 |
| Approved execution scope | `projects/**`のread-only static validation |
| Raw evidence location | 当該作業sessionのcommand output（リポジトリ未保存） |

## 2. 静的検証台帳

| ID | Category | Scope | Candidate method | Expected evidence | Status | Actual result |
| --- | --- | --- | --- | --- | --- | --- |
| VER-001 | Markdown links | `projects/**.md` relative links | PowerShell path resolution | broken-link list、tool/version | Pass | 260 links、broken 0 |
| VER-002 | Markdown structure | headings、fences、numbered docs | PowerShell line scan | error/warning list | Pass | Markdown 38 files、odd fence 0、docs 00〜24連続 |
| VER-003 | Mermaid | `diagrams/*.mmd` | Mermaid CLI 11.16.0 render | rendered files / errors | Pass | 5 filesすべてSVG render成功。一時出力は検査後に削除 |
| VER-004 | YAML syntax | install、Ansible vars/playbooks、manifests | js-yaml 5.2.1 | file-by-file parse result | Pass | 19 files、parse error 0 |
| VER-005 | Kubernetes render | `manifests/kustomization.yaml` | kubectl 1.31.4 / Kustomize 5.4.2 | rendered YAML / errors | Pass | render成功、schema/admission未確認 |
| VER-006 | Ansible syntax | inventory、playbooks、templates | pinned ansible-core / collections | syntax/lint result | Not Run | None |
| VER-007 | Parameter consistency | SCENARIO、docs、install、Ansible | review script | mismatch list | Not Run | None |
| VER-008 | Test ID consistency | test specification / results | PowerShell ID comparison | count、missing/extra ID | Pass | 仕様72 unique／結果72 unique、差分0、全結果`NOT RUN` |
| VER-009 | Publication scan | secret patterns、旧用途向け表現、local path | ripgrep + PowerShell pattern scan | redacted finding list | Pass | high-confidence secret 0、旧表現0、local user path 0。独立レビュー未実施 |

## 3. 検証記録テンプレート

```text
Verification ID:
Date/time/timezone:
Executor role:
Workspace revision:
Scope:
Tool and exact version:
Actual command/action:
Expected result:
Actual result:
Status: Not Run / Blocked / Fail / Pass
Evidence location and checksum:
Limitations:
Sanitization performed:
Related issue/change:
```

## 4. 判定規則

- 実際のcommand、tool version、target revision、actual outputがなければ`Not Run`を維持します。
- parser成功はschema validationやserver admissionを意味しません。
- server-side dry-runはruntime、network、storage、performance、recoveryを意味しません。
- warning、skip、unsupported fileを成功件数へ含めず、limitationsへ記録します。
- credential、Secret、private key、kubeconfig、組織固有データをraw evidenceとして公開しません。

## 5. 実環境検証との分離

クラスタ構築、設定変更、障害注入、backup/restore、VM移行などの記録は[実行ログ](execution-log.md)、試験ID単位の結果と証跡は[試験証跡索引](test-evidence-index.md)で管理します。
