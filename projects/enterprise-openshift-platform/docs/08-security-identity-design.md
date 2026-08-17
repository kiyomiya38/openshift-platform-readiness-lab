# 08. セキュリティ・認証認可設計書

## 文書管理

| 項目 | 内容 |
| --- | --- |
| 文書ID | `SEC-DES-001` |
| 上位要件 | `BR-007`、`REQ-IDM-001`、`REQ-IAM-001`、`REQ-SEC-001`、`REQ-AUD-001` |
| 版／状態 | 0.1／Draft（机上設計） |
| 基準日 | 2026-08-17 |
| 作成 | 生成 AI 支援ドラフト（本人レビュー前） |
| レビュー／承認 | Security/Platform／セキュリティ責任者（未実施） |

> [!IMPORTANT]
> 本書は架空の学習用設計です。IdP連携、権限付与、証明書発行、Secret登録、監査確認、脆弱性検査は未実施であり、商用経験の証明ではありません。Secret実値や実在ユーザーは記載しません。

## 1. セキュリティ原則

- 強い認証、個人ID、最小権限、職務分離、期限付き例外を基本とします。
- 人に権限を直接付けず、原則として組織グループへ付けます。
- 通常運用で`cluster-admin`を使用しません。
- 標準SCCを優先し、default SCCを編集しません。
- TLS検証を無効化せず、組織CA/Proxy CAの信頼を明示的に管理します。
- Secret実値をGit、Markdown、チケット、端末履歴へ残しません。
- 予防、検知、対応、復旧の各統制を、変更ID・個人ID・時刻で追跡します。

## 2. 設計判断

| ID | 判断 | 根拠 | Owner | 状態 |
| --- | --- | --- | --- | --- |
| SEC-001 | OpenShift内蔵OAuthと組織IdPを連携する | BR-007と一元的なID管理 | Security | IdP種類TBD |
| SEC-002 | IdP側MFAを必須候補とし、無効化・退職をIdPライフサイクルに連動 | 認証強度と迅速な失効 | Security | IdP能力TBD |
| SEC-003 | RBACはグループを単位とし、個人直接bindingを例外化 | 棚卸し・職務変更を容易にする | Security/Platform | Draft |
| SEC-004 | cluster管理、platform運用、監査、Project管理/編集/閲覧を分離 | 最小権限・職務分離 | Security | custom role詳細TBD |
| SEC-005 | 標準SCCを優先し、default SCCは編集しない | upgrade安全性と権限拡大防止 | Platform/Security | Draft |
| SEC-006 | Secretは外部Secret保管の参照IDで管理し、Gitへ平文保存しない | 漏えい防止 | Security | 製品TBD |
| SEC-007 | API/Ingress/Proxy CAを用途別に管理し、期限を監視 | 信頼境界・更新事故防止 | Security/Platform | CA/TBD |
| SEC-008 | imageは承認registry、digest/署名・脆弱性基準を検討 | supply chain risk低減 | Security/Application | policy/TBD |
| SEC-009 | API監査ログを外部保管候補へ転送し、変更記録と照合 | BR-007/008、追跡可能性 | Security | 保存先/保持TBD |
| SEC-010 | break-glassは二者承認・期限付き・使用後レビュー | IdP障害時の復旧と濫用防止 | Security | 実方式TBD |

## 3. 認証設計

### 3.1 通常認証フロー

1. 利用者はOpenShift OAuth endpointへ接続します。
2. OAuthが組織IdPへ認証を委譲します。
3. IdPで個人ID、MFA、アカウント状態を検証します。
4. OpenShift user/identity mappingを作成または参照します。
5. 同期済みGroupとRBAC bindingによりAPI認可を判断します。
6. API操作は監査ログへ個人IDとともに記録します。

### 3.2 IdPインターフェース

| 項目 | 設計方針 | Owner | 状態 |
| --- | --- | --- | --- |
| 種別 | OIDCまたはLDAP等、OCP 4.22でサポートされ組織標準に合う方式 | Security | TBD-007 |
| Endpoint/CA | TLS endpointと組織CA。実URLはSecret/環境別台帳 | Security | TBD |
| User key | 変更されない一意属性を使用 | Security | TBD |
| Display/preferred username | `/`、`:`、`%`等の制約を含め事前検証 | Security | TBD |
| Group連携 | IdPまたはLDAP group sync方式を決定 | Security/Platform | TBD |
| MFA | IdP側で強制し、OpenShift側との責任を明記 | Security | TBD |
| 失効 | 退職/異動時にIdP無効化とbinding/group同期を規定時間内に反映 | Security | OLA TBD |
| 障害 | 既存token、再login、break-glassの挙動を試験 | Security/Operations | 未実施 |

### 3.3 Bootstrap管理者とbreak-glass

- IdP設定、2名以上の個人管理者、管理者loginを確認するまで`kubeadmin` secretを削除しません。
- 確認後、変更承認を得て`kubeadmin` secretを削除します。削除は安易に元へ戻せないため、前提確認を手順化します。
- installation kubeconfig等の強権限credentialを残す場合は、暗号化保管、二者承認、取得ログ、使用期限、使用後rotationの対象とします。採否はSecurityが決定します。
- break-glass使用時はインシデントIDを発行し、実施者、承認者、理由、コマンド、開始終了、事後レビューを記録します。

