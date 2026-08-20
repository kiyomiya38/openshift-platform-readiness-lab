# 02. 前提条件・制約・未確定事項

## 文書管理

| 項目 | 内容 |
| --- | --- |
| 文書ID | `ASM-CON-001` |
| 版／状態 | 0.1／Draft（机上設計） |
| 基準日 | 2026-08-17 |
| 作成 | 文書作成チーム（サンプル） |
| レビュー／承認 | PM・各領域責任者／基盤責任者（未実施） |

> [!IMPORTANT]
> 本書は一般公開用の架空プロジェクトで使用する前提・制約のサンプルであり、実環境調査の結果ではありません。状態はDraft・未レビュー・未承認で、実機構築と試験は未実施です。共通値は [SCENARIO.md](../SCENARIO.md) を正とします。

## 1. 管理方法

- 「前提」は設計を進めるため一時的に真と置く条件です。崩れた場合は影響分析します。
- 「制約」は設計選択を制限する条件です。
- 「TBD」は決定責任者と期限を持ち、ゲート通過前に解消またはリスク受容します。
- 状態は `Open`、`Confirmed`、`Rejected`、`Risk accepted` を使用します。

## 2. 前提条件

| ID | 前提 | 根拠 | 崩れた場合の影響 | Owner | 状態 |
| --- | --- | --- | --- | --- | --- |
| ASM-001 | 対象はOCP 4.22.zである | 共通シナリオ | API、Operator、手順、サポート条件を全面再確認 | Platform | Open（z版TBD） |
| ASM-002 | Agent-based Installer、`platform: none`を用いる | 共通シナリオ | DNS/LB/ISO作成/ホスト登録方式を再設計 | Platform | Confirmed（机上） |
| ASM-003 | x86_64ベアメタル6台を利用できる | 共通シナリオ | トポロジ、サイジング、障害ドメインを再設計 | Hardware | Open |
| ASM-004 | Control Plane 3台、Worker 3台を別障害ドメインへ可能な限り分散できる | 可用性設計 | 共通障害で99.9%目標を損なう | Hardware | Open |
| ASM-005 | 外部DNS/NTP/LBが冗長で導入前に利用できる | `platform: none`要件 | インストール不能または単一障害点 | Network | Open |
| ASM-006 | インターネット接続は組織Proxy経由で許可される | connected構成 | リリースイメージ、Operator、Telemetry等の到達性を再設計 | Network/Security | Open |
| ASM-007 | OpenShift対応CSIとオブジェクトストレージを提供できる | 永続化・バックアップ要件 | Registry、PV、ログ、OADP設計を再選定 | Storage | Open |
| ASM-008 | 組織IdPが利用でき、個人IDとグループを連携できる | BR-007 | 認証・権限・監査を代替設計 | Security | Open |
| ASM-009 | アプリケーションはコンテナ化またはVM PoC対象として分類できる | BR-001/009 | 収容計画と試験範囲が未確定 | Application | Open |
| ASM-010 | 外部DBにはRPO 1時間を満たすネイティブ保護機能がある | BR-005 | OADPだけではDB整合性を保証できず目標未達 | Application/DB | Open |

## 3. 制約

| ID | 制約 | 設計への影響 | 対応方針 | Owner |
| --- | --- | --- | --- | --- |
| CON-001 | IPはIPv4、静的割当 | DHCP依存の方式を採らず、MAC/IP/hostname対応を管理 | Agent設定とDNS/PTRを事前照合 | Network/Platform |
| CON-002 | Machine `192.0.2.0/24`、Pod `10.128.0.0/14`、Service `172.30.0.0/16` | 導入後変更が困難な範囲を含む | 全接続網と重複を導入前に確認 | Network |
| CON-003 | ノードOSはRHCOS | 任意パッケージ追加や手動OS管理をしない | MachineConfig等のサポート方式を使う | Platform |
| CON-004 | Connectedだが全通信はProxy/Firewall管理下 | 外部宛先、CA、noProxyの誤りが導入・更新を阻害 | 許可リストと疎通試験を変更前に実施 | Security/Network |
| CON-005 | 実検証環境がない | 構築・試験結果を取得できない | 静的検証と机上レビューに限定し未実施表示 | Project QA |
| CON-006 | 実在情報・Secretを保存しない | 設定例に実値を埋め込めない | RFC 5737、`example.com`、Secret参照IDを使用 | 全員 |
| CON-007 | Virtualization/MTVは第2段階PoC | 第1段階受入と本番VM移行を混同しない | 別ゲート・別受入条件とする | PM |
| CON-008 | ROSA/ARO、OpenShift AI、Disconnectedは構築対象外 | 設計範囲を拡張しない | 比較・将来課題としてのみ記録 | 基盤責任者 |

