# 用語集

> [!NOTE]
> 本資料は、インフラ経験者が実務成果物を読み解くための技術リファレンスです。OpenShift に関する構成とコマンドは OpenShift Container Platform 4.22 を具体例とします。実環境へ適用する前に、対象 z-stream、プラットフォーム、権限、変更手順、製品間の互換性、サポート条件を公式資料と組織標準で確認してください。

この用語集は暗記用ではなく、会話で略語を誤解しないための入口です。製品仕様、対応バージョン、サポート条件は対象環境の公式資料で **要確認** です。

## A–C

| 用語 | 短い説明 |
| --- | --- |
| AAP | Ansible Automation Platform。自動化コンテンツの実行・管理・統制を支援する Red Hat 製品群 |
| ACM | Advanced Cluster Management for Kubernetes。複数クラスタの管理・ポリシー等を扱う製品 |
| AI Gateway | LLM 等の AI API に対し、routing、認証、制限、観測、コスト制御などを集約する層 |
| Airgap | 外部ネットワークから物理的・論理的に強く隔離された環境。案件では接続条件を具体的に定義する |
| Alertmanager | Prometheus 系アラートの grouping、routing、silence、通知を扱うコンポーネント |
| ARO | Azure Red Hat OpenShift。Microsoft と Red Hat が提供する Azure 上の managed OpenShift サービス |
| BuildConfig | OpenShift で build の入力、strategy、出力等を表す API。利用可否・推奨方式は対象版で要確認 |
| CNI | Container Network Interface。コンテナ network plugin の標準的な仕様 |
| ClusterOperator | OpenShift の主要プラットフォーム機能の状態を表す cluster-scoped resource |
| CR / CRD | Custom Resource / CustomResourceDefinition。Kubernetes API を拡張する resource とその schema |
| CRI-O | Kubernetes 向けの OCI container runtime。OpenShift の node で利用される |
| CSI | Container Storage Interface。storage system と container orchestrator の連携仕様 |

## D–I

| 用語 | 短い説明 |
| --- | --- |
| DataVolume | CDI を介した VM disk 用 data の import、clone、upload 等を表現する resource |
| Disconnected | cluster が必要な外部 source へ直接接続できない運用形態。完全隔離とは限らないため経路を明記する |
| etcd | Kubernetes API の cluster state を保持する分散 key-value store |
| GitOps | Git 等の version control を望ましい状態の source とし、継続的に同期する運用手法 |
| GPU Operator | GPU driver や周辺 component の導入・管理を自動化する Operator。対応表を要確認 |
| HAProxy | L4 / L7 proxy・load balancer。OpenShift 外部 LB や routing の文脈で使われることがある |
| HPA | HorizontalPodAutoscaler。観測 metric に基づき workload replica 数を調整する resource |
| IdP | Identity Provider。user authentication を提供する system |
| ImageStream | OpenShift が container image と tag の参照・変更を追跡するための API |
| Ingress | cluster 外から service への HTTP(S) routing を宣言する Kubernetes API |
| Ingress Controller | Ingress / Route の設定を実装し、外部 traffic を受ける controller / data plane |

## K–M

| 用語 | 短い説明 |
| --- | --- |
| KubeVirt | Kubernetes 上で VM を宣言・実行するための open source project |
| KVM | Linux kernel の virtualization 機能。hardware virtualization extension を利用する |
| Loki | label を使って log を保存・検索する log system |
| MachineConfig | OpenShift node OS の設定を宣言する resource |
| MachineConfigPool | 同じ MachineConfig 群を適用する node の集合と rollout state |
| MetalLB | bare-metal 等の Kubernetes で LoadBalancer service を実現する実装の一つ |
| MLOps | model の開発、検証、deploy、monitoring、governance を継続的に運用する考え方 |
| MTV | Migration Toolkit for Virtualization。source virtualization platform から OpenShift Virtualization への VM migration を支援する toolkit |
| Multus | Pod / VM に複数 network interface を接続するための meta CNI |

## N–R

