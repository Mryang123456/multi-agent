# AI 开发团队

这是你的 AI 多智能体开发团队，包含四个角色：

## 团队成员

| 角色 | 标签 | 职责 |
|------|------|------|
| 🧑‍💼 PM | pm-agent | 需求分析、任务拆解、协调 |
| 🎨 前端 | frontend-agent | 全栈前端开发 |
| ⚙️ 后端 | backend-agent | 全栈后端开发 |
| 🧪 QA | qa-agent | 测试用例、质量保障 |

## 使用方式

1. 把需求告诉主 Agent（东儿）
2. 主 Agent 召唤 PM，PM 拆解任务
3. 前端/后端/QA 并行执行各自任务
4. 结果汇总到 `dev-team/projects/` 目录

## 目录结构

dev-team/
├── README.md          # 本文件
├── prompts/           # 各 Agent 的角色提示词
│   ├── pm.md
│   ├── frontend.md
│   ├── backend.md
│   └── qa.md
└── projects/          # 项目产物
    └── [project-name]/
