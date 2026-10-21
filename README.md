# Kubernetes 实战作业：部署并暴露 NGINX 应用

 

> 内容包含：题目、答案、详细解析、命令示例和常见输出示例，便于你在本地 K8S 环境中部署和学习。

 

---

 

## 一、学习目标

 

目标是：

 

- 理解 Kubernetes 中核心资源的作用

- 使用 Namespace 隔离应用环境

- 部署 NGINX Pod

- 通过 Deployment 管理多副本应用

- 使用 Service 暴露应用内部访问入口

- 使用 Ingress 对外提供 HTTP 路由

- 学会查看资源、排查问题、扩容和滚动更新

 

---

 

## 二、核心概念：答案 + 解析

 

### 1. Pod

 

定义：

- Pod 是 Kubernetes 中最小的部署单元。

- 一个 Pod 通常运行一个或多个容器，并共享网络和存储。

 

作用：

- 承载应用容器

- 是调度和运行的最小单位

- 所有容器在同一个 Pod 中共享 IP、网络空间等资源

 

示例：

- NGINX 容器运行在一个 Pod 中

 

### 2. Deployment

 

定义：

- Deployment 用于声明式管理一组 Pod。

 

作用：

- 保证指定副本数始终存在

- 执行滚动更新

- 处理故障恢复

- 扩缩容

 

为什么重要：

- 生产环境里不建议直接手工维护单个 Pod，Deployment 更适合管理应用。

 

### 3. Service

 

定义：

- Service 是一组 Pod 的稳定访问入口。

 

作用：

- 为后端 Pod 提供统一访问地址

- 通过 label selector 选择目标 Pod

- 让 Pod 变化不会影响应用访问地址

 

为什么需要：

- Pod IP 是动态的，Service 提供稳定的虚拟 IP（ClusterIP）

 

### 4. Ingress

 

定义：

- Ingress 用于管理外部流量进入集群的规则。

 

作用：

- 路由 HTTP/HTTPS 请求

- 可按 Host、Path 分发到不同 Service

- 适合对外应用访问

 

例如：

- `nginx.local` -> `nginx-service`

 

### 5. Namespace

 

定义：

- Namespace 是 Kubernetes 的资源隔离空间。

 

作用：

- 将项目/环境/团队隔离

- 避免命名冲突

- 便于资源管理与权限控制

 

例如：

- dev、test、prod、training

 

### 6. Node

 

定义：

- Node 是 Kubernetes 集群中的工作节点。

 

作用：

- 执行 Pod

- 承载容器运行时

- 与控制平面协同调度任务

 

### 7. Cluster

 

定义：

- Cluster 是由多个 Node + 控制平面组成的整体。

 

作用：

- 统一管理和编排容器应用

- 提供高可用、扩展和容错能力

 

---

 

## 三、Task 2：创建 Namespace

 

### 题目要求

创建一个 Namespace：`training`

 

### YAML 文件：`namespace.yaml`

 

```yaml

apiVersion: v1

kind: Namespace

metadata:

  name: training

```

 

### 执行命令

 

```bash

kubectl apply -f namespace.yaml

kubectl get namespace training

```

 

### 参考输出示例

 

```bash

NAME       STATUS   AGE

training   Active   10s

```

 

### 解释

- `kubectl apply` 表示创建或更新对象

- `kubectl get namespace training` 用于验证该命名空间是否存在

 

---

 

## 四、Task 3：创建一个 NGINX Pod

 

### 题目要求

- Pod 名称：`nginx-pod`

- 命名空间：`training`

- 镜像：`nginx:1.25`

- 端口：80

- 标签：`app: nginx`

 

### YAML：`pod.yaml`

 

```yaml

apiVersion: v1

kind: Pod

metadata:

  name: nginx-pod

  namespace: training

  labels:

    app: nginx

spec:

  containers:

    - name: nginx

      image: nginx:1.25

      ports:

        - name: http

          containerPort: 80

          protocol: TCP

```

 

### 执行命令

 

```bash

kubectl apply -f pod.yaml

kubectl get pod -n training

kubectl describe pod nginx-pod -n training

```

 

### 参考输出示例

 

```bash

NAME        READY   STATUS    RESTARTS   AGE

nginx-pod   1/1     Running   0          15s

```

 

### 问题 1：Pod 的状态是什么？

 

答案：

- 通常为 `Running`

- 说明容器已经成功启动并在运行中

 

### 问题 2：使用的容器镜像是什么？

 

答案：

- `nginx:1.25`

 

### 问题 3：`app: nginx` 标签的作用是什么？

 

答案：

- 它用于标记该 Pod 属于 nginx 应用

- Service 和 Deployment 都可以通过 selector 找到这些 Pod

- 例如：

 

```yaml

selector:

  matchLabels:

    app: nginx

```

 

---

 

