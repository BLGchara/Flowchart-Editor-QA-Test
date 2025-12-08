# 团队任务分配（QA&测试课程项目）

## 项目背景

FreeCharts 流程图编辑器的完整软件测试实践。目标：从设计规划阶段已有的分析基础出发，补齐**实际运行/执行的测试证据**，满足课程"功能度、易用性、文档、静态、动态"五个测试维度的要求。

---

## ⚡ 关键说明：任务可以并行执行

**之前我标注 P0（优先级 0）的意思是"重要"，但不代表 T3/T4 必须等 T1/T2 完成。**

实际情况：

- **T1（编译）**→ **T2（运行日志）** 有依赖关系（必须线性）
- **T3（功能测试）** 和 **T4（易用性调查）** 可以**立即开始**，无需等 T1/T2
- **T5（性能测试）** 也可并行执行

**建议执行策略**：

```
Day 1:
  ├─ 组员 A 开始 T1（修复编译） [预计 4h]
  ├─ 组员 B 同时开始 T3（功能测试） [预计 6h，可分成多个 1h 时段]
  └─ 组员 C 同时开始 T4（邀请用户反馈） [预计 4h，主要是等待反馈回复]

Day 2:
  ├─ 组员 A 完成 T1 后，立即开始 T2（运行测试） [预计 2h]
  ├─ 组员 B/C 继续完成各自任务
  └─ 汇总所有结果，生成最终报告
```

总墙钟时间：1-2 天（而非 4+6+4=14 天的串联）。

---

## 任务总表

| 任务 ID | 任务名称                          | 优先级 | 工作量 | 负责人            | 依赖关系                        | 输出物                             |
| ------- | --------------------------------- | ------ | ------ | ----------------- | ------------------------------- | ---------------------------------- |
| T1a     | 创建测试目录并实现 Tier1 轻量测试 | P0     | 4h     | **组员 A**        | 无                              | `tests/` 源码 + `compile_log.txt`  |
| T1b     | 修复 Qt 编译/集成 & Tier2/Tier3   | P0     | 4h     | **组员 B**        | 可与 T1a 部分并行 / 依赖部分输出 | 可执行文件 + `COMPILATION_NOTES.md` |
| T2      | 运行单元测试并收集日志            | P0     | 2h     | **组员 B**        | 依赖 T1a / T1b                  | `test_run.txt` + 分析               |
| T3      | 手动执行功能测试（20 用例）       | P1     | 6h     | **组员 C**        | 无（可与 T1a/T1b 并行）         | 截屏 5+ 张 + `FUNCTIONAL_TEST_EXECUTION.md` |
| T4      | 易用性测试调查                    | P1     | 4h     | **组员 C**        | 无（可与 T1a/T1b 并行）         | `USABILITY_SURVEY_RESPONSES.md` + `USABILITY_TEST_REPORT_UPDATED.md` |
| T5      | 压力/性能测试                     | P2     | 3h     | 可选              | 无（可与 T1-T4 并行）           | 性能指标 + 日志                    |

---

## 详细任务说明

### Task T1：组员 A — 编写并运行单元测试（调整后）

**负责人**：组员 A
**优先级**：P0
**预计工作量**：4 小时

**重要说明（已更新）**：当前仓库中并**不存在** `aim/diagramscene_ultima/tests` 目录。为保证任务可重复、可验证，T1 的范围改为：

- 在仓库中创建 `aim/diagramscene_ultima/tests` 目录（如果尚不存在）；
- 由组员 A 负责设计并实现初始的单元测试套件，优先实现“与 Qt 运行时耦合最少”的轻量化测试（便于在没有完整 Qt 环境时也能编译和验证），随后再补充或迁移到基于 Qt 的集成测试（若环境可用）；
- 记录并提交所有编译命令、错误日志和最终可执行产物。

**背景（变更原因）**

- 早期文档和脚本假定 `tests` 目录存在并且能直接编译 Qt 测试，但在当前工作区该目录不存在且用户环境中 Qt/编译链存在不确定性。为避免重复失败，T1 首先保证测试目录与最小可编译测试存在，再逐步扩展到完整 Qt 测试。

