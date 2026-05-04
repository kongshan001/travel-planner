# Kanban Framework 验证日志

---

## 第4次验证-实践验证 2026-05-04 17:15

> 通过完整走通 kanban-framework 的 plan → execute → evaluate 流程，使用真实任务 "MVP核心-地图展示与POI搜索"，在实践中发现框架问题。

### 验证环境

- 项目: travel-planner
- 任务: TASK-003 "MVP核心-地图展示与POI搜索"
- 框架版本: 最新 main 分支
- 系统: Linux 6.8.0 (x64), bash, jq, git

---

### 1. 初始化确认

| 项目 | 结果 |
|------|------|
| `.kanban/` 目录结构 | ✅ 完整 |
| `config.json` | ✅ 正常 |
| `workflow.json` | ✅ 正常 |
| `index.json` | ✅ 存在 |
| `templates/` | ✅ 存在 |

---

### 2. 创建任务 (`kanban_create_task`)

**调用**: `kanban_create_task "MVP核心-地图展示与POI搜索" "...描述..."`

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| 创建任务目录 | `.kanban/tasks/TASK-003/` | 同预期 | ✅ |
| 生成 task.json | 包含完整字段 | 同预期 | ✅ |
| 生成 inbox.md | 创建收件箱文件 | 同预期 | ✅ |
| 更新 index.json | 新增 TASK-003 条目 | 同预期 | ✅ |
| 分支命名 | `feature/TASK-003` | 同预期 | ✅ |

**改进建议**: 无

---

### 3. Plan 阶段

#### 3.1 `workflow_transition "TASK-003" "plan"`

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| 状态更新 | status=planning, phase=plan | 同预期 | ✅ |
| iteration 递增 | 0 → 1 | 同预期 | ✅ |
| 创建 iteration-1 目录 | 自动创建 | 同预期 | ✅ |
| phase_lock 设置 | phase_lock=plan | 同预期 | ✅ |
| history 记录 | 新增进入记录 | 同预期 | ✅ |

#### 3.2 手动创建产物后 `guard_check_artifacts "TASK-003" "plan"`

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| 检查 requirements.md | 存在不报错 | 返回空（无缺失） | ✅ |
| 检查 task_breakdown.json | 存在不报错 | 返回空（无缺失） | ✅ |

#### 3.3 `guard_check_plan_quality "TASK-003"`

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| 评分输出 | 返回 JSON + stdout 摘要 | 同预期 | ✅ |
| 评分准确性 | 根据产物内容评分合理 | score=10.00, pass=true | ✅ |
| 维度细分 | 4个维度独立评分 | requirement_clarity=10, technical_feasibility=10, task_decomposition=10, acceptance_criteria=10 | ✅ |
| JSON 文件输出 | 写入 `.plan_quality.json` | 同预期 | ✅ |

**评价**: Plan 质量检查功能完善，评分维度合理。

#### 3.4 `workflow_complete_phase "TASK-003"` (plan 完成)

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| 初始化子任务 | 从 task_breakdown.json 读取 | Initialized 3 subtasks | ✅ |
| 更新 task_breakdown.file | 指向正确路径 | 同预期 | ✅ |

---

### 4. Execute 阶段

#### 4.1 `worktree_create "TASK-003" "feature/TASK-003"`

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| 创建 worktree | 在 `.kanban/worktrees/TASK-003` | 同预期 | ✅ |
| 更新 task.json | worktree.path 和 branch | 同预期 | ✅ |
| 幂等性 | 重复调用跳过 | 同预期 | ✅ |

**注意**: `git worktree add` 会输出 "HEAD is now at..." 提示，不影响功能但可能让用户困惑。

#### 4.2 `workflow_transition "TASK-003" "execute"`

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| Guard 检查 worktree | 检查目录存在+有效 git | 同预期 | ✅ |
| 状态更新 | status=executing, phase=execute | 同预期 | ✅ |

#### 4.3 `kanban_prepare_dispatch "TASK-003"`

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| 生成调度 JSON | 包含 task_id, title, worktree 等 | 同预期 | ✅ |
| 输出路径 | 返回文件路径 | `.kanban/dispatch/TASK-003-execute.json` | ⚠️ |

