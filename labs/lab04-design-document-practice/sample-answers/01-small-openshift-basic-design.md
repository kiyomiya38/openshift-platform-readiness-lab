# サンプル回答: 小規模 OpenShift 検証環境

> 架空の教材例です。製品 sizing や対応構成を示すものではなく、実案件ではすべて対象版・基盤で要確認です。

## 要件と方針

| 要件 | 設計案 | 根拠 | 試験 |
|---|---|---|---|
| 開発者 20 名 | 研修班ごとに Project、Quota/LimitRange を設定 | 誤った resource 消費の局所化 | Quota 超過が拒否されること |
| API 単一障害回避 | Control Plane 3 台を別 failure domain に配置する案 | etcd quorum と host 障害を考慮 | 承認済み非本番 node 障害試験 |
| 計画停止可 | 更新は月次保守枠、事前 snapshot ではなく製品対応 backup/復旧を準備 | 更新と復旧を運用に含める | 事前 cluster health/更新後試験 |

## 設計案

- 基盤/install method: 既存 virtualization 基盤への IPI/UPI 適否を要確認。
- Node: Control Plane 3、Worker 3 を初期案とするが、CPU/Memory/Disk は workload と platform overhead の測定後に決定。
- Infra workload: 小規模のため worker 同居を初期案とし、monitoring/registry 容量または分離要件があれば再検討。
- CIDR: Machine/Pod/Service の候補は書かず、network 管理者が既存・将来 route と重複確認して払い出す。
- DNS/LB: `api.<cluster>.<baseDomain>` と `*.apps...`。VIP、health check、TTL、証明書 owner は要確認。
- Egress: 組織 Proxy の FQDN/port と `noProxy` を network/security と合意。無制限 Internet は許可しない。
- Security: 組織 IdP group を Project RoleBinding へ対応。標準 SCC を優先し、Secret 実値は Git に保存しない。
- Storage: default StorageClass、RWO/RWX、snapshot/expand、Registry 要件を CSI owner と確認。
- Monitoring/logging: platform と利用者 workload の責任分界、通知先、保存期間を要確認。
- Backup: etcd、OADP、volume/backend backup の対象と RPO/RTO を分け、復元試験を実施。
- Update: channel/target version は要確認。非本番検証、Operator 互換性、CVO/CO、業務疎通を gate にする。

## 主な未決事項

| ID | 内容 | Owner | 期限 | 影響 |
|---|---|---|---|---|
| Q-01 | virtualization 基盤と install method | Platform | 基本設計 review 前 | node/LB/DNS 手順 |
| Q-02 | 三 network CIDR | Network | 詳細設計前 | 導入後変更困難 |
| Q-03 | CSI/StorageClass と backup | Storage | PoC 前 | PVC/Registry/復旧 |
| Q-04 | IdP と certificate 発行 | Security | 結合試験前 | login/TLS |

## 受入観点

1. `oc get clusterversion/co/nodes` が設計した正常条件を満たす。
2. Project の RBAC、Quota、標準 SCC admission を確認する。
3. API/Route を DNS、LB、TLS、Service、Pod まで疎通する。
4. PVC provision/expand と対象データの backup/restore を行う。
5. Alert が運用窓口へ届き、Runbook から一次切り分けできる。
