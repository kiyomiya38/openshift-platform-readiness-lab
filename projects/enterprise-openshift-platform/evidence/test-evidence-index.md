# 試験証跡索引

> [!IMPORTANT]
> 試験IDの正本は[試験仕様書](../docs/16-test-specification.md)、判定の正本は[試験結果書](../docs/17-test-results.md)です。現在、72件はすべて`NOT RUN`で、actual resultと試験証跡はありません。

## 1. 証跡管理状態

| Item | State |
| --- | --- |
| Test specification | Draft / 72 cases |
| Test result rows | Draft / 72 cases |
| Executed cases | 0 |
| `PASS` / `FAIL` / `BLOCKED` | 0 / 0 / 0 |
| `NOT RUN` | 72 |
| Evidence objects | 0 |
| Evidence repository | Not Assigned |

## 2. 正本と役割

| Information | System of record |
| --- | --- |
| Test ID、目的、前提、手順、期待結果 | [16-test-specification.md](../docs/16-test-specification.md) |
| Actual result、判定、issue ID | [17-test-results.md](../docs/17-test-results.md) |
| Raw command output、log、screenshot、packet capture | アクセス制御された証跡保管先（未割当） |
| 公開可能な索引、checksum、機密除去記録 | 本ファイルまたは承認済み公開記録 |

同じ結果を複数ファイルへ転記せず、試験IDと証跡オブジェクトを一意に対応付けます。

## 3. Evidence register

| Test ID | Execution ID | Date/time | Environment | Result | Evidence object | Checksum | Sanitized | Issue ID |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| — | — | — | — | `NOT RUN` | None | — | Not Applicable | — |

## 4. ファイル命名例

```text
<change-id>_<test-id>_<UTC timestamp>_<artifact>.<ext>
```

例: `CHG-XXXX_TST-DNS-001_YYYYMMDDThhmmssZ_dig.txt`

実host名、内部IP、account、tokenを含むraw evidenceは公開リポジトリへ置きません。公開用コピーには機密除去記録と、元証跡を追跡できる組織内referenceを付けます。

## 5. 試験証跡に必要な情報

- test ID、execution ID、change ID
- 対象環境、製品版、tool版、時刻、実施・確認役割
- preconditionとtest data
- 実際の操作とactual result
- expected resultとの差異と判定
- log、command output、metric、event、screenshot等の場所とchecksum
- issue、deviation、rollback、cleanup、final state
- 機密除去の方法と確認者

## 6. 判定上の注意

expected resultの記載、sample output、別環境の結果、静的検証を当該試験の`PASS`として扱いません。証跡が欠けて判定不能な場合は、状況に応じて`NOT RUN`または`BLOCKED`を使用します。

