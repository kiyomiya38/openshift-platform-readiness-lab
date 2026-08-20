# 13. OpenShift基盤 詳細設計書

## 文書管理

| 項目 | 内容 |
| --- | --- |
| 文書ID | `DD-OCP-001` |
| 案件 | Example Enterprise OpenShift 基盤導入 |
| 版／状態 | 0.1／Draft（机上設計） |
| 基準日 | 2026-08-17 |
| 対応要件 | `REQ-PLT-*`、`REQ-NET-*`、`REQ-IAM-001`、`REQ-SEC-001`、`REQ-OPS-001` |
| レビュー／承認 | 未レビュー／未承認 |
| 実機反映 | 未実施 |

> [!IMPORTANT]
> 本書は一般公開用の架空プロジェクトにおける詳細設計サンプルです。ここにあるIP、MAC、FQDN、サイジングは実環境の値ではありません。正確なOpenShift 4.22.z、ハードウェア、NIC、DNS、LB、Proxy CA、CSI、IdPを確定しない限り構築へ進みません。

## 1. 設計入力と状態

| ID | 入力 | 設計値 | 状態／確定方法 |
| --- | --- | --- | --- |
| DD-PLT-001 | OpenShift | 4.22.z | `TBD-001`。導入前にリリースノート、ライフサイクル、利用するinstallerの版を固定 |
| DD-PLT-002 | 導入方式 | Agent-based Installer、`platform: none` | Draft。対象z版のinstallerによるschema検証が必要 |
| DD-PLT-003 | トポロジー | Control Plane 3、Worker 3 | 架空入力。HCL・容量確認前 |
| DD-PLT-004 | ノードOS | RHCOS | 固定方針。RHELをOpenShiftノードへ直接導入しない |
| DD-NET-001 | IP方式 | IPv4、静的IP、MTU 1500 | NIC名・switch側MTUは要現物照合 |
| DD-NET-002 | CNI | OVN-Kubernetes | 4.22設計値 |
| DD-EXT-001 | 外部サービス | DNS、NTP、冗長LB、Proxy | 製品・運用方式は各Owner承認前 |

正本となる共通入力は [SCENARIO.md](../SCENARIO.md)、要件は [01-requirements.md](01-requirements.md)、TBDは [02-assumptions-constraints.md](02-assumptions-constraints.md) です。

## 2. 構成とノード

| Host | Role | IPv4 | MAC（架空） | vCPU／Memory／Boot | root device |
| --- | --- | --- | --- | --- | --- |
| `cp01.ocp-prd.example.com` | master／rendezvous | `192.0.2.21` | `02:00:00:00:00:21` | 8／32 GiB／250 GiB | `/dev/sda`（要現物確認） |
| `cp02.ocp-prd.example.com` | master | `192.0.2.22` | `02:00:00:00:00:22` | 8／32 GiB／250 GiB | 同上 |
| `cp03.ocp-prd.example.com` | master | `192.0.2.23` | `02:00:00:00:00:23` | 8／32 GiB／250 GiB | 同上 |
| `wk01.ocp-prd.example.com` | worker | `192.0.2.31` | `02:00:00:00:00:31` | 32／256 GiB／250 GiB | 同上 |
| `wk02.ocp-prd.example.com` | worker | `192.0.2.32` | `02:00:00:00:00:32` | 32／256 GiB／250 GiB | 同上 |
| `wk03.ocp-prd.example.com` | worker | `192.0.2.33` | `02:00:00:00:00:33` | 32／256 GiB／250 GiB | 同上 |

- 役割は `agent-config.yaml` で明示し、ランダム割当を避けます。
- rendezvous IPはcontrol planeの `192.0.2.21` とします。全ホストからこのホストのTCP/8090をdiscovery/bootstrap中のみ許可します。
- BMCアドレスと資格情報は本リポジトリへ格納しません。
- RAID、multipath、boot mode、Secure Boot、Firmware、NIC名、ディスク識別子はHardware teamの現物台帳と照合するまで未確定です。

## 3. Installer入力

### 3.1 install-config

| 設計ID | キー | 値 | 実装 |
| --- | --- | --- | --- |
| DD-IC-001 | `metadata.name`／`baseDomain` | `ocp-prd`／`example.com` | [install-config.yaml.example](../install/install-config.yaml.example) |
| DD-IC-002 | `controlPlane.replicas`／`compute.replicas` | 3／3 | 同上 |
| DD-IC-003 | `platform` | `none: {}` | 同上。VIPをinstallerへ設定せず外部で用意 |
| DD-IC-004 | `networking` | §4参照 | 同上 |
| DD-IC-005 | `proxy` | 組織ProxyとnoProxy | Proxy CA・許可先はTBD |
| DD-IC-006 | `pullSecret`／`sshKey` | 秘密管理領域から作業時に注入 | Gitにはプレースホルダーのみ |

