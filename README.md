# Istio 服务网格实战教科书（2026 版）

> 从零到生产：完整的 Istio 学习与运维指南

欢迎来到 **Istio 服务网格实战教科书**！本项目为学习和运维 Istio 的开发者和运维人员提供完整的知识体系和实践指南。

---

## 📚 课程结构

### 核心课程（7 章）

全面覆盖 Istio 从基础到高级的所有核心主题：

| 章节 | 标题 | 核心内容 | 文件 |
|------|------|--------|------|
| **第一章** | 服务网格的崛起与 Istio 架构基础 | 微服务痛点、Istio 架构、mTLS 自动化、实战安装 | `01-istio-foundation-lecture.md` |
| **第二章** | 数据平面的魔法 —— Sidecar 与应用接入 | Envoy 基础、Sidecar 注入、iptables 拦截、Bookinfo 部署、诊断工具 | `02-sidecar-injection-lecture.md` |
| **第三章** | 流量治理基础 —— 南北向与东西向流量管理 | VirtualService 详解、DestinationRule 详解、Gateway 配置、灰度发布、陷阱分析 | `03-traffic-management-lecture.md` |
| **第四章** | 弹性与韧性 —— 构建容错的微服务系统 | 超时、重试、熔断、限流、级联故障防护、电商场景实战 | `04-resilience-lecture.md` |
| **第五章** | 故障注入与可观测性 —— 提前发现问题 | 故障注入（延迟与错误）、流量镜像、分布式追踪、日志分析、诊断工作流 | `05-fault-injection-observability-lecture.md` |
| **第六章** | 安全基础 —— 零信任网络架构 | mTLS 自动化、PeerAuthentication、JWT 认证、AuthorizationPolicy、RBAC 配置 | `06-security-lecture.md` |
| **第七章** | 高级可观测性 —— 从数据到洞察 | 云原生可观测性概览、日志聚合、黄金指标、追踪、告警、诊断工作流 | `07-advanced-observability-lecture.md` |

### 扩展内容（超出核心大纲）

进阶主题和生态整合：

| 主题 | 内容 | 文件 |
|------|------|------|
| **Ambient Mesh** | 下一代无 Sidecar 架构（ztunnel + Waypoint） | `08-advanced-extensions.md` |
| **多集群管理** | 跨集群通信、服务发现、故障转移 | 同上 |
| **虚拟机集成** | VM + K8s 混合部署、WorkloadEntry 配置 | 同上 |
| **性能优化** | 资源调优、规则优化、高吞吐量场景 | 同上 |
| **生态整合** | Prometheus、Grafana、Jaeger、OpenTelemetry | 同上 |
| **最佳实践** | 规划、部署、安全、运维、升级流程 | 同上 |
| **学习资源** | 官方资源、社区参与、推荐学习路径 | 同上 |

---

## 🎯 学习路径

### 推荐 12 周学习计划

```
第 1-4 周：基础篇（理论 + 实践）
  ├─ 第 1-2 章：架构与数据平面
  ├─ 部署 Istio 和 Bookinfo 应用
  └─ 配置基本的流量管理

第 5-8 周：核心篇（深化 + 应用）
  ├─ 第 3-4 章：流量管理与容错
  ├─ 实验灰度发布、故障注入
  └─ 配置 mTLS 和安全策略

第 9-12 周：运维篇（完整系统）
  ├─ 第 5-7 章：可观测性与诊断
  ├─ 部署监控和告警系统
  └─ 在生产环境中实践
```

### 与 Lab Guide 的对应关系

课程讲义与实验手册紧密配合：

- 第 1-2 章 → Lab 1-2（安装与部署）
- 第 3 章 → Lab 3（流量管理）
- 第 4 章 → Lab 4（容错机制）
- 第 5 章 → Lab 5（故障注入与调试）
- 第 6 章 → Lab 7-8（安全）
- 第 7 章 → Lab 9（可观测性）

---

## 📖 内容特色

### ✨ 深度优于广度

每个知识点都深入到**原理层面**：

- 不只讲"如何用"，更讲"为什么这样设计"
- 不只给命令，更给背后的机制和权衡
- 包含实际案例和陷阱分析

### 🏗️ 清晰的架构设计

- **第 1 层**：概念与问题（为什么需要？）
- **第 2 层**：设计与原理（怎样解决？）
- **第 3 层**：配置与实践（如何应用？）
- **第 4 层**：诊断与优化（如何排查和改进？）

### 🎨 丰富的视觉化

- ASCII 架构图
- 时间线示例
- 状态机图解
- 对比表格
- 决策树

### 🔧 即用型参考

- 快速参考卡（CLI 命令、YAML 模板）
- 故障诊断检查清单
- 最佳实践速查表
- 常见陷阱与解决方案

---

## 📁 仓库结构

