---
title: "发布！设计与部署稳定的分布式系统"
date: 2022-02-13T22:51:56+08:00
draft: false
---

## 生产环境的生存法则

### 瞄准正确的目标

今天的软件设计在与现实相差甚远的环境中进行，其目标在于通过 QA 部门的验收，不能保证在未来三五年的适用性。另一方面，制造业的产品设计师一直在追求能够让人们以低成本和高质量的方式制造产品的“可制造性设计”。软件行业的“可制造性设计”，就是“为生产环境而设计”。我们既需要设计一个个彼此独立的软件系统，也需要设计由相互依赖的系统所组成的整个生态系统，从而以低成本和高质量的方式进行运维工作。

### 应对不断扩大的挑战范围

是随着用户触点的增加和系统规模的扩大，系统遭到破坏的方式也会翻新，环境会变得更加恶劣，人们对缺陷的容忍度会变得更低。这个正在不断扩大的挑战范围，即以低成本快速构建和运维对用户有益的软件，要求我们持续改进架构和设计技术。

### 多花 5 万美元来节省 100 万美元

设计决策和架构决策，也是财务决策。在选择时，必须着眼于实施成本和下游成本。在假设可以投资 5 万美元来创建不停机发布的构建流水线和部署过程。这样做至少可以避免 100 万美元的停机部署损失，而且大有可能提高系统部署频率，占领更多市场份额。

### 让"原力"与决策同在

早期决策会对系统的最终形态产生巨大的影响。最早做出的决定可能是最难以反悔的。非常具有讽刺意味的是，早期决策恰恰是在信息最不完备的时候做出的。团队在启动项目时，往往最不了解软件的最终架构，却偏偏要在那时必须做出一些最不可能更改的决定。富有远见的决策将会大大减少整个软件声明周期的总成本。

### 设计务实的架构

务实的架构中，其中每个组件都足以满足的当前的负荷，并且，当符合随着时间发生变化时，架构师知道要替换那些组件。
