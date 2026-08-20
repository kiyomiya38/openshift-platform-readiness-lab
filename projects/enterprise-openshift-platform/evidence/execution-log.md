# 実行ログ

> [!IMPORTANT]
> 本書は、承認済み環境に対して実際に行った構築、変更、移行、復旧操作を記録するためのサンプルです。本プロジェクトでは実環境操作を行っていないため、すべて`Not Run`です。

## 1. 実行対象の状態

| Scope | Environment | Change approval | Execution | Result evidence |
| --- | --- | --- | --- | --- |
| DNS / NTP / LB preparation | None | Not Approved | Not Run | None |
| Agent-based OCP install | None | Not Approved | Not Run | None |
| Cluster baseline / sample workload | None | Not Approved | Not Run | None |
| Backup / restore | None | Not Approved | Not Run | None |
| OpenShift Virtualization | None | Not Approved | Not Run | None |
| MTV VM migration / rollback | None | Not Approved | Not Run | None |
| Kong / Sysdig integration | None | Not Approved | Not Run | None |

## 2. Execution register

| Execution ID | Date | Change ID | Scope | Environment | Result | Evidence | Closure |
| --- | --- | --- | --- | --- | --- | --- | --- |
| — | — | — | — | — | Not Run | None | Open |

## 3. 実行記録テンプレート

```text
Execution ID:
Date/time/timezone:
Executor / checker roles:
Approved change ID:
Target environment and inventory revision:
Product/tool exact versions:
Preconditions and pre-check actual result:
Actual command/action:
Expected result:
Actual result:
Status: Not Run / Blocked / Fail / Pass
Stop condition encountered:
Rollback action and result:
Evidence location and checksum:
Issue / risk / deviation:
Sanitization performed:
Final state and closure approval:
```

## 4. 記録規則

- 予定したcommandではなく、実際に実行したactionと順序を記録します。
- session内に表示されたSecret、token、Authorization header、kubeconfigは保存前に除去します。
- failureやrollbackを省略せず、開始前と終了後の状態を残します。
- 実行証跡がない操作を`Pass`へ変更しません。

