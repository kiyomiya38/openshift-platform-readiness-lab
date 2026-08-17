# Lab 02: OpenShift 基本リソース

> [!IMPORTANT]
> **状態:** 演習資料: 準備済み / 本人実施: 未実施 / 実機証跡: なし


Project、Deployment、Service、Route、ConfigMap、Secret、RBAC を小さな Web アプリで関連付け、SCC と Operator は安全な read-only 確認で学ぶラボです。商用構築を模したものではなく、教材用の最小構成です。

## 学習目標

- OpenShift の Project と Kubernetes Namespace の関係を説明できる。
- Route から Service、EndpointSlice、Pod までの経路を確認できる。
- ConfigMap と Secret の用途・取扱いの違いを説明できる。
- Role/RoleBinding と ServiceAccount の関係、SCC の適用結果を確認できる。
- Operator の Subscription、CSV、管理対象 CR の関係を説明できる。

## 前提条件

- 破棄可能な OpenShift 検証環境（CRC、または利用者に割り当て済み Project を使える Developer Sandbox。利用条件は要確認）
- `oc` が対象クラスタと対応する版であること（要確認）
- 通常環境では Project の作成・削除権限。Developer Sandbox では割り当て済み Project 内のリソース作成・削除権限
- `registry.k8s.io` への Image pull。非接続環境は承認済み mirror 上の同等 Image へ変更（要確認）

## 安全上の注意

- `oc whoami --show-server` で検証環境を確認する。
- `os-basic-lab` Project 以外を変更しない。
- `manifests/03-secret-training-only.yaml` の値は教材用文字列であり、実 Secret ではない。実案件では Git に平文を置かない。
- SCC を作成・変更・付与しない。特に `anyuid` や `privileged` を安易に付与しない。
- Operator は閲覧だけにし、共有クラスタへインストールしない。

Developer Sandbox など利用者ごとに Project が割り当てられる環境では、`manifests/00-project.yaml` を適用しません。作業用コピーを作り、`manifests/01-configmap.yaml` から `06-route.yaml` までにある **すべての** `namespace: os-basic-lab` を割り当て済み Project 名へ置換します。この README の `-n os-basic-lab` と `oc project os-basic-lab` も同じ Project 名へ読み替えます。共有の割り当て済み Project 自体は削除せず、後述の cleanup ではラボで作成した namespaced resource だけを削除します。

## セットアップ

Manifest を読み、Namespace、Image、権限範囲を確認してから適用します。

```bash
oc whoami
oc whoami --show-server
oc apply -f manifests/00-project.yaml
oc apply -f manifests/01-configmap.yaml
oc apply -f manifests/02-serviceaccount-rbac.yaml
oc apply -f manifests/03-secret-training-only.yaml
oc apply -f manifests/04-deployment.yaml
oc apply -f manifests/05-service.yaml
oc apply -f manifests/06-route.yaml
```

## Exercise 1: Project とリソース範囲

```bash
oc project os-basic-lab
oc get project os-basic-lab
oc api-resources --namespaced=true
oc get all -n os-basic-lab
```

`Project` が Namespace に OpenShift の管理情報を加えたリソースであること、クラスタスコープの Node とは範囲が異なることを説明します。

## Exercise 2: ConfigMap と Secret

```bash
oc get configmap lab-app-config -n os-basic-lab -o yaml
oc get secret lab-app-secret -n os-basic-lab
oc set env --list deployment/lab-web -n os-basic-lab
```

Secret の値を画面、履歴、証跡へ出力しません。Kubernetes Secret は base64 表現だけでは暗号化にならないため、etcd 暗号化、RBAC、外部 Secret 管理、ローテーションを案件で要確認とします。

## Exercise 3: Deployment から Pod

```bash
oc rollout status deployment/lab-web -n os-basic-lab --timeout=180s
oc get deployment,replicaset,pod -n os-basic-lab -l app=lab-web
oc describe deployment lab-web -n os-basic-lab
```

