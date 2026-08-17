# 04. 要件トレーサビリティマトリクス

## 文書管理

| 項目 | 内容 |
| --- | --- |
| 文書ID | `RTM-001` |
| 版／状態 | 0.1／Draft（机上設計） |
| 基準日 | 2026-08-17 |
| 作成 | 生成 AI 支援ドラフト（本人レビュー前） |
| レビュー／承認 | PM・品質担当／基盤責任者（未実施） |

> [!IMPORTANT]
> 本表は架空案件の机上追跡です。設計・手順・試験の作成状態と、実機での合格を区別します。試験結果はすべて未実施であり、商用経験の証明ではありません。

試験IDの手順・期待結果は [16-test-specification.md](16-test-specification.md)、実施状態と証跡欄は [17-test-results.md](17-test-results.md) を参照します。

## 1. 状態定義

| 状態 | 意味 |
| --- | --- |
| Draft | 対応案はあるがレビュー・承認前 |
| TBD | 対応方式または受入値が未確定 |
| Planned | 試験仕様で確認予定 |
| Not run | 実機試験を実行していない |
| Passed/Failed | 承認された証跡に基づく結果（本版では使用しない） |

## 2. 業務要件の追跡

| 要件ID | 要約 | 詳細要件 | 設計対応 | 予定試験／確認 | Owner | 設計状態 | 実施状態 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| BR-001 | Web/APIとVMの段階収容 | REQ-PLT-001、REQ-VIR-001、REQ-MTV-001 | BD-001、BD-011、ARC-001、ARC-010 | TST-INS-001、TST-VIR-001 | Platform | Draft | Not run |
| BR-002 | 利用者100名、関係者20名 | REQ-IDM-001、REQ-IAM-001、REQ-CAP-001 | SEC-001〜004、ARC-006 | TST-IAM-001、TST-CAP-001 | Security/Platform | TBD | Not run |
| BR-003 | 24x365、計画保守承認 | REQ-OPS-001、REQ-MNT-001 | OPS-001、OPS-008、BD-010 | TST-OPS-001、TST-UPG-001 | Operations | Draft | Not run |
| BR-004 | 月間99.9% | REQ-AVL-001、REQ-AVL-002 | BD-004、ARC-007、OPS-004 | TST-HA-001〜003、TST-LB-004、TST-DNS-004、TST-NTP-001、TST-SLO-001 | 業務/Platform | TBD | Not run |
| BR-005 | RPO 1h、RTO 4h | REQ-DR-001、REQ-DR-002、REQ-BKP-001 | BKP-001〜007、BD-008 | TST-BKP-001、TST-RST-001 | Application/Platform/Storage | Draft | Not run |
| BR-006 | 責任分離 | 全領域要件 | [責任分界](03-scope-responsibility.md)、BD-012 | REV-RACI-001 | PM | Draft | Not run |
| BR-007 | 最小権限・IdP・個人ID | REQ-IDM-001、REQ-IAM-001、REQ-SEC-001、REQ-AUD-001 | SEC-001〜010 | TST-IAM-001〜004、TST-AUD-001 | Security | Draft/TBD | Not run |
| BR-008 | 変更記録 | REQ-MNT-001、REQ-AUD-001 | OPS-007〜010 | TST-CHG-001 | PM/Platform | Draft | Not run |
| BR-009 | 代表VM 3台のMTV PoC | REQ-VIR-001、REQ-MTV-001 | BD-011、ARC-010 | TST-VIR-001、TST-MTV-001〜003 | Platform/Application | TBD | Not run |
| BR-010 | 未実施を明示 | 全要件 | 全文書の文書管理・結果欄 | REV-QA-001 | 品質担当 | Draft | Not run |

## 3. プラットフォーム・ネットワーク要件

