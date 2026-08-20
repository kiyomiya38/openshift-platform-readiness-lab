# Case 01: Pod 起動失敗

> 架空・匿名化済み教材。`apps-lab` の注文 API が 09:05 から利用不可。

## 直前変更

- 08:58 Change `CHG-101`: ConfigMap `order-api-config` の DB port を更新。
- 09:02 Deployment rollout 開始。

## 証跡

```text
$ oc get pod -n apps-lab
NAME                         READY   STATUS             RESTARTS   AGE
order-api-7d8f6c9f5d-x1abc   0/1     CrashLoopBackOff   6          8m
```

```text
$ oc logs order-api-7d8f6c9f5d-x1abc -n apps-lab --previous
ERROR configuration: DB_PORT must be an integer, got "fifty-four-thirty-two"
```

```text
$ oc describe pod order-api-7d8f6c9f5d-x1abc -n apps-lab
State:          Waiting
  Reason:       CrashLoopBackOff
Last State:     Terminated
  Reason:       Error
  Exit Code:    2
Events:
  Normal  Pulled   Container image already present on machine
  Warning BackOff Back-off restarting failed container
```

```text
$ oc get configmap order-api-config -n apps-lab -o yaml
data:
  DB_HOST: db.apps-lab.svc
  DB_PORT: fifty-four-thirty-two
```

## 制約

- DB の変更は禁止。
- 直前 ConfigMap の旧値は変更記録上 `5432`。
- 変更は承認済み手順で実施する。削除や強制再起動は解答に含めない。

## 課題

1. 事実、直接原因、根本原因候補を分ける。
2. 安全な暫定対応とロールバック条件を書く。
3. 修正後に確認するコマンド／業務疎通／監視期間を書く。
4. 再発防止を「注意する」以外で二つ挙げる。
