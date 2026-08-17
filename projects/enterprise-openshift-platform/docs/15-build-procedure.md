# 15. OpenShift基盤 構築手順書

## 文書管理・実施状態

| 項目 | 内容 |
| --- | --- |
| 文書ID | `BUILD-OCP-001` |
| 対象 | Example Enterprise OpenShift 基盤導入 |
| 版／状態 | 0.1／Draft（机上手順） |
| 基準日 | 2026-08-17 |
| 対象版 | OpenShift 4.22.z（z版未確定） |
| 変更ID／作業日時 | 未採番／未設定 |
| 実施・確認・承認 | 未設定／未設定／未承認 |
| 実施結果 | **NOT RUN（実機環境なし）** |

> [!WARNING]
> Agent ISOからの導入は対象boot diskを上書きします。本書のコマンドは実行例であり、実行済みの記録ではありません。正確な4.22.z、現物、変更承認、停止・復旧計画を確認せずに実行しません。

工程は [build-flow.mmd](../diagrams/build-flow.mmd) に示します。

## 1. 役割と判定権限

| 活動 | 実施 | 確認 | Go/No-Go |
| --- | --- | --- | --- |
| DNS/LB/NTP/Firewall/Proxy | Network/Infrastructure | Platform | Network責任者 |
| Hardware/BMC/boot disk | Hardware | Platform | Hardware責任者 |
| Installer入力・ISO | Platform実施者 | 別のPlatform確認者 | Platform責任者 |
| Secret注入・証明書 | 権限保有者 | Security | Security責任者 |
| ISO boot・クラスタ導入 | Platform + Hardware | 各領域 | 変更責任者 |
| 導入後判定 | Platform | Operations/品質 | 基盤責任者 |

実施者と確認者を分け、全操作を変更ID、時刻、対象host、結果、証跡へ関連付けます。

## 2. G4構築開始判定

次の結果欄は実作業時に記録します。本版では空欄です。

| ID | 前提・承認 | 確認方法 | 必須結果 | 実績／証跡 |
| --- | --- | --- | --- | --- |
| PRE-001 | 正確な4.22.zと更新方針 | release note、lifecycle、binary版 | 承認済み | |
| PRE-002 | 6台のHCL/Firmware/資源 | vendor台帳、Red Hat catalog | 適合 | |
| PRE-003 | NIC/MAC/root disk | BMCと二者照合 | 全項目一致 | |
| PRE-004 | CIDR非重複 | IPAM/routing review | 重複なし | |
| PRE-005 | DNS正逆引き | §3.2 | 全件期待値一致 | |
| PRE-006 | LB/VIP/Firewall | §3.3、設定レビュー | 設計一致 | |
| PRE-007 | Proxy/CA/外部接続 | preflight、許可先表 | 必須宛先到達 | |
| PRE-008 | NTP | UDP/123、chrony方針 | 2系統利用可能 | |
| PRE-009 | pull secret/SSH鍵 | Secret参照ID、期限、権限 | 承認済み | |
| PRE-010 | CSI/監視/backup最低運用 | Owner、後続計画 | 受入可能 | |
| PRE-011 | 変更・停止・連絡 | 変更票、保守時間、連絡網 | 承認済み | |
| PRE-012 | 再image/復旧 | 元状態、media、担当、時間 | 復旧可能 | |

1件でも満たさなければNo-Goです。`TBD`、未確認、口頭のみをPassへ読み替えません。

## 3. 外部サービス準備

### 3.1 Ansibleの静的確認とpreflight

対象はRHEL支援サーバーのみです。RHCOSをinventoryへ加えません。

```bash
cd projects/enterprise-openshift-platform/ansible
ansible-inventory --graph
ansible-playbook --syntax-check playbooks/preflight.yml
ansible-playbook playbooks/preflight.yml
```

期待結果:

- inventoryが `lb01`、`lb02`、`bastion` のRHEL支援hostだけを示す。
- 全hostが承認済みRHEL版で、必要FQDN、wildcard、Proxy接続、chrony状態を確認できる。

中止条件:

- RHCOS/OpenShift nodeがinventoryにある。
- DNS応答が [14-parameter-sheet.md](14-parameter-sheet.md) と異なる。
- Proxy通信でTLS検証を無効化しないと成功しない。
- 既知でない時刻ずれ、到達不可、認証要求がある。

### 3.2 DNS確認

```bash
dig @192.0.2.2 api.ocp-prd.example.com A +short
dig @192.0.2.3 api-int.ocp-prd.example.com A +short
dig @192.0.2.2 canary.apps.ocp-prd.example.com A +short
dig @192.0.2.2 cp01.ocp-prd.example.com A +short
dig @192.0.2.2 -x 192.0.2.21 +short
```

