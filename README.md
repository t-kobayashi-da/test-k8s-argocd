# test-k8s-argocd

## ArgoCD & k8s 初学習ラーニングパス

### 1. ローカル環境準備

* [ ] Docker Desktop で Kubernetes を有効化
* [ ] `kubectl` コマンドをインストールして動作確認
* [ ] `kubectl get nodes` でクラスタが見えることを確認

### 2. 基本的な k8s 操作の体験

* [ ] `kubectl apply -f` で nginx Pod をデプロイしてみる
* [ ] Service を定義して `kubectl port-forward` でアクセス確認
* [ ] Pod の削除・再作成で k8s の挙動を観察

### 3. ArgoCD のインストール

* [ ] `kubectl create namespace argocd`
* [ ] `kubectl apply -n argocd -f install.yaml` で公式マニフェストを適用
* [ ] `kubectl port-forward svc/argocd-server -n argocd 8080:443` でUIにアクセス
* [ ] 初期パスワードでログイン

### 4. ArgoCD の基本操作

* [ ] GitHub にテスト用リポジトリを作成
* [ ] その中に `manifests/nginx.yaml` を置く
* [ ] ArgoCD の UI から Application を作成し GitHub を指定
* [ ] 自動デプロイ（Sync）を体験

### 5. Application in Application (App of Apps) の理解

* [ ] 親 Application (App of Apps) を作成
* [ ] 子 Application として 2つの簡単イメージ (例: nginx / whoami) をデプロイ
* [ ] ArgoCD UI で階層的に管理されていることを確認

### 6. GitOps ワークフロー体験

* [ ] GitHub のマニフェストを修正し、ArgoCD 側で自動反映されることを確認
* [ ] 修正がデプロイされない場合の「手動 Sync」も試す
* [ ] Rollback（GitHubを戻す→ArgoCDでSync）を試す

### 7. 学習ポイントの整理

* [ ] **k8sの基本概念**（Pod, Service, Namespace, Deployment）
* [ ] **ArgoCDの基本概念**（Application, Sync, App of Apps）
* [ ] **GitOpsの流れ**（GitHub更新→ArgoCD Sync→クラスタ反映）
