# 外部提出・説明境界

> [!CAUTION]
> 現在の project は本人レビュー前・実機未実施の AI 支援 draft です。そのまま「自分が設計・構築・検証した成果」として提出することは推奨しません。

## 1. Nature of this material

- 完全に架空の学習用案件です。
- 外部・host・VMの例示IPにはRFC 5737の文書用範囲、Pod/Service networkにはRFC 1918のprivate範囲を架空値として使用し、domainには予約済み例示domainの`example.com`を使用します。いずれも実環境値ではありません。
- OpenShift Container Platform 4.22.z は暫定です。
- OpenShift Virtualization は OLM 導入方針ですが未導入です。
- MTV は参考として 2.12 文書を含みますが、採用版・互換性は未確定です。
- Kong と Sysdig は設計のみで、product、version、license、support、data handling は未確定です。
- cluster build、test、VM migration、rollback、backup/restore、RPO/RTO 測定は未実施です。
- 商用 OpenShift / Kubernetes 設計構築運用経験の証明ではありません。

## 2. AI assistance disclosure

要求の整理、文書構造、draft text、sample configuration、cross-reference の作成に生成 AI の支援を使用しています。AI 出力自体は本人の知識・技能の証拠ではありません。

本人が primary official source と照合し、誤りを訂正し、自分の言葉で説明し、可能な範囲で検証した項目だけを本人の学習成果として扱います。

## 3. Claim matrix

| Claim | Current status | External wording |
| --- | --- | --- |
| 架空案件の文書 draft がある | Fact | AI 支援で学習用 draft を準備した |
| 本人が全内容を理解した | Not established | 言わない |
| 本人が primary sources を確認した | Not established | 言わない |
| OCP 4.22 cluster を構築した | False / Not Run | 言わない |
| Virtualization / MTV を導入・移行した | False / Not Run | 言わない |
| Kong / Sysdig を導入した | False / design-only | 言わない |
| backup/restore が RPO/RTO を満たす | Not tested | 目標値・試験計画としてのみ説明 |
| 商用案件で設計・構築・運用した | False for this project | 言わない |

## 4. Current safe description

> OpenShift 基盤導入の実務工程を学ぶため、完全に架空の要件を使い、要件・設計・構築手順・試験・移行・運用管理の文書 draft を AI 支援で準備しています。現時点では本人レビューと実機検証前で、商用経験や構築成功を示すものではありません。今後、公式資料との照合、訂正、tabletop、検証環境での実行を段階的に記録します。

## 5. Future wording after evidence exists

### After E1/E2 only

> 架空の OpenShift 基盤案件について、公式資料と照合しながら要件から試験・切り戻しまでの文書をレビューする学習を行いました。実機構築は未実施です。

### After E3

> 架空案件の成果物について、リンク、構文、設定値整合、手順の tabletop を実施しました。これは静的検証であり、クラスタ上の動作確認ではありません。

### After E4

> 検証環境で実施した範囲に限り、環境、version、手順、actual result、失敗と correction を提示できます。商用経験とは区別しています。

証跡が登録される前に将来形の文を使いません。

## 6. Submission Gate

| Gate | Required | Current state |
| --- | --- | --- |
| Personal data / secret scan | customer/name/token/private key なし | Not Run |
| Learner review | artifact review record complete | Not Started |
| Primary source check | exact version/date/section | Not Started |
| Claim audit | every success/experience claim backed by evidence | Not Started |
| Link / syntax / diagram check | actual tool output | Not Run |
| Reviewer approval | reviewer identity/date/scope | Not Approved |
| Known limitation list | open issue/risk reflected | Draft only |

全 Gate を通過するまで External submission readiness は `Not Ready` です。

## 7. Sanitization

提出前に次を除外・置換します。

- Secret、token、password、private key、pull secret、cookie
- 実在 customer/company/person、内部 FQDN/IP/MAC、ticket/contact
- support bundle 内の識別子、registry、cloud/account metadata
- Sysdig capture / audit、Kong access log、application payload の機密情報
- local filesystem path や user profile 名

`example.com`、RFC 5737 の外部・host・VM例、RFC 1918 のcluster内部CIDRは、用途と架空値である注記を残します。

## 8. Reviewer decision

| Item | Value |
| --- | --- |
| Learner sign-off | Not Signed |
| Technical reviewer | Not Assigned |
| Claim reviewer | Not Assigned |
| Submission decision | Not Ready |
| Decision date | — |
