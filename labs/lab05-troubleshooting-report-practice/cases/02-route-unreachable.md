# Case 02: Route アクセス不可

> 架空・匿名化済み教材。外部利用者は `https://shop.apps.lab.example` で 503 を受信。DNS 解決と TLS handshake は成功。

## 証跡

```text
$ oc get route shop -n web-lab
NAME   HOST/PORT                    PATH   SERVICES   PORT   TERMINATION
shop   shop.apps.lab.example               shop       web    edge
```

```text
$ oc describe route shop -n web-lab
Requested Host: shop.apps.lab.example
Service:        shop
Target Port:    web
Conditions:
  Type: Admitted
  Status: True
```

```text
$ oc get service shop -n web-lab -o yaml
spec:
  selector:
    app: shop-v2
  ports:
  - name: web
    port: 8080
    targetPort: http
```

```text
$ oc get pod -n web-lab --show-labels
NAME                       READY   STATUS    LABELS
shop-65bcdd774f-a1b2c       1/1     Running   app=shop
```

```text
$ oc get endpointslice -n web-lab -l kubernetes.io/service-name=shop
NAME          ADDRESSTYPE   PORTS   ENDPOINTS   AGE
shop-7h9mz    IPv4          8080    <none>      3m
```

## 直前変更

- 11:20 Service Manifest の label standardization を適用。
- 11:22 監視が HTTP 503 を検知。

## 課題

1. DNS/LB/Router が主原因ではないと判断できる事実を書く。
2. Service 以降の通信経路で不一致を特定する。
3. 修正後の Endpoint と Route 疎通の合格条件を書く。
