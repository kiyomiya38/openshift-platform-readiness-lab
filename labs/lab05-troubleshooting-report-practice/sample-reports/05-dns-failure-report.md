# Sample Report 05: DNS 不通

## 概要

- `locked-lab` の client Pod だけが Service FQDN を解決できない。
- `open-lab` では同じ FQDN を解決可能で、DNS Service と Endpoint は存在する。
- Cluster 全体の DNS service 障害より、Namespace/Pod 単位の egress 制御が原因候補。

## 原因

- client Pod は default-deny-egress の対象で、追加 policy は TCP/443 だけを許可している。
- DNS の UDP/TCP 53 が許可されず問い合わせが届かないことが直接原因と判断できる。
- 実環境では CNI の policy semantics、DNS service/namespace selector、実際の DNS IP を API から要確認。

## 対応案

- 全 egress を許可せず、対象 Pod から Cluster DNS への UDP 53 と TCP 53 だけを許可する NetworkPolicy を設計・review する。
- DNS IP を YAML に不用意に固定する代わりに、組織標準と CNI が対応する namespace/pod selector の利用可否を対象版で確認する。

## 復旧確認・再発防止

1. Service FQDN と許可した外部/内部 FQDN が解決できる。
2. DNS 以外の未許可 egress が引き続き拒否される。
3. NetworkPolicy の unit/integration test に DNS allow と deny ケースを追加する。
4. Namespace 作成 template に default deny と必要最小の DNS policy を組み込む。
