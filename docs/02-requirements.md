# 02. 要求整理

## 目的

利用者や運用組織が必要とする状態を、設計・試験で追跡できる要求へ変換します。製品機能を先に並べるのではなく、背景、対象、測定条件、優先度、責任、未確定事項を明確にします。

## 実務での使用場面

- 関係者の要望を要件ID付きで合意する
- 機能要件と非機能要件を分け、測定可能にする
- 前提、制約、依存先、責任分界を明らかにする
- 設計・試験で要求の抜けや過剰実装を防ぐ
- 変更要求の影響範囲を追跡する

## 入力

- プロジェクト目的と成功条件
- 利用部門、運用部門、セキュリティ部門の要求
- 既存ネットワーク、DNS、NTP、認証、監視、バックアップの仕様
- ワークロード特性、容量、可用性、RPO/RTO、保守時間帯
- 契約範囲、予算、納期、組織標準、法令・監査要件
- OpenShift 4.22および周辺製品のサポート条件

## 判断

要求は「実装方法」ではなく「満たすべき状態」を書きます。たとえば「HAProxyを2台置く」は方式候補です。「単一障害でAPIとアプリ公開を長時間停止させない」が要求であり、その要求を満たす方式は基本設計で判断します。

要求には次の情報が必要です。

| 項目 | 読み取る内容 |
| --- | --- |
| ID | 文書間で一意に追跡する識別子 |
| 背景・目的 | なぜ必要か、満たさない場合の影響 |
| 要求文 | 誰・何が、どの状態であるべきか |
| 判定尺度 | 値、条件、観測点、許容範囲 |
| 優先度 | Must／Should／Couldなど |
| 所有者 | 内容を決定・承認できる主体 |
| 状態 | Draft、Approved、TBD、Deferredなど |
| 検証方法 | 試験、レビュー、監視、証跡の候補 |

RPOは許容できるデータ損失時間、RTOはサービスを復旧させるまでの目標時間です。数値だけでなく、対象データ、障害シナリオ、計測開始・終了点、除外条件を定義しなければ設計や試験へ展開できません。

## 成果物例の読み方

### 要件定義書

[要件定義書](../projects/enterprise-openshift-platform/docs/01-requirements.md)では、業務要件、機能要件、非機能要件、データ保護対象を分けています。各要求について以下を確認します。

- 要求文が具体的で、実装方式そのものになっていないか
- 可用性、性能、セキュリティ、運用の判定条件があるか
- Phase 1とPhase 2、対象と対象外が混ざっていないか
- バックアップ対象が「クラスタ全体」のような曖昧な表現でないか

### 前提・制約・TBD

[前提条件・制約・未確定事項](../projects/enterprise-openshift-platform/docs/02-assumptions-constraints.md)では、仮定、変更できない条件、意思決定待ちを分離しています。特にTBD表の所有者、期限、解消条件、未解消時の影響、導入停止条件を読みます。

### スコープと責任分界

[スコープ・責任分界書](../projects/enterprise-openshift-platform/docs/03-scope-responsibility.md)では、基盤チームだけで完結しないDNS、Firewall、Storage、IdP、バックアップなどの境界を確認します。対象外であっても、クラスタが依存するサービスなら、入力・受入条件・障害時の連絡先は必要です。

### トレーサビリティ

[要件トレーサビリティマトリクス](../projects/enterprise-openshift-platform/docs/04-requirements-traceability.md)で、要求IDが設計項目と試験IDへ接続されているか確認します。状態が`Open`または`Partial`なら、未完了理由が課題・リスク管理台帳へ接続されている必要があります。

## 他文書とのつながり

| 要求領域 | 主な基本設計 | 主な後工程 |
| --- | --- | --- |
| 可用性・容量 | [基本設計](../projects/enterprise-openshift-platform/docs/05-basic-design.md)、[アーキテクチャ](../projects/enterprise-openshift-platform/docs/06-architecture-design.md) | ノード設計、障害・性能試験 |
| DNS・通信 | [ネットワーク設計](../projects/enterprise-openshift-platform/docs/07-network-dns-lb-design.md) | DNS/LB設定、疎通試験 |
| 認証・監査 | [セキュリティ設計](../projects/enterprise-openshift-platform/docs/08-security-identity-design.md) | IdP/RBAC設定、権限試験 |
| データ保護 | [ストレージ・バックアップ設計](../projects/enterprise-openshift-platform/docs/09-storage-backup-design.md) | CSI、バックアップ、復元試験 |
| 監視・保守 | [監視・運用設計](../projects/enterprise-openshift-platform/docs/10-observability-operations-design.md) | アラート試験、運用引き継ぎ |
| VM移行 | [Virtualization・MTV設計](../projects/enterprise-openshift-platform/docs/11-virtualization-mtv-design.md) | 移行・切り戻し計画 |

## レビューで指摘されやすい点

- 「高可用」「十分な性能」「セキュア」など判定不能な形容だけで終わる
- RPO/RTOの対象、起点、終点、障害条件がない
- 可用性要求と保守時停止の許容条件が矛盾する
- 要求に所有者や承認者がなく、基盤チームだけで決めている
- 対象外サービスへの依存関係と引き渡し条件がない
- TBDを空欄のまま残し、未解消時の停止判断がない
- 要件IDが設計書、試験仕様書、変更記録へ引き継がれない

## 公式一次資料

- [OpenShift 4.22 Architecture](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/architecture/)
- [OpenShift 4.22 Security and compliance](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/security_and_compliance/)
- [OpenShift 4.22 Backup and restore](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/backup_and_restore/)

製品の前提知識は[OpenShiftコア知識](../reference/technical/openshift-core-knowledge.md)、要求から設計へ展開する考え方は[基本設計](../reference/technical/openshift-basic-design.md)を参照してください。

## 次に読む章

[03. 基本設計](03-basic-design.md)へ進みます。