## 4. 認可・RBAC設計

グループ名は架空の提案値です。組織命名標準に合わせてG3で確定します。

| Principal（案） | 権限 | Scope | 用途 | 承認 | 棚卸し |
| --- | --- | --- | --- | --- | --- |
| `grp-ocp-cluster-admin` | `cluster-admin` | Cluster | 緊急・構成変更に限定 | Security + 基盤責任者 | 月次 |
| `grp-ocp-platform-ops` | cluster-reader + 承認済みcustom role | Cluster | 日常監視、node/operator確認、限定操作 | Platform lead | 四半期 |
| `grp-ocp-auditor` | `cluster-reader`を起点に監査閲覧を調整 | Cluster | 読み取り監査 | Security lead | 四半期 |
| `grp-<project>-admin` | Project管理用の限定role | Namespace | Project内RBAC/Quota管理 | Application owner | 四半期 |
| `grp-<project>-edit` | `edit` | Namespace | アプリ配備・変更 | Application owner | 四半期 |
| `grp-<project>-view` | `view` | Namespace | 閲覧・調査 | Application owner | 四半期 |
| workload service account | 必要API verbのみ | Namespace | 自動処理 | Application + Platform | releaseごと |

### 4.1 禁止事項

- 共有の通常ユーザーへ管理権限を付けない。
- `system:authenticated`等の広範なsubjectへ管理権限を付けない。
- Project要件だけの権限をClusterRoleBindingで付けない。
- CI/CD tokenを人のCLI操作へ流用しない。
- 一時権限を期限・申請IDなしに残さない。

### 4.2 権限申請

申請には申請者、対象者/Group、role、scope、業務理由、期限、データ分類、承認者を含めます。Platformが`oc auth can-i`等で許可・不許可の両方を確認し、結果を申請IDへ添付します。

## 5. SCC・Podセキュリティ

- まず標準の制限されたSCCで動作するイメージを要求します。
- default SCCを直接変更しません。変更はupgradeで上書き・競合し得るためです。
- 追加権限が必要なら、要求capability、runAsUser、SELinux、hostPath、hostNetwork、privileged、volume typeを1項目ずつリスク評価します。
- custom SCCが不可避な場合、専用ServiceAccountとNamespace scoped RBACの`use`権限に限定し、期限・Owner・例外IDを付けます。
- SCCへのアクセス付与はProject管理者が自由に行える前提にせず、Security/Platform承認を必須とします。
- `default`、`kube-*`、`openshift-*`等のsystem projectで業務workloadを実行しません。

| Check ID | 確認 | 合格条件 | 状態 |
| --- | --- | --- | --- |
| SCC-CHK-001 | imageがnon-rootで動作 | 標準SCCで起動 | 未実施 |
| SCC-CHK-002 | privilege escalation | `allowPrivilegeEscalation: false`等、要件上不要 | 未実施 |
| SCC-CHK-003 | Linux capabilities | 全dropを起点に必要分のみ | 未実施 |
| SCC-CHK-004 | host access | hostPath/hostPID/hostNetwork不要、例外は承認 | 未実施 |

## 6. Secret・鍵管理

| 種別 | 保管 | 配布 | Rotation | Owner | 状態 |
| --- | --- | --- | --- | --- | --- |
| Pull secret | 外部Secret保管 `REF-PULLSECRET` | ISO生成時のみ一時取得 | 契約/漏えい時 | Platform/Security | 実値なし |
| SSH public key | 承認済み鍵管理 `REF-INSTALL-SSH` | install configへ公開鍵のみ | 担当変更/漏えい時 | Platform | 実値なし |
| IdP client secret/bind password | 外部Secret保管 `REF-IDP` | OpenShift Secretへ安全に登録 | 期限/規程 | Security | TBD |
| Backup credential | 外部Secret保管 `REF-BACKUP` | OADP等の専用SA | 90日案、製品能力で確定 | Storage/Security | TBD |
| TLS private key | PKI/Secret管理 | 対象Secretのみ | 期限前自動または承認更新 | Security | TBD |

- Secret manifestを作る場合も、値を`<SECRET_FROM_VAULT>`等のplaceholderにします。
- 端末のshell history、一時ファイル、CI log、support archiveへの混入を確認します。
- etcd暗号化の対象と鍵rotation方針は、対象4.22.zのサポート手順でG3に決めます。

## 7. 証明書・信頼設計

| Certificate | SAN/対象 | 発行元 | 更新Owner | 期限監視 | 状態 |
| --- | --- | --- | --- | --- | --- |
| API named certificate | `api.ocp-prd.example.com` | 組織CA候補 | Security/Platform | 90/60/30日前案 | CA/TBD |
| Ingress wildcard | `*.apps.ocp-prd.example.com` | 組織CA候補 | Security/Platform | 同上 | CA/TBD |
| Proxy trusted CA | 組織Proxy | 組織CA | Security | CA変更監視 | 未受領 |
| IdP TLS CA | IdP endpoint | 組織CA | Security | CA変更監視 | TBD |
| Storage/backup CA | CSI/object endpoint | ベンダー/組織CA | Storage/Security | 期限監視 | TBD |

