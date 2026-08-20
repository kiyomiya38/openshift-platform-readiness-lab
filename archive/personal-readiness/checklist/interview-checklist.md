# 面談直前チェックリスト

## 事実確認

- [ ] 経歴書、自己紹介、回答の年月・役割・製品名が一致している
- [ ] 資格名と取得時期を正確に言える
- [ ] 実際に使った OpenShift 環境だけを挙げている
- [ ] 商用 OpenShift / Virtualization / Airgap の未経験を明示できる
- [ ] 完了した lab と、まだ読んだだけの章を区別できる

## 回答構造

- [ ] 最初の一文で経験の有無または結論を答える
- [ ] 教材・検証の具体例を一つ示す
- [ ] 未経験範囲と本番との差を示す
- [ ] 対象版確認、レビュー、エスカレーションの行動を示す
- [ ] 30 秒版と 1 分版を使い分けられる

## 技術説明

- [ ] Kubernetes と OpenShift の違い
- [ ] Project / Namespace、Route / Ingress、RBAC / SCC
- [ ] Operator / OLM / ClusterOperator
- [ ] OpenShift 基本設計の主要項目
- [ ] VirtualMachine / VMI / DataVolume / PVC
- [ ] KVM / QEMU / KubeVirt の関係
- [ ] MTV と VM 移行の棚卸し・切り戻し
- [ ] Pod、Route、PVC、Operator の初動調査
- [ ] Airgap と Mirror Registry
- [ ] Kong、Sysdig、OpenShift AI、AI ガバナンス

## NG 表現

- [ ] 根拠なく「できます」「一人で対応できます」と言わない
- [ ] 教材作成を本番設計・障害対応と言い換えない
- [ ] 「多分」「とりあえず再起動」で判断しない
- [ ] AI へ顧客 log を貼る前提で話さない
- [ ] 分からない仕様を断定せず「要確認」と言える

## 面談当日

- [ ] 質問を最後まで聞く
- [ ] 不明な用語は定義を確認する
- [ ] 長くなったら結論へ戻る
- [ ] team の成果では自分の担当部分を限定する
- [ ] 逆質問を二つ用意する

### 逆質問例

1. 参画初期に担当する成果物と、レビュー体制を教えてください。
2. 対象 OpenShift / Operator の version、導入形態、現在の工程を教えてください。
3. 検証環境、標準手順、過去の設計書を確認できる onboarding はありますか。

