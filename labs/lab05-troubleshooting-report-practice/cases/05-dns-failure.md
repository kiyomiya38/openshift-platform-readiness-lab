# Case 05: DNS 不通

> 架空・匿名化済み教材。`locked-lab` Namespace の Pod だけが Service FQDN を解決できない。他 Namespace は正常。

## 証跡

```text
$ oc exec client -n locked-lab -- getent hosts kubernetes.default.svc
# exit code 2, outputなし
```

```text
$ oc exec client -n open-lab -- getent hosts kubernetes.default.svc
172.30.0.1 kubernetes.default.svc.cluster.local
```

```text
$ oc get networkpolicy -n locked-lab
NAME                 POD-SELECTOR   POLICY-TYPES
default-deny-egress  <none>         Egress
allow-web-egress     role=client    Egress
```

```text
$ oc get networkpolicy allow-web-egress -n locked-lab -o yaml
spec:
  podSelector:
    matchLabels:
      role: client
  policyTypes: [Egress]
  egress:
    - ports:
        - protocol: TCP
          port: 443
```

```text
$ oc get service,endpointslice -n openshift-dns
NAME                  TYPE        CLUSTER-IP    PORT(S)
service/dns-default   ClusterIP   172.30.0.10   53/UDP,53/TCP
NAME                                      ENDPOINTS
endpointslice.discovery.k8s.io/dns-default 10.0.1.5,10.0.2.5
```

## 制約

- 全 egress 許可は禁止。
- DNS service IP は環境値であり、実案件では API から取得して設計する。

## 課題

1. Cluster 全体の DNS 障害ではない根拠を書く。
2. UDP/TCP 53 を必要な DNS 宛先だけ許可する設計案を述べる。
3. 修正後に許可通信と拒否通信の双方を試験する理由を書く。
