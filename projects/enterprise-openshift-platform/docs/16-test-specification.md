# 16. OpenShift基盤 試験仕様書

## 文書管理

| 項目 | 内容 |
| --- | --- |
| 文書ID | `TEST-SPEC-OCP-001` |
| 案件 | Example Enterprise OpenShift 基盤導入 |
| 版／状態 | 0.1／Draft（机上仕様） |
| 基準日 | 2026-08-17 |
| 対象版／構成 | OpenShift 4.22.z（z版TBD）／構成commit未設定 |
| 実施期間／環境 | 未設定／実機環境なし |
| 実施・確認・承認 | 未設定／未設定／未承認 |

> [!IMPORTANT]
> 本書は予定試験を定義したもので、試験を実行した証拠ではありません。全ケースの現在結果は [17-test-results.md](17-test-results.md) で `NOT RUN` としています。

## 1. 判定・開始・中止基準

| 状態 | 定義 |
| --- | --- |
| Pass | 実測したactualが期待結果を満たし、指定証跡がレビュー可能である |
| Fail | 実行できたが期待結果を満たさない。事実と課題IDを記録する |
| Blocked | 実行を開始したが前提不足で完遂できない。Failと区別する |
| NOT RUN | 実行していない。期待結果をactualへ転記しない |

開始条件:

- 対象z版、構成commit、試験環境、実施者、確認者、変更IDを固定している。
- 基盤健全性、監視、バックアップ/復旧点を事前確認している。
- 障害注入、停止、負荷、復元、更新試験は非本番で個別承認されている。
- CSI、IdP、Logging、OADP、Virtualization/MTVは採用版と互換性を確認している。

中止条件:

- 想定外の業務/共有環境影響、データ損失、quorum喪失、復旧不能の懸念。
- 重大alert、既存のDegraded ClusterOperator、証跡へのSecret露出。
- 対象、手順、復旧点、判断者の不一致。

## 2. 共通事前・事後確認

```bash
oc whoami --show-server
oc get clusterversion
oc get clusteroperators
oc get nodes
oc get events -A --sort-by=.lastTimestamp
```

事前・事後とも、対象APIが一致し、必須ClusterOperatorが `Available=True`、`Progressing=False`、`Degraded=False`、6ノードがReadyであることを確認します。既知例外は試験前に課題IDと受容者を記録します。

## 3. 構築・ノード・外部サービス

| 試験ID | 要件／設計 | 種別・優先度 | 操作・確認 | 期待結果 | 主証跡 |
| --- | --- | --- | --- | --- | --- |
| TST-INS-001 | REQ-PLT-001／DD-IC-* | 正常・High | 承認済み入力で `openshift-install ... agent create image` | errorなし、ISO生成、版/hash一致、Secret非露出 | version、生成log、hash |
| TST-INS-002 | REQ-PLT-001 | 正常・High | `agent wait-for install-complete` | install completeを返す | installer log |
| TST-INS-003 | REQ-PLT-001 | 正常・High | `oc get clusterversion,clusteroperators` | z版一致、全必須operator正常 | YAML/表形式出力 |
| TST-INS-004 | REQ-PLT-001 | 正常・High | Console Routeへ承認clientからHTTPS接続 | DNS/TLS/認証画面が正常 | curl header、画面記録 |
| TST-REG-001 | REQ-STG-001／DD-REG-001 | 正常・High | Registry config、Operator、Pod/PVCを確認し、承認済みtest imageをpush/pull | production永続storageで`Managed`、Pod/PVC正常、push/pull一致。未構成は本番受入No-Go | config、PVC、Pod、digest |
| TST-NOD-001 | REQ-PLT-002 | 正常・High | `oc get nodes -o wide --show-labels` | master 3、worker 3、hostname/IP/role一致、Ready | node出力 |
| TST-NOD-002 | REQ-PLT-003 | 正常・High | `oc debug node/<node> -- chroot /host rpm-ostree status` | 全nodeがRHCOSでMCO管理、直接RHEL導入なし | node別出力 |
| TST-DNS-001 | REQ-NET-001 | 正常・High | 2 DNSとnode/clientからAPI/API-intを `dig` | いずれも `192.0.2.10`、応答一貫 | dig出力 |
| TST-DNS-002 | REQ-NET-001 | 正常・High | canary apps名を `dig` | wildcardが `192.0.2.11` | dig出力 |
| TST-DNS-003 | REQ-NET-001 | 正常・High | APIと6 node IPを `dig -x` | 設計したPTRへ解決 | dig出力 |
| TST-DNS-004 | REQ-NET-001 | 障害・High | 非本番の隔離client/pathでprimary resolverを利用不能にし、second resolverで同じ正逆引きを確認 | 共有DNSを停止せず片系喪失を模擬し、許容時間内にsecond resolverで解決継続、復旧後に両系正常 | resolver query/timeline |
| TST-NTP-001 | REQ-AVL-002／DD-EXT-001 | 障害・High | 非本番で承認した方法により1 time sourceだけを試験nodeから利用不能にし、`chronyc sources -v`/`tracking`を確認 | 他sourceで同期継続し、offset/stratum/復旧時間がSecurity・Platform合意基準内。共有NTPは停止しない | chrony出力/timeline |
| TST-LB-001 | REQ-NET-002 | 正常/異常・High | HAProxy config確認、API `/readyz`、backend 1台停止 | L4/no persistence、異常backendを30秒以内に除外、API継続 | config、timing、LB log |
| TST-LB-002 | REQ-NET-002 | 正常/否定・High | node内部から22623、非許可clientから同portを確認 | 内部到達、外部はFW方針どおり拒否 | nc/curl、FW log |
| TST-LB-003 | REQ-NET-002 | 正常・High | HTTP/HTTPS RouteをIngress VIP経由で連続確認 | worker backendへ転送、TLS/redirect正常 | LB log、curl |
| TST-LB-004 | REQ-AVL-002 | 障害・High | 承認後、active LBのHAProxy/hostを1台停止 | VIPが片系へ移動しAPI/Ingressが許容時間内に復旧、二重VIPなし | packet/LB/VIP/timing |

