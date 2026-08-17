# 学習ログ

> [!IMPORTANT]
> このログは本人が実際に確認・説明・実施した内容だけを記録します。初期状態はすべて Not Started です。AI 生成 draft の存在を学習完了として記録しません。

## 1. Baseline

| Item | State |
| --- | --- |
| Project artifacts generated | E0 draft |
| Learner review | Not Started |
| Learner corrections | None recorded |
| Official-source check | Not Started |
| Desk/tabletop activity | Not Run |
| Lab activity | Not Run |

## 2. Learning stage tracker

| Stage | Artifacts | Intended output | Status | Date | Evidence |
| --- | --- | --- | --- | --- | --- |
| A | docs 00〜04 | requirement trace 3 本 | Not Started | — | None |
| B | docs 05〜10 | architecture / traffic / failure-domain notes | Not Started | — | None |
| C | docs 11〜12 | VM comparison / product Gate | Not Started | — | None |
| D | docs 13〜15 + build assets | value consistency / tabletop | Not Started | — | None |
| E | docs 16〜17 | test correction / evidence plan | Not Started | — | None |
| F | docs 18〜19 | migration / rollback tabletop | Not Started | — | None |
| G | docs 20〜23 | runbook / management record review | Not Started | — | None |
| H | evidence and claim audit | submission decision | Not Started | — | None |

詳細な進め方は [24-learning-guide](../docs/24-learning-guide.md) を参照します。

## 3. Daily entry template

```text
Date / duration:
Artifact and exact revision:
Primary source and version:

本人の言葉で整理した内容:

Artifact と source を照合した結果:
- 一致:
- Correction:
- Still unknown:

実際に行った作業:
- Read / desk review / static validation / lab execution:
- Command / tool / environment:
- Actual result:

Evidence:
Next action / owner / due:
Claim boundary:
```

## 4. Correction log

| ID | Date | Artifact | Original statement | Correction and source | Impact | Status |
| --- | --- | --- | --- | --- | --- | --- |
| — | — | — | None recorded | None recorded | — | Not Started |

## 5. Unknown-term log

| Term / topic | Where encountered | Current understanding | Source to read | Status |
| --- | --- | --- | --- | --- |
| — | — | None recorded | — | Not Started |

用語を調べた場合は、定義だけでなく本設計のどの decision / test / runbook に影響するかを記録します。

## 6. Experience-label rule

| Actual activity | Allowed label |
| --- | --- |
| draft を受け取っただけ | 資料あり / 未レビュー |
| 本人が source と照合 | 学習・文書レビュー |
| local static check | 静的検証 |
| disposable lab で本人実行 | 検証環境での演習 |
| commercial customer environment | 商用経験。今回該当なし |

