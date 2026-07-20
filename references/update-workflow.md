# 自动更新工作流

当用户要求「更新 moonwell-spring」或检测到 moon 版本变更时，按以下步骤执行。

## Phase 1：版本检测

```bash
moon version
```

与 SKILL.md 的 `tracked_version` 对比。版本号一致 → 跳到 Phase 4（增量检查）；不一致 → 进入 Phase 2。

## Phase 2：拉取变更信息

按顺序执行，每条记录关键变更点：

```bash
# A. moon --help 全量对比
moon --help
for cmd in $(moon --help | grep -oP '^\s{2}\w+'); do
  moon $cmd --help 2>/dev/null
done

# B. GitHub Release
gh release list --repo moonbitlang/moon --limit 5
gh release view <tag> --repo moonbitlang/moon   # 对每个比 tracked_version 新的 release

# C. 官方文档新增/变更
curl -sL "https://docs.moonbitlang.com/en/latest/toolchain/moon/" 2>/dev/null

# D. 官网 Blog / Updates
curl -sL "https://www.moonbitlang.com/blog/" 2>/dev/null
curl -sL "https://www.moonbitlang.com/updates/" 2>/dev/null

# E. 官方 skills 仓库更新
gh issue list --repo moonbitlang/skills --state all --limit 10
gh pr list --repo moonbitlang/skills --state all --limit 10

# F. moon 仓库近期 issue（已知问题/进行中功能）
gh issue list --repo moonbitlang/moon --limit 20
```

## Phase 3：差异分析与补丁生成

```
变更类型               →  处理方式
─────────────────────────────────────────────
新命令/子命令          →  新增补丁条目（如官方技能未覆盖）
命令参数变化           →  更新对应条目
配置文件格式变化        →  更新对应条目 + 标注迁移方式
属性/诊断码增减         →  更新属性列表/诊断列表
官方技能已覆盖的新特性   →  从本技能删除（不再需要补丁）
已知 bug/限制           →  新增 ⚠️ 警告条目
弃用/移除              →  新增条目 + 标注版本和替代方案
```

## Phase 4：增量检查（版本未变时）

```bash
gh issue list --repo moonbitlang/skills --state all --limit 5
gh pr list --repo moonbitlang/skills --state all --limit 5
moon --help   # 快速 diff 检查是否有新增子命令
```

## Phase 5：更新文档

1. 更新 frontmatter 中的 `tracked_version` 和 `last_updated`
2. 更新「版本追踪」表格
3. 新增/修改/删除对应补丁条目
4. 追加本次更新摘要到本文件末尾的「更新记录」
5. 更新 `## 验证` 中的检查命令（如果命令数量/属性数量变化）
6. 提交 commit：`hotfix: sync to moon <version> (<date>)`

---

## 📡 信息源列表

| 信息源 | URL | 用途 |
|:--|:--|:--|
| moon 工具链 `--help` | 本地命令 | 命令/参数权威来源 |
| GitHub Releases | `https://github.com/moonbitlang/moon/releases` | 版本变更日志 |
| 官方文档 | `https://docs.moonbitlang.com/en/latest/toolchain/moon/` | 工具链文档 |
| 官网 Blog | `https://www.moonbitlang.com/blog/` | 特性预告、发行说明 |
| 官网 Updates | `https://www.moonbitlang.com/updates/` | 更新动态 |
| 官方 Skills 仓库 | `https://github.com/moonbitlang/skills` | 技能上游变更 |
| moon 仓库 Issues | `https://github.com/moonbitlang/moon/issues` | 已知问题/进行中功能 |
| MoonBit 文档站 | `https://docs.moonbitlang.com/` | 完整文档索引 |

---

## 更新记录

### 2026-07-20：moon 0.1.20260713 → 0.1.20260717

1. `moon version` → 0.1.20260717 (438c06f 2026-07-17)
2. 命令列表无变化（30 个），属性无变化（18 个）
3. 新增选项：`moon check --fmt` / `--explain` / `--patch-file`
4. 新增选项：`moon run --build-only` / `moon test --build-only`
5. 新增能力：`moon run -` 从 stdin 读取 `.mbtx`
6. 废弃：`moon doc [SYMBOL]` → 用 `moon ide doc`
7. 新增补丁十八、十九

### 2026-07-19：初始创建（moon 0.1.20260626）

1. `moon version` → 确认版本 0.1.20260626
2. `moon --help` → 列出完整命令列表（27 个子命令）
3. 逐个读取 8 个官方 moonbit-* 技能文件
4. 逐项对比工具实际输出 vs 技能文档记录
5. `moon explain --attribute` → 18 个属性完整列表
6. `moon explain --diagnostic` → 确认诊断码列表
7. `moon new` → 确认默认创建 `moon.mod` 而非 `moon.mod.json`
8. 验证 `declare` 和 `#declaration_only` 均有效
9. 搜索官方仓库 issue/PR → 确认无他人提出相同问题
10. 生成 16 个补丁条目
