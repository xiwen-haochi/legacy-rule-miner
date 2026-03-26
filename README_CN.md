# Legacy Rule Miner

老项目规则挖掘器 — 从后端遗留代码中自动提取编码规范、架构约定和已知坑点，生成 AI 可消费的规则文件，让 AI 编程助手（Copilot / Cursor / Claude 等）修改老项目代码**一次成功**。

## 核心理念

**向前兼容 > 替换重写**

生成的规则教 AI 如何与现有代码库协作，而不是对抗它。

## 支持的语言/框架

| 语言 | 框架 |
|------|------|
| Java | Spring Boot, SSM, Spring Cloud |
| PHP | ThinkPHP, Laravel, Yii |
| Python | Django, Flask, FastAPI, Tornado |
| Node.js | Express, Koa, Egg.js, NestJS |
| Go | Gin, Beego, Echo, net/http |
| .NET | .NET Core, .NET 5+ |

## 工作流程

1. **自动扫描**（~80%）— 逐层分析项目元数据、结构、配置、API、业务逻辑、数据访问、跨切面关注点和基础设施
2. **交互式补充**（~20%）— 通过提问收集口头约定、已知坑点、业务域约束等代码分析无法检测的信息

## 输出

自动检测 IDE 类型，将规则文件生成到对应 IDE 的原生规则目录中：

| IDE/工具 | 输出目录 |
|---------|----------|
| Cursor | `.cursor/rules/` |
| Trae | `.trae/rules/` |
| GitHub Copilot (VS Code) | `.github/instructions/` |
| Claude Code | `.claude/rules/` |
| Windsurf | `.windsurf/rules/` |
| 未知 / 兜底 | `.ruleminer/` |

```
{output-dir}/
├── legacy-rules.md              # 主规则文件（项目概览 / 架构 / 命名 / 编码 / API / 数据 / 安全 / 依赖 / 坑点 / 自定义规则）
├── legacy-rules-payment.md      # 模块级规则（仅在模块有特殊约定时生成）
└── legacy-rules-auth.md         # 模块级规则（仅在模块有特殊约定时生成）
```

特性：
- 文件末尾保留 `## 自定义规则` 专区，用户可手动补充规则，重新生成时不会覆盖
- 已有 `legacy-rules.md` 时进行增量更新，而非全量覆盖
- 模块级规则已存在则更新，不存在则新建

## 安装

### 通过 skills.sh 安装（推荐）

在终端运行：

```bash
npx skills add xiwen-haochi/legacy-rule-miner
```

会自动将 Skill 安装到你 IDE 的技能目录中。

### 手动安装

1. 克隆仓库：
   ```bash
   git clone https://github.com/xiwen-haochi/legacy-rule-miner.git
   ```
2. 将整个文件夹复制到你 IDE 的技能目录：
   - **VS Code (Copilot):** `<项目根目录>/.github/skills/legacy-rule-miner/`
   - **Cursor:** `<项目根目录>/.cursor/skills/legacy-rule-miner/`
   - **Claude Code:** `<项目根目录>/.agents/skills/legacy-rule-miner/`

## 使用方式

### 快速开始

1. **在 AI 编程 IDE 中打开你的老项目**（VS Code + Copilot / Cursor / Claude Code）
2. **确保 Skill 已安装**（见上方安装步骤）
3. **开始对话**，让 AI 分析你的项目：
   ```
   分析这个老项目，生成 legacy-rules.md
   ```
4. Skill 会自动执行：
   - 自动检测 IDE 类型，选择对应的输出目录
   - 扫描项目结构、配置、依赖、源代码（~80% 自动完成）
   - 向你提问口头约定、已知坑点、业务约束等信息（~20% 交互补充）
   - 在 IDE 原生规则目录中生成 `legacy-rules.md`

### 重新运行 / 更新

- 如果 `legacy-rules.md` 已存在，Skill 会进行**增量更新**，只更新有变化的章节
- 文件末尾的 `## 自定义规则` 专区**永远不会被覆盖**，你手动添加的规则是安全的
- 如需完全重新生成，删除 `legacy-rules.md` 后重新运行即可

## 五大核心原则

1. **Copy-Paste First** — 新代码模仿最近的同类型已有代码
2. **No Surprise** — 不引入项目未使用过的模式/库/语法
3. **Version Lock** — 不升级依赖版本
4. **Gradual Improvement Only** — 允许方法内小改良，不重构架构
5. **Pitfall Awareness** — 标注所有已知坑点

## 文件结构

```
legacy-rule-miner/
├── SKILL.md                          # 主 Skill 文件
├── references/
│   ├── analysis-playbook.md          # 分析维度 & 采样策略
│   ├── rule-writing-guide.md         # 规则撰写指南
│   ├── output-templates.md           # 输出模板
│   ├── interview-questions.md        # 交互式问题库
│   ├── java-spring.md               # Java 专属
│   ├── php-thinkphp-laravel.md      # PHP 专属
│   ├── python-django-flask.md       # Python 专属
│   ├── nodejs-express-koa-egg.md    # Node.js 专属
│   ├── go-specifics.md              # Go 专属
│   └── dotnet-specifics.md          # .NET 专属
├── ref/                             # 研究参考
├── evals/                           # 测试用例
└── README.md
```