期待結果は順にAPI VIP `192.0.2.10`、API VIP、Ingress VIP `192.0.2.11`、`192.0.2.21`、`cp01.ocp-prd.example.com.` です。2台のDNS、API/API-int、wildcard、全6ノードで同じ確認を行います。

### 3.3 Load Balancerの構成

初回はcheck modeとdiffを試行します。clean RHELではcheck modeがpackageを実導入しないため、HAProxy templateのvalidator `/usr/sbin/haproxy` が存在せず停止する想定です。この依存不足をvalidation無効化で回避しません。

```bash
cd projects/enterprise-openshift-platform/ansible
ansible-playbook --check --diff playbooks/configure-load-balancers.yml \
  -e external_service_change_approved=true
```

Network責任者がpackage導入を承認した後、`packages` tagだけを実適用します。この操作はcheckではなくRHEL hostを変更します。

```bash
ansible-playbook playbooks/configure-load-balancers.yml --tags packages \
  -e external_service_change_approved=true
```

validator導入後にfull check/diffを再実行します。HAProxy、keepalived、VRRP、firewall、NIC、VIP方式、差分が承認された後だけ最後の実適用へ進みます。

```bash
ansible-playbook --check --diff playbooks/configure-load-balancers.yml \
  -e external_service_change_approved=true
ansible-playbook playbooks/configure-load-balancers.yml \
  -e external_service_change_approved=true
ansible load_balancers -b -a 'haproxy -c -f /etc/haproxy/haproxy.cfg'
ansible load_balancers -b -a 'ss -lnt'
```

期待結果:

- HAProxy構文がvalidで、TCP/6443、22623、80、443をlistenする。
- 片方のLBだけがAPI/Ingress VIPを所有し、peer停止時に承認時間内で移動する。
- クラスタ起動前はbackend DOWNが想定される。これをクラスタ障害と誤判定しない。

中止条件は両LBのVIP同時所有、想定外NIC、意図しない既存設定の上書き、APIのsession persistence、L7終端、FWの過剰開放です。

## 4. 安全なInstaller作業領域

### 4.1 binaryと作業領域

対象z版の公式配布物とchecksumを取得し、組織手順で真正性を確認します。

```bash
openshift-install version
oc version --client
butane --version
```

期待結果はinstallerとpayloadが承認済み4.22.z、Butaneが `4.22.0`入力に対応することです。

暗号化されアクセス制御された領域を環境変数へ設定してから実行します。

```bash
: "${OCP_WORK_DIR:?Set OCP_WORK_DIR to the approved secure directory}"
umask 077
install -d -m 0700 "${OCP_WORK_DIR}" "${OCP_WORK_DIR}/openshift"
cp projects/enterprise-openshift-platform/install/install-config.yaml.example \
  "${OCP_WORK_DIR}/install-config.yaml"
cp projects/enterprise-openshift-platform/install/agent-config.yaml.example \
  "${OCP_WORK_DIR}/agent-config.yaml"
chmod 0600 "${OCP_WORK_DIR}/install-config.yaml" \
  "${OCP_WORK_DIR}/agent-config.yaml"
```

pull secretと承認済みSSH公開鍵を秘密管理システムから作業コピーへ注入します。Proxy CAが必要とSecurity teamが判断した場合は、対象4.22.zのschemaに従って `additionalTrustBundle`（および対象版で必要なtrust bundle policyフィールド）を `install-config.yaml` へ追加し、証明書chainと設定差分をレビューします。現在のexampleには意図的にCAを含めておらず、単にCAファイルを作業ディレクトリへ置くだけでは設定になりません。値をterminalへechoせず、session recording、shell history、CI logへの露出を防ぎます。

### 4.2 入力レビュー

```bash
yq eval '.' "${OCP_WORK_DIR}/install-config.yaml" >/dev/null
yq eval '.' "${OCP_WORK_DIR}/agent-config.yaml" >/dev/null
```

二者で次を照合します。

- `platform: none`、3 master + 3 worker、OVN-Kubernetes、3つのCIDR。
- 6つのhostname、role、MAC、IP、prefix、gateway、DNS、root device。
- rendezvousが `cp01` のIPであること。
- noProxyに空白や欠落がなく、内部CIDR/domainを含むこと。
- pull secretが有効なJSONであること。ただしレビュー証跡へ値を残さない。
- 実値をGit差分へ追加していないこと。

MAC、root device、CIDRのいずれかが台帳と一致しなければ直ちに停止します。

### 4.3 chrony MachineConfig生成

