# From Tools to Ecosystem: Enhancements for AI Skills
### 分享演讲稿

---

## Agenda

| # | 主题 | 时长 |
|---|------|------|
| 1 | The Signals and Problems We Are Seeing | 4 min |
| 2 | Why We Started with Skill Marketplace Platform | 10 min |
| 3 | The Execution Entry Point for Skills | 12 min |
| 4 | Live Demo: release-bot | 14 min |

> **前 4 个部分共 40 分钟，之后进入 Q&A**

---

## 开场白
大家好，我是来自 HongKong SOE 团队的 Cooper Duan， 今天想跟大家分享的主题是：

**From Tools to Ecosystem: Enhancements for AI Skills**

从工具到生态：对 AI Skills 的思考。

---

## Part 1 · The Signals and Problems We Are Seeing

**（约 4 分钟）**

最近这段时间，不管是在公司内部，还是在外部社区，我们看到越来越强烈的信号：

有非常多的同事每天都在自己写 Skill、分享 Skill、使用 SKILL 来沉淀一些常用 AI 实践。
大家可以去 mfc-innersource 这一个组织下，搜索 SKILL, 到目前为止，就有 4 个不同的团队的 SKILL 仓库，每个仓库下都存放了不同团队创建的 SKILL, 而在个人和其他组织下就有更多的 SKILL 仓库了。
所以这是一个非常积极的信号，大家现在不仅仅是在用 AI，而是在提升用 AI 的能力。

但与此同时，我们也看到几个很现实的问题。

第一个，是**SKILL 重复建设**。
比如一个团队写了 code review Skill，另一个团队并不知道已经有人做过，于是又写了一个类似的。
最后同样的事情做了两遍，投入被分散了。

第二个，是**Skill 质量不稳定**。
同样一个功能，不同人写出来的 Skill，在边界、触发条件、输出质量上都可能差很多。
这会导致大家对 Skill 的预期不稳定，也会影响此 SKILL 后续推广。

第三个，是**经验沉淀不下来**。
很多很好的 AI Skill，因为没有好的分享机制，最后只停留在个人工作流里，没有真正变成团队资产，被更多人复用。

第四个，是**没有统一度量**。
我们写完一个 Skill，自己会觉得“这个还不错、挺好用”，但它到底效果怎么样、谁在用、复用率高不高，因为没有真实使用数据和各种测试场景，其实很难回答。

所以如果这些问题不解决，Skill 再多，也还是只能停留在个人工作流中，而无法快速复用，形成体系化的团队能力。

---

## Part 2 · Why We Started with Skill Marketplace Platform

**（约 10 分钟）**

看到这些问题以后，我们很快就形成了一个共识。

Skill 一定会越来越多的，我们必须有一套机制，让这些 Skill 能够在公司内部被更快发现、被相互比较、被持续复用。

devai-hub 这个网站平台就是在这种想法下产生的。

它的价值，在于把原本分散的 Skill 聚合在一起，并通过社区反馈，完全由数据驱动的方式，快速形成一个创建 SKILL - 使用 SKILL - 优化 SKILL 的生态。 

这套生态机制可以让 Skill 不只是某个人的“个人技巧”，而是整个组织同事都可以快速复用的“能力”。

大家现在看到的这个页面，是目前我们正在使用的一些 SKILL 数据汇总，这些 Skills 基本都围绕研发日常工作流展开，包括后端、前端、效率工具等不同类型。

这里我们按照使用量进行了排序整理，当然我们还有更多对SKILL 数据的整理，比如 overview 页面的图表，它基于使用数据有更多的分析报表。 比如 Project， 只要有数据，我们就能按照不同的维度进行报表分析。

回到这个 SKILL 页面，这里我们将 SKILL 分成了几个类别来看。

第一类是 **External Skills**。
这类 Skill 是我们在日常开发过程中从外部社区引入的优质的 SKILL，比如 Claude Code、Vercel 等相关能力。

第二类是 **Internal Skills**。
这类是我们自己设计的、带有公司内部知识和流程沉淀的 Skill，比如代码规范、特定流程处理、团队约定等等。

