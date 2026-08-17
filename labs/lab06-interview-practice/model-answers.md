# モデル回答

> [!IMPORTANT]
> **現在の状態（v0.1）**: このリポジトリのラボは未実施で、本人レビューも未完了です。以下は完成後の話し方を示す回答テンプレートであり、現在の技能・実施実績を表す回答ではありません。過去の経験と、実際に完了して証跡を残した演習だけに書き換えて使用してください。

## Q01. 1 分自己紹介

「インフラ講師、教材作成、学習・検証環境の構築を中心に、Linux、ネットワーク、AWS、Kubernetes の基礎を扱ってきました。OpenShift の商用設計・構築経験はありません。Developer Sandbox / CRC を用いた入門学習と社内向け教材作成で、Project、Route、Operator などを扱った範囲です。今回の資料とラボはこれから本人レビュー・演習を行い、完了した内容だけを証跡付きで説明します。」

## Q02. OpenShift の商用構築経験

「商用 OpenShift の設計・構築経験はありません。過去に Developer Sandbox / CRC を用いた入門学習と社内向け教材作成で、Project、Route、Operator などを扱いました。設計、試験、障害調査の資料初稿は準備されていますが、まだ本人レビュー・演習前です。」

## Q03. Kubernetes の経験

「Kubernetes は、教材作成と学習・検証環境の構築で、`kubeadm` と基本リソースを扱った経験があります。受講者がつまずきやすい点や質問を調査して講師間で共有しましたが、商用クラスタの運用や本番障害対応の経験ではありません。今回の障害再現ラボは未実施です。」

## Q04. OpenShift を触った範囲

「過去に Developer Sandbox / CRC を用いた入門学習と社内向け教材作成で、Project、Route、Operator などを扱いました。今回用意した基本リソースラボはまだ実施していないため、Manifest 適用や SCC の確認を実施済みとは説明しません。実施後に、環境、操作、結果を証跡へ記録して回答を更新します。」

## Q05. 担当可能範囲

「現時点では、既存手順と reviewer がある環境での Manifest 作成補助、検証、証跡整理、一次切り分け、設計書の観点整理が現実的です。Sizing、cluster-wide network/storage、production migration、重大障害の判断は支援が必要です。最初に構成・運用標準・責任分界を確認し、非本番で再現してから review を受けます。」

## Q06. Kubernetes と OpenShift

「Kubernetes は Container workload を宣言的に管理する基盤です。OpenShift は Kubernetes を中核に、Route、SCC、Operator/OLM、integrated monitoring、authentication、console など企業利用向けの機能と lifecycle を組み合わせた platform と理解しています。具体的な同梱機能は対象 version で要確認です。」

## Q07. OpenShift 基本設計

「Cluster topology と node sizing、DNS/LB、Machine/Pod/Service network、Ingress/Route、IdP/RBAC/SCC、certificate、CSI/StorageClass、Registry、Operator、monitoring/logging、backup、update、operation、test を決めます。値だけでなく、要件の根拠、外部 team との責任分界、未決事項の owner も残します。」

## Q08. 詳細設計と構築手順

「詳細設計は resource 名、parameter、label/selector、port、RBAC rule、StorageClass など再現可能な完成状態を定義します。構築手順は、その状態へ安全に到達する順序、前提、command、期待結果、中止条件、rollback、証跡を定義します。同じ設計でも自動化方式により手順は変わります。」

## Q09. RBAC と SCC

「RBAC は user、group、ServiceAccount が Kubernetes API resource に何をできるかを制御します。SCC は OpenShift で Pod がどの security context、user ID、capability、volume などで admission されるかを制御します。まず標準 SCC と最小権限を使い、カスタム付与は理由と対象を review します。」

## Q10. Route の調査

「まず影響範囲、DNS、TLS、HTTP status を分けます。次に Route の Admitted condition と host/targetPort、Service の selector/port、EndpointSlice、Pod readiness/listen port を順に確認します。Cluster 外部なら Ingress Controller、LB、Firewall、certificate も確認します。変更前に event と設定差分を保存します。」

## Q11. OpenShift Virtualization

「OpenShift 上で Container と VM を共通 API・運用基盤から管理する機能です。KubeVirt を中心に VirtualMachine/VirtualMachineInstance を扱い、実行時は virt-launcher Pod 内の QEMU が node の KVM を利用すると理解しています。Disk は DataVolume/PVC、追加 network は Multus/NAD などを使います。私は概要理解と設計演習レベルで、導入経験はありません。」

## Q12. KVM 経験

「商用 KVM 構築経験はありません。KVM が Linux kernel の virtualization 機能、QEMU が device emulation と VM process、libvirt が管理 API/tooling を提供する関係は理解しています。OpenShift Virtualization では通常 KVM host を直接 `virsh` 管理せず、OpenShift/KubeVirt の resource と Operator を通して管理する認識です。」