**具体步骤（建议）**

1. 在 `aim/diagramscene_ultima` 下创建 `tests/` 目录，并把测试源码放入（例如 `tests/test_diagramscene_light.cpp`、`tests/test_commands.cpp` 等）。
2. 先实现轻量测试（不依赖 QApplication 或 Qt Widgets）：用断言或小型测试驱动（例如简单的 main + asserts）来测试纯逻辑/数据方法（如 getter/setter、group 管理、命令的状态变更等）。
3. 如果必须调用 Qt 类型（例如 QGraphicsItem 行为），将测试拆分为：
   - a) 纯逻辑测试（优先实现）
   - b) Qt 依赖测试（在有正确 Qt/MinGW ABI 时再编译运行，放到 `tests/qt/` 子目录）
4. 在 PowerShell 中运行并收集编译输出（把 stderr/stdout 保存为 `tests/compile_log.txt`）；当遇到链接或 moc 问题时，按错误提示新增相应的源/moc 生成步骤或调整编译选项。
5. 提交以下输出物到仓库：`tests/` 源码、`tests/compile_log.txt`、最终可执行文件（或可复现的编译命令记录 `COMPILATION_NOTES.md`）。

**优先级与完成标志**

- [ ] 在仓库中创建 `aim/diagramscene_ultima/tests/` 并提交初始测试源文件
- [ ] 至少 6 个轻量单元测试（见下方建议清单）可以在无 Qt GUI环境下编译并通过
- [ ] 把所有编译/运行日志与最终命令记录在 `tests/COMPILATION_NOTES.md`

**输出物**

- `aim/diagramscene_ultima/tests/`（包含测试源）
- `tests/COMPILATION_NOTES.md`（最终编译命令与说明）
- `tests/compile_log.txt`（原始编译输出）
- `UNIT_TEST_RESULTS.md`（测试运行结果汇总）

**补充说明（迁移到 Qt 测试）**

如果环境允许（组员 A 确认能访问与项目 ABI 匹配的 Qt 编译链），再新增 `tests/qt/` 下的 QtTest 或需要 `QApplication` 的集成测试。此时可参考已有 `build_and_run.ps1` 并记录最终成功命令。

---

## 建议的单元测试清单（供组员 A 实施）

下面的清单基于对 `aim/diagramscene_ultima` 源码的扫描（涉及的类包括 `DiagramScene`、`DiagramItem`、`DiagramItemGroup`、`DiagramTextItem`、`Arrow`、`DiagramPath`、`DeleteCommand`、`SnapshotCommand`、`MyGraphicsView`、`MainWindow` 等）。为保证可复现性，我们把测试分为三个层次：

- Tier 1（轻量、尽量不依赖 Qt GUI）：优先实现，便于在任意机器上快速编译验证；
- Tier 2（依赖 QtCore/QUndoStack / 非 GUI Qt 类型）：在可用 Qt 开发环境下运行；
- Tier 3（需要 QApplication / QWidget / QGraphicsView）：GUI 集成测试，环境要求最高，晚些再做。

建议的测试用例（按优先级排序）：

