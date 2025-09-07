## 4. ArgoCD の基本操作（Application 作成）

### Step 1: GitHub にテスト用リポジトリ作成

1. GitHub 上で新規リポジトリ作成

   * 名前例：`k8s-argocd-test`
   * Private でも Public でも OK
2. ローカルでクローン

```bash
git clone <リポジトリURL>
cd k8s-argocd-test
```

---

### Step 2: manifests ディレクトリに nginx マニフェスト作成


manifests/04_argocd/nginx.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

```bash
git add .
git commit -m "Add nginx manifest"
git push
```

---

### Step 3: ArgoCD で Application 作成

1. ArgoCD UI で **NEW APP**

   * ログインは 03 参照

2. 設定項目

   * **Application Name:** `nginx-test`
   * **Project:** `default`
   * **Repository URL:** `https://github.com/t-kobayashi-da/test-k8s-argocd.git`
   * **Path:** `manifests/04_argocd`
   * **Cluster:** `https://kubernetes.default.svc`（local k8s クラスタ）
   * **Namespace:** `default`
   * **Sync Policy:** Manual / Automatic（最初は Manual で確認してから Automatic に切り替え可能）

3. **CREATE** を押す

---

### Step 4: SYNC & デプロイ確認

1. Application の画面で **SYNC** ボタンを押す
2. Pod が作成されるか確認

```bash
kubectl get pods
kubectl get svc
```

3. Sync 状態が緑色になれば成功

---

### Step 5: 自動デプロイ（オプション）

* Application の **Sync Policy → Automatic** に変更
* GitHub リポジトリで `nginx.yaml` を変更 → ArgoCD が自動で反映

---

💡 ポイント

* **Application = Git リポジトリと k8s クラスタをつなぐ窓口**
* 初めての Application 作成で、**GitHub 更新 → ArgoCD SYNC → Pod デプロイ** の GitOps 流れを体験

---

## 4 の後片付け（ArgoCD + nginx Application）

### 1. ArgoCD 経由で作った Application 削除

* UI から Application を選択 → **DELETE**
* CLI からでも削除可能

```bash
argocd app delete nginx-test --cascade
```

> `--cascade` を付けると Application によって作られたリソース（Pod / Service など）もまとめて削除

---

### 2. nginx Pod / Service が残っていないか確認

```bash
kubectl get pods
kubectl get svc
```

* 残っていれば手動で削除

```bash
kubectl delete pod <pod名>
kubectl delete svc <svc名>
```

---

### 3. （任意）ArgoCD 自体の削除

* ArgoCD もクリーンに削除したい場合

```bash
kubectl delete namespace argocd
```

* 削除後に Namespace と Pod がなくなったことを確認

```bash
kubectl get namespaces
kubectl get pods --all-namespaces
```

---

### 4. 後片付けのポイント

* Application は **ArgoCD で作ったものだけ削除すれば OK**
* ArgoCD を残しておけば Step 5 以降でも使える
* k8s クラスタ自体は Docker Desktop の設定を消さなくても問題なし

---

💡 補足

* 後で Step 5 の App-of-Apps を試す場合は **ArgoCD は残しておく** のがオススメ
* nginx-test などのテストアプリだけ削除してクリーンにしておくイメージ

---

## ArgoCD の役割

* **GitOps のコントローラ**

  * Git リポジトリに置いたマニフェストを読み込む
  * Kubernetes クラスタ上の状態を Git と一致させる
* **UI の役割**

  * Application の状態確認（Pods、Sync 状態、差分）
  * SYNC（Git の内容をクラスタに反映）
  * 差分がある場合は手動で反映するか、自動 Sync を設定できる

---

### できること / できないこと

| 項目                            | ArgoCD UI で可能か | コメント                                            |
| ----------------------------- | -------------- | ----------------------------------------------- |
| Pod の数を直接変更                   | ❌              | Replica 数の変更は Deployment マニフェストを修正して Git に push |
| マニフェストの変更                     | ❌              | Git 上のファイルを修正する必要あり                             |
| Sync / Rollback               | ✅              | Git に合わせてクラスタ状態を揃える                             |
| 状態確認（Pods、Service）            | ✅              | Pod が何個稼働しているかなど確認可能                            |
| Application 階層管理（App-of-Apps） | ✅              | UI で親子関係の確認可能                                   |

---

### まとめ

* **ArgoCD はクラスタを「コードベースの状態に一致させる管理者」**
* UI はあくまで **監視と同期操作の窓口**
* Pod 数やイメージ変更などは **Git リポジトリを更新 → ArgoCD で SYNC** という流れで管理

---

💡 ポイント

* この制約を理解すると、なぜ GitOps が安全で再現性が高いかもわかります
* UI から Pod を直接いじる運用は **Git の状態とクラスタの状態が乖離するので NG**
