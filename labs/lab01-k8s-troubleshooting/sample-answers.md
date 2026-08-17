# Lab 01 参考回答索引

> [!IMPORTANT]
> **状態:** 比較用資料 / 本人の実施結果ではない / 実機証跡として使用しない

## 修正版 Manifest

| ケース | 参考 Manifest |
| --- | --- |
| ImagePullBackOff | [01-imagepull-fixed.yaml](answers/01-imagepull-fixed.yaml) |
| CrashLoopBackOff | [02-crashloop-fixed.yaml](answers/02-crashloop-fixed.yaml) |
| Pending | [03-pending-fixed.yaml](answers/03-pending-fixed.yaml) |
| Service 疎通不可 | [04-service-fixed.yaml](answers/04-service-fixed.yaml) |
| Ingress 不通 | [05-ingress-fixed.yaml](answers/05-ingress-fixed.yaml) |

調査観点と記録例は [investigation-notes.md](answers/investigation-notes.md) を参照します。

参考回答は唯一の正解ではありません。使用する Kubernetes、Ingress Controller、Node 構成、Registry 到達性によって観測結果は変わるため、本人の環境、コマンド、実測結果を別途記録します。
