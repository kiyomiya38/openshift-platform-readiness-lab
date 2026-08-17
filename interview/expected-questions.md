# 想定質問と回答例

> [!IMPORTANT]
> 本ファイルは自己練習用の回答サンプルです。このリポジトリの本人レビュー、学習、ラボ実施、技能獲得が完了した証拠ではありません。技術説明を含むすべての回答は初稿であり、本人が内容をレビューし、自分の言葉で説明できることを確認してから使用します。

回答例は暗記用の完成台本ではありません。`【 】` を本人確認済みの事実で置き換え、未実施の内容は削除してください。今回用意された lab はすべて未実施です。

## 経験と役割

### Q1. OpenShift の商用構築経験はありますか

**回答例:** ありません。過去には CRC と Web 上の Sandbox を利用し、Project、Route、Operator などを扱う入門教材を作成しました。SCC、RBAC、`oc` の具体的な操作範囲は提出前に本人確認します。cluster の新規構築や本番変更は未経験です。

### Q2. Kubernetes はどの程度経験がありますか

**回答例:** Kubernetes の基礎教材作成・講義と、主にオンプレミスの学習・検証環境を構築した経験があります。商用 cluster の設計・構築・運用経験はありません。過去の構築方式、扱った resource、正式な資格名・取得日・有効期限は、提出前に本人確認して限定します。今回の障害 lab は未実施です。

### Q3. OpenShift はどこまで触りましたか

**回答例:** 面談メモで確認できるのは、CRC と Web 上の Sandbox を利用し、Project、Route、Operator などを入門教材で扱ったことです。SCC、RBAC、`oc` は【本人確認後の実施内容】に限定します。ROSA の利用、管理者としての install、upgrade、復旧は経験として主張しません。

### Q4. 今の対応可能範囲はどこですか

**回答例:** 現時点で実績として説明できるのは、教材作成・講義、学習・検証環境の構築、受講者のつまずきや質問の整理です。今回の lab は未実施のため、OpenShift の実機対応力はまだ証明できません。参画時は既存手順とレビュー体制の下で、資料整備や検証補助など責任範囲を限定し、cluster-wide な設計判断や本番変更は単独実施しません。

### Q5. トラブルシュート資料作成経験は、現場経験ですか、教材経験ですか

**回答例:** 現場の商用経験ではありません。受講者がつまずきやすい点や質問の多い点を調べ、他の講師へ共有する資料を作成した経験です。本番障害の影響判断、復旧、恒久対策に責任を持った実績とは分けています。今回の障害 lab も未実施です。

## OpenShift / Kubernetes

### Q6. Kubernetes と OpenShift の違いは何ですか

**回答例:** OpenShift は Kubernetes を基盤に、Operator による platform 管理、Route、統合認証・監視・Web Console、RHCOS などを組み合わせた platform です。利用可能機能と既定値は release、導入形態、subscription で変わるため案件条件を確認します。

### Q7. Project と Namespace の違いは何ですか

**回答例:** Namespace は Kubernetes resource の scope です。OpenShift の Project は Namespace を利用者向けに扱いやすくし、表示情報や access-management の操作を加えた概念です。裏側の resource と権限を確認して使います。

### Q8. Route と Ingress の違いは何ですか

**回答例:** Ingress は Kubernetes 標準の HTTP(S) routing API、Route は OpenShift 固有 API です。TLS termination や routing option の表現に違いがあります。どちらも controller と DNS / LB が必要なので、resource だけで外部公開が完結するとは考えません。

### Q9. RBAC と SCC の違いは何ですか

**回答例:** RBAC は user や ServiceAccount が API resource に対して何をできるかを認可します。SCC は Pod が要求できる user ID、privilege、volume 等の security context を制約します。起動失敗では ServiceAccount と使用可能 SCC、admission message を確認します。

### Q10. Operator とは何ですか

**回答例:** Custom Resource と controller を使い、application や platform component の install、設定、upgrade、復旧などの運用知識を自動化する pattern です。自動だから無確認でよいわけではなく、channel、InstallPlan、dependency、support、managed field を確認します。

## 設計・案件工程

### Q11. OpenShift の基本設計では何を決めますか

**回答例:** cluster と node 構成、DNS、LB、API / Ingress、Pod / Service network、authentication、RBAC / SCC、storage、Operator、registry、monitoring、logging、backup、certificate、NTP、Firewall、運用方式を要件と責任分界に基づいて決めます。

### Q12. 詳細設計と構築手順書の違いは何ですか

**回答例:** 詳細設計は resource、parameter、命名、通信、権限など「何をどの値にするか」を定義します。構築手順書は、それを安全に適用する順序、command、期待結果、中断基準、rollback、証跡を定義します。

### Q13. Node sizing では何を見ますか

**回答例:** workload の CPU / memory request、platform overhead、storage / network I/O、障害・保守時の退避余力、成長率、quota、実測値を見ます。代表値だけで保証せず、対象 workload と製品 support 条件で要確認とします。

### Q14. 設計で分からない値がある場合はどうしますか

**回答例:** 推測で確定せず、要確認事項として内容、影響、確認先、期限、暫定方針を残します。依存する設計と schedule への影響を示し、意思決定者が判断できる選択肢を用意します。

### Q15. 設計経験は商用設計経験ですか

**回答例:** 商用 OpenShift 案件の基本設計・詳細設計経験ではありません。過去には、主にオンプレミスの教材用学習・検証環境を構成した経験があります。今回の設計テンプレートは資料初稿で、本人未レビューです。レビューと架空要件での記入演習後も、顧客設計実績とは分けます。

