# 06. クラスタ構成・アーキテクチャ設計書

## 文書管理

| 項目 | 内容 |
| --- | --- |
| 文書ID | `ARCH-DES-001` |
| 上位設計 | [05-basic-design.md](05-basic-design.md) |
| 版／状態 | 0.1／Draft（机上設計） |
| 基準日 | 2026-08-17 |
| 作成 | 生成 AI 支援ドラフト（本人レビュー前） |
| レビュー／承認 | Platform/Hardware/Network／基盤責任者（未実施） |

> [!IMPORTANT]
> 架空の机上アーキテクチャです。ハードウェア互換性、性能、障害動作、インストールは未確認・未実施であり、商用経験の証明ではありません。

## 1. アーキテクチャ判断

| ID | 設計 | 根拠 | Owner | 状態 |
| --- | --- | --- | --- | --- |
| ARC-001 | 単一production想定クラスタ `ocp-prd`を構成 | 共通基盤の学習範囲を明確化 | Platform | Draft |
| ARC-002 | Agent-based Installer、`platform: none`でインストール | BD-002、外部インフラ管理 | Platform | Draft |
| ARC-003 | Control Planeを3台で構成し業務Podを配置しない | etcd quorumと管理面分離 | Platform | Draft |
| ARC-004 | Workerを3台で構成 | workload/Ingressの分散 | Platform | Draft |
| ARC-005 | DNS/NTP/LB/Proxy/IdP/CSI/backup storeをクラスタ外に置く | 責任分界と`platform: none`要件 | 各外部Owner | TBDを含む |
| ARC-006 | 現行サイジングを入力値とし、性能・容量試験後に確定 | 実負荷・VM要件が不明 | Platform/Application | 未検証 |
| ARC-007 | 物理障害ドメインへnodeとLBを分散 | 共通障害回避 | Hardware/Network | 配置TBD |
| ARC-008 | 専用infra nodeを設けず、worker容量へIngress/監視分を見込む | 3 workerという共通シナリオ | Platform | 負荷検証TBD |
| ARC-009 | Operatorは対応版・channel・承認方式を固定してから導入 | 独立したライフサイクル管理 | Platform | 版TBD |
| ARC-010 | Virtualization/MTVは第2段階で追加 | 基盤受入とPoCを分離 | Platform | 未実施 |

## 2. 構成図

- [プラットフォーム構成図](../diagrams/platform-architecture.mmd)
- [ネットワーク通信図](../diagrams/network-flow.mmd)

図中の値は設計入力で、到達性や冗長性を実証したものではありません。

## 3. 物理・論理ノード構成

| Host | Role | IPv4 | vCPU | Memory | Boot disk | 配置方針 | 状態 |
| --- | --- | --- | ---: | ---: | ---: | --- | --- |
| cp01.ocp-prd.example.com | master | 192.0.2.21 | 8 | 32 GiB | 250 GiB | 障害ドメインA（仮） | HCL/配置TBD |
| cp02.ocp-prd.example.com | master | 192.0.2.22 | 8 | 32 GiB | 250 GiB | 障害ドメインB（仮） | HCL/配置TBD |
| cp03.ocp-prd.example.com | master | 192.0.2.23 | 8 | 32 GiB | 250 GiB | 障害ドメインC（仮） | HCL/配置TBD |
| wk01.ocp-prd.example.com | worker | 192.0.2.31 | 32 | 256 GiB | 250 GiB | 障害ドメインA（仮） | HCL/配置TBD |
| wk02.ocp-prd.example.com | worker | 192.0.2.32 | 32 | 256 GiB | 250 GiB | 障害ドメインB（仮） | HCL/配置TBD |
| wk03.ocp-prd.example.com | worker | 192.0.2.33 | 32 | 256 GiB | 250 GiB | 障害ドメインC（仮） | HCL/配置TBD |

- NIC速度・本数、bond、VLAN、MTU、BMC接続、ディスク性能はTBDです。
- `platform: none`では基盤ライフサイクルをクラスタが自動管理する前提にしません。故障nodeの交換・再導入はPlatformとHardwareのRunbook対象です。
- MACは [SCENARIO.md](../SCENARIO.md) を参照し、Agentのhost mappingに使用します。

## 4. コンポーネント配置

