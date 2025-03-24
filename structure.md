# DevOps documentation structure

## 代码仓库接口

- tekton-operator
  - tekton-pipelines
  - tekton-triggers

- connector-operator
  - connectors
  - connectors-oci
  - connectors-git

- gitlab-ce-operator
  - chart
  - docker image
  - sdk

## 产品结构
Ref： https://product-doc-guide.alauda.cn/arch/site_arch.html#doc_structure

```
DevOps
- 文档导航
- 产品概览
  - 产品介绍
  - 架构
  - 快速开始
  - 发版说明
- 部署
- 升级
- 用户界面
  - 控制台
  - CLI
- 配置
  - 概览
  - 用户与角色
  - 项目管理
  - 安全与合规
- Pillar/父模块
  - 概览
  - 功能A
  - 功能B
- API文档
```

### 方案1: 每个插件独立的站点

**产品文档**

```
[...]
- Pillar/父模块:
  - 关于 Tekton 流水线 （外部链接）

```

**插件站点**

> tekton 为例

```
Tekton Pipelines
- 概览
  - 介绍
  - 架构
  - 功能总览
  - 快速开始
- 部署
- 升级
- Pipelines
  - 介绍
  - 架构
  - 部署
  - 升级
  - 核心概念
  - 快速开始
  - <功能指南>
  - 实用指南
  - 问题处理
  - 权限说明
- Triggers
- Chains
```



### 方案2: 同步/构建完整的产品文档站点

**产品文档**

```
[...]
- Pillar/父模块:
  - <tekton pipelines 完整的内容>
  - <connectors 完整的内容>

```

### 结论

使用 `方案1`

需要做的：
- 插件的仓库中解决下游仓库文档的同步
- 暂时引用文档功能过于简单，先找一个解决方案，后期在切换到框架的正式方案


## 同步下游文档内容方案

考虑了如下方案

1. `启动文档程序之前先同步所需要的文档到上游的仓库中但并不提交`：通过引用文件/目录连接的方式，实际的文件存在 .gitignore
2. `按需/事件驱动的文件驱动`: 实际进行同步到目标目录中并标记来源，支持清理/重新导入操作


如上方案都会依赖一个具体的配置来进行，如下考虑的使用场景

**上游（插件 / operator）**

- 提供配置文件来描述所需要的内容/目录结构，并不决定若干文件的引用而是决定文档类型/目录
- 解决下游“提供的”内容同步到对应的目录（看方案1或2）

**下游（项目）**

- 通过标记若干文档决定哪些文档可以共享给上游
- 不需要提供过于复杂的配置
- API也一样是下游决定那些要贡献给上游


*作为`功能`下游所需要提供的内容是：*


- 介绍
- 架构
- 部署
- 升级
- 核心概念
- 快速开始
- <功能指南>
- 实用指南
- 问题处理
- 权限说明

*额外需要提供的是:*

- API

*不需要同步：*

- Release notes? 待确认
- Proposal TEP / CEP / KEP
- Development



### 方案1：

在上游仓库中添加一个临时目录本地拷贝文件，例如 `imported-docs`，里面每个下游具有自己的目录 `tektoncd-pipelines` 内容以下游 `docs` 目录，根据每个文件的标记决定是否同步。在上游的代码仓库中添加 `imported-docs` 到 `.gitignore`中。根据配置文件添加对应的目录链接到 `docs/*` 下


**优点**

- 不需要”同步“文件变更到上游仓库，变更历史独立
- 可以方便的预览最终的文档效果，也可以验证下游的过滤条件是否准确的
- 能复用到文档的构建过程中


**缺点**

- 平台自动翻译的能力也许会有问题
- 多个仓库之间的冲突不太清晰（创建连接时？）
- 静态文件要如何处理？


#### 验证

主要需要验证的点是：

1. 通过连接方式文档能否正常构建？
2. 静态文件连接是否可用？
3. API文档有否问题？
4. 平台自动翻译成英文的问题是否存在？需要通过沟通确认

**验证方式：**

通过在一个仓库中模拟这种场景，复制 tekton 相关的文档并手动拼接这个方案验证当前 doom 文档框架能力是否满足

目录结构

```
- imported-docs
  - tektoncd-pipelines
    - (docs 内容)
- docs
 - public
 - shared
 - zh
   - apis
     - kubernetes-apis
       - pipelines
         - (tektoncd-pipelines apis内容 连接)
   - pipelines
     - (tektoncd-pipelines 剩下内容连接)
```

