# 12. 参考資料

## 目的

成果物例を読む際に必要な技術解説と、設計根拠を確認する公式一次資料への入口をまとめます。技術参考資料は理解を助ける二次的な解説であり、製品仕様、サポート、互換性、手順の最終根拠は対象版の公式資料です。

## 実務での使用場面

- 成果物中の用語、製品機能、設計判断を確認する
- 設計レビューで根拠URL、対象version、確認日を提示する
- install/update前にsupport matrixとknown issueを再確認する
- 製品版変更時に、影響する設計・設定・試験・Runbookを特定する

## 入力

- 確認したい要件ID、設計判断、パラメータ、試験ID
- 対象製品、edition、architecture、version、z-stream
- hardware、cloud、CSI、Operator、guest OSなどの組み合わせ
- 確認日と変更予定日

## 判断

情報源は次の優先順で評価します。

1. 対象versionの製品公式ドキュメント
2. 公式support policy、compatibility matrix、release notes、errata、knowledgebase
3. upstream公式documentationやdesign proposal
4. 本リポジトリの技術参考資料
5. blog、forum、個人記事は探索の入口としてのみ使用し、重要判断は一次資料で再確認

同じ製品名でもversionが違えば、API、field、default、support範囲、upgrade pathが変わる可能性があります。URLだけでなく、document title、version、section、確認日、判断への反映を[一次資料確認記録](../evidence/source-verification-record.md)へ残します。

## 技術参考資料

| 領域 | 解説 |
| --- | --- |
| 案件工程 | [インフラ案件の進め方](../reference/technical/infra-project-process.md) |
| OpenShift基礎 | [OpenShiftコア知識](../reference/technical/openshift-core-knowledge.md)、[OpenShift基本設計](../reference/technical/openshift-basic-design.md) |
| Virtualization | [OpenShift Virtualization](../reference/technical/openshift-virtualization.md)、[KVM・QEMU・KubeVirt](../reference/technical/kvm-qemu-kubevirt.md)、[MTV VM移行](../reference/technical/mtv-vm-migration.md) |
| OS・自動化 | [RHEL・Linux基礎](../reference/technical/rhel-linux-foundation.md)、[Ansible自動化](../reference/technical/ansible-automation.md) |
| 障害調査 | [Kubernetes障害調査](../reference/technical/kubernetes-troubleshooting.md)、[OpenShift障害調査](../reference/technical/openshift-troubleshooting.md) |
| 運用 | [監視・ログ・バックアップ](../reference/technical/monitoring-logging-backup.md) |
| Storage | [Storage・CSI・ODF](../reference/technical/storage-csi-odf.md) |
| Network | [Network・DNS・LB・Firewall](../reference/technical/network-dns-lb-firewall.md) |
| Disconnected | [Disconnected導入](../reference/technical/disconnected-install.md) |
| Cloud managed | [ROSA・ARO比較](../reference/technical/rosa-aro-comparison.md) |
| API・Security | [Kong API/AI Gateway](../reference/technical/kong-api-ai-gateway.md)、[Sysdig container security](../reference/technical/sysdig-container-security.md) |
| AI | [OpenShift AI概要](../reference/technical/openshift-ai-overview.md)、[AIガバナンス](../reference/technical/ai-governance.md) |
| 文書 | [設計文書ガイド](../reference/technical/design-document-guide.md)、[試験文書ガイド](../reference/technical/test-document-guide.md) |
| 用語 | [用語集](../reference/technical/glossary.md) |

## 成果物例の読み方

| 成果物 | 先に確認する技術参考 |
| --- | --- |
| 要件・責任分界・トレーサビリティ | インフラ案件、設計文書、用語集 |
| 基本・アーキテクチャ設計 | OpenShiftコア、OpenShift基本設計 |
| Network/DNS/LB/Proxy | Network・DNS・LB・Firewall |
| Storage/Registry/backup | Storage・CSI・ODF、監視・ログ・バックアップ |
| RHEL LBのAnsible | RHEL・Linux、Ansible |
| 障害Runbook | Kubernetes障害調査、OpenShift障害調査 |
| Virtualization/MTV | OpenShift Virtualization、KVM/QEMU/KubeVirt、MTV |
| Kong/Sysdig/OpenShift AI | 各製品解説、AIガバナンス |
| 試験仕様・結果 | 試験文書ガイド |

## 公式一次資料

### OpenShift Container Platform 4.22

- [Documentation top](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/)
- [Architecture](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/architecture/)
- [Agent-based Installer](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installing_an_on-premise_cluster_with_the_agent-based_installer/)
- [Networking](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/networking_overview/)
- [Storage](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/storage/)
- [Security and compliance](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/security_and_compliance/)
- [Authentication and authorization](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/authentication_and_authorization/)
- [Machine configuration](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/machine_configuration/)
- [Registry configuration](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/registry/configuring-registry-operator)
- [Backup and restore](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/backup_and_restore/)
- [Updating clusters](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/updating_clusters/)
- [Support](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/support/)
- [Virtualization](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/virtualization/)
- [Monitoring stack for Red Hat OpenShift 4.22](https://docs.redhat.com/en/documentation/monitoring_stack_for_red_hat_openshift/4.22/)

### 関連製品・サービス

- [Migration Toolkit for Virtualization](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/)
- [Red Hat OpenShift Service on AWS](https://docs.redhat.com/en/documentation/red_hat_openshift_service_on_aws/)
- [Azure Red Hat OpenShift](https://learn.microsoft.com/azure/openshift/)
- [Red Hat OpenShift AI](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/)
- [Red Hat Ecosystem Catalog](https://catalog.redhat.com/)
- [Kong Documentation](https://developer.konghq.com/)
- [Sysdig Documentation](https://docs.sysdig.com/)

### Lifecycle・互換性

- [OpenShift Container Platform Life Cycle Policy](https://access.redhat.com/support/policy/updates/openshift)
- [OpenShift Container Platform Tested Integrations](https://access.redhat.com/articles/4763741)
- [Red Hat Product Security Center](https://access.redhat.com/security/)

## 他文書とのつながり

- 調査結果で設計判断が変わる場合は、ADR、設計、パラメータ、手順、試験を同時に更新する
- versionを固定したら、構築資材と試験証跡にもversion/digestを残す
- support対象外または未確認の組み合わせは、TBD、risk、No-Goとして管理する
- 公式資料の確認履歴は[一次資料確認記録](../evidence/source-verification-record.md)へ残す

## レビューで指摘されやすい点

- 検索結果やblogだけを技術判断の根拠にする
- 対象versionと異なる最新documentationをそのまま採用する
- compatibilityを製品単体だけで見て、hardware、CSI、Operator、guest OSの組み合わせを確認しない
- access制限のあるURLだけを記載し、結論や確認日を残さない
- 公式資料の記述を引用するだけで、案件の判断・影響へ結び付けない
- release/update時に根拠を再確認しない

## 読了後の扱い

この章で番号順の主導線は終了です。以後は[架空プロジェクト](../projects/enterprise-openshift-platform/README.md)の成果物を必要な工程から参照し、用語や製品仕様で止まった箇所だけ技術参考資料と公式一次資料で確認します。採点形式にはせず、実務を想定した確認事実だけを[`evidence/`](../evidence/README.md)の様式へ記録します。
