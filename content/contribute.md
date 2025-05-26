---
title: 贡献指南
date: 2025-05-26T16:43:28+08:00
---

{{< callout type="info" >}}
  本文为团队成员撰写博客/文档提供参考指南，作为非团队成员贡献欢迎提交 [PR](https://github.com/Kawaii-Chip/Kawaii-Chip.github.io/pulls)。
{{< /callout >}}

## 步骤

{{% steps %}}

### 环境配置

1. 安装 hugo ，参考[官网教程](https://gohugo.io/installation/)。
2. 安装 Git ，参考[官网教程](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)。

{{< callout type="info" >}}
  Git 相关的配置以及初始化本文不再赘述。
{{< /callout >}}

```bash
hugo version
```

若成功输出则已经安装。

### 克隆仓库

```bash
git clone https://github.com/Kawaii-Chip/Kawaii-Chip.github.io.git Blog
cd Blog
```

### 新建页面

`content` 目录用于存放页面的源文件，通常为 markdown 格式（ `.md` 结尾的文件）。

这是刚开始的文件树：

{{< filetree/container >}}
  {{< filetree/folder name="content" >}}
    {{< filetree/folder name="assets" state="closed" >}}
      {{< filetree/file name="logo_ascii.txt" >}}
      {{< filetree/file name="logo.png" >}}
    {{< /filetree/folder >}}
    {{< filetree/file name="_index.md" >}}
    {{< filetree/file name="about.md" >}}
    {{< filetree/file name="contribute.md" >}}
  {{< /filetree/folder >}}
{{< /filetree/container >}}

```bash
hugo new [<parent_folder>/]<page_name>.md
```

这是新建后的文件树：

> <parent_folder>/ 可以为空，这将会在 `content` 目录下直接创建 markdown 文件。
> <page_name> 是路由名，建议言简意赅。

{{< filetree/container >}}
  {{< filetree/folder name="content" >}}
    {{< filetree/folder name="assets" state="closed" >}}
      {{< filetree/file name="logo_ascii.txt" >}}
      {{< filetree/file name="logo.png" >}}
    {{< /filetree/folder >}}
    {{< filetree/file name="_index.md" >}}
    {{< filetree/file name="about.md" >}}
    {{< filetree/file name="contribute.md" >}}
    {{< filetree/file name="page_name.md" >}}
  {{< /filetree/folder >}}
{{< /filetree/container >}}

### 自定义页面属性

编辑 `content/page_name.md` 。

在顶部有默认生成的 header ：

```markdown {filename="page_name.md",linenos=table}
---
title: page_name
date: 2025-04-21T20:50:39+08:00
draft: true
---
```

移除 draft 属性：

```diff {filename="page_name.md",linenos=table}
---
title: page_name
date: 2025-04-21T20:50:39+08:00
- draft: true
---
```

#### 其他属性

目前已知且正在使用的属性如下：

| 名称         | 类型   | 可选值          | 默认值       | 介绍                                                          |
|:------------:|:------:|:---------------:|:------------:|:-------------------------------------------------------------:|
| **title**    | string | ...             | 与文件名相同 | 文章标题                                                      |
| **draft**    | bool   | true, false     | true         | 是否在部署后显示该文章                                        |
| **date**     | string | ...             | 自动生成     | 创建日期                                                      |
| **comments** | bool   | true, false     | true         | 控制评论区展示                                                |
| **toc**      | bool   | true, false     | true         | 控制文章右侧 '在 GitHub 上编辑此页 →' 显示                    |
| **weight**   | int    | 0, 1, 2, 3, ... | 0            | 控制子页面在侧边栏的顺序，数字越小排序越靠前，若为 0 则不生效 |

### 添加页面入口至索引

添加入口有多种方式，这里使用 cards 为例。  
编辑父目录下的 `_index.md` 文件，在 `{{ <cards /> }}` 块中添加子 `{{ <card /> }}` 项。  

{{< callout type="warning" >}}
  这里为什么不使用代码块进行说明？  
  HTML 标签会破坏渲染，因此不使用代码块进行讲解。  
{{< /callout >}}

{{% /steps %}}

## 技巧

Hextra 主题提供了更多语法支持，可以参考[官方文档](https://imfing.github.io/hextra/zh-cn/docs/guide/)。  
如果有图标使用的需求，可以参考 [Heroicons](https://v1.heroicons.com/) 和 [icons.yaml](https://github.com/imfing/hextra/blob/main/data/icons.yaml) 。  
