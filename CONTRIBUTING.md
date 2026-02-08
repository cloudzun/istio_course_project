# 贡献指南

感谢您对 Istio 服务网格实战教科书项目的关注！

## 如何贡献

### 1. 报告问题（Issue）

发现讲义中的错误、过时信息或不清晰的内容？请提交 Issue：

- 清晰描述问题发生的位置（章节、文件名、行号）
- 说明问题的具体内容
- 提供修正建议（如果有）

### 2. 提交改进（Pull Request）

我们欢迎以下类型的贡献：

- 📝 **内容改进**：修正错误、改进表述、补充例子
- 🔧 **配置更新**：更新过时的配置或命令
- 📚 **文档完善**：添加缺失的说明或最佳实践
- 🎨 **格式优化**：改进图表、表格或代码示例

### 3. 编写规范

为保持课程的一致性和高质量，请遵循以下规范：

**Markdown 格式：**
- 使用清晰的标题层级（# ## ### #### ）
- 代码块使用语言标识（```yaml, ```bash 等）
- 使用表格展示对比性内容
- 列表使用 - 或 * 符号

**内容质量：**
- 所有命令都应该能在当前 Istio 版本（1.26+）上运行
- 提供预期的输出示例
- 包含常见的错误和解决方案
- 尽量用实际场景和例子说明概念

**图表和流程：**
- 使用 ASCII 字符绘制架构图（便于版本控制）
- 使用箭头和缩进表示流程
- 对复杂的图表提供文字说明

### 4. Git 工作流

```bash
# 1. Fork 本仓库
git clone https://github.com/your-username/istio_course_project.git
cd istio_course_project

# 2. 创建功能分支
git checkout -b fix/your-fix-description
# 或
git checkout -b enhancement/your-feature-description

# 3. 进行修改并提交
git add <修改的文件>
git commit -m "type: description

详细说明（可选）
"

# 4. 推送并创建 PR
git push origin fix/your-fix-description
```

**提交信息规范：**

使用以下前缀清晰表示修改类型：

- `fix:` - 修正错误或过时信息
- `feat:` - 新增内容或章节
- `docs:` - 文档改进
- `refactor:` - 重新组织内容
- `style:` - 格式调整
- `chore:` - 维护性改动

示例：
```
fix: 更正第三章中 VirtualService 的示例配置

之前的配置在实际测试中会导致超时。
已修正 timeout 和 retries 的参数。

Closes #123
```

### 5. 审查流程

- 所有 PR 都会经过内容和格式审查
- 请耐心等待反馈（通常 2-3 个工作日）
- 根据反馈进行修改，并重新推送
- 一旦批准，PR 将被合并到 master 分支

---

## 项目结构

```
istio_course_project/
├── 讲义（核心课程）
│   ├── 01-istio-foundation-lecture.md
│   ├── 02-sidecar-injection-lecture.md
│   ├── 03-traffic-management-lecture.md
│   ├── 04-resilience-lecture.md
│   ├── 05-fault-injection-observability-lecture.md
│   ├── 06-security-lecture.md
│   ├── 07-advanced-observability-lecture.md
│   └── 08-advanced-extensions.md
│
├── 项目文档
│   ├── README.md                    # 课程导航和快速开始
│   ├── class-note.md               # 课程大纲和学习路径
│   ├── COURSE_SUMMARY.md           # 课程完成情况说明
│   ├── CONTRIBUTING.md             # 本文件
│   └── LICENSE
│
└── notes/                           # 背景资料
    ├── background.md               # 技术背景和发展历史
    ├── course_development_specifications.md  # 开发规范
    └── istio-lab-guide-2026-2-8.md # 9 个详细的实验手册
```

---

## 常见问题

**Q: 我可以添加新的章节吗？**
A: 核心的 7 个章节已完成。如果有重要的补充内容，可以：
- 在对应章节中添加小节
- 提出 Issue 讨论是否需要新的章节
- 在扩展篇中补充高级主题

**Q: 我想翻译成其他语言可以吗？**
A: 欢迎！请创建独立的分支或仓库来进行翻译，并在项目中链接。

**Q: 我发现了一个技术错误，应该如何处理？**
A: 请立即提交 Issue，标注为 `bug` 或 `critical`。我们会优先处理。

**Q: 可以添加视频或其他媒体文件吗？**
A: 不建议在仓库中存储大型媒体文件。可以在讲义中链接到外部资源。

---

## 行为准则

- 尊重所有贡献者和维护者
- 建设性地提供反馈和建议
- 不发布不尊重、歧视或骚扰他人的言论
- 遵守开源社区的一般准则

---

## 致谢

感谢所有为这个项目做出贡献的人！你们的努力使 Istio 的学习更加易于理解和实践。

---

有问题？欢迎提交 Issue 或在 Slack 社区讨论：[Istio Slack](https://slack.istio.io)