通过测试 `tektoncd-operator` 和 `tektoncd-pipeline` 两个项目最终的结果是成功的

1. 通过连接方式文档能否正常构建？ <font style="color: green">Yes</font>
2. 静态文件连接是否可用？<font style="color: green">Yes</font>
3. API文档有否问题？<font style="color: green">Yes</font>，可以连接也可以直接修改 `doom.config.yml` 来支持多个目录获取
4. 平台自动翻译成英文的问题是否存在？**需要通过沟通确认**


如下最终的目录结构:

```bash
./docs
├── en
├── i18n.json
├── public
│   ├── logo.svg
│   └── values-yaml.png -> ../../imported-docs/tektoncd-pipeline/public/values-yaml.png
├── shared
│   ├── crds
│   │   ├── operator.tekton.dev_openshiftpipelinesascodes.yaml
│   │   ├── [...]
│   │   └── pipeline -> ../../../imported-docs/tektoncd-pipeline/shared/crds/
│   └── openapis
└── zh
    ├── apis
    │   ├── index.mdx
    │   ├── intro.mdx
    │   └── kubernetes_apis
    │       ├── index.mdx
    │       ├── openshiftpipelinesascodes.mdx
    │       ├── pipelines -> ../../../../imported-docs/tektoncd-pipeline/zh/apis/kubernetes_apis
    │       ├── [...]
    │       └── tektontriggers.mdx
    ├── development
    │   ├── component-quickstart
    │   │   └── index.md
    │   ├── index.mdx
    │   └── update-frontend
    │       ├── assets [...]
    │       └── index.md
    ├── how_to
    │   ├── customize_options.mdx
    │   ├── [...]
    │   └── index.mdx
    ├── index.mdx
    ├── overview
    │   ├── architecture.mdx
    │   ├── [...]
    │   └── release_notes.mdx
    └── pipelines -> ../../imported-docs/tektoncd-pipeline/zh
./imported-docs
└── tektoncd-pipeline
    ├── en
    ├── i18n.json
    ├── public
    │   ├── logo.svg
    │   └── values-yaml.png
    ├── shared
    │   ├── crds
    │   │   ├── resolution.tekton.dev_resolutionrequests.yaml
    │   │   ├── [...]
    │   │   ├── tekton.dev_pipelines.yaml
    │   │   └── tekton.dev_verificationpolicies.yaml
    │   └── openapis
    └── zh
        ├── 00_intro.mdx
        ├── 01_release_notes.mdx
        ├── 02_architecture.mdx
        ├── 03_concepts
        │   └── index.mdx
        ├── 04_quick_start.mdx
        ├── 05_features
        │   └── index.mdx
        ├── 06_how_to
        │   └── index.mdx
        ├── 07_trouble_shooting
        │   └── index.mdx
        ├── 08_permissions.mdx
        ├── apis
        │   └── kubernetes_apis
        │       ├── index.mdx
        │       └── tektonpipelines.mdx
        └── index.mdx
```

### 方案2:

直接复制到上游的 `docs` 目录中并提交代码


### 配置文件

需要满足的功能：

**上游**

1. 需要copy哪些下游，目标目录

**下游**

1. 如何过滤目录
2. 如何过滤若干文件


**配置文件**

**使用:** 上游

```yaml
sources:
- name: <name> # 自定义的名称
  repository: # 代码信息, 动态克隆代码进行复制
    url: https://github.com/alaudadevops/tektoncd-pipeline
    revision: main

  dir: # 目录信息 跟repository二选一
    path: ../tektoncd-pipeline

target:
  copyTo: imported-docs
  linkTo: docs
  links:
  - from: public/*.png,public/*.svg # 从下游视角，在docs目录下路径，这里使用多种文件的链接方式
    target: public/ # 在上游的docs目录下往哪里连接
  - from: shared/crds # 使用目录链接
    target: shared/crds/<name> # 使用 <name> placeholder, 来自 sources[].name
  - from: zh/apis/kubernetes_apis
    target: zh/apis/kubernetes_apis/<name>
  - from: zh
    target: zh/<name>

```


**.syncignore**

使用：下游

通过常见的 `.ignore` 文件 来决定整个仓库或者若干的目录需要忽略的内容，并且每个目录都可以单独添加的 （同 `.dockerignore`, `.gitignore`）


**单个文件过滤**

除了使用 `.syncignore` 还可以在文档 `frontmatter` 添加 `syncignore: true`

```markdown
---
title: Some interesting document
syncignore: true
tags:
- abc
- def
---

# My interesting document

```

工具根据两种配置筛选要拷贝的内容