- `tests/test_diagramitem_light.cpp`（Tier 1）: 测试 `DiagramItem` 的纯逻辑方法（若代码包含几何/字符串/属性处理函数），包括构造、getter/setter、边界值。
- `tests/test_diagramitemgroup.cpp`（Tier 1）: 测试 `DiagramItemGroup` 的 add/remove 行为、子项计数、移动是否影响成员图形的本地坐标（如实现中有相关逻辑）。
- `tests/test_commands.cpp`（Tier 1/2）: 测试 `DeleteCommand`、`SnapshotCommand` 的 undo/redo 语义（使用 `QUndoStack`，可在 Tier 2 中加入 QtCore 依赖）。
- `tests/test_arrow.cpp`（Tier 1）: 测试 `Arrow` 的端点设置、更新线段长度/角度的数学正确性（通常只需构造并调用方法，不必显示）。
- `tests/test_diagrampath.cpp`（Tier 1）: 测试 `DiagramPath`（路径创建/序列化/路径点操作等）。
- `tests/test_diagramtextitem.cpp`（Tier 1）: 测试 `DiagramTextItem` 的文本设置/获取、交互接口（如果有纯逻辑层）。
- `tests/test_diagramscene_api.cpp`（Tier 2）: 当 QtCore 可用时，测试 `DiagramScene` 的 API：添加/删除项、创建文本项、设置/获取颜色、scene 的 itemAt/selectedItems 行为；在不能使用 QApplication 时仅测试不依赖视图的部分。
- `tests/test_mygraphicsview_light.cpp`（Tier 2/3）: 测试 `MyGraphicsView` 的坐标变换、缩放/滚动处理（若实现中包含纯逻辑可以先做 Tier 2）。
- `tests/test_mainwindow_smoke.cpp`（Tier 3）: 启动主窗口的冒烟测试（需要 Qt Widgets 和 `QApplication`），检查关键控件存在并能响应最小事件序列。

每个测试文件应包含：

- 清晰的目标说明（在文件注释中写明本测试覆盖的函数/类）；
- 可重复的构建指令或 CMake 目标（把简单的 g++ 命令写入 `tests/COMPILATION_NOTES.md`）；
- 运行脚本示例（PowerShell），把运行 stdout/stderr 收集到 `tests/<testname>_run.log`。示例：

```powershell
cd 'D:\work\homework\programming\codes\SoftwareQA&testing\aim\diagramscene_ultima\tests'
& 'D:\msys2\mingw64\bin\g++.exe' -O2 -std=gnu++17 `
   -I.. ..\diagramscene.cpp test_diagramitem_light.cpp -o test_diagramitem_light.exe 2>&1 | Tee-Object compile_log.txt
