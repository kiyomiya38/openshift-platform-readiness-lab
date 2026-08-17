# Lab 02 実施ノート

> [!IMPORTANT]
> **状態:** 記録様式: 準備済み / 本人実施: 未実施 / 実機証跡: なし

このファイルは [README](README.md) の演習結果を本人が記録するための様式です。実施前は空欄を結果であるかのように埋めません。顧客情報、内部 FQDN・IP、認証情報、Secret の実値は記録しません。

## 実施条件

| 項目 | 記録 |
| --- | --- |
| 実施日 | 未実施 |
| OpenShift バージョン | 未確認 |
| `oc` バージョン | 未確認 |
| 検証環境の種類 | 未確認 |
| Context / API Server | 機密情報を除いた識別子を記録 |
| Project | 未確認 |
| 実施者の権限 | 未確認 |
| Image の置換有無 | 未確認 |

## 実施前確認

- [ ] 破棄可能または利用許可済みの検証環境である
- [ ] 対象 Context、利用者、Project を確認した
- [ ] Project 作成・削除の可否を確認した
- [ ] Manifest の Namespace と Image を確認した
- [ ] Secret が教材用の非機密値であることを確認した
- [ ] Cleanup 方法を確認した

## Manifest 適用記録

| 対象 | 実施状況 | 期待結果 | 実測結果・証跡リンク |
| --- | --- | --- | --- |
| Project / Namespace | 未実施 | 対象 Project を選択できる |  |
| ConfigMap | 未実施 | Pod から設定を参照できる |  |
| 教材用 Secret | 未実施 | 実値を公開せず参照方法を確認できる |  |
| ServiceAccount / RBAC | 未実施 | 許可・拒否を意図どおり確認できる |  |
| Deployment | 未実施 | Pod が Ready になる |  |
| Service / EndpointSlice | 未実施 | Ready Pod が Endpoint になる |  |
| Route | 未実施 | 許可された経路から応答を確認できる |  |
| SCC | 未実施 | 適用結果を read-only で確認できる |  |
| Operator | 未実施 | 既存リソースを read-only で確認できる |  |

## 観察と考察

```text
Kubernetes Namespace と OpenShift Project の関係:
Route から Pod までの経路:
ConfigMap と Secret の取扱いの違い:
RBAC と ServiceAccount の関係:
SCC で確認した内容:
Operator で確認した内容:
期待値との差:
未確認事項:
```

## 後処理

- [ ] ラボで作成した namespaced resource を削除した
- [ ] 割り当て済みの共有 Project 自体は削除していない
- [ ] 残存リソースと Event を確認した
- [ ] 機密情報が記録に含まれないことを確認した
