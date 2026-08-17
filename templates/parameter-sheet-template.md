# OpenShift パラメータシート

> 値の出所と確認状態を残す。パスワード、トークン、pull secret、秘密鍵は記載せず、Secret 管理システム上の参照 ID のみ記載する。

## 管理情報

| 項目 | 値 |
|---|---|
| 案件／環境 | `<記入>` |
| 対象バージョン | `要確認` |
| 基本／詳細設計版 | `<記入>` |
| 更新者／更新日 | `<役割>／YYYY-MM-DD` |

## クラスタ・DNS

| ID | パラメータ | 設定値 | 既定値 | 出所 | 状態 |
|---|---|---|---|---|---|
| CL-001 | clusterName | `<記入>` | - | 要件 | `要確認` |
| CL-002 | baseDomain | `<記入>` | - | DNS 設計 | `要確認` |
| CL-003 | apiVIP | `<IP>` | - | IP 管理台帳 | `要確認` |
| CL-004 | ingressVIP | `<IP>` | - | IP 管理台帳 | `要確認` |
| CL-005 | update channel | `<stable-x.y>` | - | 運用設計 | `要確認` |

## ネットワーク

| ID | パラメータ | 設定値 | 変更可否／影響 | 検証 |
|---|---|---|---|---|
| NW-001 | machineNetwork | `<CIDR>` | 導入後変更は要確認 | `ip route` |
| NW-002 | clusterNetwork | `<CIDR>/<hostPrefix>` | 重複禁止 | `oc get network.config/cluster -o yaml` |
| NW-003 | serviceNetwork | `<CIDR>` | 重複禁止 | 同上 |
| NW-004 | networkType | `<OVNKubernetes 等>` | バージョン依存 | 同上 |
| NW-005 | proxy/noProxy | `<マスク済み値>` | 到達性に影響 | `oc get proxy/cluster -o yaml` |

## ノード

| Pool | Replicas | vCPU | Memory | Disk | Zone／Rack | Labels／Taints |
|---|---:|---:|---:|---:|---|---|
| control-plane | `<値>` | `<値>` | `<GiB>` | `<GiB>` | `要確認` | `<記入>` |
| worker | `<値>` | `<値>` | `<GiB>` | `<GiB>` | `要確認` | `<記入>` |
| infra | `<値>` | `<値>` | `<GiB>` | `<GiB>` | `要確認` | `<記入>` |

## Storage

| ID | StorageClass | Provisioner | default | bindingMode | expansion | reclaimPolicy |
|---|---|---|---|---|---|---|
| ST-001 | `<名称>` | `<CSI driver>` | `<true/false>` | `<値>` | `<true/false>` | `<値>` |

## 認証・証明書・Secret 参照

| ID | 対象 | 方式／参照 ID | 有効期限 | 所有者 | 状態 |
|---|---|---|---|---|---|
| AU-001 | Identity Provider | `<OIDC/LDAP 等>` | - | `<役割>` | `要確認` |
| CE-001 | API certificate | `<発行元／Secret 名>` | `<日付>` | `<役割>` | `要確認` |
| SE-001 | pull secret | `<Vault 等の参照 ID>` | - | `<役割>` | `値を記載しない` |

## Operator・運用

| Operator／機能 | Channel | Approval | Namespace | 主要パラメータ | 根拠 |
|---|---|---|---|---|---|
| `<名称>` | `<値>` | `<値>` | `<値>` | `<値>` | `<設計 ID>` |

| 運用項目 | パラメータ | 値 | 状態 |
|---|---|---|---|
| ログ | 保存期間 | `<日数>` | `要確認` |
| 監視 | 通知先 | `<チーム名>` | `要確認` |
| バックアップ | RPO/RTO | `<時間>` | `要確認` |
| 更新 | 保守時間 | `<曜日・時間帯>` | `要確認` |

## 変更履歴

| 版 | 日付 | ID | 変更前 | 変更後 | 理由／承認 |
|---|---|---|---|---|---|
| 0.1 | `<日付>` | `<ID>` | - | `<値>` | `<記入>` |