```bash
butane --strict --pretty \
  projects/enterprise-openshift-platform/install/openshift/99-master-chrony.bu.example \
  -o "${OCP_WORK_DIR}/openshift/99-master-chrony.yaml"
butane --strict --pretty \
  projects/enterprise-openshift-platform/install/openshift/99-worker-chrony.bu.example \
  -o "${OCP_WORK_DIR}/openshift/99-worker-chrony.yaml"
```

期待結果はstrict errorなしです。生成されたMachineConfigのrole、`/etc/chrony.conf`、NTP 2台をレビューします。AnsibleやSSHによるRHCOS直接変更へ置き換えません。

## 5. Agent ISO生成とGo/No-Go

### 5.1 ISO生成

```bash
openshift-install --dir "${OCP_WORK_DIR}" agent create image
sha256sum "${OCP_WORK_DIR}/agent.x86_64.iso"
```

期待結果:

- commandがerror終了せず、`agent.x86_64.iso` が生成される。
- ISO hash、installer version、設定版、変更IDを証跡へ記録する。
- `auth/`、installer state、ISOをアクセス制御し、リポジトリへcommitしない。

生成失敗時はISOを配布せず、debug logをマスクして課題化します。schema errorを無視して別apiVersionへ推測変更しません。

### 5.2 最終Go/No-Go

| 確認 | Go条件 | 記録 |
| --- | --- | --- |
| ISO | hashと版が承認値に一致 | |
| BMC対象 | 6台のserial/hostname/roleが一致 | |
| boot disk | 各nodeで二者確認済み | |
| 外部依存 | DNS/LB/NTP/Proxy/FWが合格 | |
| 監視体制 | 作業中のconsole、LB、Network監視担当が配置 | |
| 復旧 | media解除、再image、エスカレーション担当が待機 | |
| 変更 | 承認時刻内、連絡済み | |

空欄があればNo-Goです。

## 6. ISO起動と導入監視

1. Hardware実施者が承認済み6台へ同一hashのISOをBMC virtual mediaでmountします。
2. 一時bootをISOへ指定します。恒久boot orderは変更しません。
3. まずrendezvous host、続いて残るcontrol plane、workerを承認順に起動します。
4. Agent consoleでhost inventory、network、release image pull、NTP、diskのcheckを監視します。
5. connectivity checkに赤/警告がある場合、timeoutで続行させずNo-Go判断を行います。
6. NetworkManager TUIでの手動修正は構成ドリフトとなるため、緊急変更記録なしに行いません。入力を修正してISOを再生成します。

別terminalから次を実行します。

```bash
openshift-install --dir "${OCP_WORK_DIR}" agent wait-for bootstrap-complete \
  --log-level=info
openshift-install --dir "${OCP_WORK_DIR}" agent wait-for install-complete \
  --log-level=info
```

期待結果はそれぞれbootstrap完了とinstall完了です。タイムアウト、host未登録、disk mismatch、API flap、時刻異常、予期しない再起動があれば反復bootせず停止し、§10に従います。

## 7. 導入直後の確認

資格情報を保護したterminalだけで実行します。

```bash
export KUBECONFIG="${OCP_WORK_DIR}/auth/kubeconfig"
oc whoami --show-server
oc get clusterversion
oc get clusteroperators
oc get nodes -o wide
oc get csr
oc get network.config/cluster -o yaml
oc get ingresscontroller/default -n openshift-ingress-operator
oc get configs.imageregistry.operator.openshift.io/cluster -o yaml
oc get events -A --sort-by=.lastTimestamp
```

期待結果:

- serverが `https://api.ocp-prd.example.com:6443`。
- 承認済み4.22.zで、更新中ではない。
- ClusterOperatorがすべて `Available=True`、`Progressing=False`、`Degraded=False`。例外は既知課題・承認が必要。
- master 3、worker 3が意図したhostname/IP/roleで `Ready`。
- Pending CSRを理由確認なしに一括承認しない。
- CIDRとnetworkTypeがパラメータシートに一致する。
- Ingress domainとRoute admissionが正常。
- platform noneでshareable storageがまだない場合、Image Registry Operatorは`Removed`となり得る。この状態を導入失敗と混同しない一方、本番受入可能ともみなさない。

上記が合格するまでCSI、IdP、Operator、業務workloadを追加しません。採用CSI確定後、Image Registryへ承認済みproduction永続storageを構成して`Managed`へ移し、Registry Pod/PVC、image push/pull、backupを試験します。製品・StorageClassがTBDのため、本書に推測のpatchコマンドは記載しません。未構成ならG5・本番受入はNo-Goです。