## 五、Task 4：创建 Deployment

 

### 题目要求

- Deployment 名称：`nginx-deployment`

- Namespace：`training`

- 副本数：3

- 镜像：`nginx:1.25`

- 容器端口：80

- Pod 标签：`app: nginx`

 

### YAML：`deployment.yaml`

 

```yaml

apiVersion: apps/v1

kind: Deployment

metadata:

  name: nginx-deployment

  namespace: training

  labels:

    app: nginx

spec:

  replicas: 3

  selector:

    matchLabels:

      app: nginx

  strategy:

    type: RollingUpdate

  template:

    metadata:

      labels:

        app: nginx

    spec:

      containers:

        - name: nginx

          image: nginx:1.25

          ports:

            - name: http

              containerPort: 80

              protocol: TCP

          readinessProbe:

            httpGet:

              path: /

              port: http

            initialDelaySeconds: 5

            periodSeconds: 10

          livenessProbe:

            httpGet:

              path: /

              port: http

            initialDelaySeconds: 15

            periodSeconds: 20

```

 

### 执行命令

 

```bash

kubectl apply -f deployment.yaml

kubectl get deployment -n training

kubectl get pods -n training

```

 

### 参考输出示例

 

```bash

NAME                 READY   UP-TO-DATE   AVAILABLE   AGE

nginx-deployment     3/3     3            3           20s

 

NAME                                 READY   STATUS    RESTARTS   AGE

nginx-deployment-65d5b7f56d-abcde    1/1     Running   0          20s

nginx-deployment-65d5b7f56d-fghij    1/1     Running   0          20s

nginx-deployment-65d5b7f56d-klmno    1/1     Running   0          20s

```

 

### 问题 4：有多少个 Pod 正在运行？

 

答案：

- 3 个 Pod

 

### 问题 5：如果删掉其中一个 Pod 会发生什么？

 

答案：

- Deployment 会自动检测到期望副本数和实际副本数不一致

- 它会创建一个新的 Pod，确保仍然维持 3 个副本

 

### 问题 6：Deployment selector 的作用是什么？

 

答案：

- 用于匹配被 Deployment 管理的 Pod

- 让 Deployment 能知道哪些 Pod 属于自己，以便更新、扩容和修复

 

### 问题 7：readiness 和 liveness probes 的目的是什么？

 

答案：

- readinessProbe：表示 Pod 是否已经“准备好接收流量”

- livenessProbe：表示容器是否还活着，如果不活跃则重启它

 

### 中文解释

在生产环境中，单纯启动容器并不表示应用已经好用。`readiness` 可以避免流量提前打到还没准备好的应用，而 `liveness` 则保证应用挂掉时能自动恢复。

 

---

 

## 六、Task 5：创建 Service

 

### 题目要求

- Service 名称：`nginx-service`

- Namespace：`training`

- 类型：`ClusterIP`

- 端口：80

- 目标端口：80

- 选择器：`app: nginx`

 

### YAML：`service.yaml`

 

```yaml

apiVersion: v1

kind: Service

metadata:

  name: nginx-service

  namespace: training

  labels:

    app: nginx

spec:

  type: ClusterIP

  selector:

    app: nginx

  ports:

    - name: http

      protocol: TCP

      port: 80

      targetPort: http

```

 

### 执行命令

 

```bash

kubectl apply -f service.yaml

kubectl get service -n training

kubectl describe service nginx-service -n training

kubectl get endpoints nginx-service -n training

```

 

### 参考输出示例

 

```bash

NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE

nginx-service   ClusterIP   10.96.123.45   <none>        80/TCP    15s

 

NAME            ENDPOINTS

```

 

### 问题 8：Service 如何选择正确的 Pod？

 

答案：

- 通过 `selector: app: nginx`

- 只要 Pod 上有这个标签，就会被 Service 选中

 

### 问题 9：`port` 和 `targetPort` 的区别是什么？

 

答案：

- `port`：Service 暴露出来的端口

- `targetPort`：后端 Pod 容器真正监听的端口

 

例如：

- Service 对外是 `80`

- Pod 容器里的 NGINX 也监听 `80`

- 因此可写成 `port: 80` 和 `targetPort: 80`

 

### 问题 10：为什么即使 Pod 已有 IP，Service 仍然需要？

 

答案：

- Pod 的 IP 不是稳定的，Pod 会被重建、迁移、扩缩容

- Service 提供稳定入口，客户端不需要关心底层 Pod 的动态变化

- 也能实现负载均衡

 

---

 

## 七、Task 6：创建 Ingress

 

### 题目要求

- Ingress 名称：`nginx-ingress`

- 命名空间：`training`

- Host：`nginx.local`

- 路径：`/`

- backend：`nginx-service:80`

 

### YAML：`ingress.yaml`

 

