# 試験文書テンプレートガイド（補助資料）

> [!IMPORTANT]
> 本書は [基盤案件の進め方](05-infra-project-process.md) と `templates/` の試験文書をつなぐ補助資料です。例示した試験項目は本人の実施結果ではありません。実施した試験だけを [証跡索引](../evidence/README.md) に登録します。

総合演習では、記入済みドラフトの [試験仕様書](../projects/enterprise-openshift-platform/docs/16-test-specification.md) と [試験結果報告書](../projects/enterprise-openshift-platform/docs/17-test-results.md) を対応例として読みます。全ケースの初期状態は `NOT RUN` であり、期待結果は実測結果ではありません。

## 試験の目的

試験は操作の実施記録ではなく、「設計が受入条件を満たす」という証拠を残す活動です。`templates/test-spec-template.md` で事前に判定可能な期待結果を定め、`templates/test-result-template.md` に実測値と証跡を残します。

## 仕様書と結果報告書の違い

| 文書 | 試験前に決めること | 試験後に残すこと |
| --- | --- | --- |
| 試験仕様書 | 目的、要件 ID、前提、手順、期待結果、判定基準、証跡、後処理 | 承認済みの試験条件を維持 |
| 試験結果報告書 | 対象版、期間、体制 | 実測結果、Pass / Fail / Blocked、差異、障害票、総合判定 |

## 試験レベル

### 単体試験

設定対象単位で設計どおりか確認します。

- Node と ClusterOperator が期待状態である。
- Project の Quota と RBAC が設計値どおりである。
- PVC が指定 StorageClass から払い出される。
- 監視ルールが読み込まれ、試験条件で発火する。

### 基盤結合試験

複数の基盤要素を通る経路・連携を確認します。

- client → DNS → LB → Ingress Controller / router（Route 設定を参照） → Service → Pod
- Pod → PVC → CSI → storage backend
- IdP → OAuth → group → RBAC → API
- backup controller → object storage → restore target

### システムテスト支援

アプリケーションや外部システムを含む業務シナリオで、基盤の状態、ログ、性能、障害時動作を確認します。業務機能の合否責任は体制表で明確にします。

## 良い試験項目の構造

```text
試験 ID: NET-RT-001
要件 / 設計 ID: REQ-NW-004 / BD-NW-007
目的: 外部利用者が HTTPS Route へ到達できること
前提: DNS と LB 設定済み、Pod Ready、試験元 IP は許可済み
操作: curl --fail-with-body --show-error --silent https://app.apps.example.com/healthz
期待結果: HTTP 200、応答本文 `ok`、Ingress access log に 1 件記録
証跡: コマンド出力、時刻、Route/Service/EndpointSlice、access log
後処理: なし
```

「画面が表示されること」だけでは、判定者やタイミングで結果が変わります。status code、状態値、件数、時間、許容差を定義します。

## 正常系・異常系・復旧系

| 区分 | 問い | 例 |
| --- | --- | --- |
| 正常系 | 正しい入力と正常構成で要件を満たすか | Route が 200 を返す |
| 異常系 | 依存先や入力が異常なとき安全に失敗するか | 権限のない user が変更を拒否される |
| 復旧系 | 異常除去後に想定時間で戻るか | Pod 再配置後に endpoint と応答が回復する |

破壊を伴う試験は、専用環境、影響評価、承認、監視、復旧手順を用意します。本番で無断実施しません。

## OpenShift の代表的な確認例

### 参照系の事前確認

```bash
oc whoami
oc version
oc get clusterversion
oc get clusteroperators
oc get nodes -o wide
oc get events -A --sort-by=.lastTimestamp
```

### アプリ経路

```bash
oc get route,service,endpointslice,pod -n <project>
oc describe route <route-name> -n <project>
curl --fail-with-body --show-error --verbose https://<route-host>/healthz
```

### Storage

```bash
oc get pvc,pv,storageclass -n <project>
oc describe pvc <pvc-name> -n <project>
oc get events -n <project> --sort-by=.lastTimestamp
```

コマンド結果は環境により異なります。利用可能な API、権限、プラグイン、対象リリースを事前に **要確認** とします。

## 証跡の扱い

- 環境名、対象バージョン、時刻、実行者、namespace を残す。
- 巨大なログを本文へ貼らず、改変防止された保存先と該当箇所を示す。
- Token、Secret、cookie、顧客名、内部 FQDN / IP は定められた方法でマスキングする。
- スクリーンショットだけにせず、機械可読な出力や export も残す。
- 証跡取得のために状態を変更しない。

## Fail と Blocked

- **Fail**: 手順を実行でき、実測結果が期待結果を満たさない。
- **Blocked**: 前提未成立、環境停止、権限不足などで判定まで進めない。
- **Not Run**: 未実施。
- **Pass with condition**: 組織の定義・承認がある場合だけ使用し、条件と期限を明記する。

Blocked を Pass に数えません。再試験では対象版、修正 ID、影響範囲、関連試験の再実施要否を記録します。

## 試験完了判定

- 必須項目がすべて実施され、未判定がない。
- Fail / Blocked に障害票、責任者、期限、リリース判断がある。
- 要件と試験の対応漏れがない。
- 証跡が第三者に追跡可能で、機密情報が保護されている。
- 構成差分と既知制約が結果報告に反映されている。
- 復旧・後処理が完了し、試験用権限やデータが残っていない。

## 面談での説明例

> [!NOTE]
> 次の文は回答の型です。本人が試験を計画・実施し、証跡を確認した後に、実際の事実だけを残して使用します。

> 商用 OpenShift の試験実施経験はありません。本リポジトリでは、要件と設計 ID から、前提、操作、期待結果、証跡、後処理を分ける試験演習を準備しています。本人の実施状況は証跡索引で明示します。実案件では組織の試験標準と対象環境の変更ルールを確認し、Fail と Blocked を区別して報告します。