**问题**: `dispatch_dir()` 在首次创建时返回旧格式 `.kanban/dispatch/`（因为新目录还没创建），导致 execute dispatch 文件写入了旧位置 `.kanban/dispatch/` 而非新位置 `.kanban/tasks/TASK-003/dispatch/`。这与 evaluator 的 dispatch 文件位置不一致。

**改进建议**: `dispatch_dir()` 应在返回前自动 `mkdir -p` 新格式目录，或默认使用新格式路径。

#### 4.4 `kanban_update_subtask`

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| 更新 ST-001 → in_progress | 状态变更 | 同预期 | ✅ |
| 更新 ST-001 → completed | 状态变更 | 同预期 | ✅ |
| 更新 ST-002 → in_progress | 状态变更 | 同预期 | ✅ |
| 其他子任务不受影响 | ST-003 保持 pending | 同预期 | ✅ |

---

### 5. Evaluate 阶段

#### 5.1 `evaluator_prepare_all "TASK-003"`

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| 生成4个角色 dispatch | 每个角色独立 JSON | ⚠️ 文件创建但内容为空 | ❌ |

**🐛 BUG: 模板文件名不匹配**

`evaluator_prepare_all` 中模板路径为 `$SKILL_DIR/templates/reports/${role}.json`，其中 role 使用下划线格式（`code_reviewer`），但实际模板文件名使用连字符格式（`code-reviewer.json`）。

```bash
# 代码期望:
templates/reports/code_reviewer.json  # ❌ 不存在

# 实际文件:
templates/reports/code-reviewer.json  # ✅ 存在
```

这导致 `cat` 读取失败，jq 收到空输入报错：
```
cat: .../templates/reports/code_reviewer.json: No such file or directory
jq: invalid JSON text passed to --argjson
```

**生成的 dispatch 文件为空**，后续 agent 无法获取正确的上下文信息。

**修复方案**: 在 `evaluator_prepare_all` 中添加文件名转换：
```bash
local template_name=$(echo "$role" | tr '_' '-')
local template="$SKILL_DIR/templates/reports/${template_name}.json"
```

#### 5.2 `evaluator_record_score`

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| 记录 code_reviewer | score=8.5, passed=false | 同预期 | ✅ |
| 记录 qa | score=9.0, passed=true | 同预期 | ✅ |
| 记录 pm | score=9.5, passed=true | 同预期 | ✅ |
| 记录 designer | score=8.0, passed=false | 同预期 | ✅ |
| history 追加 | 新增 score_recorded 事件 | 同预期 | ✅ |

#### 5.3 `guard_check_evaluation`

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| 检查4个角色报告 | 全部存在则 PASS | PASS | ✅ |
| 检查必需字段 | improvements, risks, 角色特有字段 | 同预期 | ✅ |

#### 5.4 `evaluator_check_pass`

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| 阈值 9.0 | code_reviewer(8.5) < 9.0 → FAIL | `FAIL:code_reviewer:score=8.5<9.0` | ✅ |

---

### 6. 其他功能测试

#### 6.1 `kanban_iteration_summary`

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| 展示迭代次数 | 1 | 同预期 | ✅ |
| 展示评分趋势 | 4个角色分数 | 同预期 | ✅ |
| 展示扣分项 | 显示 improvements | 同预期 | ✅ |
| 展示可选操作 | 4个 decide 选项 | 同预期 | ✅ |

#### 6.2 `kanban_changes_summary`

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| 需求背景 | 展示 FR 标题 | 同预期 | ✅ |
| 技术方案 | 展示 DEC 标题 | ⚠️ 展示了 `# DEC-` 而非 `### DEC-` | ⚠️ |
| 任务拆解 | 展示表格 | 同预期 | ✅ |
| 质量评估 | 展示评分趋势 | 同预期 | ✅ |
| 变更文件 | 展示 git diff | 0 commits（worktree 无实际提交） | ✅ |

