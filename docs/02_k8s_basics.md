## 2. 基本的な k8s 操作の体験（手を動かす内容）

### Step 1: nginx Pod のデプロイ

1. マニフェストファイルを作成

```yaml
# 02-1_nginx-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

2. Pod を作成

```bash
kubectl apply -f 02-1_nginx-pod.yaml
```

3. 状態確認

```bash
kubectl get pods
kubectl describe pod nginx-pod
```

---

### Step 2: Service を作ってアクセス可能にする

1. Service マニフェスト作成

```yaml
# 02-2_nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 8080       # 外部に開くポート
      targetPort: 80   # PodのコンテナPort
  type: ClusterIP      # ローカルテスト用。NodePortでも可
```

2. Service 作成

```bash
kubectl apply -f 02-2_nginx-service.yaml
```

3. 状態確認
```bash
kubectl get svc
kubectl describe svc nginx-service
```

4. ポートフォワードでブラウザ確認

```bash
kubectl port-forward svc/nginx-service 8080:8080
```

ブラウザで `http://localhost:8080` にアクセス → nginxの初期画面が見えるはず

---

### Step 3: Pod の削除・再作成

```bash
kubectl delete pod nginx-pod
kubectl get pods  # Podが消えたのを確認
kubectl apply -f 02-1_nginx-pod.yaml
kubectl get pods  # Podが再作成されたことを確認
```

> ポイント: Podは一時的な存在。DeploymentやReplicaSetで安定稼働させるのが一般的

---

### ポイントまとめ
- Pod = 実際にコンテナが動く単位
- Service = Pod への安定したアクセス入口
- Service を作っただけでは Pod は増えない
- Pod を作ってから Service を作るのが基本の順序

---

### Step 4: まとめ

* `kubectl apply -f` でマニフェストから Pod/Service を作れる
* `kubectl get` / `kubectl describe` で状態確認
* `kubectl port-forward` でローカルからアクセス
* Podは一時的、Serviceが安定した入口になる

---

## Pod に関係する主な Kubernetes オブジェクト

### 1. **Deployment**

* Pod を **複数レプリカで安定稼働** させる
* Pod が落ちても自動で再作成される
* 更新（image 変更など）も簡単に管理可能

```text
Deployment
└─ Pod (複数)
```

---

### 2. **ReplicaSet**

* Deployment が内部で作るオブジェクト
* 指定した数の Pod を常に維持する
* 基本的には **直接触ることは少ない**

```text
ReplicaSet
└─ Pod
```

---

### 3. **StatefulSet**

* Deployment に似ているが **順序や名前が重要な Pod** 向け
* データベースや永続的なストレージ付きアプリで使用

```text
StatefulSet
└─ Pod-0
└─ Pod-1
└─ Pod-2
```

---

### 4. **DaemonSet**

* 各ノードに 1 Pod を必ず配置したい場合に使用
* 例：ログ収集、監視エージェント

```text
DaemonSet
└─ Pod (Node1)
└─ Pod (Node2)
└─ Pod (Node3)
```

---

### 5. **Job / CronJob**

* 一度だけ実行する Pod → Job
* 定期実行する Pod → CronJob

```text
Job
└─ Pod (1回実行)

CronJob
└─ Job
   └─ Pod (スケジュール実行)
```

---

### 6. **ConfigMap / Secret**

* Pod 内の環境設定やシークレットを渡す
* Pod 自体ではないが、Pod の挙動に強く関係

```text
Pod
└─ uses ConfigMap / Secret
```

---

### 7. **ServiceAccount / Role / RoleBinding**

* Pod に権限を与えるためのオブジェクト
* Kubernetes API を操作する Pod はこれで認可管理

---

### 🔹 まとめ

| 種類                                  | 目的                 |
| ----------------------------------- | ------------------ |
| Pod                                 | 最小デプロイ単位、コンテナ実行    |
| Service                             | Pod への安定したアクセス入口   |
| Deployment                          | Pod のレプリカ管理・自動更新   |
| StatefulSet                         | 順序や永続性が必要な Pod の管理 |
| DaemonSet                           | 各ノードに必ず 1 Pod 配置   |
| Job/CronJob                         | 一度だけ or 定期実行する Pod |
| ConfigMap/Secret                    | Pod の設定や機密情報管理     |
| ServiceAccount / Role / RoleBinding | Pod の権限管理          |

---

💡 ポイント

* Pod 単体で使うのは学習初期だけ
* 実運用では Deployment + Service がほぼセット
* StatefulSet / DaemonSet / Job は目的に応じて学ぶ

---

### Pod と関連オブジェクトの関係図

```
Kubernetes Cluster
│
├─ Deployment
│   └─ ReplicaSet
│       └─ Pod (複数レプリカ)
│           ├─ uses ConfigMap
│           ├─ uses Secret
│           └─ uses ServiceAccount / Role / RoleBinding
│
├─ StatefulSet
│   └─ Pod-0
│   └─ Pod-1
│   └─ Pod-2
│       ├─ uses ConfigMap
│       └─ uses Secret
│
├─ DaemonSet
│   └─ Pod (Node1)
│   └─ Pod (Node2)
│   └─ Pod (Node3)
│
├─ Job
│   └─ Pod (1回実行)
│
└─ CronJob
    └─ Job
        └─ Pod (スケジュール実行)

Service
└─ Pod (ラベルで紐づくPodへのアクセス入口)
```

### ポイント

1. **Pod が最小単位**
2. **Deployment / StatefulSet / DaemonSet / Job が Pod の生成や管理を担当**
3. **Service が Pod への安定したアクセス入口**
4. **ConfigMap / Secret / ServiceAccount などが Pod に設定や権限を渡す**

---

💡 補足

* 実際のクラスタでは Pod は短命で消えたり再生成されたりするので、Deployment などで管理するのが基本
* Service は Pod の寿命に関係なくアクセスできる安定入口
