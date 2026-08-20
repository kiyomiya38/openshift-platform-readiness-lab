# 面談回答ガイド（自己確認用補助資料）

> [!IMPORTANT]
> 本書の回答例は本人の実績証明ではなく、[現在地のスキル整理](02-current-skill-position.md) と [証跡索引](../evidence/README.md) を基に回答を組み立てるための補助資料です。詳細レビュー前の回答を暗記・提出せず、利用した環境、実施内容、未経験範囲と一致する文だけを使用します。

## 回答の基本形

技術面談では、最初の一文で結論を出し、その後に事実と境界を添えます。

```text
1. 結論: 経験の有無または技術の要点
2. 具体: 実際に行った教材・検証と成果物
3. 境界: 未経験の範囲、本番との差
4. 行動: 案件での確認方法、レビュー、次の学習
```

「知っていますか」には定義だけで終わらず、案件で重要な観点を一つ加えます。「経験がありますか」には知識の説明から逃げず、経験の事実を最初に答えます。

## 主要質問と短い回答

### OpenShift の商用構築経験はありますか

> ありません。OpenShift は入門教材と検証環境で、Project、Route、RBAC、SCC、Operator、`oc` の基本を確認した段階です。商用案件の設計・構築・運用は未経験なので、参画時は対象バージョン、設計書、変更手順を確認し、有識者レビューの下で担当します。

### Kubernetes はどの程度経験がありますか

> `kubeadm` を用いた検証環境の構築と教材作成を行い、Pod、Deployment、Service、Ingress、ConfigMap、Secret、CNI、`kubectl` の基本調査を扱いました。商用クラスタの設計・運用経験はありません。資格学習で得た知識と、検証で再現した内容を分けて説明します。

### OpenShift はどこまで触りましたか

> 実際に利用した検証環境で、Project、Deployment、Service、Route、ConfigMap、Secret、RBAC の参照・作成と、SCC・Operator の基本確認を行ったレベルです。クラスタ新規構築、upgrade、重大障害対応は未経験です。

### Kubernetes と OpenShift の違いは何ですか

> OpenShift は Kubernetes を基盤に、Operator による platform component 管理、Route、統合された認証・監視・Web Console、RHCOS などを組み合わせた enterprise container platform と理解しています。具体的な既定値や対応機能は対象リリースで確認します。

### 基本設計では何を決めますか

> 要件を満たす方式と責任分界を決めます。OpenShift では cluster / node 構成、DNS、LB、network、認証・RBAC、storage、Operator、監視・logging、backup、証明書、運用方式などです。詳細設計ではそれを設定値と resource 定義へ落とします。

### 詳細設計と構築手順書の違いは何ですか

> 詳細設計は何をどの値で構成するかを定義し、構築手順書は承認済み設計を誰が再現しても同じ結果にする操作、期待結果、中断基準、切り戻しを定義します。

### OpenShift Virtualization とは何ですか

> OpenShift 上で container に加えて VM の lifecycle を Kubernetes API で管理する機能です。KubeVirt と KVM / QEMU を基盤に、VirtualMachine、VMI、DataVolume、PVC などを扱います。私は実導入経験はなく、概要理解と設計観点の学習段階です。

### 既存 VM をすぐ container 化できない場合はどうしますか

> OS、middleware、vendor support、改修費、期限などで直ちに container 化できない場合があります。まず VM として OpenShift Virtualization へ移し、container と同じ platform で運用しながら、適合する workload を段階的に modernize する選択肢を評価します。

### KVM 経験はありますか

> 商用経験はありません。KVM が Linux kernel の virtualization 機能、QEMU が device emulation と VM process、libvirt が管理 API、KubeVirt が Kubernetes API へ VM lifecycle を統合する関係を学習しています。構築を任された場合は support 条件と hardware virtualization を確認します。

### VMware から OpenShift Virtualization へ移す論点は何ですか

> VM 数だけでなく、guest OS、CPU / memory、disk、snapshot、network / VLAN、固定 IP、停止可能時間、依存関係、backup、license、移行後性能を棚卸しします。PoC と rehearsal で方式を検証し、Go / No-Go と切り戻し条件を先に決めます。

### Airgap 環境での構築経験はありますか

> ありません。概要理解レベルです。cluster が必要な external registry へ直接接続できないため、release image、Operator catalog、application image を mirror registry へ同期し、certificate、DNS、Firewall、NTP、LB、更新媒体まで設計する必要があると理解しています。

### なぜ Mirror Registry が必要ですか

> 非接続 cluster が release、Operator、application の image を internal endpoint から取得できるようにするためです。初回同期だけでなく、更新頻度、差分、容量、署名・digest、certificate、障害復旧、媒体搬送を運用として設計します。