### 3.2 agent-config

[agent-config.yaml.example](../install/agent-config.yaml.example) に6ホストのhostname、role、MAC、rootDeviceHints、NMState形式の静的ネットワークを定義します。

- `apiVersion: v1beta1` は4.22カスタマイズ導入章の例に合わせた暫定値です。4.22文書内に版の異なる例もあるため、**確定した4.22.zの `openshift-install` を最終判定元**とします。
- `interfaces[].macAddress` と `networkConfig.interfaces[].mac-address` は同じ物理NICを示す必要があります。
- `rootDeviceHints.deviceName` は対象ディスクを一意に特定できることをBMC/現物で確認します。誤りは別ディスク消去につながるため、起動前の停止条件です。
- IPv6は無効、DHCPは無効、default routeは `192.0.2.1`、DNSは2台を指定します。

## 4. ネットワーク詳細

### 4.1 CIDR

| 用途 | 値 | 変更影響 | 検証 |
| --- | --- | --- | --- |
| Machine network | `192.0.2.0/24` | 導入後変更困難 | IPAM、routing、重複照合 |
| Pod network | `10.128.0.0/14`、hostPrefix `/23` | 導入後変更は高影響 | `oc get network.config/cluster -o yaml` |
| Service network | `172.30.0.0/16` | 導入後変更は高影響 | 同上 |
| Podman既定bridge | `10.88.0.0/16` | machine networkと重複禁止 | installer hostの `podman network inspect` |

Machine、Pod、Service、組織WAN、VPN、Proxy、ストレージ、移行元ネットワークとの重複をNetwork teamが承認します。

### 4.2 DNS

| 名前 | Type | 応答先 | 解決範囲 |
| --- | --- | --- | --- |
| `api.ocp-prd.example.com` | A + PTR | `192.0.2.10` | クライアントと全ノード |
| `api-int.ocp-prd.example.com` | A + PTR | `192.0.2.10` | 全ノード |
| `*.apps.ocp-prd.example.com` | wildcard A | `192.0.2.11` | クライアントと全ノード |
| `cp01`〜`cp03` | A + PTR | `192.0.2.21`〜`.23` | 全ノード |
| `wk01`〜`wk03` | A + PTR | `192.0.2.31`〜`.33` | 全ノード |

申請例は [forward-records.example](../install/dns/forward-records.example) と [reverse-records.example](../install/dns/reverse-records.example) です。apps wildcardのPTRは要求しません。TTL、zone、serial、CNAME利用可否はDNS運用標準に従います。

### 4.3 Load Balancer

| Listener | VIP | Backend | 方式 | Health check／制約 |
| --- | --- | --- | --- | --- |
| API | `192.0.2.10:6443` | control plane 3台 `:6443` | L4、round-robin、session persistenceなし | HTTPS `/readyz`、5秒間隔、fall 3、rise 2 |
| MCS | `192.0.2.10:22623` | control plane 3台 `:22623` | L4、round-robin | TCP。内部利用に限定 |
| Ingress HTTPS | `192.0.2.11:443` | worker 3台 `:443` | L4、source balance | TCP |
| Ingress HTTP | `192.0.2.11:80` | worker 3台 `:80` | L4、source balance | TCP |

- `/readyz`異常からbackend除外まで30秒以内とします。
- HAProxy例は [haproxy.cfg.example](../install/haproxy/haproxy.cfg.example) です。SELinux enforcingを維持し、`haproxy_connect_any` を承認の上で有効化します。
- `/readyz` health checkの `verify none` はbackend node名とAPI serving証明書の名前が異なるためのcheck専用設定例で、利用者TLSはL4 passthroughのままです。採用時はNetwork/Securityが内部経路リスクをレビューします。
- 2台のLBとVIP移動を説明するkeepalived例を置きますが、VIP方式、VRRP通信、NIC名、split-brain対策は `TBD-004` です。例を採用決定とみなしません。

### 4.4 Firewall主要項目

| ID | Source | Destination | Protocol/Port | 期間／用途 |
| --- | --- | --- | --- | --- |
| DD-FW-001 | 管理クライアント、全ノード | API VIP | TCP/6443 | 常時 |
| DD-FW-002 | 全ノード | API VIP | TCP/22623 | 常時・内部のみ |
| DD-FW-003 | 利用クライアント、全ノード | Ingress VIP | TCP/80,443 | 常時 |
| DD-FW-004 | 6ノード | `cp01` rendezvous | TCP/8090 | discovery/bootstrapのみ |
| DD-FW-005 | 6ノード | NTP 2台 | UDP/123 | 常時 |
| DD-FW-006 | 6ノード、踏み台 | 組織Proxy | TCP/3128 | 常時 |
| DD-FW-007 | LB相互 | peer LB | VRRP/IP protocol 112または承認方式 | keepalived採用時 |