## 4. Cluster network・Proxy

| 試験ID | 要件／設計 | 種別・優先度 | 操作・確認 | 期待結果 | 主証跡 |
| --- | --- | --- | --- | --- | --- |
| TST-NET-001 | REQ-NET-003／DD-NET-* | 正常・High | `oc get network.config/cluster -o yaml`、IPAM照合 | OVN-K、3 CIDR、hostPrefix一致、重複なし | config、IPAM承認 |
| TST-NET-002 | REQ-NET-003 | 正常・High | sample Pod/Service/EndpointSlice、Pod間HTTP | 2 Pod Ready、2 endpoint、Service通信成功 | oc/curl出力 |
| TST-NET-003 | REQ-SEC-001／DD-WL-006 | 否定・High | 未許可namespaceからsample Pod:8080へ接続 | timeout/拒否。許可したsame namespace/Ingressだけ成功 | client log、policy |
| TST-NET-004 | REQ-NET-003 | 正常・High | RouteのHTTP/HTTPSとadmission確認 | admitted、HTTPはHTTPS redirect、HTTPS 2xx | route、curl |
| TST-PRX-001 | REQ-NET-004 | 正常・High | `oc get proxy/cluster -o yaml` | http/https/noProxyとtrustedCAが承認値 | proxy YAML |
| TST-PRX-002 | REQ-NET-004 | 正常・High | image pull、release/Operator catalog到達 | 許可先へTLS検証ありで到達 | pod/event/proxy log |
| TST-PRX-003 | REQ-NET-004 | 否定・High | internal API/Service/CSI宛通信とProxy access log照合 | noProxy対象がProxyを経由せず内部到達 | flow/proxy log |
| TST-PRX-004 | REQ-NET-004／REQ-OPS-001 | 障害・High | 非本番の専用Proxy経路または隔離test nodeでProxy不可を模擬し、検知・image pull・update check・既存workload通信を観測後に復旧 | Proxy依存操作は設計どおり失敗/alert、既存の内部workload trafficは継続、復旧後にimage/update経路が再開。共有Proxyは停止しない | alert/event/proxy log/timeline |

## 5. 認証・認可・セキュリティ

