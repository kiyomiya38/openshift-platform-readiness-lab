# 安全なサンプルワークロード

`example-web-dev` Namespaceへ非特権のHTTPサンプルを配置する、一般公開用のManifest例です。クラスタがないため適用・server-side validation・動作確認は**未実施**です。

主な安全策は次のとおりです。

- 専用Namespace、ResourceQuota、LimitRangeで影響範囲を限定
- 専用ServiceAccountのtokenを自動mountしない
- `restricted`相当のPod security context、capability全削除、非root実行
- Namespace単位のRoleによる閲覧権限だけを架空グループへ付与
- deny-by-defaultのNetworkPolicyへ、同一Namespace、OpenShift Ingress、DNSだけを加算
- 2 replicaをhostname単位に分散し、readiness/liveness/startup probeとPodDisruptionBudgetを設定
- Secret、永続データ、外部データベース接続を含めない

実環境では、イメージを承認済みRegistryのdigestへ固定し、Routeの証明書方針、NetworkPolicy、quota、利用グループをレビューします。

PodDisruptionBudgetはdrainなどの自発的disruptionを制御するもので、突然のnode障害時にPodやサービスの継続を保証するものではありません。topology spread、十分なnode余力、probe、実際の障害試験と組み合わせて評価します。

```bash
oc kustomize manifests/
oc apply --dry-run=server -k manifests/
oc diff -k manifests/
# 変更承認後だけ実適用する
oc apply -k manifests/
```

削除は影響を確認した上で `oc delete -k manifests/` とします。Namespace削除は配下の全リソースを失うため、個別の変更承認なしに実行しません。
