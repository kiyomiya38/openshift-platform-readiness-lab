# サンプル回答: Disconnected 設計観点

> 架空の教材例です。resource 名や `oc-mirror` workflow は OpenShift/oc-mirror の対象版で要確認です。

## 境界と同期

- Connected Zone の専用 mirror host だけが承認済み Registry へ egress する。
- ImageSetConfiguration は Git review し、workspace/metadata を access-controlled storage に backup する。
- OpenShift release、選定 Operator package/channel、業務 Image を digest 単位で同期する。
- 転送前に malware/image scan、signature/digest、license、承認番号を記録する。
- Disconnected Zone の Registry は TLS、RBAC、audit、capacity alert、backup/restore の対象とする。

## Cluster 依存

| 依存 | 設計案 | 試験 |
|---|---|---|
| DNS | 内部に API、wildcard Apps、Mirror Registry record | node/installer 位置から正引き |
| NTP | 内部 redundant source | 全 node の offset/同期状態 |
| LB | API/Ingress health check と冗長化 | backend 障害時の接続 |
| Certificate | 組織 CA、SAN、trust 配布、更新監視 | TLS chain/expiry |
| Firewall | Cluster から内部依存先だけ許可 | allow/deny 双方を試験 |

## Operator

1. 必要 Operator と依存 Image を棚卸しし、catalog 全体を無条件に mirror しない。
2. package/channel/version と OpenShift 互換性を対象版の公式情報で確認する。
3. 内部 CatalogSource を作り、Package/CSV/InstallPlan/Operand の Image pull を試験する。
4. Automatic/Manual approval は update 運用と rollback 可否を踏まえ決定する。

## 運用

- 毎月の差分同期と緊急 CVE 同期の二つの cadence を定義する。
- 次回 release/Operator に必要な容量と増加率を監視する。
- 更新前に mirror 完全性、cluster health、backup、Operator compatibility を gate とする。
- 更新後は CVO/ClusterOperator、catalog/CSV、sample workload、監視を確認する。
- OpenShift downgrade を当然の戻し手段とせず、対象版の supported recovery と support 連携を要確認とする。

## 受入条件

1. 外部 egress を止めた状態で installation/update/Operator install/app rollout が完了する。
2. Image 不足時に Event/log から欠落 digest を特定し、承認フローで追加同期できる。
3. Mirror Registry または metadata の復元を非本番で実証する。
4. 証明書更新、容量不足、catalog 異常の Runbook と連絡先がある。
