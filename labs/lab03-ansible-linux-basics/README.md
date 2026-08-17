# Lab 03: Ansible による Linux 基本設定

> [!IMPORTANT]
> **状態:** 演習資料: 準備済み / 本人実施: 未実施 / 実機証跡: なし


破棄可能な RHEL 系 VM に対して、Inventory、Playbook、package、user、copy、template、service、firewalld、chrony、handler、冪等性を練習します。最後に Kubernetes/OpenShift YAML の配置と、明示的に有効化した場合だけ ConfigMap を適用する演習があります。

## 学習目標

- Inventory と変数の優先関係を説明できる。
- FQCN を使った Playbook を check mode で点検できる。
- template の変更時だけ handler が動く仕組みを説明できる。
- 1 回目と 2 回目の `changed` 差から冪等性を確認できる。
- Shell で `oc apply` を直書きする場合との違いを考え、`kubernetes.core.k8s` の前提を説明できる。

## 前提条件

- 破棄可能な RHEL 8/9 または互換の検証 VM。RHCOS や OpenShift Node を対象にしない。
- Control node に `ansible-core`、対象 VM に Python 3 と sudo 権限
- SSH 公開鍵認証。Password、秘密鍵、token は Repository へ保存しない。
- `ansible.posix` Collection。任意演習には `kubernetes.core`、Python Kubernetes client、`kubernetes-validate`、`oc` login 済み環境が必要（版は要確認）。`playbooks/03-openshift-apply.yml` の `validate.fail_on_error` は `kubernetes-validate` に依存する。
- NTP server、package repository、firewalld 利用可否は環境依存（要確認）。

## 安全上の注意

- `inventory.ini` の `192.0.2.10` は文書用 TEST-NET で接続できない。自分の検証 VM へ明示的に置換する。
- 構文と対象一覧を確認し、初回は `packages` tag だけを check/apply する。その後、残りの task を check してから Playbook 全体を通常実行する。
- Playbook は package、user、firewalld、chrony 設定を変更する。本番、共有 VM、OpenShift Node には実行しない。
- `/etc/chrony.conf` は backup を作るが、復旧手順と NTP 到達性を事前確認する。
- OpenShift 任意演習は `openshift_apply_enabled=false` が既定。Context、Project、承認済み API Server を確認し、`expected_api_server` を完全一致で指定した場合だけ true にする。

## セットアップ

```bash
ansible --version
ansible-galaxy collection install ansible.posix
ansible-inventory -i inventory.ini --graph
```

`inventory.ini` の `ansible_host`、`ansible_user`、SSH key の参照を検証環境へ変更します。秘密鍵の中身や sudo password は書きません。

接続を read-only で確認します。

```bash
ansible lab_targets -i inventory.ini -m ansible.builtin.ping
ansible-playbook -i inventory.ini playbooks/00-facts.yml --limit lab_targets
```

## Exercise 1: Playbook の事前点検

```bash
ansible-playbook -i inventory.ini playbooks/01-linux-baseline.yml --syntax-check
ansible-playbook -i inventory.ini playbooks/01-linux-baseline.yml --list-hosts
```

予想外の対象、package、port、NTP server、ファイル差分があれば実行しません。変数は `answers/vars.example.yml` を自分の環境向けに複製し、`-e @<自分の変数ファイル>` で指定できます。

## Exercise 2: Linux baseline の適用

`--list-hosts` の対象が意図した破棄可能 VM だけであること、検証 VM の Snapshot/復旧手段、変更許可を再確認後、次の順序で実行します。最小構成の RHEL では、package 未導入のまま Playbook 全体を check mode で実行すると、後続の service/firewalld task が失敗することがあります。そのため、まず package の差分だけを確認して導入し、次に package 以外を check mode で確認します。`packages` tag だけを指定した場合も、対象が RHEL 系の lab VM かを検査する `always` tag の安全確認は実行されます。

```bash
ansible-playbook -i inventory.ini playbooks/01-linux-baseline.yml --limit lab_targets --tags packages --check --diff
ansible-playbook -i inventory.ini playbooks/01-linux-baseline.yml --limit lab_targets --tags packages --diff
ansible-playbook -i inventory.ini playbooks/01-linux-baseline.yml --limit lab_targets --skip-tags packages --check --diff
ansible-playbook -i inventory.ini playbooks/01-linux-baseline.yml --limit lab_targets --diff
```

