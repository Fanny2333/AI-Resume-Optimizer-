# AI Resume Optimizer / AI 简历优化器

> 输入 JD + 简历 Markdown + 头像，AI 智能润色，一键生成东方水墨风格的精美 PDF 简历。

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Flask](https://img.shields.io/badge/Flask-3.0%2B-green)
![License](https://img.shields.io/badge/License-MIT-yellow)

---

## 项目简介

**AI 简历优化器**是一个轻量级的本地 Web 应用，帮助求职者根据目标岗位 JD（Job Description）智能优化简历内容，并输出具有「东方水墨」设计风格的高质量 PDF 简历。

核心理念是：**AI 只做润色和排版，不删改你的真实经历**。

---

## 用户与场景

### 目标用户

- **求职者**：正在准备投递心仪岗位，希望简历内容与 JD 高度匹配
- **职场转型者**：需要调整简历侧重点，突出与目标方向相关的经历
- **应届生**：有实习和项目经历但不知道如何组织表达
- **猎头 / HR**：帮助候选人快速定制简历

### 典型使用场景

| 场景 | 痛点 | 解决方案 |
|------|------|----------|
| 投递不同公司，需要定制简历 | 每次手动改太慢，容易遗漏 | 输入 JD，AI 自动调整措辞重点 |
| 简历内容写好了，但排版丑 | 不会设计，Word 模板千篇一律 | 一键生成东方水墨风格 PDF |
| 担心 AI 乱改简历 | 很多 AI 工具会编造经历或删减内容 | 明确规则：只润色不增删，个人介绍贴合 JD 重写 |
| 关键数据不够醒目 | HR 扫一眼看不到亮点 | 自动用红色高亮关键结果数字 |

---

## 需求分析

在开发之前，我们调研了现有简历工具的常见问题：

1. **内容丢失**：大部分 AI 简历工具会"精简"简历，导致工作经历、项目经历被删减或合并，用户的完整履历无法保留
2. **风格单一**：生成的 PDF 排版像 Word 导出，缺乏设计感
3. **无法定制**：同一份简历投所有公司，没有针对 JD 的优化
4. **隐私顾虑**：在线工具需要上传个人信息到第三方服务器
5. **个人介绍千篇一律**：个人优势部分缺乏针对性，没有贴合目标岗位

---

## 产品解决方案

### 核心功能

```
输入端                          AI 处理                         输出端
┌─────────────┐            ┌──────────────┐            ┌─────────────┐
│  JD 岗位要求  │──┐         │              │            │  Markdown   │
│  (Markdown)  │  │         │  LLM 智能优化  │───────────→│  预览 + 编辑 │
├─────────────┤  │         │              │            ├─────────────┤
│  原始简历     │──┼────────→│  · 润色措辞   │            │  PDF 下载   │
│  (Markdown)  │  │         │  · 个人介绍重写 │            │  东方水墨风格 │
├─────────────┤  │         │  · 关键数据高亮 │            └─────────────┘
│  头像上传     │──┘         │  · 内容完整保留 │
└─────────────┘            └──────────────┘
```

### 设计原则

- **内容完整性优先**：AI 被严格约束——工作经历、项目经历、教育经历一个字都不能少，只允许润色措辞
- **个人介绍贴合 JD**：这是唯一允许 AI 根据岗位重写的部分，采用"一句话总括 + 分点陈述"的结构
- **克制使用红色**：仅高亮关键结果数据（如金额、增长率、准确率），避免满屏红色
- **本地运行**：Flask 在用户本机启动，API Key 不经过任何中间服务器
- **多模型可选**：内置 DeepSeek、通义千问、Moonshot、智谱 AI 四大服务商预设，也可自定义任意 OpenAI 兼容接口

---

## 技术实现方案

### 技术栈

| 层次 | 技术选型 | 说明 |
|------|---------|------|
| 后端 | Python 3.10+ / Flask 3.0 | 轻量 Web 框架，单文件即可启动 |
| 前端 | 原生 HTML + CSS + JavaScript | 零框架依赖，单文件 SPA |
| AI 调用 | OpenAI 兼容 API (requests) | 支持所有兼容 OpenAI 格式的国内大模型 |
| PDF 渲染 | Playwright (Chromium) | 将 HTML 高保真渲染为 PDF，支持复杂 CSS 排版 |
| 样式 | 东方水墨 CSS Design System | 自定义设计令牌（cream、vermillion、ink-black 等） |

### 架构设计

```
浏览器 (index.html)
    │
    ├── POST /api/optimize      →  Flask app.py  →  LLM API (DeepSeek / Qwen / ...)
    │    { jd, resume, apiKey }      ↓ 构造 prompt      ↓ 返回 Markdown
    │    ← { optimized }             ↑ system_prompt
    │
    ├── POST /api/generate-pdf  →  Flask app.py
    │    { markdown, photo }         ↓ markdown_to_html()
    │                                ↓ build_resume_html() + resume_template.html
    │                                ↓ Playwright Chromium 渲染
    │    ← PDF 文件下载
    │
    └── POST /api/upload-photo  →  图片转 Base64
```

### 关键实现细节

**AI 提示词工程**

系统的核心在于精心设计的 Prompt，它分为多个层级的规则：

- **最高优先级**：保留所有原始内容，禁止删除、合并、重排
- **唯一例外**：个人介绍部分根据 JD 重新撰写，要求结构化输出
- **红色高亮指令**：告诉 AI 使用 `==数据==` 语法标记关键数字，由渲染引擎转为红色
- **负面约束**：明确列出禁止操作（编造经历、缩减描述等）

**Markdown → HTML 渲染器**

自研轻量 Markdown 解析器，针对简历场景优化，支持：

- 标题层级（`#` → 姓名，`##` → 章节，`###` → 子标题）
- 列表（`-` / `*`，连续项自动合并）
- 行内语法（加粗、斜体、链接）
- 自定义语法：`==text==` → 红色高亮（`.metric` 类）

**PDF 生成**

使用 Playwright 启动无头 Chromium 浏览器，将完整 HTML（含东方水墨 CSS）渲染为 A4 PDF。通过 print-optimized CSS 控制分页、字号、间距，确保输出美观紧凑。

### 项目结构

```
resume-optimizer/
├── app.py                         # Flask 后端（路由 + AI 调用 + Markdown 渲染 + PDF 生成）
├── requirements.txt               # Python 依赖
├── .gitignore                     # Git 忽略规则
├── README.md                      # 本文件
├── static/
│   └── uploads/                   # 头像临时存储目录
│       └── .gitkeep
└── templates/
    ├── index.html                 # 前端单页应用（UI + 交互逻辑）
    └── resume_template.html       # PDF 简历模板（东方水墨 CSS）
```

---

## 快速开始

### 环境要求

- Python 3.10+
- pip

### 安装与运行

```bash
# 1. 克隆项目
git clone https://github.com/your-username/resume-optimizer.git
cd resume-optimizer

# 2. 创建虚拟环境（推荐）
python -m venv venv
source venv/bin/activate   # macOS / Linux
# venv\Scripts\activate    # Windows

# 3. 安装依赖
pip install -r requirements.txt

# 4. 安装 Playwright 浏览器（仅首次）
playwright install chromium

# 5. 启动服务
python app.py
```

打开浏览器访问 **http://localhost:5001** 即可使用。

### 使用步骤

1. **选择 AI 服务商**：从下拉菜单选择预设（DeepSeek / 通义千问 / Moonshot / 智谱），填入你的 API Key
2. **输入 JD**：粘贴目标岗位的招聘要求（Markdown 或纯文本均可）
3. **输入简历**：粘贴你的简历 Markdown（建议结构：`# 姓名` → `## 个人优势` → `## 工作经历` → `## 项目经历` → `## 教育经历`）
4. **上传头像**（可选）：支持 JPG/PNG，不超过 5MB
5. **点击「开始优化」**：AI 将逐步展示处理进度
6. **预览 & 下载**：右侧查看优化结果，满意后点击「下载 PDF」

---

## 功能特性

### AI 智能优化

- 根据 JD 润色简历措辞，突出匹配经验
- 个人介绍部分根据 JD 重新撰写（一句话总括 + 3-5 个分点陈述）
- 严格保留原始内容，不删减、不合并、不重排
- 关键结果数据自动红色高亮（如金额、增长率、准确率）

### 多模型支持

内置四大国内 AI 服务商预设，同时支持自定义任意 OpenAI 兼容 API：

| 服务商 | 默认模型 | 特点 |
|--------|---------|------|
| DeepSeek | deepseek-chat | 性价比高，中文能力强 |
| 通义千问 | qwen-plus | 阿里大模型，擅长中文 |
| Moonshot | moonshot-v1-8k | 长文本处理能力强 |
| 智谱 AI | glm-4-flash | 清华系大模型 |

### 东方水墨 PDF 样式

- 奶白底色 + 墨色正文 + 朱红点缀
- 章节标题带装饰线和圆点
- 紧凑排版，适合 3-4 页 A4 简历
- 页眉提示：「所有文字均为人工手打，仅样式设计使用 AI，请放心使用」

### 用户体验

- 左右分栏：左侧输入，右侧实时预览
- 步骤进度动画：缓解等待焦虑
- API Key 仅在前端使用，不上传任何服务器
- 支持复制优化后的 Markdown 文本

---

## 自定义与扩展

### 添加新的 AI 服务商

编辑 `app.py` 中的 `PROVIDERS` 字典：

```python
PROVIDERS = {
    "your-provider": {
        "name": "你的模型",
        "apiUrl": "https://your-api.com/v1/chat/completions",
        "model": "your-model-name",
        "description": "模型特点描述",
    },
}
```

### 修改 PDF 样式

编辑 `templates/resume_template.html` 中的 CSS。关键设计令牌：

```css
--bg-cream:    #faf8f4;   /* 奶白底色 */
--ink-black:   #1a1a1a;   /* 墨黑 */
--vermillion:  #c53030;   /* 朱红（高亮色） */
--gold-accent: #8b6914;   /* 金色点缀 */
```

### 调整 AI 优化策略

修改 `app.py` 中 `optimize()` 函数的 `system_prompt` 变量，可自定义优化规则和输出格式。

---

## Markdown 简历格式参考

```markdown
# 张三

**男 | 28岁 | 北京**
**电话：** 138xxxx0000  **邮箱：** zhangsan@email.com
**6年工作经验 | 求职意向：AI 产品经理**

---

## 个人优势

6 年 AI 产品经验，覆盖产品全生命周期。

- **AI 系统架构：** 主导过多个 Agent 系统的从 0 到 1 搭建
- **业务增长：** 负责产品累计服务 ==200 万+== 用户

---

## 工作经历

### XX 科技有限公司 — 高级产品经理

**2022.03 - 至今**

**业务背景：** 描述你负责的业务和产品...

- **成果一：** 推动产品 GMV 增长 ==150%==
- **成果二：** 用户留存率提升至 ==45%==

---

## 项目经历

### 智能客服 Agent — 产品负责人

**2023.06 - 2024.12 · XX 科技**

**背景：** 项目背景描述...
**行动：** 你的具体行动...
**结果：** 项目取得了什么成果...

---

## 教育经历

### XX 大学

**本科 · 计算机科学 · 2014-2018**
```

---

## 常见问题

**Q: API Key 安全吗？**
A: API Key 仅在浏览器中使用，直接调用你选择的 AI 服务商接口，不经过本项目任何服务器。

**Q: 为什么 PDF 生成失败？**
A: 请确保已安装 Playwright 的 Chromium 浏览器（`playwright install chromium`）。

**Q: 支持哪些 AI 模型？**
A: 任何兼容 OpenAI Chat Completions API 格式的模型均可使用，包括本地部署的 Ollama 等。

**Q: 可以自定义 API 地址吗？**
A: 可以。选择服务商后，API 地址和模型名称输入框均可自由修改。如果只填基础域名（如 `https://api.deepseek.com`），系统会自动补全路径。

---

## 贡献指南

欢迎提交 Issue 和 Pull Request。在提交 PR 之前，请确保：

1. 代码风格与现有代码保持一致
2. 新功能有相应的说明
3. 不引入不必要的依赖

---

## 许可证

本项目基于 [MIT License](LICENSE) 开源。
