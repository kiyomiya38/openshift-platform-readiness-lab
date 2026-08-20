# 一次資料確認記録

## 記録情報

| 項目 | 記入内容 |
| --- | --- |
| 確認ID | `<SRC-ID>` |
| 確認日 | `<YYYY-MM-DD TZ>` |
| 確認者 | `<role or anonymized ID>` |
| 対象案件・変更 | `<project/change ID>` |

## 確認一覧

| ID | 製品・対象version | 公式資料・section | URL | 確認した事実 | 設計への反映 | 再確認時点 |
| --- | --- | --- | --- | --- | --- | --- |
| `<SRC-001>` | `<product/version>` | `<document / section>` | `<official URL>` | `<要約。長文転載を避ける>` | `<document/ADR/test ID>` | `<install/update/review date>` |

## 互換性確認

| 組み合わせ | 確認source | 結果 | 制約・例外 | 状態 |
| --- | --- | --- | --- | --- |
| `<OCP + hardware/CSI/Operator/guest OS>` | `<official matrix>` | `未確認` | `<constraint>` | `Open` |

## 差異と判断

- リポジトリ記述との差異：`未確認`
- 変更が必要な成果物：`未確認`
- supportへの確認事項：`なし／<case ID>`
- 未解消時の影響：`<No-Go / risk / none>`

## 記録上の注意

- 対象version、section、確認日を省略しない。
- 検索結果pageではなく根拠pageへlinkする。
- 引用は最小限にし、案件での判断を自分の言葉で記録する。
- 将来変更される情報はinstall、update、renewal前に再確認する。
- subscription限定資料の本文や顧客固有Support caseを公開しない。
