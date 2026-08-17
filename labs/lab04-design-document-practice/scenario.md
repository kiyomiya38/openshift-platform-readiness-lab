# Lab 04 シナリオ索引

> [!IMPORTANT]
> **状態:** 机上演習資料: 準備済み / 本人実施: 未実施 / 本人作成成果物: なし

[README](README.md) の進め方に従い、次のいずれかを選びます。ここにある Template と Sample Answer は本人の設計実績ではありません。

| シナリオ | 作業用 Template | 主な設計論点 |
| --- | --- | --- |
| 小規模 OpenShift 検証環境 | [01-small-openshift-basic-design.md](templates/01-small-openshift-basic-design.md) | topology、sizing、DNS/LB、network、storage、identity、monitoring、backup、update |
| OpenShift Virtualization PoC | [02-virtualization-poc-basic-design.md](templates/02-virtualization-poc-basic-design.md) | 対象 VM、node、CPU/memory、storage、Multus、移行、受入、切り戻し |
| Disconnected 想定環境 | [03-disconnected-design-checklist.md](templates/03-disconnected-design-checklist.md) | mirror 対象、Registry、証明書、DNS/NTP、Firewall、同期、更新運用 |

## 本人成果物の作成条件

1. Template を別名で複製し、元ファイルを上書きしない。
2. 要件、設計判断、根拠、試験方法、未決事項を対応付ける。
3. 不明な製品値を推測で確定せず、確認先と影響を書く。
4. 実在する組織名、環境情報、認証情報を使わない。
5. 完成後に [参考設計索引](sample-design.md) と比較し、差異と修正理由を記録する。
6. 本人が作成したファイルだけを証跡索引へ登録する。
