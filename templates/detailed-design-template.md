# OpenShift 詳細設計書

> [!NOTE]
> 実務成果物例を組織の正式様式へ転記する際に使用する空テンプレートです。`<...>` は案件固有値へ置換し、承認済み標準と対象版の公式資料を優先してください。

> 基本設計の決定事項を、第三者が同じ設定へ再現できる粒度に落とす。値が環境依存なら `要確認` とする。

## 1. 文書・トレーサビリティ

| 項目 | 内容 |
|---|---|
| 案件／環境 | `<記入>` |
| 対応する基本設計版 | `<文書名・版>` |
| OpenShift／Operator 版 | `要確認` |
| 作成／レビュー | `<役割>` |

| 詳細設計 ID | 基本設計 ID | 試験 ID | 変更 ID |
|---|---|---|---|
| DD-001 | BD-001 | TS-001 | `<必要時>` |

## 2. リソース命名・ラベル

| 対象 | 規則 | 例 | 制約 |
|---|---|---|---|
| Project | `<組織>-<用途>-<環境>` | `sample-app-dev` | DNS ラベル準拠 |
| Label | `app.kubernetes.io/*` | `app.kubernetes.io/name: sample` | 必須キーは要確認 |
| Route／Service | `<規則>` | `<例>` | 重複確認 |

## 3. ノード・マシン設定

| 項目 | 対象 | 設定値 | 適用方法 | 再起動影響 |
|---|---|---|---|---|
| MachineConfigPool | `<worker/infra>` | `<selector>` | `<MachineConfig>` | `要確認` |
| taint／toleration | `<対象>` | `<key=value:effect>` | `<Manifest>` | `<影響>` |
| resource reservation | `<対象>` | `<値>` | `<KubeletConfig>` | `要確認` |

## 4. ネットワーク詳細

| 設定 ID | 対象 | 値 | 依存先 | 検証コマンド |
|---|---|---|---|---|
| NW-001 | API DNS | `<FQDN/IP>` | DNS/LB | `dig <FQDN>` |
| NW-002 | Ingress | `<domain/VIP>` | LB | `oc get ingresscontroller -n openshift-ingress-operator` |
| NW-003 | NetworkPolicy | `<namespace/通信要件>` | アプリ | `oc describe networkpolicy -n <ns>` |

### Firewall ルール

| ID | 送信元 | 宛先 | Protocol/Port | 方向 | 用途 | 所有者 |
|---|---|---|---|---|---|---|
| FW-001 | `<CIDR>` | `<FQDN/CIDR>` | `TCP/<port>` | `egress` | `<用途>` | `要確認` |

## 5. Project・ワークロード標準

```yaml
# 値は例。適用前に namespace と quota を要確認。
apiVersion: v1
kind: ResourceQuota
metadata:
  name: workload-quota
  namespace: <project-name>
spec:
  hard:
    requests.cpu: <value>
    requests.memory: <value>
    limits.cpu: <value>
    limits.memory: <value>
```

| 項目 | 設計値 |
|---|---|
| LimitRange | `<既定 request/limit>` |
| PodDisruptionBudget | `<対象・minAvailable>` |
| topology spread | `<zone/hostname 方針>` |
| probe | `<startup/readiness/liveness の基準>` |

## 6. RBAC・SCC

| Principal | Role／ClusterRole | Scope | 理由 | 承認 |
|---|---|---|---|---|
| `<group/serviceaccount>` | `<role>` | `<namespace>` | `<業務>` | `要確認` |

- カスタム SCC 名: `<不要なら「なし」>`
- 許可項目とリスク: `<runAsUser、volume、capability 等>`
- 標準 SCC で実現できない根拠: `<記入>`

## 7. Storage 詳細

| PVC 用途 | StorageClass | 容量 | AccessMode | VolumeMode | reclaimPolicy | 拡張 |
|---|---|---|---|---|---|---|
| `<用途>` | `<SC>` | `<Gi>` | `<RWO/RWX>` | `<Filesystem/Block>` | `<値>` | `要確認` |

## 8. Operator・監視・ログ

| 対象 | Namespace | Subscription channel | installPlanApproval | 設定 CR | 更新確認 |
|---|---|---|---|---|---|
| `<Operator>` | `<ns>` | `<channel>` | `<Automatic/Manual>` | `<kind/name>` | `要確認` |

| 監視項目 | 指標／条件 | 継続時間 | Severity | 通知先 | Runbook |
|---|---|---|---|---|---|
| `<対象>` | `<PromQL/条件>` | `<分>` | `<値>` | `<宛先>` | `<URL>` |

## 9. バックアップ・復元

- OADP 対象 Namespace／除外: `<記入>`
- VolumeSnapshot／ファイルバックアップ方式: `要確認`
- etcd バックアップの実施主体・保管・暗号化: `<記入>`
- 復元手順・定期試験 ID: `<記入>`

## 10. 実装・検証・差し戻し

| 順序 | Manifest／設定 | 前提 | 検証 | 失敗時 |
|---|---|---|---|---|
| 1 | `<Git path>` | `<前提>` | `oc diff -f <path>` | `<戻し方>` |

- 実値を含む Secret は Git に格納しない。
- API 廃止状況は対象バージョンの `oc api-resources` と公式文書で要確認。

## 11. レビュー記録

| 日付 | 観点 | 指摘 | 対応 | 状態 |
|---|---|---|---|---|
| `<日付>` | `セキュリティ／運用／可用性` | `<記入>` | `<記入>` | `Open` |
