# Case 03: PVC 未割当

> 架空・匿名化済み教材。新しい StatefulSet の Pod が Pending。既存 workload には影響なし。

## 証跡

```text
$ oc get pod,pvc -n data-lab
NAME             READY   STATUS    AGE
pod/report-db-0  0/1     Pending   7m

NAME                                  STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS
persistentvolumeclaim/report-db-data  Pending                                      fast-rwx
```

```text
$ oc describe pvc report-db-data -n data-lab
StorageClass:  fast-rwx
Events:
  Warning  ProvisioningFailed  storageclass.storage.k8s.io "fast-rwx" not found
```

```text
$ oc get storageclass
NAME                  PROVISIONER                  DEFAULT
standard-rwo (default) csi.example.invalid         true
shared-rwx             csi.example.invalid         false
```

```text
$ oc describe pod report-db-0 -n data-lab
Events:
  Warning FailedScheduling  0/3 nodes are available: pod has unbound immediate PersistentVolumeClaims.
```

## 制約

- Application は RWX を要件としているが、要件の妥当性と `shared-rwx` の性能は要確認。
- PVC/PV の削除は認められていない。

## 課題

1. Pod Pending と PVC Pending の因果を説明する。
2. 「CSI 障害」と断定できない根拠を書く。
3. 設計・Manifest・運用のどこを恒久対応にするか提案する。