```yaml

apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:

  name: nginx-ingress

  namespace: training

  annotations:

    nginx.ingress.kubernetes.io/rewrite-target: /

spec:

  ingressClassName: nginx

  rules:

    - host: nginx.local

      http:

        paths:

          - path: /

            pathType: Prefix

            backend:

              service:

                name: nginx-service

                port:

                  number: 80

```

 

### 执行命令

 

```bash

kubectl apply -f ingress.yaml

kubectl get ingress -n training

kubectl describe ingress nginx-ingress -n training

```

 

### 参考输出示例

 

```bash

NAME            CLASS   HOSTS          ADDRESS        PORTS   AGE

nginx-ingress  nginx   nginx.local    10.0.0.10     80      15s

```

 

### 问题 11：Ingress 的作用是什么？

 

答案：

- Ingress 用于对外暴露 HTTP/HTTPS 请求

- 按 host/path 将请求转发到后端 Service

 

### 问题 12：Ingress 如何将流量路由到 Service？

 

答案：

- 根据 `host` 和 `path` 匹配请求

- 然后将请求转发到指定 backend service 和端口

 

### 问题 13：Ingress 和 Service 的区别是什么？

 

答案：

- Service：集群内的服务发现和负载均衡入口

- Ingress：集群外部访问入口，适合 HTTP/HTTPS 路由

 

### 问题 14：为什么必须有 Ingress Controller？

 

答案：

- Ingress 只是声明式配置对象

- 真正处理流量从外部到 Service 的组件叫 `Ingress Controller`

- 如果没有控制器，Ingress 配置不会真正工作

 

---

 

## 八、Task 7：一次性应用所有 YAML

 

### 命令

 

```bash

kubectl apply -f namespace.yaml

kubectl apply -f pod.yaml

kubectl apply -f deployment.yaml

kubectl apply -f service.yaml

kubectl apply -f ingress.yaml

```

 

或者

 

```bash

kubectl apply -f .

```

 

### 验证命令

 

```bash

kubectl get all -n training

kubectl get ingress -n training

```

 

### 参考输出示例

 

```bash

NAME                          READY   STATUS    RESTARTS   AGE

pod/nginx-pod                 1/1     Running   0          1m

deployment.apps/nginx-deployment  3/3   3         3          1m

service/nginx-service         ClusterIP   10.xx.xx.xx   <none>   80/TCP

ingress.networking.k8s.io/nginx-ingress   nginx   nginx.local   10.0.0.10   80  1m

```

 

### 注意事项

- 单独 Pod 和 Deployment 同时使用 `app: nginx` 标签时

- Service 可能会同时命中两类 Pod

- 生产环境中通常只保留 Deployment，而去掉独立 Pod

 

---

 

## 九、Task 8：资源检查和故障排查

 

### 常用命令

 

```bash

kubectl get all -n training

kubectl get events -n training

kubectl describe deployment nginx-deployment -n training

kubectl describe pod <pod-name> -n training

kubectl logs <pod-name> -n training

kubectl get endpoints nginx-service -n training

```

 

### 每个命令的作用

 

#### `kubectl get all -n training`

- 查看当前命名空间中所有常见资源

 

#### `kubectl get events -n training`

- 查看最近的事件，快速定位问题

 

#### `kubectl describe deployment nginx-deployment -n training`

- 查看 Deployment 的状态、副本数、条件、事件

 

#### `kubectl describe pod <pod-name> -n training`

- 查看 Pod 是否 Ready、容器状态、Probe 状态等

 

#### `kubectl logs <pod-name> -n training`

- 查看应用日志

 

#### `kubectl get endpoints nginx-service -n training`

- 查看 Service 后端真正连接到哪些 Pod

 

### 常见问题：一个可能的问题

 

可能出现：

- Pod 没有 Ready

- Service 没有 Endpoints

- readinessProbe 失败

 

### 排查步骤

 

1. `kubectl get pods -n training`

2. `kubectl describe pod <pod-name> -n training`

3. `kubectl logs <pod-name> -n training`

4. `kubectl get endpoints nginx-service -n training`

5. 检查 selector、labels、Ports、Probe 配置

 

### 是否有 active endpoints？

 

如果 Service 正确匹配到 Pod，则有 endpoints；如果没有，说明 Service 没有有效后端，应用无法被访问。

 

---

 

## 十、Task 9：应用更新和扩缩容

 

### 方案 A：使用 kubectl 直接更新

 

#### 扩容到 5 个副本

 

```bash

kubectl scale deployment nginx-deployment --replicas=5 -n training

```

 

#### 更新镜像

 

```bash

kubectl set image deployment/nginx-deployment nginx=nginx:1.26 -n training

```

 

#### 监控滚动更新

 

```bash

kubectl rollout status deployment/nginx-deployment -n training

kubectl get pods -n training

```

 

