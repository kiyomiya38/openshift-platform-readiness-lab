# 技術リファレンス

このディレクトリは、インフラ実務経験があり OpenShift は未経験のエンジニアが、設計・構築・試験・移行・運用の成果物を読み解くための補助資料です。操作の暗記ではなく、実務で何を決め、何を確認し、どの成果物へ反映するかを扱います。

OpenShift の具体例は **OpenShift Container Platform 4.22** に固定しています。ただし、正確な z-stream、導入方式、Operator、外部製品、クラウドサービスの対応関係は変化します。実案件では採用時点の公式資料、サポートマトリクス、契約条件、組織標準を優先してください。

## 使い方

1. 記入済みの成果物例で、要件から運用までの情報の流れを確認する。
2. 成果物に現れた用語や設計判断を、本ディレクトリの該当資料で確認する。
3. 実案件では、成果物例の値を流用せず、対象環境の要件と公式資料から判断し直す。
4. 組織の正式様式へ転記するときは、[`templates/`](../templates/README.md) の空テンプレートを補助的に利用する。

コマンド例は、実行場所、対象クラスタ、権限、影響、期待結果、ロールバックを確認したうえで使用します。削除、強制操作、証明書、etcd、MachineConfig、ストレージ、Firewall に関する変更は、承認済み手順とバックアップなしに実行しないでください。

## 案件工程・文書

| 資料 | 実務で確認する内容 |
| --- | --- |
| [基盤案件の進め方](technical/infra-project-process.md) | 要件定義から運用引き継ぎまでの工程、品質ゲート、課題・変更管理 |
| [OpenShift 基本設計](technical/openshift-basic-design.md) | クラスタ、可用性、ネットワーク、認証、ストレージ、運用の設計観点 |
| [設計・管理文書ガイド](technical/design-document-guide.md) | 成果物間のトレーサビリティと記入済み例・空テンプレートの使い分け |
| [試験文書ガイド](technical/test-document-guide.md) | 試験仕様、期待値、実測値、証跡、完了判定の分離 |
| [用語集](technical/glossary.md) | 本リポジトリで使用する主要用語と混同しやすい概念 |

## OpenShift・Linux・自動化

| 資料 | 実務で確認する内容 |
| --- | --- |
| [OpenShift 基本知識](technical/openshift-core-knowledge.md) | Kubernetes との差分、Route、SCC、RBAC、Operator、ClusterOperator |
| [RHEL / Linux 基盤](technical/rhel-linux-foundation.md) | systemd、ログ、ネットワーク、時刻、TLS、RHEL と RHCOS の変更境界 |
| [Ansible / 自動化](technical/ansible-automation.md) | Inventory、Role、冪等性、秘密情報、段階適用、AAP、所有範囲 |

## ネットワーク・ストレージ・運用

| 資料 | 実務で確認する内容 |
| --- | --- |
| [Network / DNS / Load Balancer / Firewall](technical/network-dns-lb-firewall.md) | API、Ingress、Pod/Service Network、通信要件、障害境界 |
| [Storage / CSI / ODF](technical/storage-csi-odf.md) | PV/PVC、StorageClass、CSI、ODF、保護・復元、VM ストレージ |
| [監視・ログ・バックアップ](technical/monitoring-logging-backup.md) | メトリクス、通知、ログ転送、OADP、etcd、RPO/RTO |
| [Kubernetes トラブルシューティング](technical/kubernetes-troubleshooting.md) | Pod、Image、Scheduler、Service、DNS、Node、CNI の切り分け |
| [OpenShift トラブルシューティング](technical/openshift-troubleshooting.md) | SCC、OLM、ClusterOperator、Route、DNS/LB/Firewall の切り分け |

## Virtualization・移行

| 資料 | 実務で確認する内容 |
| --- | --- |
| [OpenShift Virtualization](technical/openshift-virtualization.md) | VM API、ノード、CPU、ストレージ、Multus、Live Migration、保護 |
| [KVM / QEMU / libvirt / KubeVirt](technical/kvm-qemu-kubevirt.md) | 仮想化レイヤーの責務と OpenShift での操作境界 |
| [MTV / VM 移行](technical/mtv-vm-migration.md) | 棚卸し、互換性、NetworkMap、StorageMap、Plan、切り戻し |

## 導入方式・関連製品

| 資料 | 実務で確認する内容 |
| --- | --- |
| [Disconnected Install](technical/disconnected-install.md) | Mirror Registry、oc-mirror、信頼 CA、資材搬送、更新 |
| [ROSA / ARO 比較](technical/rosa-aro-comparison.md) | クラウド別の責任分界、接続、ID、ストレージ、運用 |
| [Kong / API・AI Gateway](technical/kong-api-ai-gateway.md) | API 入口、認証・認可、流量制御、LLM ルーティング、可観測性 |
| [Sysdig / コンテナセキュリティ](technical/sysdig-container-security.md) | 脆弱性、実行時検知、Audit、SCC/RBAC、外部通信 |
| [OpenShift AI](technical/openshift-ai-overview.md) | Workbench、Pipeline、Model Serving、GPU、RAG、基盤設計 |
| [AI 利用ガバナンス](technical/ai-governance.md) | 入力禁止情報、一般化、出力検証、承認、誤投入時の初動 |

## 公開・転用時の注意

- 例示したホスト名、IP、組織名、担当名は架空値またはプレースホルダーを使用する。
- Secret、Token、Cookie、pull secret、秘密鍵、実 kubeconfig、未公開ログを登録しない。
- 外部製品の機能、ライセンス、価格、SLA、互換性は、掲載内容だけで確定しない。
- 試験の期待結果を実測結果として扱わず、未実施は `NOT RUN` と明示する。
