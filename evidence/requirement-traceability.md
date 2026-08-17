# 要求トレーサビリティ

## 現在の状態

- 文書版: v0.1 準備版
- 面談要求の匿名要約: 作成済み
- 既存教材の初回通読: docs/00〜27 を一度通読済み
- 詳細レビュー・要点整理: 未完了
- 架空の基盤導入プロジェクト: AI 支援ドラフト作成済み、本人学習は未着手
- ラボ・実機証跡: なし
- 外部提出可否: 未判定（現状の提出は非推奨）

この表の「資料初稿あり」は、本人の理解・経験を意味しません。要求、資料、今後の確認を対応付けるための管理状態です。

## 状態の定義

| 状態 | 意味 |
| --- | --- |
| 初稿あり・未レビュー | 関連資料はあるが、本人確認前 |
| 本人レビュー済み | 本人が内容と経験境界を確認済み |
| 公式資料確認済み | 対象バージョンの一次資料で確認済み |
| 検証済み | 本人が実施し、Verification Record がある |
| 説明確認済み | 本人の自己説明と質問結果を記録済み |

## 面談で明示された領域

| 要求 ID | 要求・期待 | 対応する資料 | 証跡化する行動 | 現在状態 | 証跡 ID |
| --- | --- | --- | --- | --- | --- |
| RQ-01 | OpenShift の基本概念・設計・運用 | [基本知識](../docs/06-openshift-core-knowledge.md)、[基本設計](../docs/07-openshift-basic-design.md) | 公式資料確認、lab02、基本設計記入、自己説明 | 初稿あり・未レビュー | なし |
| RQ-02 | OpenShift Virtualization と VM 移行の全体像 | [Virtualization](../docs/08-openshift-virtualization.md) | 構成・依存関係・PoC 観点の説明、可能なら安全な実機確認 | 初稿あり・未レビュー | なし |
| RQ-03 | ROSA / ARO とクラウド責任分界 | [ROSA / ARO 比較](../docs/19-rosa-aro-comparison.md) | 対象サービスの公式資料で比較表を更新 | 初稿あり・未レビュー | なし |
| RQ-04 | Kubernetes の基礎と障害調査 | [Kubernetes 障害調査](../docs/13-kubernetes-troubleshooting.md)、[lab01](../labs/lab01-k8s-troubleshooting/) | 代表障害を観測・切り分け・修正・再試験 | 初稿あり・未実施 | なし |
| RQ-05 | RHEL / Linux 基盤 | [RHEL / Linux](../docs/11-rhel-linux-foundation.md) | 調査コマンドの期待値と実測値を記録 | 初稿あり・未実施 | なし |
| RQ-06 | Ansible による自動化 | [Ansible](../docs/12-ansible-automation.md)、[lab03](../labs/lab03-ansible-linux-basics/) | Check mode、初回、再実行、失敗時の挙動を記録 | 初稿あり・未実施 | なし |
| RQ-07 | Kong / API・AI Gateway | [Kong](../docs/20-kong-api-ai-gateway.md) | 公式資料確認、用途・責任分界の自己説明 | 初稿あり・未レビュー | なし |
| RQ-08 | Sysdig / 監視・コンテナセキュリティ | [Sysdig](../docs/21-sysdig-container-security.md) | 公式資料確認、標準機能等との比較説明 | 初稿あり・未レビュー | なし |
| RQ-09 | 基盤案件の進め方 | [工程](../docs/05-infra-project-process.md)、[設計ガイド](../docs/24-design-document-guide.md)、[試験ガイド](../docs/25-test-document-guide.md) | 一組の設計・手順・試験・切り戻し成果物とレビュー記録 | 初稿あり・未実施 | なし |
| RQ-10 | 未知技術・障害の自走調査 | [OpenShift 障害調査](../docs/14-openshift-troubleshooting.md)、[lab05](../labs/lab05-troubleshooting-report-practice/) | 事実、仮説、反証、結果、未解決事項を時系列で報告 | 初稿あり・未実施 | なし |
| RQ-11 | AI 利用時の情報管理 | [AI ガバナンス](../docs/23-ai-governance-for-infra-work.md) | マスキング例、利用可否判断、公式資料での裏付けを説明 | 初稿あり・未レビュー | なし |
| RQ-12 | 継続的なキャッチアップと正確な経験説明 | [学習計画](../docs/04-learning-roadmap.md)、[経験境界](../interview/experience-boundary.md) | 学習ログ、自己説明、第三者質問、経験ラベル更新 | 初稿あり・未実施 | なし |

## 自主追加領域

| 追加 ID | 自主追加領域 | 追加目的 | 対応する資料 | 現在状態 | 証跡 ID |
| --- | --- | --- | --- | --- | --- |
| SA-01 | KVM / QEMU / KubeVirt の詳細 | OpenShift Virtualization の内部構成を補足 | [仮想化レイヤー](../docs/09-kvm-qemu-kubevirt.md) | 初稿あり・未レビュー | なし |
| SA-02 | MTV の詳細 | VM 移行計画・評価観点を具体化 | [MTV / VM 移行](../docs/10-mtv-vm-migration.md) | 初稿あり・未レビュー | なし |
| SA-03 | Airgap / Disconnected Install | 閉域環境の設計論点を補足 | [Airgap](../docs/18-airgap-disconnected-install.md) | 初稿あり・未レビュー | なし |
| SA-04 | OpenShift AI | AI 基盤との将来的な接点を整理 | [OpenShift AI](../docs/22-openshift-ai-overview.md) | 初稿あり・未レビュー | なし |
| SA-05 | Storage / ODF、監視・ログ・バックアップの詳細 | OpenShift 基本設計と障害調査を補強 | [監視等](../docs/15-monitoring-logging-backup.md)、[Storage](../docs/16-storage-csi-odf.md) | 初稿あり・未レビュー | なし |

自主追加領域は、面談側が直接要求した内容として説明しません。OpenShift Virtualization と VM 移行は RQ-02 ですが、その内部実装や特定ツールの詳細は SA-01 / SA-02 として分けます。

## 更新方法

1. 本人が対象資料をレビューし、[learning-log.md](learning-log.md) に Learning Record を作る。
2. 要求行へ Learning Record の ID とリンクを追加する。
3. 実機検証が必要な場合は [verification-record.md](verification-record.md) に Verification Record を作る。
4. 公式資料の版、URL、確認日と、本人の説明結果を Learning Record へ記録する。
5. 状態を一段ずつ更新し、飛び越えた場合は理由を記載する。
6. 外部提出前に要求漏れ、未確認の断定、経験境界を再レビューする。
