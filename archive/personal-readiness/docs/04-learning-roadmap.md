# 30 日間の学習ロードマップ

> [!IMPORTANT]
> 学習資料は準備済みですが、本人による読了、演習、実機検証は、実施記録がある項目だけを完了として扱います。

## 進め方

- 開始前に [README](../README.md) と [learning-report](../learning-report.md) を読みます。
- その後は docs/00 から docs/27 まで、番号を戻さず順番に読みます。
- 1 日 60〜120 分が目安です。終わらない場合は、次の番号へ進まず翌日に継続します。
- ラボは表で指定された日に実施します。環境がなければ、期待結果と確認方法を記載する机上演習へ置き換えます。
- 参考回答は自分の作業後に確認し、参考回答そのものを本人の証跡として扱いません。
- 日数よりも番号順と、実施事実を正確に記録することを優先します。

## 30 日の番号順学習表

| Day | 読む資料・演習 | 主な作業 | 残す記録 |
| ---: | --- | --- | --- |
| 1 | [docs/00 リポジトリ利用ガイド](00-repository-guide.md) | フォルダの役割と経験区分を確認する | 読了日、要点、不明点 |
| 2 | [docs/01 面談要求領域](01-meeting-requirements-summary.md) | 明示領域と自主追加領域を分ける | 要求領域の要約 |
| 3 | [docs/02 現在地](02-current-skill-position.md) | 自分の実際の経験との差を確認する | 経験境界と要確認事項 |
| 4 | [docs/03 スキルギャップ](03-skill-gap-analysis.md) | 優先度と不足領域を確認する | 優先課題3件 |
| 5 | [docs/04 本ロードマップ](04-learning-roadmap.md) | 時間、環境、費用、権限を確認する | 実施予定と制約 |
| 6 | [docs/05 基盤案件の進め方](05-infra-project-process.md) | 工程、成果物、完了条件を整理する | 工程ごとの入力、判断、成果物 |
| 7 | [docs/06 OpenShift基本知識](06-openshift-core-knowledge.md) | Kubernetesとの差と主要構成を整理する | APIからPod実行までの構成メモ |
| 8 | [docs/07 OpenShift基本設計](07-openshift-basic-design.md) | DNS、LB、NW、Security、Storageを整理する | 設計判断、根拠、要確認事項 |
| 9 | [lab02 OpenShift基本リソース](../labs/lab02-openshift-basic-resources/README.md) | 権限に合う環境で実施する | 環境、コマンド、期待値、実測値 |
| 10 | [docs/08 OpenShift Virtualization](08-openshift-virtualization.md) | VM、VMI、DataVolume、PVCの関係を整理する | VMライフサイクル図 |
| 11 | [docs/09 KVM・QEMU・KubeVirt](09-kvm-qemu-kubevirt.md) | 仮想化レイヤーと操作境界を整理する | 各レイヤーの責任分界 |
| 12 | [docs/10 MTV・VM移行](10-mtv-vm-migration.md) | 評価、移行方式、試験、切り戻しを整理する | 架空VMの移行可否と確認事項 |
| 13 | [docs/11 RHEL・Linux基盤](11-rhel-linux-foundation.md) | サービス、ログ、経路、容量を確認する | コマンドと確認目的 |
| 14 | [docs/12 Ansible](12-ansible-automation.md)・[lab03](../labs/lab03-ansible-linux-basics/README.md) | Check mode、変更、再実行を確認する | 差分、冪等性、復旧方法 |
| 15 | [docs/13 Kubernetes障害調査](13-kubernetes-troubleshooting.md)・[lab01](../labs/lab01-k8s-troubleshooting/README.md) | 一つ以上の障害を切り分ける | 事実、仮説、確認、対処、再試験 |
| 16 | [docs/14 OpenShift障害調査](14-openshift-troubleshooting.md) | Route、SCC、Operator等を層別に整理する | レイヤー別の確認経路 |
| 17 | [docs/15 監視・ログ・バックアップ](15-monitoring-logging-backup.md) | 検知、調査、保管、復旧を分ける | 一次対応と連絡条件 |
| 18 | [docs/16 Storage・CSI・ODF](16-storage-csi-odf.md) | PVCからBackendまでを整理する | ストレージの確認順 |
| 19 | [docs/17 Network・DNS・LB・Firewall](17-network-dns-lb-firewall.md)・[lab05](../labs/lab05-troubleshooting-report-practice/README.md) | 通信経路を使って障害報告を作る | 時系列、影響、証跡、原因、対処 |
| 20 | [docs/18 Airgap](18-airgap-disconnected-install.md) | image供給と継続更新を整理する | 導入・更新時の確認事項 |
| 21 | [docs/19 ROSA・ARO比較](19-rosa-aro-comparison.md) | OCP、ROSA、AROの管理責任を比較する | 責任分界と要確認事項 |
| 22 | [docs/20 Kong](20-kong-api-ai-gateway.md) | Gatewayとアプリの境界を整理する | 認証、制限、監査、障害時動作 |
| 23 | [docs/21 Sysdig](21-sysdig-container-security.md) | 監視、脆弱性、Runtime Securityを分ける | OpenShiftとの接点 |
| 24 | [docs/22 OpenShift AI](22-openshift-ai-overview.md) | AI基盤、モデル、アプリを分ける | 自主追加領域としての主要構成 |
| 25 | [docs/23 AIガバナンス](23-ai-governance-for-infra-work.md) | AIへ入力できる情報を判断する | マスキング例と利用可否判断 |
| 26 | [docs/24 設計文書ガイド](24-design-document-guide.md) | テンプレートと成果物追跡を整理する | 使用テンプレートと入力条件 |
| 27 | [docs/25 試験文書ガイド](25-test-document-guide.md)・[lab04](../labs/lab04-design-document-practice/README.md) | 設計・試験成果物を一組作る | 設計判断、試験条件、期待結果 |
| 28 | [docs/26 面談回答ガイド](26-interview-answer-guide.md)・[lab06](../labs/lab06-interview-practice/README.md) | 経験境界を守って説明する | 1分説明と修正点 |
| 29 | [docs/27 用語集](27-glossary.md) | 不明語を補い、弱点を3件に絞る | 自分の説明、参照先、残課題 |
| 30 | [準備度スコアカード](../checklist/readiness-scorecard.md) | 模擬面談と次期計画を行う | 根拠付き自己評価と次の計画 |

