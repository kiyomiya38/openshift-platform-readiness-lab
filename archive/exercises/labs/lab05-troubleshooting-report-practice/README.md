# Lab 05: 障害調査報告書作成練習

> [!IMPORTANT]
> **状態:** 演習資料: 準備済み / 本人実施: 未実施 / 実機証跡: なし


架空の OpenShift 障害ケースに含まれる `oc get/describe/logs/events` 相当の抜粋を読み、障害調査報告書へまとめる机上演習です。クラスタ操作はありません。ケース内の名前、IP、時刻は教材用で、実顧客情報ではありません。

## 学習目標

- 症状、影響、直接原因、根本原因を区別できる。
- ログや Event の「事実」と未検証の「仮説」を分けて書ける。
- Pod、Route、PVC、Operator、DNS の依存関係に沿って切り分けられる。
- 暫定対応、恒久対応、復旧確認、再発防止を分けて報告できる。

## 前提条件

- Kubernetes/OpenShift の Pod、Service、Route、PVC、Operator、DNS の基礎知識
- Markdown editor
- 実クラスタや認証情報は不要

## 安全上の注意

- ケースのコマンドは「取得済み証跡の説明」であり、実環境へそのまま実行しない。
- 実案件の log、host、IP、user、token、cookie、Secret を回答へ貼らない。
- 原因が確定できない場合は `要確認` とし、追加確認を具体化する。
- Pod 削除、node 再起動、Operator 再導入などを根拠なく暫定対応にしない。

## セットアップ

1. `cases/` から一つ選ぶ。
2. Repository ルートの `templates/troubleshooting-report-template.md` を作業用に複製する。
3. 15 分で事実、仮説、追加確認を記入する。
4. 10 分で原因、対応、復旧条件、再発防止を記入する。
5. `sample-reports/` と比較する。

## Exercise 1: Pod 起動失敗

[cases/01-pod-startup-failure.md](cases/01-pod-startup-failure.md) を読み、次を整理します。

- `CrashLoopBackOff` は原因か、表示状態か。
- ConfigMap 参照、Application log、直前変更がどうつながるか。
- ConfigMap 修正後に rollout と業務疎通の両方を確認する理由。

## Exercise 2: Route アクセス不可

[cases/02-route-unreachable.md](cases/02-route-unreachable.md) について、`DNS → Router → Route → Service → EndpointSlice → Pod` の各層を表にします。外部 503 と DNS エラーを混同しないようにします。

## Exercise 3: PVC 未割当

[cases/03-pvc-pending.md](cases/03-pvc-pending.md) について、PVC Event、StorageClass、CSI provisioner、backend capacity のどこまで事実があるか示します。`Pending` だけで storage 障害と断定しません。

## Exercise 4: Operator 異常

[cases/04-operator-degraded.md](cases/04-operator-degraded.md) について、ClusterOperator、Subscription、CSV、Operator Pod、管理対象 CR/Operand を区別します。このケースは OLM Operator です。

## Exercise 5: DNS 不通

[cases/05-dns-failure.md](cases/05-dns-failure.md) について、単一 Pod、単一 Namespace、全 Cluster、外部 DNS のどの範囲かを絞ります。NetworkPolicy の egress と DNS Service/Endpoint を確認します。

## 検証・採点

各報告を 0～2 点で自己採点します（計 16 点）。

| 観点 | 0 | 1 | 2 |
|---|---|---|---|
| 影響 | 不明 | 技術症状のみ | 利用者・範囲・時間を記載 |
| Timeline | なし | 一部 | 検知から復旧まで事実順 |
| Evidence | なし | コマンド名のみ | 出力と解釈を分離 |
| Hypothesis | 断定 | 候補あり | 支持・反証・追加確認あり |
| Cause | 症状を原因化 | 直接原因のみ | 根本原因と仕組みを記載 |
| Action | 危険／曖昧 | 暫定のみ | 承認・切戻し・恒久対応あり |
| Recovery | 「直った」 | 技術状態のみ | 技術・業務・監視期間あり |
| Prevention | 精神論 | 単発対応 | 設計・試験・監視・owner/期限 |

13 点以上を目標にし、足りない箇所は `REVIEW:` を付けて修正します。

## クリーンアップ

クラスタ変更はありません。回答に実環境情報が混ざっていないかを確認し、不要な作業コピーだけを削除します。

## 公式リファレンス

- [OpenShift support: gathering data about your cluster](https://docs.redhat.com/en/documentation/openshift_container_platform/)
- [Kubernetes: Debug Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/)
- [Kubernetes: Debug Services](https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/)

## 面談での説明例

> [!IMPORTANT]
> 演習完了・証跡記録後のみ、本人が作成した報告内容を過去形で説明します。現時点では本人レビュー・演習とも未実施です。

「現時点では、Pod、Route、PVC、Operator、DNS の架空ケースから障害調査報告書を作る演習資料と参考報告書を準備した段階で、本人による報告書作成と証跡記録はまだ行っていません。実施後は、事実と仮説、暫定・恒久対応、復旧判定をどう分けたかを限定して説明します。」
