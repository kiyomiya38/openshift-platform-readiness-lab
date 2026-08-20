# Sample Report 03: PVC 未割当

## 概要・影響

- 新規 `report-db-0` だけが Pending。既存 workload 影響なし。
- PVC `report-db-data` が Pending のため Pod scheduler は unbound PVC と記録。

## 原因

- PVC は `fast-rwx` を要求するが、その StorageClass は存在しない。
- CSI provisioner/backend へ provisioning request が渡る前の参照不整合であり、提示証跡から CSI 障害とは判断できない。
- `shared-rwx` が代替候補だが、性能、support、reclaim/backup、Application の RWX 要件は要確認。

## 対応案

- PVC/PV を削除せず、設計 owner と Application owner が正しい StorageClass と AccessMode を確定する。
- Manifest の class 名を修正できるか、PVC の immutable field と StatefulSet 運用を対象版で確認し、変更計画を作る。
- 修正後は PVC Bound、PV/CSI Event、Pod Ready、mount、read/write、backup を確認する。

## 再発防止

- CI で環境ごとの許可 StorageClass 一覧と Manifest を照合する。
- StorageClass catalog に性能、AccessMode、backup、owner、廃止手順を記載する。
