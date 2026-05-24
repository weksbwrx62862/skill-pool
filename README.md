<h1 align="center">Skill Pool</h1>

<p align="center"><strong>技能池向量索引 + 语义搜索 + core/pool 分类管理</strong></p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10+-blue.svg" alt="Python">
  <img src="https://img.shields.io/badge/License-MIT-green.svg" alt="License">
  <img src="https://img.shields.io/github/stars/weksbwrx62862/skill-pool?style=social" alt="Stars">
  <img src="https://img.shields.io/github/last-commit/weksbwrx62862/skill-pool" alt="Last Commit">
  <img src="https://img.shields.io/badge/Hermes-%3E%3D2.0.0-orange.svg" alt="Hermes Compat">
  <img src="https://img.shields.io/badge/version-1.0.0-blue.svg" alt="Version">
</p>

---

## 简介

Skill Pool 是 Hermes Agent 的语义技能索引插件，将技能描述编码为高维语义向量并构建 FAISS 索引，实现毫秒级语义检索。通过 core/pool 双层架构，核心技能常驻内存保障响应速度，池化技能按需加载节省资源，配合自动调优与快照机制，为 Agent 提供高效、灵活的技能调度能力。

## 功能矩阵

| 能力 | 描述 | 对应工具 |
|------|------|----------|
| 向量索引 | 将技能描述编码为语义向量并构建 FAISS 索引 | `skill_pool_build` |
| 语义搜索 | 基于向量相似度的毫秒级技能检索 | `skill_pool_search` |
| core/pool 分层 | 核心技能常驻 + 池化技能按需加载 | `skill_pool_set_core` / `skill_pool_set_pool` |
| 自动调优 | 自动优化检索参数（top_k、阈值等） | `skill_pool_auto_tune` |
| 快照管理 | 创建/恢复技能池状态快照 | `skill_pool_snapshot` |
| 技能列表 | 查看所有已索引技能及分类 | `skill_pool_list` |
| 使用统计 | 查看技能调用频次与命中率 | `skill_pool_usage` |

## 架构图

```
┌─────────────────────────────────────────────────────┐
│                   Hermes Agent                       │
│                                                      │
│  ┌───────────┐    ┌──────────────┐    ┌───────────┐ │
│  │  Query     │───▶│  Skill Pool  │───▶│  Result   │ │
│  │  Request   │    │   Plugin     │    │  Ranking  │ │
│  └───────────┘    └──────┬───────┘    └───────────┘ │
│                          │                           │
│              ┌───────────┼───────────┐               │
│              ▼           ▼           ▼               │
│     ┌────────────┐ ┌──────────┐ ┌──────────┐        │
│     │ Vector     │ │ Skill    │ │ Auto     │        │
│     │ Index      │ │ Manager  │ │ Tune     │        │
│     │ (FAISS)    │ │ (core/   │ │ Engine   │        │
│     │            │ │  pool)   │ │          │        │
│     └─────┬──────┘ └────┬─────┘ └────┬─────┘        │
│           │              │            │               │
│           ▼              ▼            ▼               │
│     ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│     │ sentence │  │ Snapshot │  │ Usage    │        │
│     │ -trans-  │  │ Store    │  │ Tracker  │        │
│     │ formers  │  │          │  │          │        │
│     └──────────┘  └──────────┘  └──────────┘        │
└─────────────────────────────────────────────────────┘
```

## 快速开始

### 前置条件

- Python 3.10+
- Hermes Agent >= 2.0.0

### 安装

```bash
git clone https://github.com/weksbwrx62862/skill-pool.git
cd skill-pool
pip install -e .
```

### 依赖安装

```bash
pip install sentence-transformers faiss-cpu pyyaml
```

### 最小示例

1. 在 Hermes 配置中注册插件：

```yaml
# hermes_config.yaml
plugins:
  - name: skill_pool
    path: ./skill-pool
```

2. 构建技能索引：

```bash
skill_pool_build --source ./skills/
```

3. 语义搜索技能：

```bash
skill_pool_search --query "调试 Python 脚本" --top_k 5
```

## 核心功能详解

### 向量索引

通过 `sentence-transformers` 将技能描述编码为语义向量，使用 FAISS 构建高效近似最近邻索引。支持增量构建与全量重建，索引构建后自动持久化到磁盘。

```bash
skill_pool_build --source ./skills/          # 全量构建
skill_pool_build --source ./skills/ --incremental  # 增量构建
```

### core/pool 分层管理

技能分为两层：

- **core 层**：核心技能，启动时全量加载到内存，保障高频技能的零延迟响应
- **pool 层**：池化技能，按需从索引中检索加载，节省内存占用

```bash
skill_pool_set_core --skill debug_tool       # 提升为核心技能
skill_pool_set_pool --skill legacy_adapter   # 降级为池化技能
skill_pool_list                               # 查看所有技能及分类
```