完全なOpenShift通信要件、Proxy許可先、CSI/IdP/監視通信は対象z版と製品確定後にFirewall申請へ展開します。

## 5. Proxy・時刻同期

- `httpProxy` と `httpsProxy` は `http://proxy.example.com:3128`。認証情報が必要ならSecret管理し、URLへ平文記録しません。
- noProxyはlocalhost、Machine/Pod/Service CIDR、`.cluster.local`、`.svc`、クラスタドメインを含めます。実環境の内部Registry、CSI、IdP、監視先を追加してから確定します。
- Proxy CAは `additionalTrustBundle` の要否をSecurity teamが判断します。現例には偽の証明書も置きません。
- chrony設定は [99-master-chrony.bu.example](../install/openshift/99-master-chrony.bu.example) と [99-worker-chrony.bu.example](../install/openshift/99-worker-chrony.bu.example) をButane 4.22でMachineConfigへ変換します。RHCOSへAnsibleやSSHで直接変更しません。

## 6. 外部RHELサービスの自動化境界

[ansible/](../ansible/README.md) はRHEL踏み台・LBだけを対象にします。

外部支援hostはRHEL 9.x（minor TBD）を前提とし、OpenShift subscriptionとは別に有効なRHEL subscriptionと承認済みrepositoryを必要とします。

| Playbook | 対象 | 変更 | 制御 |
| --- | --- | --- | --- |
| `preflight.yml` | `rhel_support` | なし | OS、DNS、Proxy、chrony確認 |
| `configure-load-balancers.yml` | `load_balancers` | package、HAProxy、keepalived、SELinux boolean | `external_service_change_approved=true` と変更承認が必要 |

RHCOSノードはinventoryへ登録しません。Ansibleの承認ガードは変更承認の代替ではありません。

## 7. Project・ワークロード標準

[manifests/](../manifests/README.md) は、クラスタ導入後の最小構成確認に使うサンプルです。

| 設計ID | 対象 | 設定 |
| --- | --- | --- |
| DD-WL-001 | Namespace | `example-web-dev`、devラベル、restricted PSAラベル |
| DD-WL-002 | Quota/Limit | CPU、memory、Pod、Service、Route、PVCの上限とcontainer既定値 |
| DD-WL-003 | Workload | 2 replicas、hostname単位topology spread（maxSkew 1/DoNotSchedule）、非root、capability全drop、3種probe、資源request/limit |
| DD-WL-004 | Service/Route | ClusterIP `8080`、edge TLS、HTTPはHTTPSへredirect |
| DD-WL-005 | 可用性 | PDB `minAvailable: 1`、rolling update `maxUnavailable: 0`、2 Podを別workerへ配置 |
| DD-WL-006 | NetworkPolicy | ingress/egress denyを起点に同一namespace、Ingress、DNSのみ許可 |

サンプルimageのtagは設計説明用です。実利用前に承認済みRegistryのimmutable digestへ固定します。

PDBが制御するのはdrainなどの自発的disruptionです。突然のnode障害に対する継続を保証しないため、topology spread、障害時の再schedule余力、probe、`TST-HA-001`の実測で判定します。

## 8. RBAC・SCC・Secret

- 架空グループ `example-app-operators` には、`example-web-dev` 内のPod/Log/Service/Deployment/Routeのread-only Roleだけを付与します。
- workload ServiceAccountにはRoleBindingを与えず、tokenの自動mountも無効にします。
- ClusterRoleBindingや `cluster-admin` はサンプルに含めません。
- 標準の `restricted-v2` SCCで動くことを目標とし、カスタムSCCは作成しません。実機では `oc adm policy who-can use scc/restricted-v2` とPod admission結果を確認します。
- pull secret、SSH秘密鍵、IdP client secret、証明書秘密鍵、バックアップ資格情報はGitへ保存しません。

## 9. Storage・Operator・運用設定

これらは実装値が未確定のため、成功した設定例を作りません。

