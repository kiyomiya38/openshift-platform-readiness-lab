# Linux baseline Playbook 解説

| 要素 | このラボでの役割 | 冪等性の観点 |
|---|---|---|
| Inventory | 対象と接続変数を定義 | 対象を狭くし `--limit` で再確認 |
| `package` | chrony/firewalld を導入 | installed なら変更なし |
| `user` | 非 login の教材 account | 属性が同じなら変更なし |
| `copy` | 固定の教材 marker を配置 | checksum が同じなら変更なし |
| `template` | NTP source から設定生成 | 出力差分がある時だけ changed |
| `service` | enabled/started を保証 | 既に desired state なら変更なし |
| `firewalld` | 教材 port を許可 | 同じ rule があれば変更なし |
| handler | chrony 設定変更時だけ再起動 | notify されなければ動かない |

## 注意点

- check mode は module と環境により予測精度が異なり、接続性や runtime 動作までは保証しません。
- chrony server、firewalld zone、package repository は環境依存のため要確認です。
- OpenShift Node は MachineConfig 等で管理されるため、この Playbook の直接適用対象にしません。
