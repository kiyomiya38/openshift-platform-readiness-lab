# 検証チェックリスト（解答例）

- [ ] Project `os-basic-lab` が Active である。
- [ ] Deployment の Available replica が 1 である。
- [ ] Pod が Ready で、適用 SCC annotation と非 0 の実行 UID を確認できる。
- [ ] Service selector と Pod label が一致する。
- [ ] EndpointSlice に Ready な endpoint が一つ以上ある。
- [ ] Route の `Admitted` condition が True である（Controller がある環境）。
- [ ] ConfigMap の非機密設定を確認できる。
- [ ] Secret 値を表示・記録していない。
- [ ] Role は namespaced read-only で、Secret の read 権限を含まない。
- [ ] 異常 Event がある場合は原因と環境依存事項を記録した。

## 要確認になり得る項目

- Image tag/digest、外部 Registry 到達性
- Route の証明書、DNS、Ingress Controller、外部 LB
- クラスタ既定 SCC と組織の RBAC ポリシー
- 利用可能な Operator と閲覧権限
