# 14. OpenShift基盤 パラメータシート

## 文書管理

| 項目 | 値 |
| --- | --- |
| 文書ID | `PARAM-OCP-001` |
| 案件／環境 | Example Enterprise OpenShift 基盤導入／架空production想定 |
| 版／状態 | 0.1／Draft |
| 基準日 | 2026-08-17 |
| 対応詳細設計 | [13-detailed-design.md](13-detailed-design.md) 0.1 |
| 実機反映 | 未実施 |

> [!IMPORTANT]
> `架空値` は学習シナリオの入力、`TBD` は未決定、`未実施` は確認していないことを表します。Secret実値は意図的に記載していません。

## 1. クラスタ・Installer

| ID | パラメータ | 設定値 | 出所 | 状態／検証 |
| --- | --- | --- | --- | --- |
| CL-001 | cluster name | `ocp-prd` | SCENARIO | 架空値 |
| CL-002 | base domain | `example.com` | SCENARIO | 架空値 |
| CL-003 | cluster domain | `ocp-prd.example.com` | CL-001/002 | 架空値 |
| CL-004 | OCP version | `4.22.z` | 要件 | z版TBD |
| CL-005 | architecture | `amd64` | SCENARIO | HCL確認前 |
| CL-006 | install method | Agent-based Installer | SCENARIO | 未実施 |
| CL-007 | platform | `none` | SCENARIO | 未実施 |
| CL-008 | AgentConfig API | `v1beta1` | 4.22 customization例 | 対象installerで要検証 |
| CL-009 | publish | `External` | 架空設計 | 要レビュー |
| CL-010 | FIPS | 未指定（false相当） | 要件なし | Security判断TBD |
| CL-011 | update channel | `stable-4.22`候補 | 運用要件 | 対象z版・EUS確認前 |
| CL-012 | rendezvous IP | `192.0.2.21` | 詳細設計 | 架空値 |

## 2. ネットワーク

| ID | パラメータ | 設定値 | 変更影響／注意 | 状態 |
| --- | --- | --- | --- | --- |
| NW-001 | networkType | `OVNKubernetes` | 4.22での設計値 | Draft |
| NW-002 | machineNetwork | `192.0.2.0/24` | RFC 5737、実利用不可 | 架空値 |
| NW-003 | clusterNetwork | `10.128.0.0/14` | 他網と重複禁止 | 重複確認未実施 |
| NW-004 | hostPrefix | `/23` | 1ノード当たりのPod subnet | Draft |
| NW-005 | serviceNetwork | `172.30.0.0/16` | 他網と重複禁止 | 重複確認未実施 |
| NW-006 | IP family | IPv4 single-stack | シナリオ決定 | Draft |
| NW-007 | node addressing | static | DHCP不使用 | 未実施 |
| NW-008 | default gateway | `192.0.2.1` | 現物routing要確認 | 架空値 |
| NW-009 | node interface | `enp1s0` | 全ノード同名想定 | NIC名TBD |
| NW-010 | MTU | `1500` | switch、CSI、VM網と統一 | 実測未実施 |
| NW-011 | Podman bridge注意 | `10.88.0.0/16` | machineNetworkに含めない | 設計レビュー未実施 |

## 3. ノード

| ID | Host | Role | IPv4 | MAC（架空） | vCPU | Memory | Boot disk | 状態 |
| --- | --- | --- | --- | --- | ---: | ---: | ---: | --- |
| ND-001 | `cp01.ocp-prd.example.com` | master/rendezvous | `192.0.2.21` | `02:00:00:00:00:21` | 8 | 32 GiB | 250 GiB `/dev/sda` | 現物未確認 |
| ND-002 | `cp02.ocp-prd.example.com` | master | `192.0.2.22` | `02:00:00:00:00:22` | 8 | 32 GiB | 250 GiB `/dev/sda` | 現物未確認 |
| ND-003 | `cp03.ocp-prd.example.com` | master | `192.0.2.23` | `02:00:00:00:00:23` | 8 | 32 GiB | 250 GiB `/dev/sda` | 現物未確認 |
| ND-004 | `wk01.ocp-prd.example.com` | worker | `192.0.2.31` | `02:00:00:00:00:31` | 32 | 256 GiB | 250 GiB `/dev/sda` | 現物未確認 |
| ND-005 | `wk02.ocp-prd.example.com` | worker | `192.0.2.32` | `02:00:00:00:00:32` | 32 | 256 GiB | 250 GiB `/dev/sda` | 現物未確認 |
| ND-006 | `wk03.ocp-prd.example.com` | worker | `192.0.2.33` | `02:00:00:00:00:33` | 32 | 256 GiB | 250 GiB `/dev/sda` | 現物未確認 |

