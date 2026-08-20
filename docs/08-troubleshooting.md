# 08. 障害調査

## 目的

申告された症状を再現可能な事実へ変換し、依存関係とレイヤーに沿って切り分け、影響抑制、復旧、原因分析、恒久対策まで記録する方法を理解します。コマンドを順に試すのではなく、仮説と観測結果で調査範囲を狭めます。

## 実務での使用場面

- アラート、利用者申告、変更失敗からIncidentを開始する
- クラスタ、node、network、storage、workload、外部サービスを切り分ける
- 証拠を保全しながら暫定復旧を行う
- Red Hat Supportや製品ベンダーへ必要情報を渡す
- 原因、再発防止、監視・Runbook・設計の改善を管理する

## 入力

- 発生・検知日時、申告者、対象、症状、影響、直前変更
- 正常時baselineと期待構成
- ClusterOperator、node、MCP、event、Pod、route、PVCなどの状態
- DNS、NTP、LB、Proxy、Firewall、IdP、Storageの外部観測
- 監視metric、alert、log、audit、作業・変更履歴

## 判断

### 最初の事実化

「OpenShiftが遅い」をそのまま調査項目にしません。誰が、どこから、どのURL/APIへ、いつから、どの程度、常時または断続的に、どんなerrorで失敗するかを記録します。影響範囲と重大度を決め、変更停止やエスカレーションが必要か判断します。

### レイヤー別の切り分け

| レイヤー | 代表観測 | 主な依存 |
| --- | --- | --- |
| 利用経路 | client DNS、TLS、HTTP status、応答時間 | 外部DNS、FW、LB、Ingress |
| OpenShift公開 | Route、IngressController、router Pod | Service、EndpointSlice、certificate |
| Workload | Deployment、Pod、event、probe、resource | image、ConfigMap、Secret、quota、policy |
| Cluster service | ClusterOperator、API、authentication、DNS | control plane、etcd、network |
| Node | Ready、condition、MCP、disk、time | hardware、RHCOS、NTP、network |
| Storage | PVC/PV、CSI、attachment、latency | storage system、network、credential |
| 外部基盤 | DNS、NTP、LB、Proxy、IdP、registry | 他teamと責任分界 |

上位から症状を確認し、下位へ一段ずつ降ります。仮説を立てたら「仮説が正しければ何が観測されるか」を先に決め、結果を記録します。無関係な再起動や設定変更は原因証拠を失い、影響を拡大するため避けます。

### 復旧と原因分析の分離

サービス復旧を優先する場合でも、変更前に時刻、状態、event、log、関連設定を保全します。暫定復旧は原因確定ではありません。復旧後にtimeline、直接原因、寄与要因、検知不足、再発防止、owner、期限を整理します。

## 成果物例の読み方

[監視・ログ・運用設計](../projects/enterprise-openshift-platform/docs/10-observability-operations-design.md)のalert catalogとIncident flowから、検知条件、通知先、Runbookを確認します。[運用引き継ぎ書](../projects/enterprise-openshift-platform/docs/20-operations-handover.md)には、次の代表Runbookがあります。

- ClusterOperator `Degraded`
- Node `NotReady`／hardware failure
- API／Ingress unavailable
- PVC `Pending`／storage latency
- Internal Image Registry unavailable
- VM停止／migration failure
- backup failure／restore request
- certificate expiry／authentication failure

Runbookは原因を一つに決め打ちする文書ではありません。入口条件、影響確認、安全な観測、分岐、暫定回避、エスカレーション、完了条件を示すものとして読みます。

[インシデント記録](../evidence/incident-record.md)は、申告、事実、timeline、仮説、操作、結果、復旧、原因、改善を分離する汎用様式です。机上で想定した内容は実障害記録とせず、`机上例`と明記します。

## 他文書とのつながり

- 正常時の期待値は[基本設計](../projects/enterprise-openshift-platform/docs/05-basic-design.md)、[詳細設計](../projects/enterprise-openshift-platform/docs/13-detailed-design.md)、[試験結果](../projects/enterprise-openshift-platform/docs/17-test-results.md)から取得する
- 直前変更は[変更管理台帳](../projects/enterprise-openshift-platform/docs/22-change-register.md)から確認する
- 既知問題は[課題・リスク管理台帳](../projects/enterprise-openshift-platform/docs/21-issue-risk-register.md)と照合する
- 恒久対策が設計を変える場合はADR、設計、手順、試験を更新する
- 新しい検知条件や切り分け手順は監視設計とRunbookへ反映する

## レビューで指摘されやすい点

- 症状、影響、原因、対処を混ぜて記録する
- 発生時刻、timezone、対象version、直前変更がない
- 仮説と事実を区別せず、最初のerrorを根本原因と断定する
- 影響と承認を確認せず、Pod削除、node再起動、Operator再導入を行う
- `oc adm must-gather`だけを取得し、利用経路や外部依存の証跡を残さない
- Secret、token、証明書秘密鍵、個人情報をlog bundleに含める
- 暫定復旧後に原因・再発防止・監視改善を閉じない

## 公式一次資料

- [OpenShift 4.22 Support](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/support/)
- [Gathering cluster data](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/support/gathering-cluster-data)
- [OpenShift 4.22 Nodes](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/nodes/)
- [OpenShift 4.22 Networking](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/networking_overview/)

切り分けの補足は[Kubernetes障害調査](../reference/technical/kubernetes-troubleshooting.md)と[OpenShift障害調査](../reference/technical/openshift-troubleshooting.md)を参照してください。

## 次に読む章

[09. Virtualization・VM移行](09-virtualization-migration.md)へ進みます。
