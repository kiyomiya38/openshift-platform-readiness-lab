# Sample Report 02: Route アクセス不可

## 概要

- `shop.apps.lab.example` は DNS 解決と TLS handshake に成功するが HTTP 503。
- Route は Admitted、backend Service/port 名も解決可能。
- Service の EndpointSlice は作成されているが endpoint が `<none>` で、backend がないため Router が転送できない状態。

## 原因

- Service selector は `app=shop-v2`、Ready Pod label は `app=shop` で不一致。
- 直前の Service label standardization 変更との時間的・設定上の一致がある。直接原因は selector 不一致。
- 根本原因候補は Deployment label と Service selector の結合試験不足。変更手順と review 記録は要確認。

## 対応・確認

1. source of truth と intended label を確認し、Service selector または workload label のどちらを戻すか決定する。
2. 承認済み変更後、EndpointSlice に Ready Pod IP が登録されることを確認する。
3. Cluster 内 Service 疎通、Route 経由 HTTPS、監視 probe の順で確認する。
4. DNS/LB/Router は今回の主原因ではないが、復旧判定では外部経路まで確認する。

## 再発防止

- CI で Service selector と rendered workload labels の一致を検査する。
- 非本番 rollout で EndpointSlice と Route smoke test を自動化する。