| 層 | 主なコンポーネント | 配置 | 永続性／注意 |
| --- | --- | --- | --- |
| Control plane | kube-apiserver、controller manager、scheduler、etcd、各Operator | cp01〜03 | etcdはquorum。手動データ操作禁止 |
| Ingress | Ingress Controller/router | wk01〜03上に分散する想定 | 実replica/配置は導入後確認 |
| Registry | Image Registry Operatorとregistry Pod | worker | 本構成では自動共有storageがないため導入時は`Removed`でbootstrap。導入後にサポートされる永続storageを設定して`Managed`へ移行 |
| Platform monitoring | CMO、Prometheus、Alertmanager等 | worker（標準配置を起点） | PV、retention、anti-affinityを要確認 |
| User monitoring | user workload monitoring | worker | 有効化・資源量TBD |
| Logging | collector、Lokiまたは外部forwarder | worker | 対応版・object storage・保持TBD |
| Application | Web/API workload | worker | 原則複数replica、PDB、spread、probe |
| Virtualization（第2段階） | HyperConverged Operator、VM launcher等 | worker | CPU支援・CSI・ネットワーク適合が前提 |

## 5. インストールアーキテクチャ

1. 管理端末で対象4.22.zの`openshift-install`と`oc`を検証済み経路から取得します。
2. `install-config.yaml`と`agent-config.yaml`を作成し、機密値は外部Secret保管から一時注入します。
3. Agent ISOを生成し、6ホストをISO/PXE相当の承認済み方式で起動します。
4. 本机上設計ではcp01（`192.0.2.21`）をrendezvous hostとし、全hostからdiscovery/bootstrap中のTCP/8090到達を確認します。実施前にcp01のMAC/IP、起動可否、要件適合を再確認します。
5. DNS、LB、NTP、Proxy、CA、network CIDR、host inventoryが合格してからインストールを開始します。
6. インストール完了後、ClusterOperator、node、CSR、MCP、Ingress、registry、monitoringを確認します。`platform: none`で共有object storageを自動提供しない本構成では、Image Registry Operatorが`managementState: Removed`であることと、installer完了自体を区別します。
7. 外部CSI等のサポートされるproduction向け永続storageを構成し、Image Registry Operatorを`Managed`へ変更して、Pod、PVC、push/pull、再起動後の永続性を確認します。registry未構成のまま基盤本番受入へ進みません。
8. 組織IdPと代替管理者を確認してからbootstrap認証情報を廃止します。

> [!WARNING]
> rendezvous hostの選定、exact CLI、生成物、ポートは対象z版の公式文書で再確認します。この手順は実行結果ではありません。

## 6. 制御・データフロー

| Flow ID | 起点 | 終点 | 目的 | 設計上の保護 |
| --- | --- | --- | --- | --- |
| FLW-001 | 管理者/CI | API VIP → control plane | API操作 | TLS、IdP、RBAC、監査 |
| FLW-002 | 利用者 | Ingress VIP → router → Service/Pod | Web/API利用 | TLS、Route、NetworkPolicy、アプリ認証 |
| FLW-003 | node/Operator | Proxy → Red Hat endpoints | image/update/catalog | Proxy CA、egress制御、ログ |
| FLW-004 | Pod/node | CSI endpoint | PV操作/I/O | CSI権限、storage network、暗号化 |
| FLW-005 | OADP/backup job | object storage 192.0.2.15 | バックアップ保管 | TLS、専用credential、保持 |
| FLW-006 | OAuth | 組織IdP | 認証 | TLS、MFA責任、個人ID |
| FLW-007 | collector | log store/SIEM | ログ転送 | TLS、最小権限、buffer/欠損監視 |

## 7. 可用性と障害モード

| 障害 | 設計上の期待 | 必要条件 | 試験 | 残余リスク |
| --- | --- | --- | --- | --- |
| Control Plane 1台停止 | API/etcd継続 | 他2台正常、network/LB正常 | TST-HA-002 | 2台同時喪失でquorum喪失 |
| Worker 1台停止 | 複数replica workload再配置 | spare capacity、PDB、CSI attach | TST-HA-001 | 単一replica・RWO再attach時間 |
| LB node 1台停止 | VIPをもう1台へ引継ぎ | HA方式、health check、state同期 | TST-LB-004 | 製品/切替時間TBD |
| DNS/NTP 1系停止 | もう1系を利用 | resolver/時刻同期clientの複数設定 | TST-DNS-004、TST-NTP-001 | 両系同一障害ドメインなら無効 |
| CSI経路/装置障害 | 冗長経路・装置でI/O継続 | 製品対応、multipath/fencing | TST-STG-002、TST-STG-006（製品障害試験は追加TBD） | 未選定 |
| Proxy停止 | 稼働中workloadは継続、更新/取得に影響 | 既存image、内部通信 | TST-PRX-004 | Proxy冗長性未定義 |

