**中期测试汇报**

**项目**：FreeCharts 流程图编辑器 — 测试中期汇报

**日期**：2025-12-12

**作者**：测试小组

**一、执行摘要**

- 目标：对项目的单元测试、功能测试、性能测试与易用性测试的已完成工作、缺口与下一步计划进行汇总。
- 结论：项目已有大量测试材料（Qt Test 源、功能测试截图、性能脚本、易用性问卷），但缺少可复现的运行日志、JUnit/XML 报表、非 GUI 的轻量单元测试和 CI 集成；建议在短期内补齐这些可验证证据以满足课程要求。

**二、已完成证据（路径）**

- 单元测试（Qt Test 源与结果截图）：
  - 源文件：`aim/diagramscene_ultima/diagramscene_test/tests/` 下的 `tst_*.cpp`
  - 截图：`aim/diagramscene_ultima/diagramscene_test/result/*.png`
  - 测试说明：`aim/diagramscene_ultima/diagramscene_test/test_report.md`
- 功能测试（手动用例与截图）：
  - 用例文档：`funtional_test/TEST_REPORT_FUNCTIONAL.md`
  - 执行表：`funtional_test/FUNCTIONAL_TEST_EXECUTION.md`
  - 截图：`funtional_test/TEST_REPORT_PICTURES/TC_001_*` … `TC_020_*`
- 压力/性能测试：
  - 脚本：`stress_performance_test/run_*.py`
  - 报告与图像：`stress_performance_test/PERFORMANCE_TEST_RESULTS.md`、`stress_performance_test/result*.png` 等
- 易用性测试：
  - 原始问卷：`usability_test/USABILITY_SURVEY_RESPONSES.md`
  - 汇总报告：`usability_test/USABILITY_TEST_REPORT_UPDATED.md`
  - 截图/照片：`usability_test/image1.png` …
- 项目文档与状态追踪：
  - 任务分配：`project_docs/TEAM_TASK_ASSIGNMENT.md`
  - 静态分析：`project_docs/STATIC_TEST_REPORT.md`, `project_docs/PROJECT_ANALYSIS.md`

**三、缺失与风险**

- 缺少可复现的运行日志：多数测试仅有截图或报告，缺少 stdout/stderr 日志文件（如 `*.log`）和 QTest 导出的 JUnit XML（`*.xml`）。
- 缺少 Tier1（非 GUI）轻量单元测试用于 CI 验证（建议使用 Catch2/GoogleTest 或小型 main+assert）。
- 缺少内存/泄露检测报告（Dr. Memory / ASAN）与性能量化数据（CSV/JSON）。
- CI 集成缺失：当前无 `.github/workflows` 或其他 CI 配置，导致测试无法自动化运行与持续验证。
- 构建环境风险：Qt 版本、编译器（MinGW/MSVC）差异会导致本地可运行性不一致。

**四、短期建议（优先级 P0/P1）**

1. P0 — 生成并提交运行日志与 XML：对 `aim/diagramscene_ultima/diagramscene_test` 下的每个 `tst_*.cpp` 运行一次并保存 `tst_*.log`（stdout/stderr），如可行使用 QTest 的 `-o junitxml:...` 导出 XML，路径放 `aim/diagramscene_ultima/diagramscene_test/result/`。
2. P0 — 添加 Tier1 轻量测试：在 `aim/diagramscene_ultima/tests/tier1/` 添加 4 个不依赖 GUI 的单元测试样板，并把编译命令与运行脚本写入 `tests/COMPILATION_NOTES.md`。
3. P1 — 为功能测试优先 5 个用例编写可复现的自动化脚本（PowerShell/AutoIt/Sikuli），把截图与运行日志关联至 `funtional_test/`。
4. P1 — 改造压力测试脚本使其产出机器可读指标（CSV），并保存原始运行日志到 `stress_performance_test/logs/`。
5. P1 — 在本地对关键测试运行内存检测（Dr. Memory），把报告放 `tests/logs/drmemory_*`。
6. P1 — 创建简单 CI（GitHub Actions）只运行 Tier1 测试，后续扩展到运行 QTest（如有可用的 Qt runner）。

**五、责任分配（建议）**

- 组员 A：创建 `tests/tier1/` 并实现 4 个轻量单元测试（T1a）。
- 组员 B：修复 Qt 编译/集成、运行 `tst_*.cpp` 并收集 `*.log` 与 `junitxml`（T1b + T2）。
- 组员 C：执行功能用例与易用性测试的脚本化与截图整理（T3 + T4）。
- 可选：一人负责在 Windows 上运行 Dr. Memory 并收集报告（可交给组员 B）。

**六、里程碑（示例）**

- M1（+1 天）：`tests/tier1/` 提交并在本地编译通过；CI（basic）通过。
- M2（+2 天）：`tst_*.log` 與 `junitxml` 提交；`UNIT_TEST_RESULTS.md` 完成。
- M3（+3 天）：功能测试 5 个用例脚本化并提交日志/截图；压力测试 CSV 输出。

**七、附录 — 关键运行命令示例**

```powershell
# 在 diagramscene_test 目录下运行 QTest 并导出 junit xml（示例）
cd 'd:\work\homework\programming\codes\SoftwareQA&testing\aim\diagramscene_ultima\diagramscene_test'
.\tst_diagramscene.exe -o junitxml:result\tst_diagramscene.xml 2>&1 | Tee-Object result\tst_diagramscene.log

# 编译并运行 Tier1 示例（不依赖 Qt）
cd 'd:\work\homework\programming\codes\SoftwareQA&testing\aim\diagramscene_ultima\tests\tier1'
$gpp = 'D:\msys2\mingw64\bin\g++.exe'
& $gpp -O2 -std=gnu++17 -I.. ..\arrow.cpp test_arrow_light.cpp -o test_arrow_light.exe 2>&1 | Tee-Object ..\logs\compile_tier1_arrow.txt
.\test_arrow_light.exe 2>&1 | Tee-Object ..\logs\test_arrow_light_run.log
```

---

如需，我可以把本报告另存为 `project_docs/MIDTERM_REPORT.md`（或你指定的路径），并/或马上在仓库创建 `aim/diagramscene_ultima/tests/tier1/` 与 4 个样板测试文件并提交。请确认是否需要我立刻执行这两个动作中的任意一项。