| 試験ID | 要件／設計 | 種別・優先度 | 操作・確認 | 期待結果 | 主証跡 |
| --- | --- | --- | --- | --- | --- |
| TST-IAM-001 | REQ-IDM-001 | 正常・High | 有効な個人IDでOAuth login、MFA | 個人Userとしてloginでき、MFAはIdP方針どおり | マスク済み認証記録 |
| TST-IAM-002 | REQ-IDM-001 | 否定・High | 無効化済み試験ID、IdP一時不可 | 無効IDは拒否、障害時挙動が設計どおり、既存token方針を確認 | IdP/OAuth log |
| TST-IAM-003 | REQ-IAM-001 | 正常・High | `oc auth can-i` をoperator groupで実施 | namespace read権限だけが期待どおり許可 | can-i一覧 |
| TST-IAM-004 | REQ-IAM-001 | 否定・High | 同groupでSecret取得、cluster変更、他namespace変更を照会 | すべて `no` | can-i出力 |
| TST-SEC-001 | REQ-SEC-001 | 正常/否定・High | sample workloadをadmitし、privileged Podを別の承認済み否定fixtureで試す | sampleは標準SCC、privileged要求は拒否 | Pod/SCC event |
| TST-SEC-002 | REQ-SEC-001 | 静的・High | repo/成果物をSecret scannerと目視で確認 | token、private key、pull secret実値なし | scan report |
| TST-SEC-003 | REQ-SEC-001 | 正常・Medium | API/Ingress証明書のchain、SAN、期限を確認 | 承認CA、正しいSAN、期限alert閾値内 | openssl/alert |
| TST-SEC-004 | REQ-SEC-001 | 否定・High | workload SAのtoken mountと権限を確認 | token自動mountなし、不要API権限なし | Pod spec、can-i |
| TST-AUD-001 | REQ-AUD-001 | 正常・High | 個人IDで承認済み変更を行いaudit/変更IDと照合 | user、verb、resource、timestampを変更記録へ追跡可能 | audit event、変更票 |

## 6. Storage

`TBD-006`解消後、採用CSIのサポート条件・データ保護手順で実行します。

| 試験ID | 要件 | 種別・優先度 | 操作・確認 | 期待結果 | 主証跡 |
| --- | --- | --- | --- | --- | --- |
| TST-STG-001 | REQ-STG-001 | 正常・High | StorageClass指定PVCを作成 | 動的PVがBound、provisioner/暗号化が設計どおり | PVC/PV/CSI event |
| TST-STG-002 | REQ-STG-001 | 異常・High | RWO volume利用Podのnode移動 | detach/attach後にデータ整合、二重mountなし | event、checksum |
| TST-STG-003 | REQ-STG-001 | 正常・High | RWX volumeを複数nodeからmount | 対応方式で同時read/writeと整合性を確認 | Pod/node、checksum |
| TST-STG-004 | REQ-STG-001 | 復旧・High | VolumeSnapshot作成・別PVCへrestore | 対応版でreadyToUse、復元データ一致 | snapshot/PVC/checksum |
| TST-STG-005 | REQ-STG-001 | 正常・Medium | PVC online expansion | filesystemを含め承認容量へ拡張、データ保持 | PVC/df/checksum |
| TST-STG-006 | REQ-STG-001 | 性能・High | 承認済みfio profileを隔離環境で実行 | Applicationと合意したIOPS/latencyを満たす | profile/raw result |

## 7. 監視・ログ

| 試験ID | 要件 | 種別・優先度 | 操作・確認 | 期待結果 | 主証跡 |
| --- | --- | --- | --- | --- | --- |
| TST-MON-001 | REQ-MON-001 | 正常・High | Cluster Monitoring target/rule状態 | 必須target Up、rule errorなし | Prometheus/API出力 |
| TST-MON-002 | REQ-MON-001 | 異常・High | 非本番で承認済みsample alert発火 | severity、label、通知先、起票が設計どおり | alert/通知/issue |
| TST-MON-003 | REQ-MON-001 | 復旧・Medium | alert原因を解消 | resolve通知、issue更新、時刻追跡可能 | alert timeline |
| TST-MON-004 | REQ-MON-001 | 異常・High | 通知先1系統を停止 | 欠損検知または代替通知が設計どおり | notification log |
| TST-LOG-001 | REQ-LOG-001 | 正常・High | infrastructure logを検索 | node/pod/namespace/timeで検索可能 | query result |
| TST-LOG-002 | REQ-LOG-001 | 正常・High | application marker logを検索 | markerが欠損せず権限内で閲覧可能 | query result |
| TST-LOG-003 | REQ-LOG-001 | 正常・High | audit marker operationを検索 | 操作主体・verb・resourceを追跡可能 | query result |
| TST-LOG-004 | REQ-LOG-001 | 異常・High | 非本番でforward先を一時停止 | buffer/欠損alert/復旧後転送が設計どおり | collector/receiver log |

## 8. Backup・restore・可用性

