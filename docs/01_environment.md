## 1. ローカル環境準備（手を動かす内容）

### Step 1: Docker Desktop で Kubernetes を有効化

1. Docker Desktop を開く
2. メニューから **Settings → Kubernetes** を選択
3. **Enable Kubernetes** にチェックを入れる
4. **Apply & Restart** を押す
   → 再起動後、Kubernetes クラスタが Docker Desktop 上に立ち上がる

---

### Step 2: kubectl コマンドをインストール

1. macOS なら Homebrew でインストール

   ```bash
   brew install kubectl
   ```
2. バージョン確認

   ```bash
   kubectl version --client
   ```

   → クライアントバージョンが表示されればOK

---

### Step 3: Kubernetes クラスタへの接続確認

1. Docker Desktop の設定で Kubernetes が立ち上がったら、クラスタを確認

   ```bash
   kubectl config get-contexts
   ```

   → `docker-desktop` があることを確認

2. ノードが見えるかチェック

   ```bash
   kubectl get nodes
   ```

   → `docker-desktop Ready` と出ればOK

---

### 完了条件

* Docker Desktop 上で Kubernetes が有効化されている
* `kubectl` コマンドが使える
* `kubectl get nodes` で `docker-desktop` が Ready になっている

---

## 🔹 Kubernetes クラスタの概念（最低限）

Kubernetes (k8s) は **コンテナを効率よく動かすためのオーケストレーションツール**。
クラスタは大きく 2つの役割に分かれています。

### 1. コントロールプレーン (Control Plane)

クラスタ全体を管理する中枢。

* **API Server**: kubectl や ArgoCD からのリクエストを受ける入口
* **etcd**: クラスタの設定・状態を保存する分散DB
* **Scheduler**: どのノードでPodを動かすか決める
* **Controller Manager**: Pod数が減ったら復旧するなどの調整

（Docker Desktopの場合、このコントロールプレーンは1台のノードに全部入っている）

---

### 2. ワーカーノード (Worker Nodes)

実際にコンテナ（Pod）を動かすところ。

* **kubelet**: ノード上でPodを管理するエージェント
* **kube-proxy**: ネットワークをつなぐ役割
* **Container Runtime**: Docker や containerd など、コンテナを実際に動かす

（Docker Desktopのクラスタは最小構成なので **1台に全部入り**）

---

### 3. Pod とは？

* k8sで最小のデプロイ単位
* 基本は「1 Pod = 1コンテナ」だけど、密結合コンテナなら複数もOK
* Podは寿命が短く、消えたり再生成されたりする → 「サービス（Service）」で安定した入口を作る

---

## 🔹 Step1 で押さえておくポイント

1. **kubectl がクラスタの窓口**

   * `kubectl` コマンド = Control Plane の API Server へのリモコン
   * 実際の状態確認はいつも `kubectl` から

2. **docker-desktop クラスタは学習用の最小構成**

   * Control Plane と Worker が同じ1ノードに入っている
   * 本番では Control Plane 複数 + Worker 複数 の分散構成が普通

3. **クラスタの状態確認の基本コマンド**

   * `kubectl get nodes` → ノードの一覧
   * `kubectl get pods -A` → 全NamespaceのPod確認（システムPod含む）
   * `kubectl describe node docker-desktop` → ノードの詳細

### クラスタ構成のイメージ図

```
Kubernetes Cluster
│
├─ Control Plane (管理層)
│   ├─ API Server       ← kubectl / ArgoCD からの操作受付窓口
│   ├─ etcd             ← クラスタ状態を保存するDB
│   ├─ Scheduler        ← Podをどのノードに置くか決定
│   └─ Controller Manager ← Pod数や状態を自動調整
│
└─ Worker Node(s) (実働層)
    ├─ kubelet          ← Podの状態管理エージェント
    ├─ kube-proxy       ← ネットワーク接続を管理
    └─ Container Runtime← Docker / containerd がコンテナを実行
        │
        └─ Pod(s)
            ├─ Pod A → コンテナ1 (例: nginx)
            └─ Pod B → コンテナ2 (例: whoami)
```

### ポイント

* クラスタ = Control Plane + Worker Node の集合
* Pod = 最小デプロイ単位（1 Podに1コンテナ以上も可）
* Control Plane は Worker Node 上に1台でも、複数ノードに分かれても構わない