更新前に新旧CAのoverlap、Secret形式、Ingress/API rollout影響、client trust、切り戻しを検証します。

## 8. Audit・ログ設計

- API監査ログ、OAuth/IdPログ、RBAC/SCC変更、Secretメタデータ操作、certificate変更、break-glass使用を監査対象とします。
- audit policyの変更はデータ量と機密情報露出をSecurityがレビューします。
- 監査ログは改ざん耐性のある外部保存先へTLS転送する候補です。保存先と保持期間は `TBD-010` です。
- 監査ログにtoken、Secret本文、不要な個人データを収集しない設定を確認します。
- 変更IDとの自動関連付けが難しい場合、実施時刻・個人ID・対象resourceを変更記録で照合します。

## 9. Image・脆弱性・ランタイム

| 統制 | 方針 | Owner | 状態 |
| --- | --- | --- | --- |
| Source | 承認registry/repositoryのみ | Application/Security | list TBD |
| Tag | productionはmutable tagだけに依存せずdigest固定候補 | Application | policy TBD |
| Scan | build時と定期再scan、重大度と例外期限を定義 | Security | tool/TBD |
| Signature/provenance | 対応能力と組織標準を確認 | Security | TBD |
| Runtime | 標準監視に加えSysdig候補を評価 | Product owner/Security | 版/TBD |
| Quarantine | 高リスクimage/workloadの停止・隔離・証跡手順 | Security/Platform | Runbook TBD |

Sysdig導入は、必要なcluster-wide権限、host access、kernel/eBPF機能、外部通信、データ所在をレビューしてから承認します。

## 10. セキュリティ試験計画

| 試験ID | 確認 | 期待結果 | 証跡 | 状態 |
| --- | --- | --- | --- | --- |
| TST-IAM-001 | 組織IdP正常login/MFA | 有効な個人IDのみ成功 | login/IdP log | 未実施 |
| TST-IAM-002 | 無効化・IdP障害 | 無効ID拒否、障害時手順どおり | error/audit/runbook | 未実施 |
| TST-IAM-003 | role別許可 | 必要操作のみ成功 | `oc auth can-i` | 未実施 |
| TST-IAM-004 | role別否定 | scope外操作を拒否 | 同上 | 未実施 |
| TST-SEC-001 | 標準SCCとprivileged否定 | 標準workloadは起動し、未承認のprivileged要求は拒否 | Pod/SCC/admission event | 未実施 |
| TST-SEC-002 | Secret漏えい確認 | Git/log/support dataに実値なし | scan結果 | 未実施 |
| TST-SEC-003 | certificate確認 | 承認CA、SAN、chain、期限が設計どおり | cert/connection log | 未実施 |
| TST-SEC-004 | ServiceAccount最小権限 | 不要なtoken自動mountとAPI権限がない | Pod spec/`can-i` | 未実施 |
| TST-AUD-001 | 変更追跡 | 個人ID、対象、verb、時刻を変更IDと照合 | audit + ticket | 未実施 |

## 11. 例外管理

Security例外には `SEC-EXC-nnn` を採番し、標準との差、業務理由、対象、脅威、代替統制、Owner、承認者、期限、撤去条件を記録します。期限切れ例外は自動継続せず再審査します。

## 12. TBDと中断条件

| ID | 内容 | Owner | 期限 | 中断条件 |
| --- | --- | --- | --- | --- |
| TBD-SEC-001 | IdP種別、属性、MFA、group sync | Security | G2前 | 個人ID/RBAC試験不能なら利用開始不可 |
| TBD-SEC-002 | CA、証明書申請・更新 | Security | G3前 | trust chain不明なら公開不可 |
| TBD-SEC-003 | Secret管理製品とrotation | Security | G3前 | 平文配布しかない場合は構築中断 |
| TBD-SEC-004 | 監査保存先・保持・閲覧者 | Security | G2前 | 変更追跡不能なら本番化不可 |
| TBD-SEC-005 | Sysdig権限・通信・版 | Product owner | 第2段階前 | 過剰権限未受容なら導入しない |

## 13. 公式根拠

- [OpenShift 4.22 Authentication and authorization（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/authentication_and_authorization/)
- [Understanding identity provider configuration（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/authentication_and_authorization/understanding-identity-provider)
- [Managing security context constraints（Red Hat公式）](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/authentication_and_authorization/managing-pod-security-policies)

## 14. 承認・変更履歴

| 役割 | 判定 | 日付 | コメント |
| --- | --- | --- | --- |
| Security lead | 未レビュー | - | IdP/CA/監査未確定 |
| Platform lead | 未レビュー | - | 実装未実施 |
| 基盤責任者 | 未承認 | - | 架空案件 |

| 版 | 日付 | 内容 | 作成者 |
| --- | --- | --- | --- |
| 0.1 | 2026-08-17 | 初版 | 生成 AI 支援ドラフト |