復元試験は本クラスタを直接壊さず、隔離したrecovery環境と匿名化データで行います。

| 試験ID | 要件 | 種別・優先度 | 操作・確認 | 期待結果 | 主証跡 |
| --- | --- | --- | --- | --- | --- |
| TST-BKP-001 | REQ-BKP-001 | 正常・High | etcd backupを承認手順で取得・外部保管 | snapshot完了、暗号化、hash、保持情報あり | backup log/catalog |
| TST-BKP-002 | REQ-BKP-001 | 正常・High | OADPで対象namespace resourceをbackup | 対象/除外/完了状態が設計どおり | Backup CR/log |
| TST-BKP-003 | REQ-BKP-001 | 正常・High | PV backup/snapshot | アプリ整合点、snapshot、保管先を追跡可能 | snapshot/catalog |
| TST-BKP-004 | REQ-BKP-001 | 正常・High | 外部DB backup/PITR点確認 | RPO 1時間以内の復旧点と整合手順あり | DB catalog（マスク） |
| TST-RST-001 | REQ-DR-001 | 復旧・High | 隔離環境でetcd restore演習 | supported手順でAPI/resource状態を復旧 | timeline/log |
| TST-RST-002 | REQ-DR-001 | 復旧・High | namespace resource restore | Deployment/Service/Route/RBACが期待版 | diff/oc出力 |
| TST-RST-003 | REQ-DR-001 | 復旧・High | PV/DBを同じ整合点へ復元 | checksum/業務整合性合格、data loss <=1h | checksum/業務判定 |
| TST-RST-004 | REQ-DR-002 | 復旧・High | 障害宣言から業務再開承認まで計時 | RTO <=4h。全工程時刻と判断者あり | timeline/承認 |
| TST-HA-001 | REQ-AVL-002 | 障害・High | 事前に2 Podが別workerであることを確認し、worker 1台を承認済み手順でdrain/停止 | PDBはdrainを保護し、既存sample提供継続、復旧後に別workerへ再配置。突然のnode障害は別途実測しPDBだけで保証しない | alert/Pod配置/curl |
| TST-HA-002 | REQ-AVL-002 | 障害・High | control plane 1台だけ停止 | etcd quorum/API継続、復旧後member正常 | etcd/API/timeline |
| TST-HA-003 | REQ-AVL-002 | 障害・High | Ingress backend 1台停止 | Route提供継続、LBが異常backend除外 | LB/curl/timeline |
| TST-SLO-001 | REQ-AVL-001 | 運用・High | 合意した外形SLIを月次集計 | 99.9%判定、計画保守/除外を再現可能 | SLI report |

## 9. 容量・性能・運用・変更

| 試験ID | 要件 | 種別・優先度 | 操作・確認 | 期待結果 | 主証跡 |
| --- | --- | --- | --- | --- | --- |
| TST-CAP-001 | REQ-CAP-001 | 容量・High | node停止時のrequests/usage/退避余力を計算 | 1 worker障害後も承認閾値内 | capacity sheet |
| TST-CAP-002 | REQ-CAP-001 | 容量・Medium | PV消費傾向と増設lead timeを評価 | alert閾値前に増設可能 | trend/forecast |
| TST-PER-001 | REQ-PER-001 | 性能・High | 合意済み負荷modelでAPI/業務応答を測定 | 合意目標を満たしbottleneckを記録 | test config/raw data |
| TST-OPS-001 | REQ-OPS-001 | 運用・High | alertから一次切分け・escalation演習 | OLA内に起票、必要情報、担当引継ぎ | ticket/timeline |
| TST-OPS-002 | REQ-OPS-001 | 運用・Medium | Runbookだけで別担当が確認 | 前提、判断、停止、証跡が再現可能 | review record |
| TST-UPG-001 | REQ-MNT-001 | 更新・High | 非本番で承認z版へupdate演習 | precheck/backup/進捗/postcheck/戻せない境界を記録 | CV/history |
| TST-CHG-001 | BR-008 | 変更・High | sample変更を申請から結果まで追跡 | 申請、review、承認、実施、確認、切戻し判定が同一ID | change record |
| REV-RACI-001 | BR-006 | Review・High | [03-scope-responsibility.md](03-scope-responsibility.md)を各Ownerレビュー | 各成果物/境界にA/Rが合意される | review minutes |
| REV-QA-001 | BR-010 | Review・High | 期待値、実績、未実施、架空値を横断確認 | 未実施をPass扱いした記載が0件 | QA report |

