# 07. ネットワーク・DNS・Load Balancer設計書

## 文書管理

| 項目 | 内容 |
| --- | --- |
| 文書ID | `NET-DES-001` |
| 上位設計 | [05-basic-design.md](05-basic-design.md)、[06-architecture-design.md](06-architecture-design.md) |
| 版／状態 | 0.1／Draft（机上設計） |
| 基準日 | 2026-08-17 |
| 作成 | 生成 AI 支援ドラフト（本人レビュー前） |
| レビュー／承認 | Network/Platform/Security／基盤責任者（未実施） |

> [!IMPORTANT]
> 架空の文書用IPとドメインによる机上設計です。DNS登録、LB設定、Firewall開通、疎通試験は未実施であり、商用経験の証明ではありません。ポート一覧は代表フローであり、実装前に対象4.22.zの公式一覧を完全照合します。

## 1. 設計判断

| ID | 判断 | 理由 | Owner | 状態 |
| --- | --- | --- | --- | --- |
| NET-001 | Cluster名 `ocp-prd`、base domain `example.com` | 共通シナリオ | Platform/Network | 机上確定 |
| NET-002 | API/API-intは`192.0.2.10`、apps wildcardは`192.0.2.11`へ解決 | APIとIngressの責務分離 | Network | 未登録 |
| NET-003 | 全nodeにA/PTR、APIにA/PTR、API-int/appsに必要なforward recordを用意 | RHCOS hostname/CSRとplatform none要件 | Network | 未登録 |
| NET-004 | DNS TTLは300秒を暫定値とする | 変更時の収束とDNS負荷のバランス | Network | TBD（G3） |
| NET-005 | API LBはL4、stateless、session persistenceなし | Red Hat platform none要件 | Network | 製品TBD |
| NET-006 | API backendはcp01〜03、Ingress backendはwk01〜03 | control plane/Ingress配置 | Network/Platform | Draft |
| NET-007 | API health checkは`/readyz`を用い、異常から30秒以内にpool除外可能な設計 | 公式要件 | Network | 実装方式TBD |
| NET-008 | LB node 2台でVIPを冗長化 | 単一障害点回避 | Network | 製品/HA TBD |
| NET-009 | IPv4・静的IP、OVN-Kubernetesを利用 | 共通シナリオ | Platform/Network | Draft |
| NET-010 | Machine/Pod/Service CIDRは相互および接続先と重複させない | routing不整合防止 | Network | 外部網照合未実施 |
| NET-011 | tenant namespaceはdefault denyを基本とする | 最小通信 | Security/Platform | アプリ要件TBD |
| NET-012 | cluster system namespaceへ一律default denyを適用しない | 基盤通信の破壊防止 | Platform | Draft |
| NET-013 | 外部HTTP(S)は組織Proxy経由 | connected構成とegress統制 | Network/Security | Draft |
| NET-014 | cluster内部、Machine/Pod/Service範囲、`.svc`/`.cluster.local`等をnoProxy候補とする | 内部通信のproxy迂回 | Platform/Network | 正確な値TBD |
| NET-015 | Proxy CAを信頼bundleとして管理する | TLS検証を無効化しない | Security/Platform | CA/TBD |

## 2. アドレス設計

| 用途 | 値 | 管理者 | 変更影響 |
| --- | --- | --- | --- |
| Machine network | `192.0.2.0/24` | Network | host/VIP/LB/bastion全体 |
| Gateway | `192.0.2.1` | Network | 全node外部疎通 |
| Pod network | `10.128.0.0/14`、hostPrefix `/23` | Platform | 導入後変更困難。全接続網と非重複必須 |
| Service network | `172.30.0.0/16` | Platform | 導入後変更困難。全接続網と非重複必須 |
| API VIP | `192.0.2.10` | Network | API/内部API |
| Ingress VIP | `192.0.2.11` | Network | 全Route |
| LB nodes | `192.0.2.12`、`192.0.2.13` | Network | VIP冗長化 |
| Bastion | `192.0.2.14` | Platform | 管理起点。アクセス制御TBD |
| Backup endpoint | `192.0.2.15` | Storage | OADP/backup送信先 |

ノードIP/MACは [SCENARIO.md](../SCENARIO.md) を正とします。これらはRFC 5737の文書用アドレスで、実環境へ設定しません。

## 3. DNS設計

