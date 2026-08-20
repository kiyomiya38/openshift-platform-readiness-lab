# 技術図

このディレクトリには、OpenShift 基盤の構成、通信、移行、障害調査、AI 利用統制を説明する Mermaid ソースを収録しています。図は技術リファレンスや実務成果物例の補助であり、特定の顧客環境や実装済み構成を表すものではありません。

| ファイル | 用途 |
| --- | --- |
| [基盤案件工程](infra-project-process.mmd) | 要件定義から運用引き継ぎまでの工程を示す |
| [OpenShift 全体構成](openshift-platform-overview.mmd) | 利用者、API、Ingress、ノード、外部サービスの関係を示す |
| [基本設計項目](openshift-basic-design-items.mmd) | 要件から各設計領域への展開を示す |
| [OpenShift Virtualization](openshift-virtualization-overview.mmd) | VM 管理 API、実行 Pod、ネットワーク、ストレージの関係を示す |
| [VM 移行フロー](vm-migration-flow.mmd) | 棚卸しから移行、確認、切り戻しまでの流れを示す |
| [Disconnected 導入](disconnected-install-overview.mmd) | 外部接続環境から分離環境への資材搬送を示す |
| [障害調査フロー](troubleshooting-flow.mmd) | 事実保全から切り分け、復旧、再発防止までを示す |
| [AI 利用判断](ai-governance-flow.mmd) | 機密情報と利用許可に基づく AI 利用判断を示す |

## 利用上の注意

- 実案件へ転用するときは、責任分界、ゾーン、FQDN、通信方向、冗長化、障害点を対象設計に合わせて更新する。
- 図だけで仕様を確定せず、要件 ID、設計書、パラメータ、通信要件表、試験項目へ対応付ける。
- 実ホスト名、IP、顧客名、アカウント、Secret、非公開構成を公開用の図へ記載しない。
- Mermaid のレンダリング結果を確認し、文字切れ、誤った矢印、凡例不足がない状態で成果物へ添付する。
