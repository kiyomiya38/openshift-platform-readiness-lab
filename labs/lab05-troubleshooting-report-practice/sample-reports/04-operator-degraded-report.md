# Sample Report 04: Operator 異常

## 概要

- OLM の Subscription は最新既知 CSV を指し、CSV phase は Succeeded。
- ただし Operator Pod 自体は CrashLoopBackOff で、管理対象を reconcile できない。
- 提示された OpenShift ClusterOperator は Available=True/Degraded=False。OpenShift 本体の Operator 異常ではない。

## 原因

- Operator log は必須 ConfigMap `example-metrics-settings` がないと示す。
- Namespace cleanup で同 ConfigMap が削除された事実があるため、直接原因は必要設定の削除。
- 根本原因候補は resource ownership/label、削除対象 review、backup/source of truth の不備。詳細は要確認。

## 対応案

1. Operator の公式仕様、CSV/Deployment reference、Git/backup から期待する ConfigMap の内容と version を確認する。
2. Secret が含まれる場合は安全な保管元を用い、報告書へ値を書かない。
3. 承認後に ConfigMap を復元し、Operator Pod Ready、log、管理対象 CR status、Operand、alert を確認する。

## 再発防止

- Operator 管理資源を label/owner として inventory 化し、cleanup selector から保護する。
- 削除前の dry-run/diff、owner 承認、復元試験を手順へ追加する。
