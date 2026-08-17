# RHEL外部サービス用Ansible

このAnsible資材の対象は、踏み台および外部Load Balancerなどの**RHEL支援サーバーだけ**です。OpenShiftノードはRHCOSであり、AnsibleでSSH接続して直接変更しません。RHCOSの構成変更は、OpenShiftが管理するMachineConfigなどのサポートされた方法を使います。

本シナリオではRHEL 9.x（minor TBD）を設計前提とし、各LB/踏み台に別途有効なRHEL subscription、承認済みrepository、更新・脆弱性管理が必要です。

## Playbook

| Playbook | 目的 | 変更 |
| --- | --- | --- |
| `playbooks/preflight.yml` | RHEL、名前解決、Proxy、時刻同期の事前確認 | なし |
| `playbooks/configure-load-balancers.yml` | HAProxy/keepalivedの導入と設定 | あり。明示承認変数が必要 |

## 安全な実行順（実行は未実施）

```bash
ansible-playbook --syntax-check playbooks/preflight.yml
ansible-playbook playbooks/preflight.yml
# clean RHELではHAProxy validatorが未導入のため、この初回checkが
# template検証段階で失敗することを依存不足として確認する
ansible-playbook --check --diff playbooks/configure-load-balancers.yml \
  -e external_service_change_approved=true
# 変更承認後、packageだけを先に導入する（この行は実変更）
ansible-playbook playbooks/configure-load-balancers.yml --tags packages \
  -e external_service_change_approved=true
# validator導入後に完全なcheck/diffを再実行し、承認後だけ実適用する
ansible-playbook --check --diff playbooks/configure-load-balancers.yml \
  -e external_service_change_approved=true
ansible-playbook playbooks/configure-load-balancers.yml \
  -e external_service_change_approved=true
```

- `inventory/hosts.example.yml` は文書用IPです。承認済みインベントリを別のアクセス制御領域に作成します。
- SSH鍵、sudoパスワード、Vaultパスワードは格納しません。
- `external_service_change_approved=true` は承認そのものではなく、誤操作防止のガードです。承認番号は別の変更記録へ残します。
- keepalived利用、NIC名、VRRP通信、split-brain防止、VIPフェイルオーバーはNetwork/Infrastructure teamの承認対象です。
- clean RHELでの最初のfull checkは、check modeではpackageを実導入しないため、`/usr/sbin/haproxy` validator不在で停止する想定です。これを回避するためvalidationを無効化せず、承認後に`packages` tagだけを実適用してからfull checkを再実行します。
