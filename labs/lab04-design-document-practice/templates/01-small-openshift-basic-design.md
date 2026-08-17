# 小規模 OpenShift 検証環境 基本設計ワークシート

## 1. シナリオの理解

- 目的: `<誰が何を検証するか>`
- 利用者／workload: `<人数・種類・増加見込み>`
- 可用性／停止許容: `<記入>`
- データ区分: `<機密性・保持期間>`
- 対象外: `<記入>`

## 2. 要件から設計への対応

| 要件 | 設計判断 | 根拠 | 試験方法 | 未決事項 owner／期限 |
|---|---|---|---|---|
| 開発者 20 名 | `<記入>` | `<記入>` | `<記入>` | `<要確認>` |
| API の単一障害回避 | `<記入>` | `<記入>` | `<記入>` | `<要確認>` |

## 3. Cluster / Node

| 項目 | 設計案 | 算定根拠・確認方法 |
|---|---|---|
| Platform／install method | `<要確認>` | `<既存基盤、対応版>` |
| Control Plane／Worker | `<台数・配置>` | `<failure domain、capacity>` |
| CPU／Memory／Disk | `<値または要確認>` | `<workload 見積、monitoring overhead>` |
| Infra workload | `<worker 同居／infra node>` | `<規模と運用>` |
| Update channel／window | `<要確認>` | `<事前検証、EUS 等>` |

## 4. Network / DNS / LB

| 項目 | 設計案 | 依存先／試験 |
|---|---|---|
| Machine/Pod/Service CIDR | `<要確認>` | `<既存 route と重複確認>` |
| API / `*.apps` DNS | `<命名・TTL>` | `<DNS owner>` |
| API / Ingress LB | `<health check・source>` | `<LB owner>` |
| Proxy / noProxy | `<要確認>` | `<egress allowlist>` |
| NetworkPolicy | `<default と例外>` | `<DNS/monitoring 通信>` |

## 5. Security / Storage / Operation

| 領域 | 設計案 | 確認・試験 |
|---|---|---|
| IdP/RBAC | `<group と role>` | `<最小権限試験>` |
| SCC | `<標準 SCC 方針>` | `<admission 試験>` |
| Certificate | `<発行・更新・期限監視>` | `<TLS 試験>` |
| StorageClass | `<backend/RWO/RWX>` | `<provision/expand/restore>` |
| Monitoring/Logging | `<対象・保存・通知>` | `<alert test>` |
| Backup | `<OADP/etcd/volume>` | `<RPO/RTO/restore test>` |

## 6. 受入試験

| ID | 観点 | 前提 | 合格条件 | 証跡 |
|---|---|---|---|---|
| T-01 | Cluster health | `<記入>` | `<oc get co 等>` | `<記入>` |
| T-02 | Node failure | `<承認済み条件>` | `<継続条件>` | `<記入>` |
| T-03 | Route | `<DNS/LB>` | `<HTTP/TLS 条件>` | `<記入>` |
| T-04 | Restore | `<backup>` | `<RPO/RTO>` | `<記入>` |

## 7. 課題

| ID | 要確認事項 | 影響 | Owner | 期限 | 決まらない場合の扱い |
|---|---|---|---|---|---|
| Q-01 | `<記入>` | `<記入>` | `<役割>` | `<日付>` | `<保留／仮定>` |