### Ansible 経験はありますか

> 教材・検証レベルです。Inventory、Playbook、variable、template、handler、冪等性を学び、package、service、firewalld、chrony 等の小さな例を整理しています。大規模な本番 automation や AAP 運用経験はありません。

### Pod が起動しない場合はどう調べますか

> まず namespace、Pod status、container state、event を確認し、`describe`、current / previous log、resource request、image、volume、probe、node の順に仮説を絞ります。いきなり再起動せず、時刻と観測事実を残し、変更が必要なら影響と rollback を確認します。

### Route でアクセスできない場合はどう調べますか

> client 側の DNS、TCP / TLS、external LB から始め、Ingress Controller、Route、Service、EndpointSlice、Pod readiness と application listen port まで層を分けます。同じ hostname でも内部と外部で結果が違うかを確認し、network 担当との境界を明確にします。

### Operator 異常はどう見ますか

> まず影響範囲と変更履歴を確認し、ClusterOperator や ClusterServiceVersion、Subscription、InstallPlan、Operator Pod、event、log、catalog source の状態を見ます。Operator 管理 resource を直接直す前に、管理主体と公式の recovery 手順を確認します。

### PVC が Bound にならない場合はどう見ますか

> PVC event、StorageClass、access mode、request 容量、provisioner / CSI controller、backend 容量と credential、topology、node condition を確認します。`WaitForFirstConsumer` の場合は Pod scheduling と一緒に見ます。

### 監視・ログ・バックアップ経験はありますか

> 商用 OpenShift での運用経験はありません。Prometheus、Grafana、Alertmanager、logging、OADP / Velero、etcd backup の役割と設計観点を教材で整理しています。実案件では RPO / RTO、保持、通知先、復元試験まで確認します。

### Kong とは何ですか

> API Gateway として reverse proxy、routing、authentication / authorization、rate limit、request / response 処理、logging などの横断機能を提供する製品です。AI Gateway では model routing、token、latency、cost、guardrail の観測・制御も論点になります。導入経験はなく概要理解です。

### Sysdig とは何ですか

> container / Kubernetes の observability と security を扱う製品群です。image scanning、runtime detection、Kubernetes audit、Falco 由来の rule、agent などが論点です。Prometheus / Grafana や RHACS と目的・責任範囲を比較します。導入経験はありません。

### OpenShift AI とは何ですか

> data scientist と ML engineer が Workbench、pipeline、model serving、registry、model governance などを使うための OpenShift 上の AI / MLOps platform です。製品機能は version / subscription で確認が必要です。私は概要理解レベルです。

### Dify と OpenShift AI の違いは何ですか

> OpenShift AI は model と data science workload を企業基盤として動かす土台、Dify は RAG、workflow、agent など AI application を組み立てる上位 layer の一例と整理しています。OpenShift AI 上で serve した model endpoint を Dify から呼ぶ構成も考えられますが、認証、network、audit、data governance が別途必要です。

### AI を障害調査に使うときの注意は何ですか

> 顧客情報、個人情報、IP、hostname、log、configuration、credential を無断で入力しません。承認された AI と利用規程を確認し、一般化・masking した論点に限定します。回答は仮説として扱い、最終判断は対象版の公式資料、実機、担当者レビューで行います。

### トラブルシューティング資料作成は現場経験ですか

> 現場の商用障害対応経験ではありません。教材作成中に意図的な障害を再現し、症状、event、log、仮説、切り分け、復旧、再発防止を文書化した経験です。そこから得た調査の型は説明できますが、本番責任を負った実績とは分けます。

### 今の自分の対応可能範囲はどこですか

> 既存設計・手順とレビュー体制がある前提で、情報整理、検証、手順・試験項目の作成補助、参照系の初動調査から対応します。cluster 全体へ影響する設計判断や本番変更は単独で行わず、責任者へ確認します。

## 回答を自分用に直すチェック

- 実際に利用していない製品・環境を削除したか。
- 「作りました」「確認しました」に成果物または観測事実があるか。
- 商用、本番、顧客、運用という語を事実以上に使っていないか。
- 質問へ最初の一文で答えているか。
- 未経験だけで終わらず、理解している観点と安全な進め方を述べたか。

## 面談での説明例

> [!NOTE]
> 次の文は回答方針の例です。本人確認後にのみ使用し、完了していない学習や演習を過去形で説明しません。

> 回答では、最初に経験の有無を明確にし、次に教材・検証で実際に行ったこと、未経験の境界、案件での確認方法を話します。分からない仕様は推測で断定せず、対象バージョンと環境条件を確認すると答えます。
