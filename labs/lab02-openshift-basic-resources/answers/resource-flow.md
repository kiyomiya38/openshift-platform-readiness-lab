# リソース関係の解答例

```text
利用者
  -> Route lab-web (TLS edge termination)
  -> Service lab-web:8080
  -> EndpointSlice（selector app=lab-web で Pod IP を登録）
  -> Deployment が管理する Pod の agnhost netexec（containerPort 8080）

Deployment
  -> ConfigMap lab-app-config を環境変数として参照
  -> Secret lab-app-secret を環境変数として参照（値は表示しない）

ServiceAccount lab-viewer
  -> RoleBinding lab-resource-reader
  -> Role lab-resource-reader（os-basic-lab 内の read-only 権限）
```

`lab-viewer` は教材アプリ Pod に割り当てていません。RBAC の主体として分離して観察するためです。アプリ Pod は token を必要としないため `automountServiceAccountToken: false` としています。

アプリコンテナは固定タグの HTTP テストイメージを使い、`runAsUser` は Manifest で固定していません。restricted SCC が Project の UID range から非 0 UID を割り当てる前提です。