| 要件ID | 設計ID／文書 | 実装成果物（予定） | 試験ID（予定） | 受入の焦点 | Owner | 状態 |
| --- | --- | --- | --- | --- | --- | --- |
| REQ-PLT-001 | BD-001/002、ARC-001/002 | install-config、agent-config、構築手順 | TST-INS-001〜004 | 4.22.z固定、install-complete、ClusterOperator正常 | Platform | z版TBD |
| REQ-PLT-002 | ARC-003/004 | host inventory、agent-config | TST-NOD-001 | 3 control plane + 3 worker、役割・IP一致 | Platform/Hardware | Draft |
| REQ-PLT-003 | BD-003 | Agent ISO、RHCOS image | TST-NOD-002 | 全ノードOSと更新管理方式 | Platform | Draft |
| REQ-NET-001 | NET-001〜004 | DNS申請、前提チェック | TST-DNS-001〜004 | API/API-int/apps/nodeの正逆引きと片系障害 | Network | Not run |
| REQ-NET-002 | NET-005〜008 | LB設定依頼、Firewall申請 | TST-LB-001〜004 | L4、port、readyz、failover | Network | 製品TBD |
| REQ-NET-003 | NET-009〜012 | install-config、NetworkPolicy | TST-NET-001〜004 | CIDR非重複、OVN-K、Pod通信 | Network/Platform | Not run |
| REQ-NET-004 | NET-013〜015 | Proxy/CA設定 | TST-PRX-001〜004 | proxy/noProxy、registry/Operator到達、障害復旧 | Network/Security | CA/TBD |

## 4. セキュリティ・データ・運用・第2段階要件

| 要件ID | 設計ID／文書 | 実装成果物（予定） | 試験ID（予定） | 受入の焦点 | Owner | 状態 |
| --- | --- | --- | --- | --- | --- | --- |
| REQ-IDM-001 | SEC-001/002 | OAuth設定（Secret実値除外） | TST-IAM-001/002 | 正常/無効ユーザー、MFA、IdP障害 | Security | 方式TBD |
| REQ-IAM-001 | SEC-003〜005 | Group、RoleBinding | TST-IAM-003/004 | 最小権限、否定試験、棚卸し | Security/Platform | Draft |
| REQ-STG-001 | STG-001〜006 | CSI/StorageClass定義、Registry永続化 | TST-REG-001、TST-STG-001〜006 | Registry、provision、RWO/RWX、Snapshot、拡張、性能 | Storage | 製品TBD |
| REQ-BKP-001 | BKP-001〜008 | etcd/OADP/DB runbook | TST-BKP-001〜004、TST-RST-001〜004 | 分離保護、保管、復元、整合性 | 複数 | 方式TBD |
| REQ-MON-001 | MON-001〜006 | CMO設定、通知設定、Runbook | TST-MON-001〜004 | 基盤/業務メトリクス、通知、欠損 | Platform/Operations | Draft |
| REQ-LOG-001 | LOG-001〜006 | Logging/forwarder設定 | TST-LOG-001〜004 | 3ログ種別、保持、権限、転送停止 | Platform/Security | 版・保存先TBD |
| REQ-SEC-001 | SEC-005〜010 | SCC/RBAC/証明書/Secret手順 | TST-SEC-001〜004 | 標準SCC、平文なし、期限、監査 | Security | Draft |
| REQ-AUD-001 | SEC-009、OPS-009 | audit転送、変更記録 | TST-AUD-001 | 個人ID・変更ID・API操作の照合 | Security/PM | TBD |
| REQ-VIR-001 | DES-VIRT-001、ADR-004/005 | OLM Subscription/CSV、HyperConverged CR、compatibility記録 | TST-VIR-001 | CPU/Firmware/CSI/network/Operator前提 | Platform/Hardware/Storage | TBD・Not run |
| REQ-MTV-001 | DES-VIRT-001、MIG-PLAN-001、RB-PLAN-001、ADR-006、ADR-013 | provider、mapping、wave plan、rollback evidence | TST-MTV-001〜003 | 3 VMの変換、cutover計時、source復帰 | Migration/Platform/Application | TBD・Not run |
| REQ-INT-001 | DES-INT-001、ADR-008 | Kong integration pointのみ。実装なし | **現行72 IDのbaseline外**。`KONG-T01〜07`はlocal planned check | 製品選定Gate後に中央試験へ採番・承認 | Product owner/Platform/Security | Deferred・Not run |
| REQ-INT-002 | DES-INT-001、ADR-009 | Sysdig integration pointのみ。実装なし | **現行72 IDのbaseline外**。`SYSDIG-T01〜07`はlocal planned check | 製品選定Gate後に中央試験へ採番・承認 | Product owner/Platform/Security | Deferred・Not run |