这里我举一个例子，就是 **release-bot** 这个 Skill。
开发者只需要确认版本号和发版信息，Skill 就能自动完成检查、打包、发版整套流程。
我稍后在 demo 中会具体演示它的使用效果。

另外还有一类，是 **Personal Skills**，这些还没有进入组织共享范围，只存在于个人仓库里。
我们的SKILL开发流程是这样的：先在个人仓库里创建和迭代，等 Skill 相对稳定、质量足够好了之后，再提交到组织仓库。
一旦进入组织仓库，其他同事就可以复用，同时它也会自动出现在 Internal 类别里。


页面右侧还有一组很关键的数据——**feedback & installs**，代表每个 Skill 的使用反馈和安装量。
这些数据不是我们手动收集的，而是开发者和用户在使用我们的工具时自动回传的，完全由真实使用场景驱动。
有了这些真实数据，我们就能清楚地看到：哪些 Skill 在被持续使用、哪些反馈最好、更值得投入，从而做出最合理的优化决策。

我们后续也会建立一套对 SKILL 内容检查的机制，并将每个 SKILL的评分展示在平台上。


那么我们 在使用 SKILL 的时候，为什么数据会自动流向 devai-hub 被我们收集呢？

---

## Part 3 · The Execution Entry Point for Skills

**（约 12 分钟）**

接下来介绍
SKILLs 的统一执行入口，我们通过 devai 这个工具来实现数据收集。

1. devai 是一个 cli 工具， 通过执行 npx devai skills add 命令，将远程 skill 安装在本地使用时，它会自动上传在这过程中产生的数据。你可以通过 devai 来安装 skill， 也可以检查更新 SKILL.

2. 我们也提供了 npx devai setup-copilot 指令，将项目开发相关的 SKILL 预制好，只需要在项目中执行这个指令， 你就有拥有一套较为完善的 SKILL 和 Copilot 配置。 目前我们针对 java react angular 这三种技术类型的项目都有对应的配置。

使用devai 指令之前，你需要配置你的电脑： 第一：必须有 nodejs 环境 第二： 必须配置公司内部的 npm 镜像仓库地址。 具体说明你都可以在网站文档中找到。

刚刚我们讲到如何使用 devai 安装 SKILLS

那如何创建了一个 skill， 如何共享给其他同事使用呢？
  - 将 skill 上传到 公司的团队仓库中。如： https://github.com/mfc-innersource/skills
  - 通过 npx devai add https://github.com/mfc-innersource/skills 执行后，此SKILL 会自动出现在  Internal 类别中。这样其他的同事可以一键安装。

好了，到目前为止，我分享的这个平台做了两件事情

1. 使用 devai 安装 skill, 收集数据。 这里既可以是内部 Skill，也可以是外部社区的 Skill。
2. devai-hub 平台本身，它能快速分享 SKILL, 并通过数据，社区反馈，把优质 Skill 进一步放大。

所以这个闭环很清楚：
从创建 Skill，到使用 Skill，再到通过真实使用数据持续优化 Skill。

---

## Part 4 · A SKILL Live Demo: release-bot

**（约 14 分钟）**

接下来，我准备了一个 demo:
我将执行 **release-bot** 这个 SKILL，演示怎么把发版流程做成一键执行；

在演示之前，我先回顾一下日常发布版本时需要执行的动作：写发布日志，打 tag，提交 commit，更新版本号，发布 npm 包。

我想通过这个 Demo，让大家更直观地感受到 SKILL 的能力，一句话来说：它可以帮助我们将复杂的事情简单化。
在这个视频里，我只需要确认发布的版本号，（在这个视频中因为我选择了 AutoPilot 模式）它就会自动帮我们完成发版的整个流程。
让我打开这个视频

在这个 DEMO 中，我只演示了使用一个 SKILL ，那如果遇到多个 SKILL ， 他们互相有冲突，依赖，我们应该怎么处理呢？
接下来，请 Ian Chen 来继续分享， 看看我们对 SKILL 冲突，复用问题的思考结果，现阶段我们的解决方案。

在 Ian 分享之前，如果大家对我的分享有任何问题，可以写在 Q & A 中，谢谢。