## 10. Virtualization・MTV第2段階

| 試験ID | 要件 | 種別・優先度 | 操作・確認 | 期待結果 | 主証跡 |
| --- | --- | --- | --- | --- | --- |
| TST-VIR-001 | REQ-VIR-001 | 前提・High | CPU/Firmware/CSI/network/Operator互換性確認 | 全前提合格、未確定がPoC開始を阻害しない | compatibility matrix |
| TST-MTV-001 | REQ-MTV-001 | 正常・High | 代表VM 1台をtest migration | 変換・起動・network・disk・tool課題を記録 | Plan/VM/event |
| TST-MTV-002 | REQ-MTV-001 | 切替・High | 承認windowでwarm/cold cutover候補を計時 | 停止時間と手順がPoC基準内 | timeline |
| TST-MTV-003 | REQ-MTV-001 | 復旧・High | sourceを保持した切り戻し演習 | 二重稼働を防ぎ、source側で整合して再開 | checklist/業務判定 |

これらは本番VM移行の承認試験ではありません。詳細条件はVirtualization/MTV設計・移行計画を正とします。

### 10.1 Kong / Sysdig のbaseline境界

本版の中央試験baselineは、本書に採番済みの72 IDです。[Kong・Sysdig連携設計](12-kong-sysdig-integration-design.md)の `KONG-T01〜07` と `SYSDIG-T01〜07` は、製品選定後の試験設計に備えた local planned-check ID であり、この72 IDには含みません。

製品、版、license、topology / agent方式、data handling、support条件を確定し、[変更管理台帳](22-change-register.md)の承認を得た後に、要件トレーサビリティ、本書、試験結果記録を同一revisionで改訂します。それまではlocal checkを実行結果やcurrent acceptanceとして扱わず、状態はdesign-only / `Not Run`です。

## 11. 障害試験の安全手順

### 11.1 共通

1. 変更ID、対象、開始/終了、判断者、連絡先、復旧点を記録する。
2. 事前健全性と業務影響ゼロを確認する。
3. 一度に注入する障害は1つとし、control planeは同時に複数停止しない。
4. 操作開始、検知、切替、復旧、正常化の時刻を単一基準時計で記録する。
5. 中止条件に達したら追加操作を止め、復旧を優先する。
6. 事後健全性が事前状態へ戻るまで次ケースへ進まない。

DNS/NTP/Proxyの片系試験は共有サービスそのものを停止せず、非本番の専用経路、隔離client、Network teamが承認したfailure injectionを使います。方式を安全に隔離できなければ実行せずBlockedとして課題化します。

### 11.2 NetworkPolicy否定試験

- 専用test Pod/Namespaceを使い、timeout値を設定する。
- 成功すべき経路を先に確認し、同じdestinationへ未許可sourceから接続する。
- 接続失敗だけでなくpolicy、source/destination、port、CNI eventを保存する。
- 調査目的でdefault-denyを外す場合も別の承認変更とする。

### 11.3 復元・RTO試験

- backupの成否とrestoreの成否を別ケースにする。
- RTOの開始は障害宣言、終了はApplication責任者の業務再開承認とする。
- RPOはbackup時刻ではなく、復元データの最後の整合済みtransaction時点で判定する。
- etcd、Kubernetes resource、PV、外部DBの復旧順と整合点を記録する。

## 12. 証跡管理

- 保存先: 承認済みアクセス制御領域（未設定）。公開リポジトリへ実環境証跡を置かない。
- 命名: `<test-id>_<UTC timestamp>_<artifact>`。
- 必須metadata: cluster、version、config commit、変更ID、実施者、確認者、時刻、command。
- `kubeconfig`、token、cookie、Authorization、pull secret、private key、個人情報を除去する。
- raw dataを保持し、画像だけで判定しない。加工手順を記録する。
- 保持期間・廃棄方法はSecurity/品質担当のTBDです。

## 13. 完了条件

- HighケースがすべてPassし、Fail/Blocked/NOT RUNが0件である。
- Mediumの未実施は影響、期限、Owner、リスク受容者が承認されている。
- `REQ-DR-*`、`REQ-AVL-*`、`REQ-SEC-*` の代替確認は不可とし、実測証跡がある。
- 試験後のCV/CO/Node/Eventが受入状態へ戻っている。
- 結果、defect、change、設計差異をトレーサビリティへ反映している。

現時点では開始条件を満たさず、完了判定は行いません。