## 8. サンプルworkloadの段階適用

これは導入後の非本番確認です。本番用途・性能確認ではありません。

```bash
oc kustomize projects/enterprise-openshift-platform/manifests
oc apply --dry-run=server \
  -k projects/enterprise-openshift-platform/manifests
oc diff -k projects/enterprise-openshift-platform/manifests
```

意図した新規resourceだけであり、image digest、group、Route certificate、quota、NetworkPolicyを承認した後に限り実適用します。

```bash
oc apply -k projects/enterprise-openshift-platform/manifests
oc rollout status deployment/example-web -n example-web-dev --timeout=5m
oc get pod,service,route -n example-web-dev -o wide
oc get endpointslice -n example-web-dev \
  -l kubernetes.io/service-name=example-web
```

期待結果は2 Pod Ready、Endpoint 2件、Route admitted、HTTPからHTTPSへのredirect、HTTPS応答です。詳細判定は [16-test-specification.md](16-test-specification.md) を使用します。

## 9. 完了判定

| 項目 | 必須結果 | 実績／証跡 |
| --- | --- | --- |
| installer | install-complete | |
| CV/CO/Node | 全必須条件合格 | |
| DNS/LB/Route | 正常・冗長性試験合格 | |
| Internal Image Registry | production永続storage、Managed、Pod/PVC、push/pull試験合格 | |
| Security | Secret漏えいなし、権限否定試験合格 | |
| Storage/backup | 採用方式の試験合格または受入前No-Go | |
| 記録 | 変更票、構成、hash、試験結果、課題を格納 | |

本版はすべて空欄で、完了判定は行っていません。

## 10. 中止・ログ採取・切り戻し

### 10.1 中止条件

- 対象host、MAC、root disk、IP、version、ISO hashの不一致。
- Secret漏えい、未承認変更、想定外の既存サービス影響。
- Agent connectivity check不合格、host登録欠落、API不安定、重大alert。
- データ損失、split-brain、二重VIP、復旧不能の懸念。

### 10.2 失敗時の証跡

```bash
openshift-install --dir "${OCP_WORK_DIR}" agent wait-for bootstrap-complete \
  --log-level=debug
ssh core@192.0.2.21 agent-gather -O >agent-gather.tar.xz
```

bootstrap後にAPIへ到達できる場合は、承認された保管場所で `oc adm must-gather` を実行します。採取物からtoken、cookie、kubeconfig、個人情報を除外し、`auth/` をsupport添付へ含めません。

### 10.3 段階別の戻し方

| 段階 | 戻し方 | 注意 |
| --- | --- | --- |
| 外部LB変更 | Ansibleが作成したremote backupまたは承認済み前版を復元し、HAProxy検証後にservice reload | VIPとclient影響をNetworkが確認 |
| ISO生成前後 | 配布を停止し、作業領域・ISO・資格情報を組織の安全な廃棄/失効手順へ渡す | `rm`だけで安全消去済みとはみなさない |
| disk書込み前 | bootを停止しvirtual mediaを解除、恒久boot順を確認 | 対象hostを二者確認 |
| disk書込み開始後 | installerのundoはない。media解除、ログ保全、変更責任者No-Go後に承認済みimageから再構築 | 元データがある場合は別途backupから復元 |
| workload更新 | 直前のGit commitを再適用、Deploymentは承認後に `oc rollout undo` | 初回Namespace削除は配下全消去のため別承認 |

クラスタ構築失敗時に「無理に完成させる」ことを切り戻しと呼びません。復旧点、責任者、業務影響が不明なら停止状態を維持します。

## 11. 証跡命名

`<change-id>_<step-id>_<UTC timestamp>_<artifact>` とし、例は `CHG-XXXX_PRE-005_YYYYMMDDThhmmssZ_dns.txt` です。実host/IPを含む証跡は本公開リポジトリではなくアクセス制御領域へ保存します。標準出力を保存する前にSecret、token、cookie、Authorization header、kubeconfigをマスクします。

## 12. 公式資料

- [Agent imageの生成とboot](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installing_an_on-premise_cluster_with_the_agent-based_installer/installing-with-agent-based-installer)
- [Agent-based installationの進捗確認・ログ採取](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installing_an_on-premise_cluster_with_the_agent-based_installer/installing-with-agent-based-installer)
- [MachineConfigによるchrony設定](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/machine_configuration/machine-configs-configure)

## 13. 作業記録

| 開始 | 終了 | 結果 | 変更ID | 構成commit | 証跡 | 残課題 |
| --- | --- | --- | --- | --- | --- | --- |
| | | NOT RUN | | | | |
