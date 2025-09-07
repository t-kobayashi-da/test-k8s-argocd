## 3. ArgoCD のインストール（Docker Desktop + k8s）

### Step 1: ArgoCD 用 Namespace 作成

```bash
kubectl create namespace argocd
```

* Namespace を分けることで k8s 内の ArgoCD 関連リソースをまとめられる
* 確認：

```bash
kubectl get namespaces
```

---

### Step 2: ArgoCD マニフェスト適用

公式マニフェストで一括インストールします。

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

* Deployment、Service、ConfigMap など必要なリソースがまとめて作成される
* 状態確認：

```bash
kubectl get pods -n argocd
kubectl get svc -n argocd
```

> `argocd-server` が立ち上がるまで少し時間がかかります

---

### Step 3: ArgoCD UI にアクセス

1. ポートフォワード

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

2. ブラウザでアクセス

```
https://localhost:8080
```

3. 初期ログイン情報

* ユーザー名：`admin`
* パスワード：初期は `admin` Pod のシークレットに入っている

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

---

### Step 4: ArgoCD CLI インストール（任意）

* macOS / Homebrew でインストール可能

```bash
brew install argocd
```

* CLI でログイン

```bash
argocd login localhost:8080
# ユーザー名 admin、パスワードは上記で取得
```

---

### 完了条件

* ブラウザで ArgoCD UI にアクセスできる
* CLI で `argocd version` が確認できる
* `argocd get app` などで後でアプリ管理が可能
