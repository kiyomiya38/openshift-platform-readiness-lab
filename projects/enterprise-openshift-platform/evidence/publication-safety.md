# 公開・機密除去基準

> [!CAUTION]
> 本書は、架空のOpenShift実務成果物サンプルを一般公開する際の情報分類と機密除去基準です。公開可否は、採用組織の情報管理、契約、法務、セキュリティ基準を優先して判断します。

## 1. 公開サンプルの状態表示

公開版では、次の事実を隠さず明記します。

- 架空プロジェクトの成果物サンプルである。
- 成果物はDraft・未レビュー・未承認である。
- 実機構築、試験、移行、復旧は実施していない。
- 試験結果72件はすべて`NOT RUN`である。
- exact version、互換性、製品選定、契約条件に未確定事項がある。

## 2. 使用できる例示値

| 種別 | 公開版で使用する値 | 条件 |
| --- | --- | --- |
| Domain | `example.com`配下 | 予約済み例示domainである旨を保持 |
| Host / external / VM IPv4 | RFC 5737の文書用範囲 | `192.0.2.0/24`等を実環境へ使用しない |
| Pod / Service CIDR | RFC 1918 private range | 架空のクラスタ内部値と明記 |
| MAC | locally administeredの架空値 | 実機から取得しない |
| Credential | `<PLACEHOLDER>`または参照ID | 実値、形式を模した有効値を置かない |
| Person / organization | team role、Example等 | 実在名を使用しない |

## 3. 公開禁止情報

- password、token、pull secret、cookie、Authorization header
- SSH private key、certificate private key、kubeconfig、service account token
- 実在する組織名、担当者名、メール、電話、chat room、会議URL
- 内部FQDN、IP、MAC、VLAN、routing、Firewall rule、proxy、registry
- account、subscription、Cluster ID、serial、asset ID、ticket ID
- raw log、support bundle、must-gather、packet capture、screenshot内の識別子
- Sysdig capture/audit、Kong access log、application payload、DB内容
- local user profile、workspace absolute path、端末固有情報
- 契約、価格、license key、脆弱性の未公開情報

## 4. 機密除去方法

1. 元データを公開作業領域へ直接コピーしない。
2. identifierを種類別の一貫したplaceholderへ置換する。
3. 値だけでなくmetadata、filename、image、link targetも検査する。
4. free text、code block、comment、Git historyを対象にpattern scanする。
5. secret scannerと目視レビューを併用する。
6. 置換後も構成関係が理解できる最小限の情報だけを残す。
7. 検査対象revision、tool、結果、未解決事項を記録する。

## 5. 公開前Gate

| Gate | Required condition | Current state |
| --- | --- | --- |
| Artifact status | Draft / Not Reviewed / Not Approved / Not Runを明示 | Draft |
| Secret scan | credential patternのfindingが0件 | Static scan Pass／独立レビュー未実施 |
| Identifier scan | 実在組織・内部値・local pathのfindingが0件 | Static scan Pass／独立レビュー未実施 |
| Link review | 内部・private URLを含まず、relative linkが解決する | Relative links Pass／URL到達性Not Run |
| License / source review | 再配布条件と引用範囲を確認 | Not Reviewed |
| Technical claim review | success、compatibility、supportの未検証断定がない | Not Reviewed |
| Independent release review | 指定reviewerがscopeと結果を記録 | Not Approved |

Gateの状態は、[成果物レビュー記録](artifact-review-record.md)と[検証記録](verification-record.md)へ対応付けます。

## 6. 公開版の安全な説明

> このリポジトリは、OpenShift 4.22基盤導入で使用される要求、設計、構築、試験、移行、運用、管理成果物の構成を示す、一般公開用の架空プロジェクトサンプルです。成果物はDraft・未レビュー・未承認で、実機作業と試験は実施していません。記載値は例示値であり、実案件では再設計と正式承認が必要です。