## Virtualization / Migration / Airgap

### Q16. OpenShift Virtualization とは何ですか

**回答例:** OpenShift 上で VM を Kubernetes API により宣言・管理・実行する機能として、KubeVirt、KVM / QEMU、VirtualMachine、VMI、DataVolume、PVC、Multus などを扱う説明初稿があります。私は実導入・検証経験がなく、本人レビューもこれからです。現時点では技能として主張しません。

### Q17. KVM 経験はありますか

**回答例:** 商用・検証経験は確認できません。KVM、QEMU、libvirt、KubeVirt の役割を扱う資料初稿はありますが、本人レビューと演習はこれからです。【本人が実施した場合のみ、その環境・操作・結果】を後から追記します。

### Q18. VMware から OpenShift Virtualization へ移行する論点は何ですか

**回答例:** guest OS と driver、CPU / memory、disk / snapshot、network / VLAN / fixed IP、停止可能時間、依存関係、backup、license、性能を棚卸しします。代表 VM で PoC と rehearsal を行い、acceptance と rollback 条件を決めます。

### Q19. Airgap 環境での構築経験はありますか

**回答例:** ありません。Airgap / disconnected 環境の説明資料初稿はありますが、本人レビューと検証はこれからです。使用前に、internal registry、image / metadata の同期、証明書、更新・復旧方式を本人が説明できるか確認します。

### Q20. Airgap で Mirror Registry が必要な理由は何ですか

**回答例:** cluster が external registry へ直接接続できないため、release、Operator catalog、application image を internal source から取得させるためです。初回だけでなく、継続的な mirror、容量、certificate、媒体搬送、監査、更新失敗時の復旧を設計します。

## Ansible / 自動化

### Q21. Ansible 経験はありますか

**回答例:** 商用環境の構成管理・運用自動化経験はありません。面談では Ansible と Terraform の基礎部分を把握していると申告しましたが、具体的な過去の実行内容は本人再確認が必要です。今回の Ansible lab は未実施です。今後実施した場合は、環境、Playbook、check / diff、実行結果を記録した範囲だけを説明します。

## 障害調査と運用

### Q22. Pod が起動しない場合はどう調べますか

**回答例:** 対象と影響を確認し、Pod status、container state、`describe` の event、current / previous log を見ます。その後 image、command、probe、resource、volume、security、node の仮説を絞ります。変更前に証拠を保存します。

### Q23. Route でアクセスできない場合はどう調べますか

**回答例:** client DNS、TCP / TLS、external LB、Ingress Controller、Route admission、Service、EndpointSlice、Pod readiness、application listen port を順に確認します。内部・外部からの差と変更履歴で、担当境界を絞ります。

### Q24. Operator 異常はどう見ますか

**回答例:** ClusterOperator、CSV、Subscription、InstallPlan、CatalogSource、Operator Pod、event、log と直前の更新を確認します。Operator-managed resource の直接変更は避け、対象版の recovery と support 手順を確認します。

### Q25. PVC が Bound にならない場合はどう見ますか

**回答例:** PVC event、StorageClass、provisioner、access mode、request 容量、backend 容量・credential、topology を確認します。`WaitForFirstConsumer` なら Pod scheduling と node topology も一緒に確認します。

### Q26. 監視・ログ・バックアップ経験はありますか

**回答例:** 商用 OpenShift での経験はありません。Prometheus、Grafana、Alertmanager、logging、OADP / Velero、etcd backup を扱う資料初稿はありますが、本人レビューと演習はこれからです。現時点では経験や理解済みの領域として主張しません。

## 周辺製品と AI

### Q27. Kong とは何ですか

**回答例:** API Gateway として routing、authentication / authorization、rate limit、変換、logging などを扱う説明初稿があります。Kong は面談側が示した対象領域で、私は導入・利用経験がなく、本人レビューもこれからです。

### Q28. Sysdig とは何ですか

**回答例:** container / Kubernetes の observability と security を扱う製品群として、image scanning、runtime detection、Kubernetes audit、agent などを扱う説明初稿があります。Sysdig は面談側が示した対象領域で、私は導入・利用経験がなく、本人レビューもこれからです。

### Q29. OpenShift AI とは何ですか

**回答例:** OpenShift 上の AI / MLOps platform として、Workbench、pipeline、model serving、registry、governance などを扱う説明初稿があります。私は導入・利用経験がなく、本人レビューもこれからです。現時点では概要理解を主張しません。

### Q30. Dify と OpenShift AI の違いは何ですか

**回答例:** 資料初稿では、OpenShift AI を model / data science workload の platform、Dify を RAG、workflow、agent などの AI application を組み立てる layer の一例として区別しています。本人レビュー前のため、現時点の技能説明には使用しません。

### Q31. AI を使って障害調査するときの注意は何ですか

**回答例:** 顧客情報、個人情報、IP、hostname、log、configuration、credential を無断入力しません。承認済み tool と rule を確認し、一般化・masking した論点だけに使います。出力は仮説として扱い、公式資料、実測、有識者レビューで確認します。

## 自己採点

各回答を 0〜2 点で評価します。

- 0: 経験境界が曖昧、または質問に答えていない
- 1: 結論と境界は正しいが、具体例・確認観点が弱い
- 2: 60 秒以内で、結論、事実、境界、案件での行動を説明できる