## Q13. VM 移行の論点

「対象 VM の OS/support、CPU/memory、disk 容量と性能、network/VLAN/固定 IP、driver/agent、license、依存先、停止時間、RPO/RTO を棚卸しします。移行先の StorageClass、Multus、backup、monitoring、capacity と mapping し、代表 VM で PoC と rollback rehearsal をします。MTV の対応 source/target は対象版で要確認です。」

## Q14. Mirror Registry

「Disconnected cluster は外部 Registry から release、Operator、application Image を直接 pull できないため、接続可能 zone で承認済み Image を内部 Mirror Registry へ同期し、cluster の参照先を内部へ向けます。Registry の TLS、容量、backup、catalog、pull secret、DNS/NTP/LB、定期更新まで設計対象です。私は設計観点の学習レベルです。」

## Q15. Ansible

「Ansible は基礎理解の範囲で、商用案件での実装・運用経験はありません。今回、Inventory、Playbook、package、template、handler、冪等性を確認する演習資料が用意されていますが、まだ実行していません。破棄可能な VM で check/diff と複数回実行の結果を記録した後に、検証できた範囲を回答へ加えます。」

## Q16. Pod が起動しない

「最初に cluster/context、namespace、影響範囲と Pod status を確認します。`get` と `describe` で Container state/reason、schedule、probe、mount、Event を見て、起動した Container なら current/previous log を確認します。Image、command/config、resource、PVC、SCC のどの層か仮説を立て、一つずつ反証します。」

## Q17. PVC が Bound にならない

「PVC の request、StorageClass、AccessMode、capacity と Event を確認します。次に StorageClass の存在、provisioner、binding mode、selected node、CSI controller/node Pod、backend 容量・認証・network を順に見ます。まず `Pending` の層を特定し、根拠なく PVC/PV を削除しません。」

## Q18. Operator 異常

「OpenShift 本体の ClusterOperator か、OLM Operator かを最初に分けます。OLM なら Subscription、InstallPlan、CSV、Operator Deployment/Pod log、管理対象 CR status、Operand の順に確認します。Version/channel、直前 update、catalog/Image pull、RBAC、webhook/certificate も候補です。再導入は影響を確認してからです。」

## Q19. Node NotReady

「まず node condition、taint、event と影響 workload を確認し、複数 node か一台かを分けます。次に resource pressure、kubelet、CRI、network/CNI、certificate、MachineConfig の状態を見ます。OpenShift node への直接 service 操作や reboot は、保守手順、workload eviction、quorum、承認を確認して行います。」

## Q20. Kong、Sysdig、OpenShift AI

「いずれも商用導入経験はなく概要理解レベルです。Kong は API Gateway として routing、authentication、rate limit、observability を担い、Sysdig は Container/Kubernetes の runtime visibility、security、monitoring に関係します。OpenShift AI は model development/serving/pipeline などの AI platform 機能です。案件では edition/version と責任分界を先に確認します。」

## Q21. OpenShift AI と Dify

「OpenShift AI は OpenShift 上で notebook、pipeline、model serving、model lifecycle などを提供する platform layer、Dify は LLM application、workflow、RAG、agent を組み立てる application layer と理解しています。OpenShift AI 上の model endpoint を Dify から使う構成は考えられますが、product support、authentication、data governance は要確認です。」

## Q22. AI 利用時の注意

「最初に顧客と組織の AI 利用可否を確認し、顧客名、個人情報、host/IP、log、configuration、Secret を入力しません。必要な事象は匿名化・一般化します。出力は仮説として扱い、対象 version の公式文書、実環境の read-only 確認、非本番試験、人の review で検証してから変更管理へ載せます。」

## Q23. 未経験製品への対応

「未経験であることを最初に伝えます。その上で、責任分界、対象 version、既存設計、support matrix、Runbook を確認し、非本番で最小構成を再現します。既知の Linux/network/Kubernetes の層へ分解し、review が必要な判断を明示して、手順・試験・証跡から担当範囲を広げます。」

## Q24. NG 表現の修正

| NG | 誤認を避ける例 |
|---|---|
| OpenShift の構築はできます | 「商用構築経験はありません。過去の入門学習範囲と、今後の検証計画を分けて説明します」 |
| 障害対応は得意です | 「商用インシデント対応経験はありません。教材・講義でのつまずき調査経験があり、今回の再現演習は未実施です」 |
| Airgap は分かります | 「実構築経験はなく、資料初稿をこれから確認する段階です。確認後も概要理解と実機検証を分けて説明します」 |
| AI で調べれば対応できます | 「AI は匿名化した一般調査の補助に限定し、公式文書、対象環境、非本番試験、人の review で最終確認します」 |
