---
layout: blog
title: "Kubernetes 配置最佳实践"
date: 2025-11-25T00:00:00+00:00
slug: configuration-good-practices
evergreen: true
author: Kirti Goyal
translator: >
  [Xin Li](https://github.com/my-git9) (DaoCloud)
---
<!--
layout: blog
title: "Kubernetes Configuration Good Practices"
date: 2025-11-25T00:00:00+00:00
slug: configuration-good-practices
evergreen: true
author: Kirti Goyal
-->

<!--
Configuration is one of those things in Kubernetes that seems small until it's not. Configuration is at the heart of every Kubernetes workload.
A missing quote, a wrong API version or a misplaced YAML indent can ruin your entire deploy. 

This blog brings together tried-and-tested configuration best practices. The small habits that make your Kubernetes setup clean, consistent and easier to manage. 
Whether you are just starting out or already deploying apps daily, these are the little things that keep your cluster stable and your future self sane. 

_This blog is inspired by the original *Configuration Best Practices* page, which has evolved through contributions from many members of the Kubernetes community._
-->
在 Kubernetes 中，配置是那种乍看之下微不足道，但实际上却非常重要的东西。
配置是每个 Kubernetes 工作负载的核心，
缺少引号、错误的 API 版本或 YAML 缩进不当都可能导致整个部署失败。

本博客汇集了经过实践检验的 Kubernetes 配置最佳实践。
这些小习惯能让你的 Kubernetes 配置更简洁、更一致、更易于管理。
无论你是刚刚起步还是已经每天部署应用程序，这些小事都能让你的集群保持稳定，也能让你未来的自己安心。

**此博客灵感来源于最初的“配置最佳实践”页面，该页面在 Kubernetes 社区众多成员的贡献下不断发展完善。**

<!--
## General configuration practices

### Use the latest stable API version 
Kubernetes evolves fast. Older APIs eventually get deprecated and stop working. So, whenever you are defining resources, make sure you are using the latest stable API version. 
You can always check with
-->
## 通用配置实践

### 使用最新的稳定 API 版本

Kubernetes 发展迅速，旧版 API 最终会被弃用并停止工作。因此，在定义资源时，请务必使用最新的稳定版 API。

你可以随时通过以下方式进行检查：

```bash
kubectl api-resources
```

<!--
This simple step saves you from future compatibility issues. 
-->
这个简单的步骤可以避免将来出现兼容性问题。

<!--  
### Store configuration in version control 
Never apply manifest files directly from your desktop. Always keep them in a version control system like Git, it's your safety net. 
If something breaks, you can instantly roll back to a previous commit, compare changes or recreate your cluster setup without panic.
-->
### 将配置存储在版本控制系统中

切勿直接从桌面应用清单文件。务必将其保存在 Git 等版本控制系统中，这是你的安全保障。
如果出现问题，你可以立即回滚到之前的提交，比较更改，或重新创建集群设置，而无需惊慌。

<!--
### Write configs in YAML not JSON
Write your configuration files using YAML rather than JSON. Both work technically, but YAML is just easier for humans. It's cleaner to read and less noisy and widely used in the community. 
-->
### 使用 YAML 而不是 JSON 编写配置

使用 YAML 而不是 JSON 编写你的配置文件。虽然两种格式在技术上都可行，
但 YAML 对人类来说更易于使用。它更易读、更简洁，并在社区中被广泛使用。

<!--
YAML has some sneaky gotchas with boolean values: 
Use only `true` or `false`. 
Don't write `yes`, `no`, `on` or  `off`.
They might work in one version of YAML but break in another. To be safe, quote anything that looks like a Boolean (for example `"yes"`).
-->
YAML 在处理布尔值时有一些需要注意的地方：
- 只使用 `true` 或 `false`。
- 不要写 `yes`、`no`、`on` 或 `off`。
- 这些值可能在某一版本的 YAML 中工作，但在另一版本中无效。
  为安全起见，请将所有看起来像布尔值的内容（例如 `"yes"`）用引号括起来。

<!--
###	Keep configuration simple and minimal
Avoid setting default values that are already handled by Kubernetes. Minimal manifests are easier to debug, cleaner to review and less likely to break things later. 
-->
### 保持配置简洁

避免设置 Kubernetes 已经处理的默认值。
简洁的清单文件更容易调试、更易于审查，并且以后出现问题的可能性也更小。

<!--
###	Group related objects together
If your Deployment, Service and ConfigMap all belong to one app, put them in a single manifest file.  
It's easier to track changes and apply them as a unit. 
See the [Guestbook all-in-one.yaml](https://github.com/kubernetes/examples/blob/master/web/guestbook/all-in-one/guestbook-all-in-one.yaml) file for an example of this syntax.

You can even apply entire directories with:
-->
### 将相关对象分组

如果您的 Deployment、Service 和 ConfigMap 都属于同一个应用，请将它们放在同一个清单文件中。
这样更容易跟踪更改并作为一个整体应用它们。
有关此语法的示例，请参阅 [Guestbook all-in-one.yaml](https://github.com/kubernetes/examples/blob/master/web/guestbook/all-in-one/guestbook-all-in-one.yaml)。

你甚至可以使用以下命令应用整个目录：

```bash
kubectl apply -f configs/
```

<!--
One command and boom everything in that folder gets deployed. 
-->
一个命令，砰！那个文件夹中的所有内容都会被部署。

<!--
###	Add helpful annotations
Manifest files are not just for machines, they are for humans too. Use annotations to describe why something exists or what it does. A quick one-liner can save hours when debugging later and also allows better collaboration.  

The most helpful annotation to set is `kubernetes.io/description`. It's like using comment, except that it gets copied into the API so that everyone else can see it even after you deploy.
-->
### 添加有用的注解

清单文件不仅供机器使用，也供人使用。使用注解来描述某个功能存在的原因或用途。
一行简洁的注解就能在后续调试时节省数小时的时间，还能促进更好的协作。

最有用的注解是 `kubernetes.io/description`。
它类似于注释，但不同之处在于，它会被复制到 API 中，以便其他人即使在部署后也能看到它。

<!--
## Managing Workloads: Pods, Deployments, and Jobs

A common early mistake in Kubernetes is creating Pods directly. Pods work, but they don't reschedule themselves if something goes wrong.

_Naked Pods_ (Pods not managed by a controller, such as [Deployment](/docs/concepts/workloads/controllers/deployment/) or a [StatefulSet](/docs/concepts/workloads/controllers/statefulset/)) are fine for testing, but in real setups, they are risky.
-->
## 管理工作负载：Pod、Deployment 和 Job

Kubernetes 中一个常见的早期错误是直接创建 Pod。
Pod 可以运行，但如果出现问题，它们不会自动重新调度。

**裸 Pod**（即未由控制器管理的 Pod，例如 [Deployment](/zh-cn/docs/concepts/workloads/controllers/deployment/)
或 [StatefulSet](/zh-cn/docs/concepts/workloads/controllers/statefulset/)）在测试环境中可以接受，
但在实际部署中则存在风险。

<!--
Why?
Because if the node hosting that Pod dies, the Pod dies with it and Kubernetes won't bring it back automatically. 
-->
为什么？

因为如果托管该 Pod 的节点崩溃，Pod 也会随之崩溃，Kubernetes 不会自动将其恢复。

<!--
### Use Deployments for apps that should always be running
A Deployment, which both creates a ReplicaSet to ensure that the desired number of Pods is always available, and specifies a strategy to replace Pods (such as [RollingUpdate](/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment)), is almost always preferable to creating Pods directly.
You can roll out a new version, and if something breaks, roll back instantly.
-->
### 对于需要始终运行的应用，请使用 Deployment

Deployment 会创建一个 ReplicaSet 来确保始终有所需数量的 Pod 可用，
并指定 Pod 的替换策略（例如 [RollingUpdate](/zh-cn/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment)），
这几乎总是优于直接创建 Pod。

你可以快速部署新版本，如果出现问题，可以立即回滚。

<!--
### Use Jobs for tasks that should finish
A [Job](/docs/concepts/workloads/controllers/job/) is perfect when you need something to run once and then stop like database migration or batch processing task.
It will retry if the pods fails and report success when it's done. 
-->
### 使用 Job 来执行需要完成的任务

当你需要运行某些一次性任务，例如数据库迁移或批处理任务时，
[Job](/zh-cn/docs/concepts/workloads/controllers/job/) 是完美的选择。

如果 Pod 失败，它会重试，并在完成后报告成功。

<!--
## Service Configuration and Networking

Services are how your workloads talk to each other inside (and sometimes outside) your cluster. Without them, your pods exist but can't reach anyone. Let's make sure that doesn't happen.
-->
## Service 配置与网络

Service 是你的工作负载在集群内部（有时也在外部）相互通信的方式。
没有它们，你的 Pod 将存在但无法与任何其他 Pod 通信。让我们确保这种情况不会发生。

<!--
### Create Services before workloads that use them
When Kubernetes starts a Pod, it automatically injects environment variables for existing Services.
So, if a Pod depends on a Service, create a [Service](/docs/concepts/services-networking/service/) **before** its corresponding backend workloads (Deployments or StatefulSets), and before any workloads that need to access it.

For example, if a Service named foo exists, all containers will get the following variables in their initial environment:
-->
### 在使用 Service 的工作负载之前创建 Service

当 Kubernetes 启动一个 Pod 时，它会自动为现有的 Service 注入环境变量。
因此，如果 Pod 依赖于某个 Service，则必须在创建其对应的后端工作负载（Deployment 或 StatefulSet）之前，
以及在任何需要访问该 Service 的工作负载之前，创建该
[Service](/zh-cn/docs/concepts/services-networking/service/)。

例如，如果存在名为 foo 的 Service，则所有容器的初始环境中都会包含以下变量：

```
FOO_SERVICE_HOST=<the host the Service runs on>
FOO_SERVICE_PORT=<the port the Service runs on>
```

<!--
DNS based discovery doesn't have this problem, but it's a good habit to follow anyway.
-->
基于 DNS 的发现机制不会出现这个问题，但无论如何，遵循上述做法是一个好习惯。

<!--
### Use DNS for Service discovery
If your cluster has the DNS [add-on](/docs/concepts/cluster-administration/addons/) (most do), every Service automatically gets a DNS entry. That means you can access it by name instead of IP:
-->
### 使用 DNS 进行服务发现

如果你的集群安装了 DNS [插件](/zh-cn/docs/concepts/cluster-administration/addons/)（大多数集群都已安装），
每个 Service 都会自动获得一个 DNS 条目。这意味着你可以通过名称而不是 IP 地址来访问它：

```bash
curl http://my-service.default.svc.cluster.local
``` 

<!--
It's one of those features that makes Kubernetes networking feel magical. 

### Avoid `hostPort` and `hostNetwork` unless absolutely necessary
You'll sometimes see these options in manifests:
-->
这是那些让 Kubernetes 网络感觉神奇的特性之一。

### 除非绝对必要，否则请避免使用 `hostPort` 和 `hostNetwork`。

你有时会在清单文件中看到这些选项：

```yaml
hostPort: 8080
hostNetwork: true
```

<!--
But here's the thing:
They tie your Pods to specific nodes, making them harder to schedule and scale. Because each <`hostIP`, `hostPort`, `protocol`> combination must be unique. If you don't specify the `hostIP` and `protocol` explicitly, Kubernetes will use `0.0.0.0` as the default `hostIP` and `TCP` as the default `protocol`.
Unless you're debugging or building something like a network plugin, avoid them. 

If you just need local access for testing, try [`kubectl port-forward`](/docs/reference/kubectl/generated/kubectl_port-forward/):
-->
但问题是：
它们将你的 Pod 绑定到特定节点，使其更难以调度和扩缩。
因为每个 <`hostIP`, `hostPort`, `protocol`> 组合必须是唯一的。
如果你没有明确指定 `hostIP` 和 `protocol`，
Kubernetes 将使用 `0.0.0.0` 作为默认的 `hostIP` 并使用 `TCP` 作为默认的 `protocol`。
除非你在调试或构建类似网络插件的东西，否则请避免使用它们。

但问题在于：

它们会将你的 Pod 绑定到特定的节点，这使得调度和扩缩更加困难。
因为每个 `hostIP`、`hostPort` 和 `protocol` 的组合都必须是唯一的。
如果你没有显式指定 `hostIP` 和 `protocol`，
Kubernetes 会使用 `0.0.0.0` 作为默认的 `hostIP`，`TCP` 作为默认的 `protocol`。
除非你在进行调试或构建类似网络插件之类的东西，否则请避免使用它们。

如果你只是需要本地访问进行测试，尝试 [`kubectl port-forward`](/zh-cn/docs/reference/kubectl/generated/kubectl_port-forward/)：

```bash
kubectl port-forward deployment/web 8080:80
```

<!--
See [Use Port Forwarding to access applications in a cluster](/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) to learn more.
Or if you really need external access, use a [`type: NodePort` Service](/docs/concepts/services-networking/service/#type-nodeport). That's the safer, Kubernetes-native way. 
-->
请参阅[使用端口转发访问集群中的应用程序](/zh-cn/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)了解更多信息。
或者，如果你确实需要外部访问，使用
[`type: NodePort` Service](/zh-cn/docs/concepts/services-networking/service/#type-nodeport)。
这是更安全、Kubernetes 原生的方式。

<!--
### Use headless Services for internal discovery 
Sometimes, you don't want Kubernetes to load balance traffic. You want to talk directly to each Pod. That's where [headless Services](/docs/concepts/services-networking/service/#headless-services) come in.

You create one by setting `clusterIP: None`.
Instead of a single IP, DNS gives you a list of all Pods IPs, perfect for apps that manage connections themselves. 
-->
### 对于内部发现使用无头服务（Headless Service）

有时，你可能不希望 Kubernetes 负责流量的负载均衡。你想要直接与每个 Pod 通信。
这时就轮到[无头服务（Headless Service）](/zh-cn/docs/concepts/services-networking/service/#headless-services)登场了。

通过设置 `clusterIP: None` 来创建一个无头服务。
DNS 不再返回单一 IP 地址，而是提供所有 Pod 的 IP 地址列表，
这对于那些自行管理连接的应用来说是完美的。

<!--
## Working with labels effectively 

[Labels](/docs/concepts/overview/working-with-objects/labels/) are key/value pairs that are attached to objects such as Pods.
Labels help you organize, query and group your resources.
They don't do anything by themselves, but they make everything else from Services to Deployments work together smoothly. 

### Use semantics labels
Good labels help you understand what's what, even after months later. 
Define and use [labels](/docs/concepts/overview/working-with-objects/labels/) that identify semantic attributes of your application or Deployment.
For example;
-->
## 有效地使用标签

[标签](/zh-cn/docs/concepts/overview/working-with-objects/labels/)是附加到对象（例如：Pod）的键值对。
标签可以帮助你组织、查询和分组资源。
它们本身不做任何事情，但它们使从服务到部署的所有工作都能顺畅地协同。

### 使用语义标签

好的标签可以帮助你即使在几个月后也能轻松理解各个组件的含义。。
定义并使用[标签](/zh-cn/docs/concepts/overview/working-with-objects/labels/)来标识应用程序或部署的语义属性。
例如：

```yaml
labels:
  app.kubernetes.io/name: myapp
  app.kubernetes.io/component: web
  tier: frontend
  phase: test
```

  <!--
  - `app.kubernetes.io/name` : what the app is
  - `tier` : which layer it belongs to (frontend/backend)
  - `phase` : which stage it's in (test/prod)
  -->

  - `app.kubernetes.io/name`：应用是什么
  - `tier`：它属于哪一层（前端/后端）
  - `phase`：它处于哪个阶段（测试/生产）

<!--
You can then use these labels to make powerful selectors.
For example:
-->
然后，你可以使用这些标签创建强大的选择器。
例如：

```bash
kubectl get pods -l tier=frontend
```

<!--
This will list all frontend Pods across your cluster, no matter which Deployment they came from. 
Basically you are not manually listing Pod names; you are just describing what you want. 
See the [guestbook](https://github.com/kubernetes/examples/tree/master/web/guestbook/) app for examples of this approach.
-->
这将列出集群中所有前端 Pod，无论它们来自哪个 Deployment。
实际上，你无需手动列出 Pod 名称；只需描述你想要的内容即可。
请参阅 [Guestbook](https://github.com/kubernetes/examples/tree/master/web/guestbook/)
应用，了解此方法的示例。

<!--
### Use common Kubernetes labels
Kubernetes actually recommends a set of [common labels](/docs/concepts/overview/working-with-objects/common-labels/). It's a standardized way to name things across your different workloads or projects.
Following this convention makes your manifests cleaner, and it means that tools such as [Headlamp](https://headlamp.dev/), [dashboard](https://github.com/kubernetes/dashboard#introduction), or third-party monitoring systems can all
automatically understand what's running.
-->
### 使用通用的 Kubernetes 标签

Kubernetes 实际上推荐了一组[通用标签](/zh-cn/docs/concepts/overview/working-with-objects/common-labels/)。
这是一种标准化的方式，用于在不同的工作负载或项目中命名事物。
遵循此约定可以使你的清单文件更简洁，这意味着诸如
[Headlamp](https://headlamp.dev/)、[Dashboard](https://github.com/kubernetes/dashboard#introduction)
或第三方监控系统等工具都可以自动理解正在运行的内容。

<!--
###	Manipulate labels for debugging 
Since controllers (like ReplicaSets or Deployments) use labels to manage Pods, you can remove a label to “detach” a Pod temporarily.

Example:
-->
### 操作标签进行调试

由于控制器（例如 ReplicaSet 或 Deployment）使用标签来管理 Pod，
因此你可以移除标签来暂时“分离”Pod。

示例：

```bash
kubectl label pod mypod app-
```

<!--
The `app-` part removes the label key `app`.
Once that happens, the controller won’t manage that Pod anymore.
It’s like isolating it for inspection, a “quarantine mode” for debugging. To interactively remove or add labels, use [`kubectl label`](/docs/reference/kubectl/generated/kubectl_label/).

You can then check logs, exec into it and once done, delete it manually.
That’s a super underrated trick every Kubernetes engineer should know.
-->
`app-` 部分移除了标签键 `app`。
一旦完成，控制器将不再管理那个 Pod。
这就像是将其隔离以进行检查，一种用于调试的“隔离模式”。
要交互式地移除或添加标签，请使用 [`kubectl label`](/zh-cn/docs/reference/kubectl/generated/kubectl_label/)。

之后，你可以检查日志，执行进入它，完成后手动删除它。
这是一个被严重低估的技巧，每个 Kubernetes 工程师都应该掌握。

<!--
## Handy kubectl tips 

These small tips make life much easier when you are working with multiple manifest files or clusters.

### Apply entire directories
Instead of applying one file at a time, apply the whole folder:
-->
## 便捷的 kubectl 小技巧

这些小技巧能让你在处理多个清单文件或集群时事半功倍。

### 应用整个目录

与其一次应用一个文件，不如一次性应用整个文件夹：

<!--
```bash
# Using server-side apply is also a good practice
kubectl apply -f configs/ --server-side
```
This command looks for `.yaml`, `.yml` and `.json` files in that folder and applies them all together.
It's faster, cleaner and helps keep things grouped by app. 
-->
```bash
# 使用 server-side 部署也是一个好习惯
kubectl apply -f configs/ --server-side
```

此命令会在该文件夹中查找 `.yaml`、`.yml` 和 `.json` 文件，并将它们一起部署。
这种方法速度更快、更简洁，并且有助于按应用程序对内容进行分组。

<!--
### Use label selectors to get or delete resources
You don't always need to type out resource names one by one.
Instead, use [selectors](/docs/concepts/overview/working-with-objects/labels/#label-selectors) to act on entire groups at once:
-->
### 使用标签选择器获取或删除资源

你并不总是需要逐一输入资源名称。
相反，可以使用[选择器](/zh-cn/docs/concepts/overview/working-with-objects/labels/#label-selectors)一次性对整个组进行操作：

```bash
kubectl get pods -l app=myapp
kubectl delete pod -l phase=test
```

<!--
It's especially useful in CI/CD pipelines, where you want to clean up test resources dynamically. 

### Quickly create Deployments and Services
For quick experiments, you don't always need to write a manifest. You can spin up a Deployment right from the CLI:
-->
它在 CI/CD 流水线中尤其有用，可以用来动态清理测试资源。

### 快速创建 Deployment 和 Service

对于快速实验，你并不总是需要编写清单，也可以直接从 CLI 启动 Deployment：

```bash
kubectl create deployment webapp --image=nginx
```

<!--
Then expose it as a Service:
```bash
kubectl expose deployment webapp --port=80
```
This is great when you just want to test something before writing full manifests. 
Also, see [Use a Service to Access an Application in a cluster](/docs/tasks/access-application-cluster/service-access-application-cluster/) for an example.
-->
然后将其作为 Service 公开：

```bash
kubectl expose deployment webapp --port=80
```

当你只想在编写完整清单文件之前测试某些东西时，这非常有用。
另外，你可以参阅[使用 Service 访问集群中的应用](/zh-cn/docs/tasks/access-application-cluster/service-access-application-cluster/)以获取示例。

<!--
## Conclusion

Cleaner configuration leads to calmer cluster administrators. 
If you stick to a few simple habits: keep configuration simple and minimal, version-control everything,
use consistent labels, and avoid relying on naked Pods, you'll save yourself hours of debugging down the road.

The best part? 
Clean configurations stay readable. Even after months, you or anyone on your team can glance at them and know exactly what’s happening.
-->
## 结论

更简洁的配置能让集群管理员更加从容。
如果你养成了一些简单的习惯：
保持配置简单和最小化、对所有内容进行版本控制、使用一致的标签、避免依赖裸 Pod，
那么你将省去许多调试的时间。

最棒的是？
干净的配置保持了可读性。即使过了几个月，你或团队中的任何人都能快速查看并准确知道发生了什么。