### 方案 B：修改 `deployment.yaml` 后再 apply

 

```yaml

spec:

  replicas: 5

```

 

```yaml

image: nginx:1.26

```

 

然后执行：

 

```bash

kubectl apply -f deployment.yaml

```

 

### 查看历史

 

```bash

kubectl rollout history deployment/nginx-deployment -n training

```

 

### 回滚

 

```bash

kubectl rollout undo deployment/nginx-deployment -n training

```

 

### 中文解释

- 扩容：增加副本数量，处理更高流量

- 更新镜像：触发滚动更新，逐步替换旧 Pod

- `kubectl rollout` 是 Kubernetes 的版本管理机制

 

---

 

## 十一、Task 10：清理资源

 

### 删除整个 Namespace

 

```bash

kubectl delete namespace training

```

 

### 验证是否删除成功

 

```bash

kubectl get namespace training

```

 

### 参考输出

 

```bash

Error from server (NotFound): namespaces "training" not found

```

 

---

 

## 十二、可直接复制的完整 YAML 文件

 

### 1) `namespace.yaml`

 

```yaml

apiVersion: v1

kind: Namespace

metadata:

  name: training

```

 

### 2) `pod.yaml`

 

```yaml

apiVersion: v1

kind: Pod

metadata:

  name: nginx-pod

  namespace: training

  labels:

    app: nginx

spec:

  containers:

    - name: nginx

      image: nginx:1.25

      ports:

        - name: http

          containerPort: 80

          protocol: TCP

```

 

### 3) `deployment.yaml`

 

```yaml

apiVersion: apps/v1

kind: Deployment

metadata:

  name: nginx-deployment

  namespace: training

  labels:

    app: nginx

spec:

  replicas: 3

  selector:

    matchLabels:

      app: nginx

  strategy:

    type: RollingUpdate

  template:

    metadata:

      labels:

        app: nginx

    spec:

      containers:

        - name: nginx

          image: nginx:1.25

          ports:

            - name: http

              containerPort: 80

              protocol: TCP

          readinessProbe:

            httpGet:

              path: /

              port: http

            initialDelaySeconds: 5

            periodSeconds: 10

          livenessProbe:

            httpGet:

              path: /

              port: http

            initialDelaySeconds: 15

            periodSeconds: 20

```

 

### 4) `service.yaml`

 

```yaml

apiVersion: v1

kind: Service

metadata:

  name: nginx-service

  namespace: training

  labels:

    app: nginx

spec:

  type: ClusterIP

  selector:

    app: nginx

  ports:

    - name: http

      protocol: TCP

      port: 80

      targetPort: http

```

 

### 5) `ingress.yaml`

 

```yaml

apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:

  name: nginx-ingress

  namespace: training

  annotations:

    nginx.ingress.kubernetes.io/rewrite-target: /

spec:

  ingressClassName: nginx

  rules:

    - host: nginx.local

      http:

        paths:

          - path: /

            pathType: Prefix

            backend:

              service:

                name: nginx-service

                port:

                  number: 80

```

 

---

 

## 十三、完整命令清单（可直接复制执行）

 

```bash

kubectl apply -f namespace.yaml

kubectl get namespace training

 

kubectl apply -f pod.yaml

kubectl get pod -n training

kubectl describe pod nginx-pod -n training

 

kubectl apply -f deployment.yaml

kubectl get deployment -n training

kubectl get pods -n training

 

kubectl apply -f service.yaml

kubectl get service -n training

kubectl describe service nginx-service -n training

kubectl get endpoints nginx-service -n training

 

kubectl apply -f ingress.yaml

kubectl get ingress -n training

kubectl describe ingress nginx-ingress -n training

 

kubectl get all -n training

kubectl get events -n training

kubectl logs <pod-name> -n training

 

kubectl scale deployment nginx-deployment --replicas=5 -n training

kubectl set image deployment/nginx-deployment nginx=nginx:1.26 -n training

kubectl rollout status deployment/nginx-deployment -n training

kubectl rollout history deployment/nginx-deployment -n training

kubectl rollout undo deployment/nginx-deployment -n training

 

kubectl delete namespace training

kubectl get namespace training

```

 

---

 

## 十四、架构简图

 

```text

Client

  ↓

Ingress

  ↓

Service

  ↓

Deployment

  ↓

Pods

```

 

---

 

## 十五、结论总结

 

这份 Kubernetes 实验训练的核心价值在于：

 

- 了解资源之间的关系

- 学会如何通过 YAML 定义应用

- 掌握部署方式、服务暴露方式和路由方式

- 学会通过 `kubectl` 观察、诊断和修复应用

- 形成从部署到扩容、更新和回滚的完整认知

 

如果本地 K8S 环境已经可用，你可以直接按上述命令执行，并验证各个资源是否正常运行。

 