Kong / Sysdig の local planned-check ID は [連携設計](12-kong-sysdig-integration-design.md) にだけ定義します。製品選定、互換性、license、data handling、変更承認が完了するまでは [試験仕様書](16-test-specification.md) と [試験結果記録](17-test-results.md) の中央baselineへ追加せず、未実施・設計のみとして扱います。

## 5. 非機能要件の追跡

| 要件ID | 設計対応 | 試験・計測 | 未確定／残余リスク | Owner |
| --- | --- | --- | --- | --- |
| REQ-AVL-001 | BD-004、OPS-004 | TST-SLO-001 | SLI地点と除外条件 `TBD-014` | 業務/Operations |
| REQ-AVL-002 | ARC-003/007、NET-006 | TST-HA-001〜003、TST-LB-004、TST-DNS-004、TST-NTP-001 | 物理障害ドメイン `TBD-003` | Platform/Hardware |
| REQ-DR-001 | BKP-003〜006 | TST-RST-001〜003 | CSI/DB方式 `TBD-006/009` | Application/Storage |
| REQ-DR-002 | BKP-007 | TST-RST-004 | 実データ量・復旧速度未測定 | 業務/Platform |
| REQ-CAP-001 | ARC-006/008、STG-004、OPS-006 | TST-CAP-001/002 | 負荷・増設LT未確認 | Platform/Storage |
| REQ-PER-001 | ARC-006、STG-005 | TST-PER-001 | 性能目標 `TBD-013` | Application |
| REQ-OPS-001 | OPS-001〜010 | TST-OPS-001/002 | 実運用組織・OLA未確定 | Operations |
| REQ-MNT-001 | BD-009、OPS-007/008 | TST-UPG-001、TST-CHG-001 | 更新チャネル `TBD-001` | Platform |

## 6. ギャップ管理

| Gap ID | 内容 | 関連要件 | 対応 | Owner | 期限 | 状態 |
| --- | --- | --- | --- | --- | --- | --- |
| GAP-001 | 正確な4.22.zとOperator互換性未固定 | REQ-PLT-001ほか | 互換表とリリースノートをG3で凍結 | Platform | G3前 | Open |
| GAP-002 | 99.9%測定定義なし | REQ-AVL-001 | SLI/除外条件を業務合意 | 業務 | G2前 | Open |
| GAP-003 | CSI・バックアップ実現性未検証 | REQ-STG-001、REQ-DR-* | ベンダー確認と復元PoC | Storage | G4前 | Open |
| GAP-004 | 実環境がなく全試験未実施 | 全要件 | 環境確保後に承認済み試験を実行 | PM | G4前 | Open |

## 7. 変更統制

要件変更時は、該当行の要件、設計、実装、試験、運用Runbookを同じ変更IDに紐付けます。追跡先がない要件はG2を通過できません。試験が未実施の場合は `Not run` のまま保持し、期待結果を実績へ転記しません。

## 8. 承認・変更履歴

| 役割 | 判定 | 日付 | コメント |
| --- | --- | --- | --- |
| PM | 未承認 | - | 後続文書との照合前 |
| 品質担当 | 未レビュー | - | 実試験なし |

| 版 | 日付 | 内容 | 作成者 |
| --- | --- | --- | --- |
| 0.1 | 2026-08-17 | 初版 | 生成 AI 支援ドラフト |
