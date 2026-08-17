# OpenShift Virtualization PoC 基本設計ワークシート

## 1. PoC の問い

- PoC で証明すること: `<技術適合性・運用適合性>`
- PoC では証明しないこと: `<本番 SLA、全 VM 移行等>`
- 合格後の次段階: `<追加 PoC／基本設計／移行計画>`

## 2. 対象 VM 選定

| VM ID | 選定理由 | OS | CPU/RAM | Disk | Network/VLAN | 依存先 | 停止可能時間 |
|---|---|---|---|---|---|---|---|
| VM-01 | `<代表性>` | `<要確認>` | `<値>` | `<値>` | `<要確認>` | `<DB等>` | `<要確認>` |

### 除外条件

- `<未対応 guest OS、物理 device、特殊 driver、license、停止不可等>`

## 3. OpenShift Virtualization 設計

| 項目 | 設計案 | 根拠／確認 |
|---|---|---|
| Operator／OpenShift 版 | `<要確認>` | `<support matrix>` |
| Virtualization worker | `<台数・CPU virtualization>` | `<BIOS/firmware、failure domain>` |
| VirtualMachine / InstanceType | `<値>` | `<source VM と capacity>` |
| DataVolume / StorageClass | `<値>` | `<performance、snapshot、migration>` |
| Default Pod Network | `<用途>` | `<masquerade 等>` |
| Multus / VLAN | `<NAD と IPAM>` | `<既存 VLAN、IP、MTU>` |
| Live Migration | `<要否>` | `<shared storage、network、capacity>` |
| Backup / Monitoring | `<方式>` | `<restore/alert test>` |

## 4. MTV 移行設計

| 項目 | 設計案 | 要確認先 |
|---|---|---|
| Source provider | `<vCenter 版等>` | `<support matrix>` |
| Migration type | `<cold/warm>` | `<停止/RPO>` |
| Mapping | `<network/storage mapping>` | `<NW/Storage owner>` |
| Credential | `<Secret 参照・rotation>` | `<Security owner>` |
| Conversion | `<driver/agent/OS>` | `<VM owner>` |
| Rollback | `<source keep/restart>` | `<data consistency>` |

## 5. 試験と合格条件

| ID | Test | 合格条件 | 証跡／測定 |
|---|---|---|---|
| POC-01 | Import / Boot | `<OS/driver 正常>` | `<記入>` |
| POC-02 | Network | `<VLAN/IP/DNS/LB 疎通>` | `<記入>` |
| POC-03 | Storage | `<IO/expand/snapshot>` | `<記入>` |
| POC-04 | Live Migration | `<時間・停止影響>` | `<記入>` |
| POC-05 | Backup/Restore | `<RPO/RTO>` | `<記入>` |
| POC-06 | Rollback | `<source VM 復帰と整合>` | `<記入>` |

## 6. リスク・残課題

| Risk/Question | 影響 | 確認方法 | Owner | 期限 |
|---|---|---|---|---|
| `<記入>` | `<記入>` | `<PoC/公式確認>` | `<役割>` | `<日付>` |
