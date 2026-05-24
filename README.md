# Skill Pool

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10+-blue.svg" alt="Python">
  <img src="https://img.shields.io/badge/License-MIT-green.svg" alt="License">
  <img src="https://img.shields.io/badge/Hermes-%3E%3D2.0.0-orange.svg" alt="Hermes">
  <img src="https://img.shields.io/badge/version-1.0.0-blue.svg" alt="Version">
</p>

技能池向量索引 + 语义搜索 + core/pool 分类管理。为 Hermes Agent 提供技能的向量化索引和高效检索能力。

## 核心能力

- **向量索引**：将技能描述编码为语义向量，支持语义搜索
- **core/pool 分层**：核心技能常驻，pool 技能按需加载
- **自动调优**：`skill_pool_auto_tune` 自动优化检索参数
- **快照管理**：`skill_pool_snapshot` 创建技能池状态快照

## 安装

### 前置条件

- Python 3.10+
- [Hermes Agent](https://github.com/weksbwrx62862/hermes) >= 2.0.0

### 从源码安装

```bash
git clone https://github.com/weksbwrx62862/skill-pool.git
cd skill-pool
pip install -e .
```

### 依赖

```bash
pip install sentence-transformers faiss-cpu pyyaml
```

## 使用

```yaml
# hermes_config.yaml
plugins:
  - name: skill_pool
    path: ./skill-pool
```

### 构建技能池

```yaml
skill_pool_build --source ./skills/
```

### 搜索技能

```yaml
skill_pool_search --query "调试 Python 脚本" --top_k 5
```

## 提供的工具

| 工具 | 功能 |
|------|------|
| `skill_pool_build` | 构建/重建技能向量索引 |
| `skill_pool_search` | 语义搜索技能 |
| `skill_pool_list` | 列出所有索引技能 |
| `skill_pool_set_core` | 将技能标记为核心技能 |
| `skill_pool_set_pool` | 将技能移至按需池 |
| `skill_pool_auto_tune` | 自动优化检索参数 |
| `skill_pool_snapshot` | 创建状态快照 |
| `skill_pool_usage` | 查看使用统计 |

## 项目结构

```
skill-pool/
├── plugin.yaml         # 插件声明
├── __init__.py          # 主入口
├── vector_index.py     # 向量索引引擎
└── skill_manager.py    # 技能分类管理
```

## 开发

```bash
git clone https://github.com/weksbwrx62862/skill-pool.git
cd skill-pool
pip install -e .
# 通过 Hermes 运行时测试
```

## License

MIT