## ラボと記録先

| 記録内容 | 記録先 |
| --- | --- |
| 日々の読了、要点、不明点 | [学習ログ](../evidence/learning-log.md) |
| ラボ、コマンド、実測結果 | [検証記録](../evidence/verification-record.md) |
| 要求と成果物の対応 | [要求トレーサビリティ](../evidence/requirement-traceability.md) |
| 最終的な提出可否 | [提出準備判定](../evidence/submission-readiness.md) |

実施していないラボは未実施のまま記録します。机上演習と実機検証済みを同じ状態にしません。

## 遅れた場合の扱い

1. 現在の番号を終えるまで次の資料へ進みません。
2. 時間が少ない日は、要点と不明点の記録だけを行います。
3. 実機環境がない場合は、期待結果、確認コマンド、要確認事項を記録します。
4. 再開時は、最後に完了した番号の次から始めます。

## 全章を一度読んだ後の次の段階

全章を一度通読した後は、[架空のエンタープライズ OpenShift 基盤導入プロジェクト](../projects/enterprise-openshift-platform/README.md) を進めます。プロジェクト内の文書も `docs/00` から番号順で、憲章、要件、基本設計、詳細設計、構築、試験、移行、運用、管理へ進みます。

実機がない間は、設計判断の理由、未決事項、期待結果、確認方法を机上で整理します。構築済み・試験合格とは記録せず、実機を用意した後に期待結果と実測結果を比較します。

## 公式資料

- [OpenShift Container Platform documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Ansible Documentation](https://docs.ansible.com/ansible/latest/)

## 面談での説明例

> 資料は番号順に確認し、読了だけでなく、机上成果物または許可された環境での検証結果を記録しています。実施していない領域と商用経験のない領域は分けて説明します。
