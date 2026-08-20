# Sample Report 01: Pod 起動失敗

## 概要・影響

- 09:05 から `apps-lab` の注文 API が利用不可。
- 新 Replica の Container が exit code 2 で終了し、再起動を繰り返した。
- 影響利用者数と復旧時刻はケース情報だけでは不明のため `要確認`。

## 事実と原因

| 区分 | 内容 |
|---|---|
| 事実 | previous log は `DB_PORT` が整数でないと出力。ConfigMap 値は `fifty-four-thirty-two`。直前に同 ConfigMap を変更。Image pull は成功。 |
| 直接原因 | Application が不正な DB port 設定を読み、exit 2 で終了した。 |
| 根本原因候補 | ConfigMap の型・値を admission/CI で検証せず rollout できたこと。承認記録と pipeline は要確認。 |

`CrashLoopBackOff` は kubelet の backoff 状態であり、原因そのものではない。

## 対応案

1. 変更 ID と Git/source of truth の旧値 `5432` を再確認し、承認を得る。
2. ConfigMap を承認済み値へ戻す。Application の読込方式に応じ rollout が必要か要確認。
3. `rollout status`、Pod Ready/restartCount、current/previous log、Service endpoint を確認する。
4. 注文 API の非破壊な業務疎通を行い、定めた時間 Alert 再発がないことを監視する。

## 再発防止

- ConfigMap schema/値の CI validation を追加し、port は 1～65535 の整数に限定する。
- rollout 前後の smoke test と自動停止 gate を追加する。
- Change review と ConfigMap owner を明確にし、GitOps 差分を証跡化する。