CPU virtualization extension、NUMA、disk性能、RAID/multipath、Secure Boot、Firmware、rack/電源/ToR分散はすべてTBDです。

## 4. DNS・Load Balancer

| ID | 対象 | 値 | Backend／確認 | 状態 |
| --- | --- | --- | --- | --- |
| DNS-001 | DNS server 1/2 | `192.0.2.2`／`192.0.2.3` | `dig`正逆引き | 架空値・未実施 |
| DNS-002 | API | `api.ocp-prd.example.com -> 192.0.2.10` | A + PTR | 未実施 |
| DNS-003 | internal API | `api-int.ocp-prd.example.com -> 192.0.2.10` | A + PTR | 未実施 |
| DNS-004 | apps wildcard | `*.apps.ocp-prd.example.com -> 192.0.2.11` | canary名でA確認 | 未実施 |
| DNS-005 | node records | ND-001〜006のFQDN/IP | A + PTR | 未実施 |
| LB-001 | LB nodes | `192.0.2.12`、`192.0.2.13` | RHEL 9.x（minor TBD）、NIC、別途subscription | 未実施 |
| LB-002 | API VIP | `192.0.2.10:6443` | cp 3台、L4、no persistence、`/readyz` | 未実施 |
| LB-003 | MCS | `192.0.2.10:22623` | cp 3台、L4、内部限定 | 未実施 |
| LB-004 | Ingress VIP | `192.0.2.11:80,443` | worker 3台、L4 | 未実施 |
| LB-005 | VIP HA | keepalived unicast例、VRID 51 | 正式方式・FW・fencing | TBD |

## 5. Proxy・NTP・Firewall

| ID | パラメータ | 値 | 状態／備考 |
| --- | --- | --- | --- |
| PX-001 | HTTP/HTTPS Proxy | `http://proxy.example.com:3128` | 架空値、認証方式TBD |
| PX-002 | noProxy | localhost、Machine/Pod/Service CIDR、`.svc`、`.cluster.local`、cluster domain | 内部宛先追加レビュー前 |
| PX-003 | Proxy CA | Secret管理領域の参照IDのみ記載予定 | 発行元TBD、値なし |
| NT-001 | NTP 1/2 | `192.0.2.4`／`192.0.2.5`、UDP/123 | chrony同期未実施 |
| FW-001 | API | TCP/6443 | 未開通確認 |
| FW-002 | MCS | TCP/22623 | 未開通確認 |
| FW-003 | Ingress | TCP/80,443 | 未開通確認 |
| FW-004 | Assisted Service | 6ノード -> rendezvous TCP/8090 | bootstrap期間のみ、未確認 |
| FW-005 | NTP | 6ノード -> NTP UDP/123 | 未確認 |
| FW-006 | VRRP | LB peer間 IP protocol 112候補 | keepalived採用時、TBD |

## 6. 認証・証明書・Secret

