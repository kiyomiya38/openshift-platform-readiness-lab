# Lab 04: 基盤設計書作成練習

> [!IMPORTANT]
> **状態:** 演習資料: 準備済み / 本人実施: 未実施 / 実機証跡: なし


三つの架空シナリオについて、基本設計の判断と根拠を文章化する机上演習です。実環境の構築は行いません。`templates/` を複製して記入し、`sample-answers/` は完成後のレビュー観点として使います。

## 学習目標

- 要件、設計判断、根拠、試験方法、未決事項を結び付けられる。
- OpenShift のクラスタ、DNS/LB、network、storage、security、operation の設計観点を列挙できる。
- OpenShift Virtualization PoC の成功条件と対象 VM 選定を説明できる。
- Disconnected 環境で Image 同期以外に必要な依存関係を整理できる。

## 前提条件

- Markdown editor
- OpenShift の Project、Route、Operator、StorageClass の基礎知識
- OpenShift Virtualization、MTV、Disconnected Installation は概要理解でよい
- 製品版ごとの正確な値は、対象バージョンの公式文書で要確認

## 安全上の注意

- 架空の host、IP、組織名だけを使用し、顧客情報や実環境の構成を転記しない。
- IP、証明書、pull secret、認証情報を記入しない。必要なら参照 ID の欄だけを作る。
- サンプル回答は唯一の正解ではない。前提が変われば設計判断も変わる。
- 実案件で使う場合は、セキュリティ、運用、製品サポートのレビューを受ける。

## セットアップ

次の三つから一つ選び、対応 Template を作業用ファイルへ複製します。

1. 小規模 OpenShift 検証環境: [templates/01-small-openshift-basic-design.md](templates/01-small-openshift-basic-design.md)
2. OpenShift Virtualization PoC: [templates/02-virtualization-poc-basic-design.md](templates/02-virtualization-poc-basic-design.md)
3. Disconnected 想定環境: [templates/03-disconnected-design-checklist.md](templates/03-disconnected-design-checklist.md)

ファイル名例は `work-01-small-openshift.md` です。元 Template と Sample Answer は上書きしません。

## Exercise 1: 小規模 OpenShift 検証環境

次の要求を設計へ変換します。

- 開発者 20 名が教育用 workload を実行する。
- 計画停止は可能だが、Control Plane の単一障害で直ちに API を失わない構成とする。
- 既存 DNS/NTP/virtualization/storage の詳細は未確定。
- 機密データは置かず、Internet 接続は組織 Proxy 経由の想定。

最低限、cluster topology、sizing の決め方、CIDR 重複、API/Ingress DNS/LB、IdP/RBAC/SCC、StorageClass、monitoring/logging、backup、update、試験を記入します。数値を推測した箇所は `要確認` に戻します。

## Exercise 2: OpenShift Virtualization PoC

次の要求を設計へ変換します。

- VMware 上の三種類の代表 VM を PoC 候補とする。
- VM 起動だけでなく network、storage、backup、monitoring、migration/rollback を評価する。
- 本番移行の承認ではなく、追加調査事項を明らかにする PoC とする。

KVM/QEMU の直接操作手順ではなく、VirtualMachine/DataVolume/PVC/Multus/MTV を OpenShift API から管理する構成として整理します。CPU virtualization、共有 storage、Live Migration の要件は環境依存として確認します。

## Exercise 3: Disconnected 設計チェック

次の要求を設計へ変換します。

- Cluster から Internet へ直接接続できない。
- 接続可能 Zone で Image を取得し、承認後に内部 Registry へ持ち込む。
- OpenShift release、Operator、業務 Image を継続更新する。

Mirror Registry、`oc-mirror`、ImageSetConfiguration、Operator catalog、証明書、pull secret、DNS/NTP/LB、Firewall、持込み審査、容量、backup、更新失敗時の戻しを含めます。対象版で使う mirror 関連 resource 名は公式文書で要確認とします。

## レビューと検証

各回答を次の観点で自己レビューします。

| 観点 | 合格条件 |
|---|---|
| Traceability | 要件ごとに設計項目と試験方法がある |
| Decision | 方針だけでなく決める値・責任者が分かる |
| Rationale | 可用性、security、operation の根拠がある |
| Boundary | platform、network、storage、application の責任分界がある |
| Uncertainty | 未確定事項を推測せず `要確認` として owner/期限を置く |
| Operability | monitoring、backup、update、incident、rollback を含む |

最後に Sample Answer と比較し、抜けた観点を別の色または `REVIEW:` prefix で追記します。サンプルの値を無条件にコピーしないでください。

## クリーンアップ

クラスタ変更はありません。作業ファイルに実情報が混ざっていないことを確認し、不要な作業コピーだけを削除します。Template と Sample Answer は残します。

## 公式リファレンス

- [OpenShift Container Platform architecture](https://docs.redhat.com/en/documentation/openshift_container_platform/)
- [OpenShift Virtualization](https://docs.redhat.com/en/documentation/openshift_container_platform/)
- [Disconnected environments](https://docs.redhat.com/en/documentation/openshift_container_platform/)

## 面談での説明例

> [!IMPORTANT]
> 演習完了・証跡記録後のみ、本人が作成した設計内容を過去形で説明します。現時点では本人レビュー・演習とも未実施です。

「現時点では、小規模 OpenShift、Virtualization PoC、Disconnected 環境の架空要件を設計へ落とす演習資料と参考回答を準備した段階で、本人による設計書作成と証跡記録はまだ行っていません。実施後は、自分で判断した事項、根拠、要確認事項を限定して教材・机上演習として説明します。」
