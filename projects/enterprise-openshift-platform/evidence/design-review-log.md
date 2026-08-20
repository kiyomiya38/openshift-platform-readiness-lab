# 設計レビュー実施記録

> [!IMPORTANT]
> 本書は、設計レビュー会議または非同期レビューの論点、判断、actionを時系列で記録するためのサンプルです。現在、実施済みの設計レビュー記録はありません。

## 1. Review register

| Review ID | Date | Artifact / revision | Review scope | Reviewer roles | Decision | Open actions | Status |
| --- | --- | --- | --- | --- | --- | ---: | --- |
| — | — | — | — | — | — | 0 | Not Reviewed |

## 2. Review record template

```text
Review ID:
Date/time/timezone:
Target artifact and exact revision:
Purpose / scope:
Required reviewer roles:
Participants by role:
Inputs and primary sources:

Findings:
- ID / severity / location / finding / impact

Decisions:
- Decision / authority / effective scope / condition

Actions:
- Action / owner role / due / related issue or change

Open questions:
Review result: Not Reviewed / In Review / Needs Correction / Reviewed
Approval result: Not Approved / Approved
Evidence location and checksum:
Sanitization performed:
```

## 3. レビュー観点

- 上位要件、前提、責任分界との整合
- 対象OpenShift 4.22.zと関連製品の公式仕様・互換性
- availability、security、operations、failure recoveryの成立性
- parameter、構成図、設定例、手順、試験の一貫性
- TBD、assumption、risk、change、ADRのownerと期限
- expected resultとactual result、DraftとApprovedの区別
- 公開用サンプルへ含められない識別子・秘密情報の除去

## 4. 記録上の注意

review meetingを開催しただけでは`Reviewed`としません。指摘、判断、action、未解決事項を記録し、承認はレビューとは別の状態として管理します。

