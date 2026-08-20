# OpenShift／OpenShift Virtualization 移行計画書

> [!NOTE]
> 実務成果物例を組織の正式様式へ転記する際に使用する空テンプレートです。`<...>` は案件固有値へ置換し、承認済み標準と対象版の公式資料を優先してください。

> VM・アプリ移行の共通ひな型。MTV の対応ソース／対象バージョン、ネットワーク、停止方式は案件環境で `要確認`。

## 1. 目的・範囲

| 項目 | 内容 |
|---|---|
| 移行元 | `<VMware／Kubernetes／その他、版>` |
| 移行先 | `<OpenShift cluster/project、版>` |
| 対象 | `<VM／namespace／データ>` |
| 対象外 | `<記入>` |
| 方式 | `cold／warm／段階移行（要確認）` |
| 希望停止時間 | `<分。要確認>` |

## 2. 成功条件・責任分界

- 成功条件: `<業務疎通、データ整合、性能、監視、バックアップ>`
- RPO／RTO: `<要確認>`

| 領域 | 移行元担当 | OpenShift 担当 | アプリ担当 | 確認者 |
|---|---|---|---|---|
| VM／OS | `<役割>` | `<役割>` | `<役割>` | `<役割>` |
| Network／DNS／LB | `<役割>` | `<役割>` | `<役割>` | `<役割>` |
| Data／Backup | `<役割>` | `<役割>` | `<役割>` | `<役割>` |

## 3. 対象棚卸し

| ID | VM／アプリ | OS／版 | vCPU/RAM | Disk／使用量 | VLAN/IP/DNS | 依存先 | 停止可否 |
|---|---|---|---|---|---|---|---|
| MIG-001 | `<名称>` | `<記入>` | `<値>` | `<値>` | `<マスク済み>` | `<DB等>` | `要確認` |

- Guest OS と MTV の対応状況: `対象バージョンの公式文書で要確認`
- 物理デバイス、特殊ドライバ、ライセンス、時刻同期、固定 IP: `<調査結果>`
- コンテナ化しない理由／将来方針: `<記入>`

## 4. 移行先設計

| 項目 | 設計値 | 確認 |
|---|---|---|
| Namespace／ResourceQuota | `<記入>` | `要確認` |
| InstanceType／CPU model | `<記入>` | `要確認` |
| StorageClass／AccessMode | `<記入>` | Live Migration 要件と照合 |
| NetworkAttachmentDefinition | `<名称／VLAN>` | Multus・既存 VLAN 接続を確認 |
| IP／DNS／LB | `<切替方式・TTL>` | `要確認` |
| Backup／Monitoring | `<製品・対象>` | 復元試験を実施 |

## 5. PoC・リハーサル

| ケース | 目的 | 対象 | 合格条件 | 結果／課題 |
|---|---|---|---|---|
| POC-001 | 変換・起動 | `<低リスク VM>` | OS 起動、driver 正常 | `<記入>` |
| POC-002 | データ／性能 | `<対象>` | `<閾値>` | `<記入>` |
| POC-003 | 切戻し | `<対象>` | `<時間内に復旧>` | `<記入>` |

## 6. 本番移行タイムライン

| 相対時刻 | 操作 | 担当 | 完了条件 | Go/No-Go |
|---|---|---|---|---|
| T-7d | バックアップ／復元性確認 | `<役割>` | 完了記録あり | - |
| T-0 | 変更凍結・停止 | `<役割>` | 書込み停止 | Go 1 |
| T+ | `<MTV plan 実行／データ同期>` | `<役割>` | `<条件>` | Go 2 |
| T+ | DNS／LB 切替 | `<役割>` | TTL・疎通正常 | Go 3 |
| T+ | 業務試験・監視 | `<役割>` | 成功条件達成 | 完了 |

## 7. リスク・切戻し

| リスク | 検知 | 予防 | 発生時対応 | Owner |
|---|---|---|---|---|
| データ差分 | `<整合性試験>` | `<凍結／同期>` | `<切戻し>` | `<役割>` |

- 切戻し判断期限／判断者: `<記入>`
- 移行元の保持期限・再開条件: `<記入>`
- 詳細: `<rollback-plan へのリンク>`

## 8. 参照

- [Migration Toolkit for Virtualization documentation](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/)
- [OpenShift Virtualization documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/)