**小问题**: 技术方案提取使用了 `grep -E '^#{2,3} DEC-'`，但我们的 execution_decisions.md 中用的是 `## DEC-`（二级标题）。正则能匹配，但输出格式不太美观。不算 bug。

#### 6.3 `kanban_show_task`

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| 展示任务详情 | ID, 状态, 子任务, 评分 | 同预期 | ✅ |
| 子任务进度 | 1/3 completed | 同预期 | ✅ |
| 评分展示 | 4个角色 + PASS/FAIL | 同预期 | ✅ |

#### 6.4 `self_improve_evolve_skills`

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| 列出候选 | 显示 iteration pitfalls | ⚠️ `seq` 报错 | ⚠️ |

**问题**: `seq` 收到空字符串参数（`iteration` 从 task_file 读取失败时），导致 `seq: invalid floating point argument: ''`。

**改进建议**: 添加空值保护：`local iter=${iter:-0}` 或在调用 seq 前检查。

#### 6.5 `kanban_archive_task` (无用户批准)

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| 拒绝归档 | 提示需要用户确认 | `ERROR: 归档需要用户确认` | ✅ |

安全检查工作正常。

#### 6.6 `kanban_knowledge_*`

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| kanban_knowledge_add | 添加 K001 条目 | 同预期 | ✅ |
| kanban_knowledge_list | 列出条目 | 同预期 | ✅ |
| kanban_knowledge_search | 搜索匹配 | 同预期 | ✅ |

#### 6.7 `kanban_version_*`

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| kanban_version_init | 创建 CHANGELOG.md | 同预期 | ✅ |
| kanban_version_record | 记录版本 + git tag | ⚠️ config.json 版本未写入 | ⚠️ |
| kanban_version_list | 列出版本 | 显示但 `current: v`（版本号为空） | ❌ |

**🐛 BUG: `kanban_version_record` config.json 写入失败**

`jq --arg v "$ver_num" '. + {version: $v}' "$cfg"` 命令未能成功更新 config.json。推测原因是 `_update_index` 覆盖导致的 tmp 文件冲突，或 jq 的 mv 操作在并发场景下失败。

#### 6.8 Dashboard

| 项目 | 预期 | 实际 | 状态 |
|------|------|------|------|
| 启动服务 | localhost:3000 | ✅ 启动成功 | ✅ |
| `/api/config` | 返回项目配置 | 同预期 | ✅ |
| `/api/tasks` | 返回任务列表 | 返回空数组 `[]` | ❌ |

**🐛 BUG: Dashboard 不兼容 ST-011 内聚目录结构**

`server.js` 的 `/api/tasks` 端点使用 `files.filter(f => f.endsWith('.json'))` 扫描 `tasks/` 目录，但 ST-011 新布局将 task.json 放在 `tasks/TASK-NNN/task.json` 子目录中。顶层目录不再有 `.json` 文件，所以 dashboard 找不到任何任务。

**修复方案**: 修改 server.js 的 task 读取逻辑：
```javascript
// 旧: 只扫描 .json 文件
const files = fs.readdirSync(tasksDir).filter(f => f.endsWith('.json'));

// 新: 同时扫描子目录中的 task.json
const entries = fs.readdirSync(tasksDir, { withFileTypes: true });
const tasks = entries
  .filter(e => e.isDirectory() && e.name.startsWith('TASK-'))
  .map(e => {
    const taskFile = path.join(tasksDir, e.name, 'task.json');
    if (fs.existsSync(taskFile)) {
      return JSON.parse(fs.readFileSync(taskFile, 'utf-8'));
    }
    return null;
  })
  .filter(Boolean);
```

---

### 🔴 严重问题汇总

#### BUG-1: `_update_index` 被 workflow.sh 覆盖为空操作

**文件**: `lib/workflow.sh` 第 325-329 行

workflow.sh 在文件末尾重新定义了 `_update_index()`：
```bash
_update_index() {
  if type _update_index_core >/dev/null 2>&1; then
    _update_index_core
  fi
}
```