.\test_diagramitem_light.exe 2>&1 | Tee-Object test_diagramitem_light_run.log
```

（以上命令为示例；组员 A 请根据本地 Qt/编译链调整包含路径与链接选项。）

实现顺序建议：

1. 先实现并提交 Tier 1 的 4-6 个轻量测试（可在无 Qt GUI 环境下快速验证）；
2. 验证通过后，添加 Tier 2 的命令/QUndoStack 测试与少量 QtCore 依赖测试；
3. 最后在有 Qt Widgets 的开发环境下补充 Tier 3 的 GUI 集成测试并记录运行截图/日志。

---

把上述测试清单和初始测试文件提交到 `aim/diagramscene_ultima/tests/`，并在 `UNIT_TEST_RESULTS.md` 中把每次运行的状态和日志路径记录清楚，方便复核与 grading。

---

### Task T2：运行单元测试并收集日志

**负责人**：组员 B（T1a/T1b 完成后接手）
**优先级**：P0
**预计工作量**：2 小时

**背景**

- T1 完成后，执行编译生成的测试程序
- 收集运行日志并分析测试结果

**具体步骤**

1. 在 PowerShell 中运行：
   ```powershell
   cd 'D:\work\homework\programming\codes\SoftwareQA&testing\aim\diagramscene_ultima\tests'
   .\test_diagramscene_real.exe 2>&1 | Tee-Object -FilePath '..\\test_run_log.txt'
   ```
2. 检查 exit code：
   - `0` = 所有断言通过
   - 非 0 = 某些测试失败，查看输出定位问题
3. 解析输出，统计"PASS / FAIL"数量
4. 如有失败，记录失败的具体测试点和错误信息
5. 把日志保存为 `TEST_DIAGRAMSCENE_REAL_RUN.log`，并在 `UNIT_TEST_RESULTS.md` 中汇总

**完成标志**

- [ ] 测试程序成功运行（无崩溃）
- [ ] 运行日志已收集（exit code 已记录）
- [ ] 测试结果已分析（通过率、失败点）

**输出物**

- `test_run_log.txt`（原始运行输出）
- `UNIT_TEST_RESULTS.md`（分析报告，包含通过/失败统计和说明）

---

### Task T3：手动执行功能测试（20 用例）

**负责人**：组员 C
**优先级**：P1
**预计工作量**：6 小时（约 ~18 分钟/用例，部分并行）

**背景**

- 用例清单已准备好（见 `TEST_REPORT_FUNCTIONAL.md` 的 20 个功能点）
- 需要在实际 GUI 上逐一执行，并截屏/记录结果
- 目标：补齐"实际 GUI 交互"的证据

**具体步骤**

1. 启动 FreeCharts 应用程序（需要能看到 GUI；若看不到，先联系组员 B 排查 Qt 运行环境）
2. 对照 `TEST_REPORT_FUNCTIONAL.md` 的用例表，逐一执行：
   - **示例用例 1**："启动与主界面" → 运行程序 → 截屏主窗口 → 记录 ✓
   - **示例用例 2**："新建流程图" → 点击 File > New → 截屏空白场景 → 记录 ✓
   - **示例用例 3**："插入图形" → 选择矩形工具 → 点击场景 → 截屏 → 记录 ✓
   - ...以此类推 20 个用例
3. 对每个用例：
   - 记录是否"通过"、"失败"或"部分通过"
   - 如有异常（崩溃、错误对话框），截屏并记录
   - 把关键截屏保存为 `TC_00X_<用例名>_<时间戳>.png`
4. 把所有结果汇总到 `FUNCTIONAL_TEST_EXECUTION.md`（表格形式）

**用例清单（摘自 TEST_REPORT_FUNCTIONAL.md）**

| #   | 功能              | 测试操作       | 预期     | 实际  |
| --- | ----------------- | -------------- | -------- | ----- |
| 1   | 启动与主界面      | 运行程序       | 窗口显示 | ✓/✗ |
| 2   | 新建流程图        | File → New    | 空白场景 | ✓/✗ |
| 3   | 插入图形（20 种） | 选工具 → 点击 | 图形出现 | ✓/✗ |
| 4   | 图形移动          | 拖拽图形       | 位置变化 | ✓/✗ |
| ... | ...               | ...            | ...      | ...   |

**完成标志**

- [ ] 20 个用例均已测试
- [ ] 关键截屏至少 5 张（展示不同功能）
- [ ] 所有结果记录在 `FUNCTIONAL_TEST_EXECUTION.md`
- [ ] 若有失败，已描述失败原因

**输出物**

- `TC_001_<用例>.png`～`TC_020_<用例>.png`（关键截屏，至少 5 张）
- `FUNCTIONAL_TEST_EXECUTION.md`（用例执行表 + 分析）

---

### Task T4：易用性测试调查

**负责人**：组员 C
**优先级**：P1
**预计工作量**：4 小时

**背景**

- 易用性测试需要收集用户反馈，而非仅单人操作
- 目标：邀请 5+ 人（同学、朋友、师兄师姐等）实际使用程序并记录反馈

**具体步骤**

1. 准备用户调查问卷（问卷模板已生成，见下方）
2. 邀请 5+ 测试员，给他们一个简单的任务：
   - "请用这个程序画一个简单的流程图（3 个图形 + 2 条连线）"
   - "请尽量在不看文档的情况下完成"
3. 在他们操作时观察：
   - 是否能快速找到工具？
   - 是否对某些功能感到困惑？
   - 是否遇到意外行为？
   - UI 是否美观/专业？
4. 操作后，请他们填写问卷（5 分制或开放式反馈）
5. 统计结果并汇总为 `USABILITY_TEST_REPORT_UPDATED.md`

**用户问卷模板**

```
=== 易用性反馈问卷 ===
测试员编号: ___
使用时间: ___ 分钟
是否第一次使用: 是 / 否

1. 工具栏布局是否清晰？(1-5) ___
   反馈：___

2. 操作是否直观？(1-5) ___
   反馈：___

3. 图形绘制是否简便？(1-5) ___
   反馈：___

4. 颜色/样式选择是否容易找到？(1-5) ___
   反馈：___

