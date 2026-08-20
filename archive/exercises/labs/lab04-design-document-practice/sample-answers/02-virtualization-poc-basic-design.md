# サンプル回答: OpenShift Virtualization PoC

> 架空の教材例です。導入・移行実績の主張ではありません。対応 guest/source/target は対象版の support matrix で要確認です。

## PoC の目的

- 三つの代表 VM について、MTV での変換、起動、network、storage、backup/restore、operation の課題を抽出する。
- 本番性能や全 VM の移行可否はこの PoC だけで保証しない。

## 対象選定案

| VM | 代表性 | 選定理由 | 未決事項 |
|---|---|---|---|
| VM-A | Linux Web | 小容量、依存が少なく移行経路確認に適する | OS/VMware/MTV 対応版 |
| VM-B | Windows middleware | driver、license、固定 IP の観点を含む | License/agent/再認証 |
| VM-C | Stateful Linux | 大容量 disk と backup/整合性を確認できる | 停止時間/RPO |

## 移行先設計案

- Virtualization worker: CPU virtualization extension、node 数、failure domain、予備 capacity を要確認。
- VM: InstanceType/Preference 利用可否を確認し、CPU topology と memory は source 実使用量を測って決める。
- Storage: DataVolume/PVC の StorageClass、VolumeMode、snapshot、performance、Live Migration 条件を検証する。
- Network: 管理用 default network と既存 VLAN 用 Multus/NAD を分離。IPAM、MTU、固定 IP、DNS/LB 切替 owner は要確認。
- Security: Migration provider credential は Secret 管理、最小 RBAC、監査対象とし実値を文書へ書かない。
- Operation: VM state、guest agent、node、storage、backup job を監視し Runbook を作る。

## MTV 計画案

1. Source provider、inventory、mapping を作る前に互換性と credential 取扱いを承認する。
2. VM-A で cold migration をリハーサルし、変換ログと所要時間を記録する。
3. VM-B/C は依存先、data consistency、停止時間に応じ cold/warm の適否を要確認。
4. 移行元は書込みを再開しない状態で一定期間保持し、Go 判定後に廃止計画へ進む。

## 合格条件

| Test | 条件 |
|---|---|
| Boot | OS 起動、guest agent/driver、時刻同期が正常 |
| Network | 必要 segment、DNS、依存先だけ疎通し禁止通信は拒否 |
| Storage | 容量、IO、snapshot、restore が設計値内 |
| Migration | 計測した停止時間が対象 VM の許容内 |
| Live Migration | 対象 VM/Storage で対応する場合のみ、業務影響が基準内 |
| Rollback | 判断期限内に source 側へ戻し、split-brain と data 差分を管理 |

## 残課題

- 対応 Matrix、source VMware 版、guest OS は要確認。
- VLAN/IP、CSI、backup 製品、license owner を確定する。
- 本番対象全 VM を棚卸しし、PoC の三台からの外挿が妥当か評価する。