Deployment の selector と Pod label、ReplicaSet の世代、readiness/liveness probe、request/limit を確認します。教材では固定タグの `agnhost netexec` を HTTP サーバーとして起動します。`runAsUser` は固定せず restricted SCC に namespace 固有の非 0 UID 割当てを委ねており、商用環境では Image tag/digest と Registry mirror の方針を要確認とします。

## Exercise 4: Service と Route

```bash
oc get service lab-web -n os-basic-lab -o yaml
oc get endpointslice -n os-basic-lab -l kubernetes.io/service-name=lab-web
oc get route lab-web -n os-basic-lab
oc describe route lab-web -n os-basic-lab
```

Route host を取得し、検証端末から到達できる環境のみ HTTP 疎通を確認します。証明書、DNS、Proxy、外部 Firewall は環境依存です（要確認）。

```bash
oc get route lab-web -n os-basic-lab -o jsonpath='{.spec.host}{"\n"}'
```

## Exercise 5: RBAC と SCC

```bash
oc get serviceaccount lab-viewer -n os-basic-lab
oc get role lab-resource-reader -n os-basic-lab -o yaml
oc get rolebinding lab-resource-reader -n os-basic-lab -o yaml
oc get pod -n os-basic-lab -l app=lab-web -o jsonpath='{range .items[*]}{.metadata.name}{" SCC="}{.metadata.annotations.openshift\.io/scc}{" UID="}{.spec.containers[0].securityContext.runAsUser}{"\n"}{end}'
oc auth can-i create pods -n os-basic-lab
```

`lab-viewer` へは read-only の namespaced Role だけを結びます。`--as` を使う確認には impersonate 権限が必要な場合があるため、管理者の許可なしに実行しません。

## Exercise 6: Operator の read-only 観察

権限がある場合のみ、既存 Operator を観察します。

```bash
oc get subscription -A
oc get clusterserviceversion -A
oc get installplan -A
oc get clusteroperator
```

OLM Operator の `Subscription → InstallPlan → CSV → CR/Operand` と、OpenShift 本体の `ClusterOperator` は同じものではない点を整理します。表示できない場合は権限不足を結果として記録し、権限拡大はしません。

## 検証

```bash
oc get deployment/lab-web -n os-basic-lab
oc get pod -n os-basic-lab -l app=lab-web
oc get endpointslice -n os-basic-lab -l kubernetes.io/service-name=lab-web
oc get route/lab-web -n os-basic-lab
oc get events -n os-basic-lab --sort-by=.lastTimestamp
```

期待する関連は [answers/resource-flow.md](answers/resource-flow.md)、合格条件は [answers/verification-checklist.md](answers/verification-checklist.md) と比較します。

## クリーンアップ

次は専用 Project 内の全教材リソースを削除します。対象クラスタと Project を再確認します。

```bash
oc whoami --show-server
oc get all -n os-basic-lab
oc delete project os-basic-lab
```

Developer Sandbox の割り当て済み Project を使用した場合は、上の `oc delete project` を実行しません。namespace を置換した作業用 manifest の `01-configmap.yaml` から `06-route.yaml` を `oc delete -f` で削除し、Project は残します。

## 公式リファレンス

- [OpenShift: Projects](https://docs.redhat.com/en/documentation/openshift_container_platform/)
- [OpenShift: Routes](https://docs.redhat.com/en/documentation/openshift_container_platform/)
- [OpenShift: Authentication and authorization](https://docs.redhat.com/en/documentation/openshift_container_platform/)
- [Kubernetes: ConfigMaps and Secrets](https://kubernetes.io/docs/concepts/configuration/)

## 面談での説明例

> [!IMPORTANT]
> 演習完了・証跡記録後のみ、実施結果を過去形で説明します。現時点では本人レビュー・演習とも未実施です。

「現時点では、Project、Deployment、Service、Route、ConfigMap、教材用 Secret、RBAC、SCC、Operator を確認する演習資料を準備した段階で、本人による OpenShift 環境での適用・確認と証跡記録はまだ行っていません。実施後は、利用した環境、権限、作成・参照したリソース、実測結果を限定して説明します。」
