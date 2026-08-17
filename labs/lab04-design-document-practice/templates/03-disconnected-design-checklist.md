# Disconnected OpenShift 設計チェックワークシート

## 1. 接続境界

| Zone | Internet 接続 | 許可 Protocol | データ持込み／持出し | Owner |
|---|---|---|---|---|
| Connected | `<有/制限>` | `<要確認>` | `<審査方式>` | `<役割>` |
| Transfer | `<有無>` | `<要確認>` | `<媒体/中継>` | `<役割>` |
| Disconnected | `なし` | `<内部のみ>` | `<承認済み経路>` | `<役割>` |

## 2. 同期対象

| 対象 | Source | Destination | Version/Channel | 頻度 | 容量見積 | 検証 |
|---|---|---|---|---|---|---|
| OpenShift release | `<registry>` | `<mirror>` | `<要確認>` | `<記入>` | `<記入>` | `<digest/signature>` |
| Operator catalog | `<catalog>` | `<mirror>` | `<選定 package/channel>` | `<記入>` | `<記入>` | `<CatalogSource>` |
| Application Image | `<source>` | `<mirror>` | `<digest>` | `<記入>` | `<記入>` | `<scan/sign>` |

## 3. Mirror 基盤

- Registry topology/HA: `<要確認>`
- Storage capacity/growth/backup: `<記入>`
- TLS certificate/trust distribution/renewal: `<記入>`
- Authentication/RBAC/audit: `<記入>`
- `oc-mirror` workspace/metadata backup: `<記入>`
- ImageSetConfiguration owner/review: `<記入>`
- pull secret: `<安全な参照 ID。値を書かない>`

## 4. Cluster 依存

| 依存 | 設計 | 冗長化 | Firewall | 試験 |
|---|---|---|---|---|
| DNS | `<api/*.apps/mirror>` | `<記入>` | `<記入>` | `<正逆引き>` |
| NTP | `<内部 source>` | `<記入>` | `<記入>` | `<offset>` |
| Load Balancer | `<API/Ingress>` | `<記入>` | `<記入>` | `<health>` |
| IdP | `<内部 IdP>` | `<記入>` | `<記入>` | `<login>` |
| Registry | `<FQDN>` | `<記入>` | `<記入>` | `<pull>` |

## 5. Cluster mirror 設定

- Release Image source mapping: `<対象版の resource/設定を要確認>`
- Operator CatalogSource: `<name/namespace/polling>`
- Application Image mapping: `<対象版の resource/設定を要確認>`
- Samples Operator: `<管理状態と mirror 方針>`
- Proxy/noProxy: `<Disconnected 内の経路>`

## 6. 更新・障害・戻し

| Scenario | 検知 | 対応 | Rollback/Recovery | Owner |
|---|---|---|---|---|
| Image 不足 | `<event/log>` | `<追加同期>` | `<前 metadata>` | `<役割>` |
| Catalog 異常 | `<CSV/packagemanifest>` | `<catalog 修正>` | `<前 catalog>` | `<役割>` |
| Registry 障害 | `<alert>` | `<HA/restore>` | `<RTO>` | `<役割>` |
| Update 失敗 | `<CVO/CO>` | `<support/runbook>` | `downgrade 可否は要確認` | `<役割>` |

## 7. 受入チェック

- [ ] Cluster node から外部 Internet へ直接出ないことを確認した。
- [ ] Release、Operator、Application Image を内部 Registry から取得できた。
- [ ] 証明書、DNS、NTP、LB が冗長性と監視を含めて動作した。
- [ ] Mirror metadata と Registry data の backup/restore を試験した。
- [ ] 定期同期、脆弱性対応、容量監視、承認フローを運用へ引き継いだ。
