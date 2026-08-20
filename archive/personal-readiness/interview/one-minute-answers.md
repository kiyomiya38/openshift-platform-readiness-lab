# 1 分回答集

> [!IMPORTANT]
> 本ファイルは自己練習用の回答サンプルです。このリポジトリの本人レビュー、学習、ラボ実施、技能獲得が完了した証拠ではありません。技術説明は本人レビュー後、自分の言葉で説明できる場合にだけ使用してください。

## OpenShift とは

> OpenShift は Kubernetes を基盤とする enterprise container platform です。Kubernetes の workload orchestration に加え、Operator による platform component 管理、Route、認証、監視、Web Console、RHCOS などを統合します。この説明は練習用初稿です。私の過去経験は CRC と Web 上の Sandbox で Project、Route、Operator などを扱った入門教材の範囲で、商用設計・構築・運用経験はありません。SCC、RBAC、`oc` の具体的な到達範囲は本人再確認が必要です。

## Kubernetes 経験

> Kubernetes は基礎教材の作成・講義と、主にオンプレミスの学習・検証環境を構築した経験があります。商用 cluster の設計・構築・運用経験はありません。過去の構築方式、扱った resource、資格名と有効期限は提出前に本人確認します。今回用意された障害 lab は未実施であり、過去の教材・検証経験とは分けて説明します。

## OpenShift Virtualization

> OpenShift Virtualization は、OpenShift 上で container と VM を Kubernetes API の管理モデルで扱う機能です。KubeVirt と KVM / QEMU、VirtualMachine、VMI、DataVolume、PVC などを扱う説明初稿が準備されています。ただし、私は実導入・検証経験がなく、本人レビューもこれからです。現時点では技能として主張しません。

## Airgap

> Airgap / disconnected 環境では、cluster が必要な external source へ直接接続できないため、release、Operator catalog、application image を internal mirror registry へ供給します。この説明と設計観点は資料初稿で、私は構築・検証経験がなく、本人レビューもこれからです。

## Ansible

> Ansible は、Inventory で対象を定義し、Playbook と module で望ましい状態を宣言する automation tool です。面談では Ansible と Terraform の基礎部分を把握していると説明しましたが、商用自動化経験は確認できません。今回の Ansible lab は未実施で、実行結果を伴う技能証拠ではありません。

## 障害調査

> 障害調査の手順初稿では、影響範囲と変更履歴を押さえ、status、event、log、metric を時系列で保存し、層ごとに仮説を分ける流れを示しています。ただし、過去に行ったのは受講者のつまずきや質問を調べて講師間で共有する活動で、商用本番の障害対応ではありません。今回の障害 lab も未実施です。

## AI 利用ガバナンス

> AI は一般的な error や公開仕様の論点整理に使えますが、顧客名、IP、hostname、log、configuration、credential、個人情報を無断で入力しません。会社と案件の利用規程、data classification、承認済み tool を先に確認します。AI の出力は仮説であり、最終判断は対象版の公式資料、実機結果、担当者レビューで行い、利用した prompt と判断根拠を必要に応じて監査可能にします。

## 基盤案件の進め方

> 要件定義、基本設計、詳細設計、構築、試験、移行、運用引き継ぎの関係を説明する資料初稿があります。ただし、私は商用 OpenShift 案件の遂行経験がなく、今回のテンプレートも本人未レビューです。レビュー後に架空要件で記入し、成果物間のつながりを演習する予定です。
