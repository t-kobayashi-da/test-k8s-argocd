## Step 5: App-of-Apps（親子 Application）体験

### 前提

* Docker Desktop の Kubernetes が有効
* `kubectl` でクラスタ確認済み
* ArgoCD はまだインストールしていない（App-of-Apps で自分も管理する構成）

---

### ディレクトリ構成（GitHub リポジトリ例）

```
k8s-argocd-apps/
├─ argocd/
│   └─ install.yaml          # ArgoCD 自身の Deployment/Service
├─ nginx/
│   └─ nginx.yaml            # nginx Deployment/Service
└─ parent-app.yaml           # 親 Application
```

---

#### 1. nginx 子アプリ（nginx/nginx.yaml）

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: default
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
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: default
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

---

#### 2. ArgoCD 子アプリ（argocd/install.yaml）

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: argocd
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: argocd-server
  namespace: argocd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: argocd-server
  template:
    metadata:
      labels:
        app: argocd-server
    spec:
      containers:
      - name: argocd-server
        image: argoproj/argocd:v2.9.13
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: argocd-server
  namespace: argocd
spec:
  selector:
    app: argocd-server
  ports:
    - protocol: TCP
      port: 443
      targetPort: 8080
  type: ClusterIP
```

> ⚠ 注意
>
> * Docker Desktop の k8s は最小リソースなので、本番環境の ArgoCD install.yaml ほど複雑にせず簡易化しています。
> * 外部アクセス（UI）は `kubectl port-forward svc/argocd-server -n argocd 443:8080` で確認可能です。

---

#### 3. 親 App-of-Apps（parent-app.yaml）

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: parent-app
  namespace: default
spec:
  project: default
  source:
    repoURL: <GitHubリポジトリURL>
    targetRevision: main
    path: .
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

* 親 Application は **親ディレクトリ（`.`）を参照**
* ディレクトリ内の `argocd/` と `nginx/` がそれぞれ子アプリとして認識される
* 自動 Sync で親 → 子が順にデプロイされる

---

💡 ポイント

1. **親アプリが Git リポジトリを監視**
2. **子アプリがそれぞれ Pod/Service を作成**
3. **ArgoCD 自身も Git で管理可能** → 「ArgoCD の状態もコードで管理」できる
4. Docker Desktop でも小規模なら動作確認可能（リソース注意）

---

### Step 5-1: GitHub リポジトリに push

```bash
git init
git add .
git commit -m "Add App-of-Apps structure with ArgoCD and nginx"
git branch -M main
git remote add origin <GitHubリポジトリURL>
git push -u origin main
```

* リポジトリ URL を後で親 Application の `repoURL` に指定

---

### Step 5-2: 親 Application の作成

```yaml
# parent-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: parent-app
  namespace: default
spec:
  project: default
  source:
    repoURL: <GitHubリポジトリURL>
    targetRevision: main
    path: .
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

* `path: .` にしてディレクトリ直下の `argocd/` と `nginx/` を子として管理

---

### Step 5-3: 親 Application を ArgoCD で作成

1. **UI で NEW APP**

   * Name: `parent-app`
   * Project: `default`
   * RepoURL: GitHub リポジトリ
   * Path: `.`
   * Cluster: `https://kubernetes.default.svc`
   * Namespace: `default`
   * Sync Policy: Manual（最初は手動で確認）
2. CREATE → SYNC

* この SYNC によって **ArgoCD 自身（argocd/install.yaml）** と **nginx（nginx/nginx.yaml）** が順にデプロイされる

---

### Step 5-4: デプロイ状況確認

```bash
# Namespace 確認
kubectl get namespaces

# ArgoCD Pod
kubectl get pods -n argocd

# nginx Pod
kubectl get pods -n default

# サービス確認
kubectl get svc -n argocd
kubectl get svc -n default
```

* ArgoCD UI はポートフォワードで確認可能

```bash
kubectl port-forward svc/argocd-server -n argocd 443:8080
```

* ブラウザで `https://localhost:8080`

---

### Step 5-5: 体験ポイント

1. 親 Application を SYNC → 子アプリが順にデプロイ
2. 子 Application のマニフェストを変更 → 親を SYNC → 自動反映
3. ArgoCD 自身も Git で管理される状態を体験

---

💡 ポイント

* **App-of-Apps = 階層的 GitOps 管理**
* Docker Desktop 上でも小規模なら十分確認可能
* 後で Helm や複数サービスに拡張可能

---

必要であれば、**この最小構成で動かす YAML ファイルの完全版（そのままコピー可能）** も作れます。
作りますか？
