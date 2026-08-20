# Lab 01: Kubernetes トラブルシューティング

> [!IMPORTANT]
> **状態:** 演習資料: 準備済み / 本人実施: 未実施 / 実機証跡: なし


このラボは、教材用クラスタで 5 種類の典型障害を再現し、`get → describe → events/logs → 依存先確認` の順で切り分ける演習です。`manifests/*-broken.yaml` は意図的に壊しています。本番環境では適用しないでください。

## 学習目標

- Pod の状態表示を症状ではなく調査の入口として扱える。
- ImagePullBackOff、CrashLoopBackOff、Pending の証跡を採取できる。
- Service と Ingress の通信経路を上流から Pod まで追える。
- 事実、仮説、修正、修正後確認を分けて記録できる。

## 前提条件

- 破棄可能な Kubernetes 検証クラスタと、Namespace を作成・削除できる権限
- `kubectl`（Server と Client の対応版は要確認）
- Ingress 演習のみ Ingress Controller と到達可能な Host 設定（環境依存のため要確認）
- 外部 Registry へ到達できること。非接続環境では利用可能 Image へ置換する（要確認）

## 安全上の注意

- `kubectl config current-context` と `kubectl cluster-info` で検証クラスタであることを確認する。
- 専用 Namespace `os-readiness-lab` 以外を変更しない。
- YAML は障害再現用であり、監視通知が発生し得る。共有環境では事前に管理者へ連絡する。
- `answers/` は原因を自分で記録した後に参照する。
- Secret や実環境のログを外部サービスへ貼り付けない。

## セットアップ

```bash
kubectl config current-context
kubectl apply -f manifests/00-namespace.yaml
kubectl get namespace os-readiness-lab
```

端末を二つ使える場合は、片方でイベントを観測します。

```bash
kubectl get events -n os-readiness-lab --watch
```

## 共通の調査手順

1. `kubectl get pods -n os-readiness-lab -o wide` で症状と範囲を確認する。
2. `kubectl describe pod <pod> -n os-readiness-lab` の State、Reason、Events を読む。
3. 起動済み Container がある場合だけ `kubectl logs` と `kubectl logs --previous` を比較する。
4. Service 系は `Service → EndpointSlice → Pod label → containerPort` の順で照合する。
5. 変更前の仮説と、修正後に何を確認するかをメモする。

## Exercise 1: ImagePullBackOff

```bash
kubectl apply -f manifests/01-imagepull-broken.yaml
kubectl get pod imagepull-broken -n os-readiness-lab
kubectl describe pod imagepull-broken -n os-readiness-lab
```

確認課題:

- Container の Waiting reason と Event のメッセージは何か。
- Image 名、tag、Registry 到達性、imagePullSecret のどこを順に確認するか。
- この教材で失敗が確実になる仕掛けは何か。

原因を記録後、`answers/01-imagepull-fixed.yaml` を適用します。修正版は比較用の別名 `imagepull-fixed` で作成されます。

## Exercise 2: CrashLoopBackOff

```bash
kubectl apply -f manifests/02-crashloop-broken.yaml
kubectl get pod crashloop-broken -n os-readiness-lab -w
kubectl logs crashloop-broken -n os-readiness-lab
kubectl logs crashloop-broken -n os-readiness-lab --previous
kubectl describe pod crashloop-broken -n os-readiness-lab
```

終了コード、再起動回数、current/previous log を区別します。修正版 `answers/02-crashloop-fixed.yaml` は、Pod の immutable field を無理に変更せず、比較用の別名で作成します。

## Exercise 3: Pending

```bash
kubectl apply -f manifests/03-pending-broken.yaml
kubectl get pod pending-broken -n os-readiness-lab
kubectl describe pod pending-broken -n os-readiness-lab
kubectl get nodes
```

Scheduler Event と request 値を確認します。実クラスタでは taint、Affinity、PVC、Quota も候補です。修正版 `answers/03-pending-fixed.yaml` は比較用の別名で作成します。

## Exercise 4: Service 疎通不可

```bash
kubectl apply -f manifests/04-service-broken.yaml
kubectl get pod,service,endpointslice -n os-readiness-lab -l training-case=service
kubectl describe service service-broken -n os-readiness-lab
```

Service selector と Pod label を比較し、EndpointSlice に endpoint がない理由を説明します。クラスタ内の一時的な確認 Podを作る権限がある場合のみ、組織で承認された Image から Service へ HTTP 接続します（Image は要確認）。正解は `answers/04-service-fixed.yaml` です。

## Exercise 5: Ingress 疎通不可

```bash
kubectl apply -f manifests/05-ingress-broken.yaml
kubectl get ingress,service,endpointslice,pod -n os-readiness-lab -l training-case=ingress
kubectl describe ingress ingress-broken -n os-readiness-lab
```

IngressClass、Host/DNS、Controller、backend Service/port、Endpoint の順に確認します。Ingress Controller がない環境では、YAML と `describe` の机上確認までとします。正解は `answers/05-ingress-fixed.yaml` です。

## 全体検証

修正版を一つずつ適用し、状態を確認します。

```bash
kubectl apply -f answers/01-imagepull-fixed.yaml
kubectl apply -f answers/02-crashloop-fixed.yaml
kubectl apply -f answers/03-pending-fixed.yaml
kubectl apply -f answers/04-service-fixed.yaml
kubectl apply -f answers/05-ingress-fixed.yaml
kubectl get pods -n os-readiness-lab -l training-result=fixed
kubectl get endpointslice -n os-readiness-lab
```

`*-broken` Pod が失敗状態のまま残るのは意図した結果です。修正版に `training-result=fixed` label が付き、こちらだけが Ready であることを確認します。

Ingress の外部到達性は Controller、DNS、LB に依存するため、`Ingress resource が作成された` だけで合格にせず環境の合格条件を要確認とします。調査観点は [answers/investigation-notes.md](answers/investigation-notes.md) と比較できます。

## クリーンアップ

次のコマンドは専用 Namespace 内の全教材リソースを削除します。対象 Context と Namespace を再確認してから実行します。

```bash
kubectl config current-context
kubectl get all -n os-readiness-lab
kubectl delete namespace os-readiness-lab
```

## 公式リファレンス

- [Kubernetes: Debug Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/)
- [Kubernetes: Debug Services](https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/)
- [Kubernetes: Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

## 面談での説明例

> [!IMPORTANT]
> 演習完了・証跡記録後のみ、実施結果を過去形で説明します。現時点では本人レビュー・演習とも未実施です。

「現時点では、ImagePullBackOff、CrashLoopBackOff、Pending、Service、Ingress の障害調査演習資料を準備した段階で、本人によるクラスタ上の実施と証跡記録はまだ行っていません。実施後は、状態、Event、log、EndpointSlice など、実際に確認した事実だけを教材・検証経験として説明します。」