5. 保存/加载功能是否清楚？(1-5) ___
   反馈：___

6. 总体满意度（1-5）: ___

7. 最喜欢的功能：___
8. 最需要改进的地方：___
9. 其他建议：___
```

**完成标志**

- [ ] 至少 5 个用户反馈已收集
- [ ] 问卷已填写并保存
- [ ] 数据已汇总（平均分、常见反馈）

**输出物**

- `USABILITY_SURVEY_RESPONSES.md`（原始反馈记录）
- `USABILITY_TEST_REPORT_UPDATED.md`（汇总分析）

---

### Task T5：压力/性能测试（可选，时间允许）

**负责人**：可选
**优先级**：P2
**预计工作量**：3 小时

**背景**

- 验证程序在大量数据下的性能表现
- 可手动测试或编写自动化脚本

**具体步骤**

1. 编写或录制一个脚本，批量插入 1000+ 个图形
2. 记录：
   - 插入时间
   - 内存占用
   - 是否卡顿/崩溃
3. 测试保存/加载大文件的性能

**输出物**

- `PERFORMANCE_TEST_RESULTS.md`（性能指标和分析）

---

## 提交与汇总

### 最终交付物清单

- [ ] T1: `test_diagramscene_real.exe` + `COMPILATION_NOTES.md`
- [ ] T2: `test_run_log.txt` + `UNIT_TEST_RESULTS.md`
- [ ] T3: 截屏 5+ 张 + `FUNCTIONAL_TEST_EXECUTION.md`
- [ ] T4: `USABILITY_SURVEY_RESPONSES.md` + `USABILITY_TEST_REPORT_UPDATED.md`
- [ ] T5: `PERFORMANCE_TEST_RESULTS.md`（可选）

### 提交流程

1. 每个任务完成后，在对应 `.md` 文件中标注"完成日期"和"负责人"
2. 把所有输出物推送到 GitHub `main` 分支
3. 更新 `HONEST_TEST_STATUS.md` 中的状态表（从 🔴 改为 🟢）
4. 主要负责人（或 leader）审阅并在 `FINAL_TEST_SUMMARY.md` 中生成最终报告

---

## 时间表与 Milestone

### 并行执行策略

- **第 1 天**：T1, T3, T4 同时开始（互不阻塞）
- **第 1 天晚 / 第 2 天**：T2（T1 完成后开始）
- **第 2 天**：汇总 T1-T4 结果，可选执行 T5

### 关键节点

| 里程碑       | 目标完成日期 | 涉及任务 | 说明                                    |
| ------------ | ------------ | -------- | --------------------------------------- |
| M1: T1 完成  | [日期]       | T1       | 单元测试编译成功；解除 T2 阻塞          |
| M2: T2 完成  | [日期]       | T2       | 单元测试运行结果反馈                    |
| M3: T3 完成  | [日期]       | T3       | 功能测试证据完整（可与 M1-M2 并行完成） |
| M4: T4 完成  | [日期]       | T4       | 用户反馈收集完成（可与 M1-M2 并行完成） |
| M5: 最终报告 | [日期]       | 所有     | 综合所有证据生成最终测试报告            |

---

## 联系与支持

- **技术问题**（编译、运行）：组员 A
- **功能测试**问题：组员 B
- **用户调查**问题：组员 C
- **总体协调**：项目 leader

---

## 附录：快速参考

### 启动应用

```powershell
cd 'D:\work\homework\programming\codes\SoftwareQA&testing\aim\diagramscene_ultima\build\Desktop_Qt_6_10_1_MinGW_64_bit-Debug\debug'
.\FreeCharts.exe
```

### 编译测试

```powershell
cd 'D:\work\homework\programming\codes\SoftwareQA&testing\aim\diagramscene_ultima\tests'
.\build_and_run.ps1
```

### 文档位置

- 项目分析：`PROJECT_ANALYSIS.md`
- 功能用例清单：`TEST_REPORT_FUNCTIONAL.md`（表中的 20 个用例）
- 代码位置：`aim/diagramscene_ultima/`