| ID | 対象 | 方式／参照 | 所有者 | 状態 |
| --- | --- | --- | --- | --- |
| AU-001 | Identity Provider | OIDC/LDAP候補 | Security | 方式TBD |
| AU-002 | MFA | 組織IdP側 | Security | TBD |
| AU-003 | break-glass | 個人別保管・利用監査 | Security/Platform | 手順TBD |
| CE-001 | API certificate | 組織CAまたは既定証明書 | Security | 方針TBD |
| CE-002 | Ingress certificate | wildcard証明書候補 | Security | 方針TBD |
| SE-001 | pull secret | `<APPROVED_SECRET_STORE_REFERENCE>` | Platform/Security | 実値記載なし |
| SE-002 | SSH public key | `<APPROVED_KEY_REFERENCE>` | Platform | 公開鍵も例では未設定 |
| SE-003 | Proxy CA | `<PKI_REFERENCE>` | Security | 実値記載なし |
| SE-004 | backup credential | `<BACKUP_SECRET_REFERENCE>` | Platform/Storage | 実値記載なし |

## 7. Storage・バックアップ・運用

| ID | 項目 | 設計値 | 状態 |
| --- | --- | --- | --- |
| ST-001 | CSI product/provisioner | 要確認 | `TBD-006` |
| ST-002 | default StorageClass | 要確認 | 未作成 |
| ST-003 | RWO/RWX/Snapshot/expansion | 要確認 | 未検証 |
| REG-001 | internal Image Registry managementState | 導入直後 `Removed` の可能性。production storage構成後 `Managed` | CSI/PVC/容量TBD、未構成 |
| REG-002 | internal Image Registry storage | shareable production永続storage | StorageClass/PVC/backup未確定 |
| BK-001 | App RPO/RTO | 1時間／4時間 | 机上目標、未実測 |
| BK-002 | etcd | cluster外暗号化保管 | 頻度・保持TBD |
| BK-003 | Kubernetes resources/PV | OADP候補 | 版・方式TBD |
| BK-004 | external DB | DB-native backup/PITR候補 | Application/DB設計TBD |
| OP-001 | monitoring | Cluster Monitoring | 通知先・Runbook未設定 |
| OP-002 | logging | infrastructure/application/audit分離 | 製品版・保持TBD |
| OP-003 | maintenance window | 事前承認制 | 曜日・時間TBD |
| OP-004 | evidence retention | アクセス制御領域 | 期間TBD |

## 8. サンプルワークロード

| ID | パラメータ | 値 | 状態 |
| --- | --- | --- | --- |
| WL-001 | Namespace | `example-web-dev` | Manifest作成済み、未適用 |
| WL-002 | Deployment/replicas | `example-web`／2 | 未適用 |
| WL-003 | image | `registry.access.redhat.com/ubi9/httpd-24:latest` | 学習用。digest固定前 |
| WL-004 | Service | ClusterIP、TCP/8080 | 未適用 |
| WL-005 | Route | edge、redirect、`example-web-dev.apps...` | 未適用 |
| WL-006 | RBAC group | `example-app-operators`／namespace read-only | IdP group連携前 |
| WL-007 | PDB | `minAvailable: 1` | 未適用 |
| WL-008 | NetworkPolicy | default deny + same namespace/Ingress/DNS allow | 未適用・通信試験前 |
| WL-009 | topology spread | hostname、maxSkew 1、DoNotSchedule、app label | 未適用。2 Pod別worker配置を要確認 |

## 9. 実装ファイルと検証状態

| ファイル群 | 静的作成 | 構文検証 | server/実機検証 |
| --- | --- | --- | --- |
| `install/*.yaml.example` | 済 | YAML parser予定 | 未実施 |
| `install/openshift/*.bu.example` | 済 | YAML parser予定、Butane未実施 | 未実施 |
| `install/haproxy/*` | 済 | HAProxy/keepalived実機validator未実施 | 未実施 |
| `ansible/` | 済 | tooling有無に応じてsyntax-check予定 | 未実施 |
| `manifests/` | 済 | YAML parser予定 | server dry-run未実施 |

## 10. 変更履歴

| 版 | 日付 | ID | 変更 | 承認 |
| --- | --- | --- | --- | --- |
| 0.1 | 2026-08-17 | 初版 | 架空値とTBDを分離 | 未承認 |
