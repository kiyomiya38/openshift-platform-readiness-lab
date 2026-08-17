# Case 04: Operator 異常

> 架空・匿名化済み教材。OLM で導入した `example-metrics-operator` の管理対象が更新されない。OpenShift の ClusterOperator は正常。

## 証跡

```text
$ oc get co
NAME             VERSION   AVAILABLE   PROGRESSING   DEGRADED
authentication   4.x       True        False         False
network          4.x       True        False         False
```

```text
$ oc get subscription -n openshift-operators example-metrics-operator -o yaml
status:
  installedCSV: example-metrics-operator.v1.4.2
  currentCSV: example-metrics-operator.v1.4.2
  state: AtLatestKnown
```

```text
$ oc get csv -n openshift-operators example-metrics-operator.v1.4.2
NAME                              DISPLAY                   VERSION   PHASE
example-metrics-operator.v1.4.2   Example Metrics Operator  1.4.2     Succeeded
```

```text
$ oc get pod -n openshift-operators -l app=example-metrics-operator
NAME                                        READY   STATUS             RESTARTS
example-metrics-operator-6f8b7d6cc5-q7xyz   0/1     CrashLoopBackOff   9
```

```text
$ oc logs -n openshift-operators example-metrics-operator-6f8b7d6cc5-q7xyz --previous
ERROR failed to load configuration: configmap "example-metrics-settings" not found
```

## 直前変更

- Namespace cleanup 作業で不要 ConfigMap を削除。削除一覧に `example-metrics-settings` が含まれていた。

## 課題

1. Subscription/CSV が正常でも機能が止まる理由を説明する。
2. OpenShift ClusterOperator 異常との違いを書く。
3. ConfigMap を復元する前に確認すべき source of truth と変更承認を書く。
