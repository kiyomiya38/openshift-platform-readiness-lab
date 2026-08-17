# Lab 03 実施ノート

> [!IMPORTANT]
> **状態:** 記録様式: 準備済み / 本人実施: 未実施 / 実機証跡: なし

このファイルは [README](README.md) に沿った本人の実施記録用です。参考回答や想定 recap を、実測結果として転記しません。

## 実施条件

| 項目 | 記録 |
| --- | --- |
| 実施日 | 未実施 |
| Control node OS | 未確認 |
| `ansible-core` バージョン | 未確認 |
| Collection バージョン | 未確認 |
| Managed node OS / バージョン | 未確認 |
| 対象 Inventory group | `lab_targets` を予定 |
| Snapshot / 復旧方法 | 未確認 |

## 実施前確認

- [ ] 対象が破棄可能な RHEL 系検証 VM である
- [ ] RHCOS または OpenShift Node ではない
- [ ] `inventory.ini` の文書用 IP を検証 VM へ置換した
- [ ] 秘密鍵、Password、Token を Repository に保存していない
- [ ] Snapshot または復旧方法を確認した
- [ ] NTP server、Repository、firewalld の利用条件を確認した

## 実行記録

| 段階 | 実施状況 | 期待結果 | 実測結果・証跡リンク |
| --- | --- | --- | --- |
| Inventory graph | 未実施 | 対象 host だけが表示される |  |
| Ping / facts | 未実施 | 対象へ read-only で接続できる |  |
| Syntax check | 未実施 | 構文エラーがない |  |
| Package task check | 未実施 | 予定変更を確認できる |  |
| Package bootstrap | 未実施 | 必要 package が導入される |  |
| 残りの task check / diff | 未実施 | 変更差分を事前確認できる |  |
| 1 回目の通常実行 | 未実施 | 設計した状態へ収束する |  |
| 2 回目の通常実行 | 未実施 | 環境外の変化がなければ `changed=0` |  |
| OpenShift 任意演習 | 未実施 | Context guard を通過した対象だけへ適用する |  |

## 考察

```text
Inventory と変数の関係:
check mode だけでは確認できなかった点:
handler が動いた条件:
1 回目と 2 回目の差:
冪等にならなかった task と理由:
OpenShift Node を対象外にする理由:
未確認事項:
```

## 後処理

- [ ] 変更内容と残存 user / file / firewall rule を確認した
- [ ] Snapshot へ戻すか、承認済みの復旧手順を実行した
- [ ] Inventory と出力から実情報をマスキングした
