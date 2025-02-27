
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

