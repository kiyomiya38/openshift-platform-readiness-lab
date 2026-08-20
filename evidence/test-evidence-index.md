# 試験証跡索引

## 対象情報

| 項目 | 記入内容 |
| --- | --- |
| 試験計画／revision | `<document / revision>` |
| 対象環境 | `<environment ID>` |
| OCP／関連製品version | `<versions / digest>` |
| 実施期間 | `未実施` |
| 証跡保管先 | `<access-controlled repository ID>` |
| 状態 | `NOT RUN` |

## 証跡索引

| 試験ID | 要求・設計ID | 実施日時 | 実施者 | 結果 | 証跡ID・file | 不具合ID | 再試験ID |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `<TST-001>` | `<REQ/DES IDs>` | `未実施` | `—` | `NOT RUN` | `—` | `—` | `—` |

## 証跡metadata

各証跡には、可能な範囲で次を付与します。

- 試験ID、取得日時とtimezone、対象環境、対象version
- 実行commandまたは操作、exit code、観測点
- 期待結果と実測結果の対応
- 取得者、原本file名、checksum、保管先
- mask/redactionの有無と方法

## 公開・秘匿確認

| 確認項目 | 状態 |
| --- | --- |
| Token、password、private keyを含まない | `未確認` |
| Secret objectのdataを含まない | `未確認` |
| 顧客host、IP、URL、accountを匿名化した | `未確認` |
| 個人情報、業務data、license情報を含まない | `未確認` |
| 証跡原本のaccess権と保持期限を確認した | `未確認` |

## 総合判定

- `PASS`：`0`
- `FAIL`：`0`
- `BLOCKED`：`0`
- `NOT RUN`：`未集計`
- `N/A`：`0`
- 未解決の重大不具合：`未集計`
- 総合判定：`NOT RUN`