## 4. TBD管理表

| ID | 未確定事項 | 決定に必要な情報 | 影響 | Owner | 期限 | 状態 |
| --- | --- | --- | --- | --- | --- | --- |
| TBD-001 | 正確なOCP 4.22.z、更新チャネル、EUS/サポート期間 | リリースノート、ライフサイクル、組織更新標準 | 全構築・Operator互換性 | Platform | G3前 | Open |
| TBD-002 | サーバーHCL、Firmware、BIOS、CPU仮想化支援 | Red Hat Ecosystem Catalog、機器仕様、ベンダー推奨 | 導入可否、Virtualization | Hardware | G2前 | Open |
| TBD-003 | ラック、電源、ToRの障害ドメイン | 物理配置図 | 可用性目標 | Hardware/Network | G2前 | Open |
| TBD-004 | LB製品、HA方式、VIP引継ぎ、health check | 製品仕様と運用標準 | API/Ingress可用性 | Network | G2前 | Open |
| TBD-005 | Proxy CA、許可先、認証方式 | Proxy・PKI標準、Red Hat必須URL | 導入・更新・Operator | Security/Network | G3前 | Open |
| TBD-006 | CSI製品、StorageClass、RWO/RWX、Snapshot、暗号化 | 互換表、性能・容量試験 | Registry、アプリ、VM、バックアップ | Storage | G2前 | Open |
| TBD-007 | IdP方式、MFA、属性、グループ、障害時運用 | IAM標準、IdP仕様 | 認証・RBAC・監査 | Security | G2前 | Open |
| TBD-008 | API/Ingress証明書のCA、SAN、更新責任 | PKI標準、証明書申請手順 | 利用者接続・更新停止 | Security | G3前 | Open |
| TBD-009 | OADP版、バックアップ方式、オブジェクトストレージAPI | OCP/CSI互換性、RPO試験 | データ保護 | Platform/Storage | G3前 | Open |
| TBD-010 | Logging版、保存先、保持期間、外部SIEM | 互換表、監査要件、容量試算 | 障害調査・監査 | Security/Platform | G2前 | Open |
| TBD-011 | Sysdig/Kongの製品版、方式、ライセンス、サポート | ベンダー仕様、契約 | 周辺連携 | Product owner | 第2段階設計前 | Open |
| TBD-012 | VMware版、権限、ネットワーク、データストア、停止可能時間 | 移行元調査 | MTV PoC可否 | Application/Platform | PoC計画承認前 | Open |
| TBD-013 | 業務性能目標、ピーク負荷、データ量 | アプリ性能実績 | サイジング・RTO | Application | G2前 | Open |
| TBD-014 | 99.9%のSLI地点・除外条件 | サービスカタログ、業務合意 | 月次判定 | 業務/Operations | G2前 | Open |

## 5. 導入開始前の停止条件

次のいずれかが未解消なら、Agent ISOによるノード起動へ進みません。

- OCP z版、pull secret参照、SSH鍵管理方法が未承認。
- HCL、Firmware、必要CPU機能、ディスク、NICが未確認。
- API／API-int／apps wildcard／全ノードの正逆引きが不合格。
- LBの`6443`、`22623`、`80`、`443`のbackendとhealth checkが未確認。
- Machine／Pod／Serviceネットワークに重複がある。
- Proxy、Proxy CA、時刻同期、イメージ取得経路の疎通が不明。
- CSI、バックアップ、監視の最低限の運用責任者が決まっていない。
- 承認済みの構築・中断・切り戻し手順がない。

## 6. 公式根拠

- [Agent-based Installerのplatform none要件（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installing_an_on-premise_cluster_with_the_agent-based_installer/preparing-to-install-with-agent-based-installer)
- [OpenShift 4.22インストール方式と対応プラットフォーム（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installation_overview/ocp-installation-overview)
- [OpenShift 4.22 Backup and restore（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/backup_and_restore/)

## 7. 承認・変更履歴

| 役割 | 判定 | 日付 | コメント |
| --- | --- | --- | --- |
| PM | 未承認 | - | TBDの期限・ゲートを要確認 |
| 基盤責任者 | 未承認 | - | 架空案件 |

| 版 | 日付 | 内容 | 作成者 |
| --- | --- | --- | --- |
| 0.1 | 2026-08-17 | 初版 | 文書作成チーム（サンプル） |