### 语义搜索

基于向量相似度的语义检索，支持自然语言查询，返回 top_k 最相关技能及相似度分数。

```bash
skill_pool_search --query "解析 JSON 配置文件" --top_k 5
```

### 自动调优

`skill_pool_auto_tune` 根据历史使用数据自动优化检索参数，包括 top_k 值、相似度阈值、core/pool 分配策略等，持续提升检索精度与资源效率。

```bash
skill_pool_auto_tune                         # 执行一轮自动调优
```

### 快照管理

`skill_pool_snapshot` 创建技能池完整状态快照，包含索引数据、分类配置和使用统计，支持从快照恢复。

```bash
skill_pool_snapshot --create --tag v1.0      # 创建快照
skill_pool_snapshot --restore --tag v1.0     # 恢复快照
```

### 使用统计

`skill_pool_usage` 提供技能调用频次、命中率、平均检索延迟等运行时指标，辅助调优决策。

```bash
skill_pool_usage                             # 查看使用统计
```

## 技术栈

```
┌──────────────────┬──────────────────────────────┐
│ 类别             │ 技术                          │
├──────────────────┼──────────────────────────────┤
│ 语言             │ Python 3.10+                  │
│ 向量编码         │ sentence-transformers         │
│ 向量索引         │ faiss-cpu                     │
│ 配置管理         │ PyYAML                        │
│ 插件框架         │ Hermes Agent >= 2.0.0         │
└──────────────────┴──────────────────────────────┘
```

## 项目结构

```
skill-pool/
├── plugin.yaml          # 插件声明（名称、版本、工具列表）
├── __init__.py          # 插件主入口
├── vector_index.py      # 向量索引引擎（编码 + FAISS 索引）
└── skill_manager.py     # 技能分类管理（core/pool 分层）
```

## 开发指南

### 环境搭建

```bash
git clone https://github.com/weksbwrx62862/skill-pool.git
cd skill-pool
pip install -e .
```

### 开发流程

1. 从 main 创建 feature 分支：`git checkout -b agent/<task-id>-<description>`
2. 编写代码并确保通过 Hermes 运行时测试
3. 提交遵循 Conventional Commits 规范
4. 创建 PR 并等待审查合并

### 扩展指南

- 新增检索策略：在 `vector_index.py` 中实现自定义 FAISS 索引类型
- 新增分类规则：在 `skill_manager.py` 中扩展 core/pool 分配逻辑
- 新增工具：在 `plugin.yaml` 的 `provides_tools` 中声明，并在 `__init__.py` 中注册

## 路线图

- [ ] 支持多模型向量编码切换（OpenAI / Cohere 等）
- [ ] GPU 加速索引构建（faiss-gpu）
- [ ] 分布式索引支持
- [ ] 技能衰减机制（长期未用技能自动降级）
- [ ] Web UI 管理面板
- [ ] 技能依赖图可视化

## 常见问题

**Q: FAISS 索引占用内存过大怎么办？**

A: 可使用 FAISS 的 IVF/PQ 压缩索引，或通过 core/pool 分层将低频技能移至 pool 层按需加载。

**Q: 语义搜索结果不准确怎么办？**

A: 运行 `skill_pool_auto_tune` 自动调优检索参数，或检查技能描述是否足够清晰具体。

**Q: 如何从快照恢复到之前的状态？**

A: 使用 `skill_pool_snapshot --restore --tag <标签名>` 即可恢复到指定快照状态。

**Q: 是否支持增量构建索引？**

A: 支持，使用 `skill_pool_build --source ./skills/ --incremental` 仅索引新增或变更的技能。

## Contributing

欢迎贡献！请遵循以下流程：

1. Fork 本仓库
2. 创建 feature 分支（`git checkout -b agent/<task-id>-<description>`）
3. 提交变更（遵循 Conventional Commits 规范）
4. 推送到分支（`git push origin agent/<task-id>-<description>`）
5. 创建 Pull Request

请确保 PR 描述包含：变更摘要、设计决策、测试覆盖情况。

## License

[MIT](LICENSE)

## Security

如发现安全漏洞，请**不要**在公开 Issue 中报告。请通过以下方式私下联系维护者：

- 在 GitHub 上创建 Security Advisory
- 发送邮件至仓库维护者

请勿在代码、配置或日志中暴露 API Key、Token 等敏感信息。

## 致谢

- [sentence-transformers](https://www.sbert.net/) — 高质量语义向量编码
- [FAISS](https://github.com/facebookresearch/faiss) — 高效向量相似性搜索
- [Hermes Agent](https://github.com/weksbwrx62862/hermes) — 插件运行框架

---

<p align="center"><em>Skill Pool — 让 Agent 精准找到每一个技能</em></p>