## 8. 容量・性能方針

- Workerの総入力値は96 vCPU／768 GiBです。ただしOS、Ingress、監視、ログ、DaemonSet、障害退避用を差し引いていません。
- 1 worker喪失後も重要workloadを収容できることを容量試験で確認します。単純な総量だけで合格にしません。
- CPU/Memory request・limit、PV使用量、Pod数、API負荷、network帯域、storage latency/IOPSを観測します。
- しきい値案: 平常時request 60%以下、障害時80%以下、PV使用率70%警告/85%重大。これは暫定で、負荷試験と増設リードタイムにより確定します。
- VM PoCのCPU overcommit、memory overcommit、hugepages、live migration用帯域は第2段階で別途決定します。

## 9. Scheduling方針

- 業務workloadはworkerのみへ配置します。
- 重要workloadは2replica以上を原則とし、`topologySpreadConstraints`またはpod anti-affinityを設計します。
- PDBは保守性を高めますが、強すぎる設定がdrainを妨げるため、replica数と合わせてレビューします。
- system namespaceの標準配置を根拠なく変更しません。
- Virtualization導入後のVM配置はCPU機能、storage access、eviction strategyを含めて再設計します。

## 10. Operator・ライフサイクル方針

| Operator/機能 | 導入段階 | 方針 | 確定事項 |
| --- | --- | --- | --- |
| 基盤標準Operator | 第1段階 | OCPリリース管理下 | OCP z版 |
| CSI Operator/driver | 第1段階 | ベンダー/Red Hat対応表に固定 | 製品・版・更新順 |
| OADP | 第1段階 | アプリ保護用。etcd代替にしない | 版、plugin、object store |
| Logging/Loki | 第1段階候補 | 短期調査と外部転送を分離 | 対応版、容量、保持 |
| OpenShift Virtualization | 第2段階 | 対応channel/版をPoC前固定 | CPU/CSI/MTV互換性 |
| MTV | 第2段階 | VM 3台のPoC限定 | source版・権限・停止時間 |

install planのAutomatic/Manualは、セキュリティ修正速度と互換性レビューの両方を考慮し、G3で製品ごとに決めます。

## 11. アーキテクチャ受入基準

- 6nodeのHCL、資源、Firmware、BIOS、NIC、障害ドメインが確認済み。
- 1 Control Plane、1 Worker、1 LB nodeの各単一障害試験が期待どおり。
- CIDR非重複、DNS正逆引き、LB/Proxy/NTP/CSI疎通が合格。
- 平常時および1 worker喪失時のcapacityが目標内。
- Operator互換性表と更新順序が承認済み。
- 実測証跡が試験ID・変更IDに紐づく。本版はすべて未実施。

## 12. 公式根拠

- [Agent-based Installer supported platforms and platform none requirements（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installing_an_on-premise_cluster_with_the_agent-based_installer/preparing-to-install-with-agent-based-installer)
- [Cluster Network Operator / OVN-Kubernetes（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/networking_operators/cluster-network-operator)
- [Control plane backup and restore（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/backup_and_restore/control-plane-backup-and-restore)
- [Configuring the Image Registry Operator（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/registry/configuring-registry-operator)

## 13. 承認・変更履歴

| 役割 | 判定 | 日付 | コメント |
| --- | --- | --- | --- |
| Platform lead | 未レビュー | - | HCL/性能未確認 |
| Hardware/Network lead | 未レビュー | - | 外部構成未確定 |
| 基盤責任者 | 未承認 | - | 架空案件 |

| 版 | 日付 | 内容 | 作成者 |
| --- | --- | --- | --- |
| 0.1 | 2026-08-17 | 初版 | 生成 AI 支援ドラフト |