| 用語 | 短い説明 |
| --- | --- |
| Namespace | namespaced resource を分離・整理する Kubernetes の scope |
| NetworkPolicy | 選択した Pod に対する ingress / egress traffic の許可を宣言する API。実装能力は CNI に依存 |
| OADP | OpenShift APIs for Data Protection。アプリ resource と persistent data の保護を支援する Operator / 機能群 |
| OCP | OpenShift Container Platform |
| ODF | OpenShift Data Foundation。OpenShift 上で data service / software-defined storage を提供する製品 |
| OLM | Operator Lifecycle Manager。Operator の install、dependency、upgrade lifecycle を管理する仕組み |
| Operator | Kubernetes API と controller を使い、application / platform component の運用知識を自動化する pattern と実装 |
| OperatorHub | 利用可能な Operator を検索・導入する catalog UI / experience |
| OVN-Kubernetes | OpenShift で利用される Kubernetes network implementation。既定・機能は対象版で要確認 |
| PV / PVC | PersistentVolume / PersistentVolumeClaim。永続 volume と利用側からの要求 |
| Project | OpenShift が Namespace に annotation や access-management の使い勝手を加えた概念・resource |
| QEMU | machine emulator / virtualizer。KVM と組み合わせて VM を実行することが多い |
| RAG | Retrieval-Augmented Generation。検索した外部 knowledge を prompt context として生成へ利用する方式 |
| RBAC | Role-Based Access Control。Role と binding で権限を制御する方式 |
| RHCOS | Red Hat Enterprise Linux CoreOS。OpenShift node 向けに管理される immutable 志向の OS |
| RHACS | Red Hat Advanced Cluster Security for Kubernetes。workload / image / runtime 等の security を扱う製品 |
| ROSA | Red Hat OpenShift Service on AWS |
| Route | OpenShift 固有の HTTP(S) 公開 API。host と service の routing、TLS termination 等を表現 |
| RPO | Recovery Point Objective。障害時に許容する data loss の時点・量 |
| RTO | Recovery Time Objective。service を復旧させる目標時間 |

## S–Z

| 用語 | 短い説明 |
| --- | --- |
| SCC | Security Context Constraints。OpenShift で Pod が要求できる privilege / security context を制御する仕組み |
| Service | 変動する Pod 集合に安定した network endpoint を提供する Kubernetes API |
| ServiceAccount | Pod や automation が Kubernetes API へ認証する identity |
| StorageClass | dynamic provisioning の provisioner と parameter、policy を表す resource |
| TrustyAI | AI model / application の explainability、fairness、risk 管理等を支援する Red Hat の技術群 |
| VMI | VirtualMachineInstance。KubeVirt で実行中の VM instance を表す resource |
| vLLM | LLM inference / serving engine の一つ |
| VM | Virtual Machine。仮想 hardware 上で guest OS を実行する単位 |
| Velero | Kubernetes resource と persistent volume data の backup / restore、migration を支援する open source tool |

## よく混同する組み合わせ

| 組み合わせ | 区別 |
| --- | --- |
| Namespace / Project | Kubernetes の scope と、OpenShift が利用性・access control の情報を加えた表現 |
| Ingress / Route | Kubernetes 標準 API と OpenShift 固有 API。controller、TLS、機能差は対象版で確認 |
| RBAC / SCC | API 操作の認可と、Pod が許される security context の制約 |
| VM / VMI | 望ましい VM 定義・lifecycle と、現在実行中の instance |
| PV / PVC / StorageClass | volume resource、利用要求、dynamic provisioning policy |
| Monitoring / Logging | 数値・状態の時系列観測と、event / message の収集・検索 |
| Backup / DR | data・resource の複製と、service を目標時間で復旧する人・場所・手順を含む計画 |
| OpenShift AI / Dify | model / data science workload の platform と、AI application / workflow を作る上位 layer の一例 |

## 公式資料

- [OpenShift Container Platform documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/)
- [Kubernetes glossary](https://kubernetes.io/docs/reference/glossary/)
- [KubeVirt user guide](https://kubevirt.io/user-guide/)

## 実務での説明要点

- 略語は正式名称だけでなく、どの層の何を管理し、どの成果物や運用へ影響するかで説明する。
- 例えば RBAC は API 操作の認可、SCC は Pod の security context 制約であり、目的と適用箇所が異なる。
- 機能、API、推奨構成、サポート条件は OCP 4.22 と各 Operator の公式資料で確認する。
