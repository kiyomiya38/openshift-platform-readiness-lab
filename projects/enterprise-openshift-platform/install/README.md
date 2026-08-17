# Agent-based Installer 入力例

## 状態と安全上の注意

このディレクトリは `Example Enterprise OpenShift 基盤導入` の机上演習資材です。実機での生成・起動・導入は**未実施**です。`*.example` はそのまま投入する完成品ではありません。

- `install-config.yaml.example` の pull secret と SSH 公開鍵はダミーです。実値は秘密管理システムから、アクセス制御した一時作業ディレクトリへ注入します。
- `agent-config.yaml.example` のIPアドレスとMACアドレスは文書用です。NIC名、MAC、root device、疎通性をBMC画面と現物で照合します。
- Agent-based Installerの設定入力には、対象の4.22.zと同じリリースの `openshift-install` を使用します。
- インストーラーは入力を変換・消費し、`auth/` に資格情報を生成します。Git作業ツリーでは実行せず、承認済みの暗号化された作業領域へコピーして実行します。
- `platform: none` では、API/Application Ingress用の外部DNSと外部Load Balancerを事前に準備します。VIPの所有方式はネットワーク担当との未確定事項です。
- `openshift/*.bu.example` はButane入力例です。対象リリース用ButaneでMachineConfigへ変換し、差分レビュー後にインストール作業ディレクトリの `openshift/` へ置きます。

## ファイル対応

| ファイル | 目的 | 現在の状態 |
| --- | --- | --- |
| `install-config.yaml.example` | クラスタ全体の入力 | 架空値・Secret要置換 |
| `agent-config.yaml.example` | 6ノードの役割と静的ネットワーク | 架空値・現物照合前 |
| `dns/*.example` | 正引き・逆引きレコードの申請例 | DNS投入未実施 |
| `haproxy/haproxy.cfg.example` | API/MCS/IngressのL4振り分け例 | 検証未実施 |
| `haproxy/keepalived-*.conf.example` | 2つのVIPをLBノード間で移動する例 | 方式承認前 |
| `openshift/*.bu.example` | RHCOSのchrony設定例 | Butane変換未実施 |

## 公式資料

- [OpenShift 4.22: Agent-based Installerによるカスタマイズ導入](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/installing_an_on-premise_cluster_with_the_agent-based_installer/installing-with-agent-based-installer)
- [OpenShift 4.22: MachineConfigによるchrony設定](https://docs.redhat.com/en/documentation/openshift_container_platform/4.22/html/machine_configuration/machine-configs-configure)

