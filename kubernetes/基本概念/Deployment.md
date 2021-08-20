Deployment是kubernetes1.2版本引入的新概念，用于更好地解决Pod的编排问题，属于一种kubernetes资源对象。

**Deployment在内部使用了Replica Set来实现目的，**无论从Deployment的作用于目的、yaml定义还是从它的具体命令行操作来看，我们都可以把它看作RC的一次升级，两者的相似度超过90%。

# Deployments

一个 *Deployment* 为 [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 和 [ReplicaSets](https://kubernetes.io/zh/docs/concepts/workloads/controllers/replicaset/) 提供声明式的更新能力。

你负责描述 Deployment 中的 *目标状态*，而 Deployment [控制器（Controller）](https://kubernetes.io/zh/docs/concepts/architecture/controller/) 以受控速率更改实际状态， 使其变为期望状态。你可以定义 Deployment 以创建新的 ReplicaSet，或删除现有 Deployment， 并通过新的 Deployment 收集其资源。

## 创建 Deployment

下面是 Deployment 示例。**其中创建了一个 ReplicaSet，负责启动三个 `nginx` Pods**：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
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
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```





在该例中：

- 创建名为 `nginx-deployment`（由 `.metadata.name` 字段标明）的 Deployment。
- 该 Deployment 创建三个（由 `replicas` 字段标明）Pod 副本。

- `selector` 字段定义 Deployment 如何查找要管理的 Pods。 在这里，你选择在 Pod 模板中定义的标签（`app: nginx`）。 不过，更复杂的选择规则是也可能的，只要 Pod 模板本身满足所给规则即可。

  > **说明：**
  >
  > `spec.selector.matchLabels` 字段是 `{key,value}` 键值对映射。 在 `matchLabels` 映射中的每个 `{key,value}` 映射等效于 `matchExpressions` 中的一个元素， 即其 `key` 字段是 “key”，`operator` 为 “In”，`values` 数组仅包含 “value”。 在 `matchLabels` 和 `matchExpressions` 中给出的所有条件都必须满足才能匹配。

- ```
  template
  ```

   

  字段包含以下子字段：

  - Pod 被使用 `labels` 字段打上 `app: nginx` 标签。
  - Pod 模板规约（即 `.template.spec` 字段）指示 Pods 运行一个 `nginx` 容器， 该容器运行版本为 1.14.2 的 `nginx` [Docker Hub](https://hub.docker.com/)镜像。
  - 创建一个容器并使用 `name` 字段将其命名为 `nginx`。