確認する項目:

- `chrony` と `firewalld` package が present か。
- `chronyd` と `firewalld` service が enabled/started か。
- `labobserver` user が対話 login 不可で作成されたか。
- `/etc/motd.d/99-readiness-lab` が copy module で配置されたか。
- chrony template が変化した時だけ `Restart chronyd` handler が通知されたか。
- 8080/tcp が検証用 zone に設定されたか。既存ルールへの影響は要確認。

## Exercise 3: 冪等性

同じ Playbook をもう一度実行します。

```bash
ansible-playbook -i inventory.ini playbooks/01-linux-baseline.yml --limit lab_targets
```

環境外の変化がなければ 2 回目の `changed=0` が期待値です。0 でなければ、常に変更扱いの task、動的な template、service/firewall 状態を調べ、[answers/baseline-explained.md](answers/baseline-explained.md) と比較します。

## Exercise 4: YAML 配置

```bash
ansible-playbook -i inventory.ini playbooks/02-place-manifest.yml --limit lab_targets --check --diff
ansible-playbook -i inventory.ini playbooks/02-place-manifest.yml --limit lab_targets
```

対象 VM の `/tmp/os-readiness-lab/training-configmap.yaml` に非機密 YAML が配置されます。`copy` は内容が同じなら変更しません。

## Exercise 5: OpenShift への任意適用

これは Control node で実行します。検証クラスタ、`os-ansible-lab` Project、`kubernetes.core`、Python Kubernetes client、`kubernetes-validate` がある場合だけ行います。`validate.fail_on_error` を有効にしているため、`kubernetes-validate` がない環境では適用前の検証で失敗します。

```bash
oc whoami --show-server
oc project os-ansible-lab
ansible-playbook playbooks/03-openshift-apply.yml --check -e openshift_apply_enabled=true -e expected_api_server=https://api.lab.example.com:6443
ansible-playbook playbooks/03-openshift-apply.yml -e openshift_apply_enabled=true -e expected_api_server=https://api.lab.example.com:6443
oc get configmap ansible-training -n os-ansible-lab
```

例の API Server は文書用です。`oc whoami --show-server` の出力と、管理者が承認した検証クラスタの API Server が一致することを別経路でも確認してから、同じ完全な URL に置き換えます。Playbook は適用前にも現在の選択先と `expected_api_server` を比較し、不一致なら停止します。

`kubernetes.core.k8s` は API の desired state を扱います。Shell/command で `oc apply` を呼ぶ場合は、Context、戻り値、changed 判定、認証情報、冪等性を別途設計する必要があります。

## 検証

```bash
ansible lab_targets -i inventory.ini -b -m ansible.builtin.service -a 'name=chronyd state=started' --check
ansible lab_targets -i inventory.ini -b -m ansible.builtin.stat -a 'path=/etc/motd.d/99-readiness-lab'
ansible-playbook -i inventory.ini playbooks/01-linux-baseline.yml --limit lab_targets --check
```

最後の recap、変更差分、handler の実行条件を [answers/expected-run.md](answers/expected-run.md) に沿って記録します。

## クリーンアップ

Linux 設定の削除は環境の既存値を壊す可能性があるため、自動削除 Playbook は用意していません。破棄可能 VM を Snapshot へ戻すか廃棄します。OpenShift 任意演習を行った場合は対象 Context を確認し、専用 ConfigMap/Project の所有者方針に従って削除します（要確認）。

## 公式リファレンス

- [Ansible: Inventory](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html)
- [Ansible: Playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html)
- [Ansible: Check mode and diff mode](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_checkmode.html)
- [kubernetes.core.k8s module](https://docs.ansible.com/ansible/latest/collections/kubernetes/core/k8s_module.html)

## 面談での説明例

> [!IMPORTANT]
> 演習完了・証跡記録後のみ、実施結果を過去形で説明します。現時点では本人レビュー・演習とも未実施です。

「現時点では、Inventory、package、service、copy、template、firewalld、chrony、handler と、check/diff・冪等性を確認する演習資料を準備した段階で、本人による RHEL 系 VM での実行と証跡記録はまだ行っていません。実施後は、対象環境、Playbook、1 回目と 2 回目の実測差を限定して説明します。」