| Record ID | Name | Type | Value | PTR | Owner | 状態 |
| --- | --- | --- | --- | --- | --- | --- |
| DNS-001 | `api.ocp-prd.example.com` | A | `192.0.2.10` | 同じAPI VIPのPTR ownerへ`api`を登録 | Network | 未登録 |
| DNS-002 | `api-int.ocp-prd.example.com` | A | `192.0.2.10` | 同じAPI VIPのPTR ownerへ`api-int`も登録 | Network | 未登録（複数PTRの実装可否はDNS担当確認） |
| DNS-003 | `*.apps.ocp-prd.example.com` | wildcard A | `192.0.2.11` | 不要（要レビュー） | Network | 未登録 |
| DNS-004 | `cp01.ocp-prd.example.com` | A | `192.0.2.21` | 同FQDN | Network | 未登録 |
| DNS-005 | `cp02.ocp-prd.example.com` | A | `192.0.2.22` | 同FQDN | Network | 未登録 |
| DNS-006 | `cp03.ocp-prd.example.com` | A | `192.0.2.23` | 同FQDN | Network | 未登録 |
| DNS-007 | `wk01.ocp-prd.example.com` | A | `192.0.2.31` | 同FQDN | Network | 未登録 |
| DNS-008 | `wk02.ocp-prd.example.com` | A | `192.0.2.32` | 同FQDN | Network | 未登録 |
| DNS-009 | `wk03.ocp-prd.example.com` | A | `192.0.2.33` | 同FQDN | Network | 未登録 |

### 3.1 事前確認

管理端末、LB、各nodeの想定segmentから次を確認します。以下は予定コマンドで、実行済みではありません。

```bash
dig +short api.ocp-prd.example.com A
dig +short api-int.ocp-prd.example.com A
dig +short test.apps.ocp-prd.example.com A
dig +short cp01.ocp-prd.example.com A
dig +short -x 192.0.2.21
```

合格条件は、全resolverで同一結果、timeoutなし、A/PTRの対応一致です。split DNSを使う場合のviewと外部公開範囲は `TBD-NET-001` です。

## 4. Load Balancer設計

| Pool ID | VIP:Port | Mode | Backend | Health check | 公開範囲 | 状態 |
| --- | --- | --- | --- | --- | --- | --- |
| LB-API | `192.0.2.10:6443` | L4 TCP、非sticky | cp01〜03:`6443` | HTTPS `/readyz`相当、30秒以内に反映 | internal + 承認済み管理網 | 製品設定TBD |
| LB-MCS | `192.0.2.10:22623` | L4 TCP | cp01〜03:`22623` | TCP（正確な方式は要確認） | cluster internal | 製品設定TBD |
| LB-ING-HTTP | `192.0.2.11:80` | L4 TCP | wk01〜03:`80` | TCP/HTTP check（方式TBD） | 利用者網 | 製品設定TBD |
| LB-ING-HTTPS | `192.0.2.11:443` | L4 TCP passthrough | wk01〜03:`443` | TCP/HTTPS check（方式TBD） | 利用者網 | 製品設定TBD |

- TLSはOpenShift API/Ingress側で処理し、LBで証明書内容を変更しない方針です。
- API LBへsession persistenceを設定しません。
- Ingress Controller podの実配置とreplicaを確認してからbackendを有効化します。
- LB 2台のVIP方式、state監視、split-brain対策、設定同期、保守手順は `TBD-004` です。
- timeout、connection limit、source IP保持、ログ項目は負荷・監査要件と合わせてG3で決定します。

## 5. 代表通信要件

| FW ID | Source | Destination | Protocol/Port | 用途 | 期間 | Owner | 状態 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| FW-001 | 管理網/Bastion | API VIP | TCP/6443 | API/CLI | 常設 | Network | 未開通 |
| FW-002 | cluster node | API VIP/control plane | TCP/6443 | Kubernetes API | 常設 | Network | 未開通 |
| FW-003 | cluster node | API VIP/control plane | TCP/22623 | Machine Config Server | 常設 | Network | 未開通 |
| FW-004 | 利用者網 | Ingress VIP | TCP/80,443 | Route | 常設 | Network | 未開通 |
| FW-005 | 全cluster host | cp01 `192.0.2.21`（rendezvous host） | TCP/8090 | Agent discovery/bootstrap | 導入時 | Platform/Network | 未開通 |
| FW-006 | cluster node/LB | DNS 192.0.2.2, .3 | UDP/TCP 53 | 名前解決 | 常設 | Network | 未開通 |
| FW-007 | cluster node/LB | NTP 192.0.2.4, .5 | UDP/123 | 時刻同期 | 常設 | Network | 未開通 |
| FW-008 | cluster node/Operator | proxy.example.com | TCP/3128 | 外部HTTP(S) proxy | 常設 | Network/Security | 未開通 |
| FW-009 | backup component | 192.0.2.15 | TCP/443（暫定） | object storage | 常設 | Storage | API/TLS TBD |
| FW-010 | OAuth components | 組織IdP | TCP/443または636（TBD） | 認証 | 常設 | Security | endpoint TBD |
| FW-011 | logging/agent | 外部log/Sysdig | TCP/443等（TBD） | telemetry/log/security | 常設 | Security/Product owner | 製品TBD |
| FW-012 | cluster nodes相互 | cluster nodes | OCP 4.22.z公式必須port一式 | control plane/OVN/Kubelet等 | 常設 | Platform/Network | 明細TBD |

`FW-012`は省略許可ではありません。対象z版確定後、Red Hat公式の全ポート表から送信元・宛先・方向を展開し、Firewall申請と1対1照合することがG3の完了条件です。

