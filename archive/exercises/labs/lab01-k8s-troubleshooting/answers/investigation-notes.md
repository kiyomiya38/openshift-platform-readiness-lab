# 調査観点（解答例）

| Case | 主要な事実 | 原因 | 修正後の確認 |
|---|---|---|---|
| ImagePullBackOff | Waiting reason と pull 失敗 Event | `example.invalid` の存在しない Registry | Image pull 完了、Pod Ready |
| CrashLoopBackOff | 終了コード 42、previous log | command が意図的に終了 | restartCount が増えず Running |
| Pending | FailedScheduling と request 値 | Node 容量を超える request | Pod が schedule され Ready |
| Service | EndpointSlice に endpoint がない | selector と Pod label の不一致 | Endpoint に Pod IP が登録 |
| Ingress | backend が Service にない port 9999 | backend port 不一致 | backend 解決。外部疎通は環境依存 |

## 切り分けの型

1. 観測事実を時刻付きで保存する。
2. 直前変更と障害範囲を確認する。
3. 一つの仮説につき、支持・反証する read-only 確認を選ぶ。
4. 修正の影響、承認、切戻しを決める。
5. 技術状態と利用者疎通の両方で復旧を判定する。