| 設計ID | 項目 | 現在の設計 | 構築開始条件 |
| --- | --- | --- | --- |
| DD-STG-001 | CSI／StorageClass | 製品、provisioner、RWO/RWX、Snapshot、暗号化、reclaimPolicyはTBD | `TBD-006`解消と互換性確認 |
| DD-REG-001 | 内部Image Registry | shareable storage未構成のplatform noneでは初期状態がRemovedとなり得る。CSI確定後にproduction永続storageを割り当てManagedへ変更 | StorageClass/PVC、容量、backup、push/pull試験承認 |
| DD-BKP-001 | etcd | Platform teamが定期取得し、クラスタ外へ暗号化保管 | 手順・保管・復元試験承認 |
| DD-BKP-002 | App/PV | OADP + CSI Snapshot/データムーバー候補 | `TBD-009`解消、RPO試験 |
| DD-BKP-003 | 外部DB | DBネイティブ方式 | Application/DB teamの整合点定義 |
| DD-MON-001 | 監視 | 標準Cluster Monitoringを起点に通知連携 | 通知先・severity・Runbook承認 |
| DD-LOG-001 | ログ | infrastructure/application/auditを分離 | Logging版・保存先・保持期間確定 |
| DD-IDM-001 | IdP | 組織IdP連携 | 方式、属性、MFA、break-glass確定 |

RPO 1時間/RTO 4時間はアプリデータの目標であり、etcdバックアップだけでは満たしません。PV、外部DB、Kubernetesリソースを同じ整合点へ復元し、業務再開承認まで実測します。

## 10. 実装順序と差し戻し

| 順序 | 成果物／操作 | 前提 | 検証 | 失敗時 |
| ---: | --- | --- | --- | --- |
| 1 | DNS/LB/NTP/Proxy/Firewall | 各Owner承認 | preflight、設定レビュー | 先へ進まない |
| 2 | installer入力を安全領域へ複製・Secret注入 | z版固定、HCL確認 | YAML、installer、Butane検証 | 作業コピーを破棄し設計へ戻る |
| 3 | Agent ISO生成 | 入力レビュー済み | hash、生成ログ | ISOを配布しない |
| 4 | 6台をISO起動 | G4承認、boot disk二者照合 | Agent console checks | disk書込み前は停止・media解除 |
| 5 | bootstrap/install監視 | 6台登録、疎通正常 | installer wait-for、ログ | 反復再起動せずログ採取・No-Go |
| 6 | クラスタ健全性 | install-complete | CV/CO/Node/CSR | 後続設定を止める |
| 7 | CSI/IdP/監視/backup等 | 各TBD解消 | 個別試験 | 変更単位で差し戻す |
| 8 | サンプルworkload | 非本番承認 | server dry-run、rollout、Route | workloadだけを直前版へ戻す |

ISO起動後のRHCOS disk書込みは単純なundoができません。No-Go時はログを保全し、BMCからmediaを外し、ネットワーク広告を戻した上で、Hardware team承認の再imageまたは元システム復元を行います。詳細は [15-build-procedure.md](15-build-procedure.md) に記載します。

## 11. レビュー・未確定事項

| 観点 | 必須レビュー | 状態 |
| --- | --- | --- |
| Installer | 正確な4.22.z、AgentConfig schema、入力生成結果 | 未実施 |
| Hardware | HCL、Firmware、NIC、root disk、障害ドメイン | 未実施 |
| Network | DNS、LB、VIP、readyz、Firewall、Proxy、MTU | 未実施 |
| Security | Proxy CA、Secret注入、RBAC、IdP、証明書 | 未実施 |
| Storage/DR | CSI、Snapshot、復元、RPO/RTO | 未実施 |
| Operations | 監視、ログ、backup、更新、連絡、証跡 | 未実施 |

## 12. 公式根拠

- [OpenShift 4.22 Agent-based Installer: customizations](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installing_an_on-premise_cluster_with_the_agent-based_installer/installing-with-agent-based-installer)
- [OpenShift 4.22 Agent-based Installer: platform noneのDNS/LB要件](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html-single/installing_an_on-premise_cluster_with_the_agent-based_installer/index)
- [OpenShift 4.22 MachineConfig: chrony](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/machine_configuration/machine-configs-configure)
- [OpenShift 4.22 NetworkPolicy](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/network_security/network-policy)

## 13. 承認・変更履歴

| 役割 | 判定 | 日付 | コメント |
| --- | --- | --- | --- |
| Platform責任者 | 未レビュー | - | 机上設計 |
| Network責任者 | 未レビュー | - | 外部サービス方式TBD |
| Security責任者 | 未レビュー | - | Secret/IdP/CA未確定 |

| 版 | 日付 | 内容 | 作成者 |
| --- | --- | --- | --- |
| 0.1 | 2026-08-17 | 初版（構築資材と対応付け） | 文書作成チーム（サンプル） |