## 6. Proxy設計

| 項目 | 暫定値／方針 | 状態 |
| --- | --- | --- |
| HTTP/HTTPS Proxy | `proxy.example.com:3128` | 共通シナリオ |
| 認証 | Secret参照方式。実値を文書へ記載しない | 方式TBD |
| trusted CA | 組織Proxy CA bundleを登録 | CA未受領 |
| noProxy候補 | `.cluster.local,.svc,localhost,127.0.0.1,192.0.2.0/24,10.128.0.0/14,172.30.0.0/16`、API/apps内部名 | 4.22.z要件で再確認 |
| 外部許可先 | Red Hat registry、quay、update/catalog等 | 正確なFQDN一覧TBD |

合格条件は、TLS検証を無効化せず、release image、Operator catalog、必要な外部APIへ到達でき、cluster内部通信がProxyへ迂回しないことです。

## 7. OVN-Kubernetes・NetworkPolicy

- `networkType: OVNKubernetes`、cluster network `10.128.0.0/14`、hostPrefix `/23`を指定します。
- 既存WAN、VPN、storage、IdP、VM移行元を含む全route tableとCIDR重複を確認します。
- MTUはunderlay、encapsulation overhead、storage/VM networkを測定して決めます。暫定値を推測しません。
- tenant namespaceには、DNS、Ingressからの必要通信、監視scrape、アプリ間、DB/外部API egressを個別許可してからdefault denyを適用します。
- `default`、`kube-*`、`openshift-*`等のsystem namespaceへ一括policyを適用しません。
- 第2段階のVM secondary network、Multus、VLAN、live migration networkは別設計とします。

## 8. ネットワーク試験計画

| 試験ID | 確認 | 期待結果 | 証跡 | 状態 |
| --- | --- | --- | --- | --- |
| TST-DNS-001 | API/API-int/apps forward lookup | 設計VIPへ解決 | `dig`出力 | 未実施 |
| TST-DNS-002 | node A/PTR | 全nodeでFQDN/IP一致 | `dig`出力 | 未実施 |
| TST-DNS-004 | DNS片系障害 | second resolverで許容時間内に正逆引きを継続 | resolver query/timeline | 未実施 |
| TST-NTP-001 | NTP片系障害 | 他sourceで同期を継続しoffset/stratumが合意値内 | `chronyc`出力/timeline | 未実施 |
| TST-LB-001 | API readyzとbackend除外 | unhealthy nodeを30秒以内に除外 | LB/API log | 未実施 |
| TST-LB-004 | LB node failover | 接続継続または合意時間内復旧 | packet/log/time | 未実施 |
| TST-NET-001 | CIDR非重複 | 重複なし | route/IPAMレビュー | 未実施 |
| TST-NET-002 | Pod-to-Pod/Service/egress | 許可通信のみ成功 | test pod/log | 未実施 |
| TST-NET-003 | NetworkPolicy否定試験 | 未許可通信が失敗 | command/event | 未実施 |
| TST-PRX-001 | Proxy/CA経由のimage pull | TLS成功、image取得 | node/operator log | 未実施 |
| TST-PRX-004 | Proxy障害・復旧 | 依存操作を検知し、内部workload継続、復旧後に外部経路再開 | alert/event/proxy log | 未実施 |

## 9. TBD・中断条件

| ID | 内容 | Owner | 期限 | 中断条件 |
| --- | --- | --- | --- | --- |
| TBD-NET-001 | split DNS/view/TTL | Network | G3前 | resolver結果不一致なら導入中断 |
| TBD-NET-002 | LB製品・HA・check・timeout | Network | G3前 | 単一障害試験不能なら導入中断 |
| TBD-NET-003 | MTU/NIC/bond/VLAN | Network/Hardware | G3前 | packet loss/fragmentation懸念未解消なら中断 |
| TBD-NET-004 | 公式全portのFirewall明細 | Platform/Network | G3前 | FW-012未展開なら導入中断 |
| TBD-NET-005 | Proxy CA/許可先/noProxy | Security/Network | G3前 | image/update到達未確認なら中断 |

## 10. 公式根拠

- [Agent-based Installer: platform none DNS/LB requirements（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installing_an_on-premise_cluster_with_the_agent-based_installer/preparing-to-install-with-agent-based-installer)
- [OpenShift 4.22 Networking overview（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html-single/networking_overview/index)
- [OpenShift 4.22 OVN-Kubernetes network plugin（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html-single/ovn-kubernetes_network_plugin/index)

## 11. 承認・変更履歴

| 役割 | 判定 | 日付 | コメント |
| --- | --- | --- | --- |
| Network lead | 未レビュー | - | LB/FW/DNS未設定 |
| Platform/Security | 未レビュー | - | 公式port/Proxy CA未確認 |
| 基盤責任者 | 未承認 | - | 架空案件 |

| 版 | 日付 | 内容 | 作成者 |
| --- | --- | --- | --- |
| 0.1 | 2026-08-17 | 初版 | 生成 AI 支援ドラフト |