`_update_index_core` 从未在任何地方定义，因此 `_update_index` 变成空操作。当 `kanban_init_env` 加载所有 lib 后，kanban.sh 中正常的 `_update_index` 被 workflow.sh 的空壳版本覆盖。

**影响**:
- 所有 `workflow_transition` 调用后 `index.json` 不更新
- `kanban_status` 看不到新任务
- Dashboard 依赖 index.json 的功能失效
- 在**同一 bash 进程**中连续操作时，后续依赖 `_update_index` 的操作全部失效

**复现**: 在一个 bash 进程中 source 所有 lib 后，调用 `workflow_transition`，然后检查 index.json，发现未更新。

**修复方案**: 删除 workflow.sh 末尾的 `_update_index` 覆盖，或改为直接调用 kanban.sh 中的原始版本：
```bash
# 方案A: 删除 workflow.sh 末尾的 _update_index
# 方案B: 不覆盖，让 kanban.sh 的定义生效
# 方案C: 如果需要委托，使用不同函数名
```

#### BUG-2: 评估模板文件名格式不匹配

**文件**: `lib/evaluator.sh` 第 38 行

代码: `$SKILL_DIR/templates/reports/${role}.json`
实际文件: `templates/reports/code-reviewer.json`（连字符）

**修复**: `local template_name=$(echo "$role" | tr '_' '-');`

#### BUG-3: Dashboard 不兼容内聚目录结构

**文件**: `dashboard/server.js` 第 244-266 行

Dashboard 只扫描 `tasks/*.json`，不支持 `tasks/TASK-NNN/task.json`。

**影响**: 所有使用内聚目录结构的项目，Dashboard 无法显示任务。

---

### 🟡 中等问题汇总

#### ISSUE-4: dispatch_dir() 首次调用返回旧路径

`dispatch_dir()` 基于目录是否存在来决定返回新/旧路径。首次调用时新目录不存在，所以返回旧路径 `.kanban/dispatch/`。后续 `kanban_prepare_dispatch` 创建了新目录但文件写到了旧位置。

**改进**: `dispatch_dir()` 应始终返回新格式路径并自动创建目录。

#### ISSUE-5: `kanban_version_record` config.json 更新不可靠

版本号未能成功写入 config.json，导致 `kanban_version_list` 显示 `current: v`（空版本）。

#### ISSUE-6: `self_improve_evolve_skills` 中 seq 空参数

当 task_file 找不到或 iteration 为空时，`seq 1 ""` 报错。

---

### ✅ 功能正常

| 函数 | 状态 |
|------|------|
| `kanban_create_task` | ✅ 完美 |
| `kanban_update_subtask` | ✅ 完美 |
| `kanban_show_task` | ✅ 完美 |
| `kanban_init_subtasks` | ✅ 完美 |
| `workflow_transition` (plan/execute/evaluate) | ✅ 核心逻辑正确 |
| `guard_check` | ✅ 三层检查完善 |
| `guard_check_artifacts` | ✅ 正确 |
| `guard_check_plan_quality` | ✅ 评分维度合理 |
| `guard_check_evaluation` | ✅ 正确 |
| `worktree_create` | ✅ 幂等，回退机制好 |
| `evaluator_record_score` | ✅ 正确 |
| `evaluator_collect_scores` | ✅ 正确 |
| `evaluator_check_pass` | ✅ 正确 |
| `kanban_iteration_summary` | ✅ 信息完整 |
| `kanban_changes_summary` | ✅ 功能丰富 |
| `kanban_knowledge_add/list/search` | ✅ 正确 |
| `kanban_archive_task` 安全检查 | ✅ 正确拒绝未授权归档 |

---

### 建议提交的 Issues

1. **BUG**: `_update_index` 被 workflow.sh 覆盖导致 index.json 不更新
2. **BUG**: evaluator 模板文件名使用连字符但代码使用下划线
3. **BUG**: Dashboard server.js 不兼容 ST-011 内聚目录结构
4. **改进**: `dispatch_dir()` 应默认使用新格式路径
5. **改进**: `kanban_version_record` config.json 更新逻辑需加固
6. **改进**: `self_improve_evolve_skills` 需空值保护