```
istio_course_project/
├── 01-istio-foundation-lecture.md          # 第一章：基础
├── 02-sidecar-injection-lecture.md         # 第二章：数据平面
├── 03-traffic-management-lecture.md        # 第三章：流量管理
├── 04-resilience-lecture.md                # 第四章：弹性与韧性
├── 05-fault-injection-observability-lecture.md  # 第五章：故障注入
├── 06-security-lecture.md                  # 第六章：安全
├── 07-advanced-observability-lecture.md    # 第七章：可观测性
├── 08-advanced-extensions.md               # 扩展篇：进阶主题
├── notes/                                  # 背景资料
│   ├── background.md                       # 技术背景与发展历史
│   ├── course_development_specifications.md # 课程开发规范
│   ├── class-note.md                       # 课程大纲与知识点映射
│   └── istio-lab-guide-2026-2-8.md        # 完整的实验手册（9 个 Lab）
├── exercises/                              # 练习题与作业
├── modules/                                # 模块化内容（待扩展）
├── slides/                                 # 演示文稿（待制作）
├── resources/                              # 辅助资源（图表、脚本等）
├── README.md                               # 本文件
├── CONTRIBUTING.md                         # 贡献指南
└── TASKS.md                                # 开发进度追踪
```

---

## 🚀 快速开始

### 1. 阅读顺序

**第一次学习：按顺序读 7 章**

```
01 → 02 → 03 → 04 → 05 → 06 → 07
```

**有经验的学员：可跳过/快速浏览**

```
已知 K8s 和 Docker：快速浏览第 1-2 章，重点看第 3-4 章
有 Nginx 经验：快速浏览第 3 章，重点看第 4-6 章
安全专家：快速浏览第 6 章，重点看第 7 章和扩展篇
```

### 2. 配合实验手册

每读完一章，做对应的 Lab：

```bash
阅读第 3 章（流量管理）
  ↓
完成 Lab 3（创建 VirtualService 和 DestinationRule）
  ↓
回到讲义，复习高级用法
  ↓
自己设计和实施一个灰度发布方案
```

### 3. 动手实践

```bash
# 克隆项目
git clone https://github.com/cloudzun/istio_course_project.git
cd istio_course_project

# 阅读讲义（Markdown）
# 推荐使用 VS Code + Markdown Preview 或其他 Markdown 阅读器

# 参考实验手册（notes/istio-lab-guide-2026-2-8.md）
# 在 Kubernetes 集群上完成每个 Lab

# 查看快速参考（文档末尾的命令和 YAML 模板）
```

---

## 📊 课程统计

| 指标 | 数值 |
|------|------|
| **核心章节** | 7 章 |
| **扩展内容** | 1 篇（bonus） |
| **总字数** | ~100KB |
| **平均每章** | 14KB |
| **覆盖的 Lab** | 9 个（Lab 1-9） |
| **代码示例** | 200+ 个 YAML |
| **诊断工作流** | 10+ 个 |
| **快速参考** | 5+ 个 |

---

## 🤝 贡献与反馈

### 如何贡献

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/improvement`)
3. 提交改进 (`git commit -m 'Add: ...'`)
4. 推送到分支 (`git push origin feature/improvement`)
5. 开启 Pull Request

详见 `CONTRIBUTING.md`。

### 反馈与建议

- 发现错误或遗漏？提交 Issue
- 有新的学习资源推荐？提交 PR
- 想讨论某个主题？创建 Discussion

---

## 📝 许可证

本项目使用 CC BY 4.0 许可证。详见 LICENSE 文件。

---

## 🔗 相关资源

### 官方资源
- [Istio 官方文档](https://istio.io/)
- [Istio GitHub 仓库](https://github.com/istio/istio)
- [Istio 官方博客](https://istio.io/latest/blog/)

### 推荐学习资源
- 书籍：《Istio in Action》（O'Reilly）
- 课程：Linux Foundation 的 Istio 认证课程
- 社区：[Istio Slack](https://slack.istio.io)

---

## 👥 鸣谢

本课程由 Abraham Cheng（[@cloudzun](https://github.com/cloudzun)）精心设计和编写。

感谢 Istio 社区的持续贡献和优秀的开源项目！

---

## 📞 联系方式

- 📧 Email：[联系方式]
- 💬 Slack：[Istio Slack 社区]
- 🐙 GitHub：[@cloudzun](https://github.com/cloudzun)

---

## 🎓 学完后的下一步

```
✓ 理解 Istio 架构和原理
✓ 能够部署和配置 Istio
✓ 能够排查常见问题
  ↓
下一步可以：
  1. 在生产环境中部署 Istio
  2. 贡献代码到 Istio 社区
  3. 学习相关的 CNCF 项目（Prometheus、Kubernetes 等）
  4. 在组织内推广服务网格文化
  5. 参加 Istio 和服务网格的技术大会
```

---

**最后的话：**

服务网格是云原生时代的必修课。希望这门课程能帮助你在微服务的旅途中更自信、更高效。

**祝学习愉快！** 🚀

---

*最后更新：2026-02-08*