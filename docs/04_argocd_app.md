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
