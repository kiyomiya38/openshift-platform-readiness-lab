# 実行結果の見方

## 1 回目

- `ok`: 既に desired state、または read-only task
- `changed`: package、file、user、service、firewall の変更
- `failed`: 接続、sudo、repository、service 名、firewall zone、template の問題を事実から切り分ける
- `unreachable`: SSH/DNS/route/firewall/user/key を確認し、Playbook の task failure と区別する

## 2 回目

外部から設定が変化していなければ `changed=0` を期待します。異なる場合は `--diff -vv` の機密情報に注意しながら、どの task が常に変わるか確認します。

## 記録する項目

1. Inventory graph と対象 host
2. `--syntax-check`、`--list-hosts` の結果
3. `packages` tag の check/apply と、`--skip-tags packages --check --diff` の結果
4. Playbook 全体を通常実行した recap
5. 2 回目の recap
6. handler が動いた条件
7. 環境依存で `要確認` とした値
