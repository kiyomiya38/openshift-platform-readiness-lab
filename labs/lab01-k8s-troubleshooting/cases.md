# Lab 01 ケース索引

> [!IMPORTANT]
> **状態:** 演習資料: 準備済み / 本人実施: 未実施 / 実機証跡: なし
>
> `*-broken.yaml` は意図的に壊した教材です。[README](README.md) の安全条件を確認し、許可された破棄可能な検証環境以外へ適用しません。

## ケース一覧

| ケース | 障害 Manifest | 主な観察対象 |
| --- | --- | --- |
| ImagePullBackOff | [01-imagepull-broken.yaml](manifests/01-imagepull-broken.yaml) | Pod status、Event、Image 名、Registry 到達性 |
| CrashLoopBackOff | [02-crashloop-broken.yaml](manifests/02-crashloop-broken.yaml) | Container state、終了理由、現在・直前ログ |
| Pending | [03-pending-broken.yaml](manifests/03-pending-broken.yaml) | Scheduler Event、request、Node の割当可能量 |
| Service 疎通不可 | [04-service-broken.yaml](manifests/04-service-broken.yaml) | selector、EndpointSlice、Pod label、targetPort |
| Ingress 不通 | [05-ingress-broken.yaml](manifests/05-ingress-broken.yaml) | Ingress rule、Service、EndpointSlice、Controller、DNS |

Namespace の準備は [00-namespace.yaml](manifests/00-namespace.yaml) を使用します。ケースごとの調査コマンド、合格条件、後処理は [README](README.md) を参照してください。

## 記録する内容

各ケースについて、次を自分の言葉で記録します。

1. 症状と影響範囲
2. 変更前に取得した事実
3. 原因仮説と、それを確認するコマンド
4. 修正内容と判断根拠
5. 修正後の実測結果
6. 未確認事項と後処理

参考回答は、本人の調査記録を作成した後に [sample-answers.md](sample-answers.md) から参照します。
