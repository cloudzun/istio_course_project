# Istio课程规划文档

## 第一章：服务网格的崛起与 Istio 架构 (基础篇)
**目标：**
建立对 Istio 的全貌认知，理解服务网格的由来和核心框架。

**规划：**
- 预计学习时长：6小时
- **重点：**
  1. 微服务架构的痛点与服务网格基础知识。
  2. Istio 架构从 1.0 演进到最新版本（1.26）。
  3. 实战：使用 istioctl 安装基础环境，并验证安装结果。

**实验：**
1. 用 Istioctl 在 Kubernetes 中至少安装一个 Workload。
2. 深入解读服务网格相关的管理资源（ConfigMap、Service）。

---

## 第二章：数据平面的魔法 —— Sidecar 与应用接入 (接入篇)
**目标：**
理解 Sidecar 的核心功能与工作机制。

**规划：**
- 预计学习时长：5小时
- **重点：**
  1. 掌握 Sidecar 的注入机制及配置。
  2. 实践练习Bookinfo安装。理解与灰度发布相关内容。

**实验：**
1. 基于多种版本负载工作负载，初步适配全量新功能。
2. 优化 Envoy rib发行，42线程部署=`xprox-checkoned`对比历史ICP .

[Moreовор Collapse Auto ÇолntextsFeedback Evalsegment.]