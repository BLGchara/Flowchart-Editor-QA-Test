# FreeCharts 项目结构详解

## 📂 目录结构

```
diagramscene_ultima/
│
├── 主要源文件
│   ├── main.cpp                    # 入口点，创建 QApplication 和 MainWindow
│   ├── mainwindow.cpp/h            # 主窗口（1,500 行 - 最复杂）
│   ├── diagramscene.cpp/h          # 图形场景管理（490 行）
│   ├── diagramitem.cpp/h           # 图形元素基类（550 行）
│   ├── mygraphicsview.cpp/h        # 自定义 QGraphicsView
│   └── diagramtextitem.cpp/h       # 文本编辑项
│
├── 连接和变换
│   ├── arrow.cpp/h                 # 直线箭头（60 行）
│   ├── diagrampath.cpp/h           # 曲线连接（90 行）
│   └── diagramitemgroup.cpp/h      # 项组合（180 行）
│
├── 编辑功能
│   ├── deletecommand.cpp/h         # 删除命令（撤销/重做）
│   ├── snapshotcommand.cpp/h       # 快照命令
│   └── findreplacedialog.cpp/h     # 查找替换对话框
│
├── UI 和资源
│   ├── mygraphicsview.ui           # Qt Designer 文件
│   ├── diagramscene.qrc            # 资源配置
│   └── newpic.qrc                  # 新图片资源
│
├── 项目配置
│   ├── diagramscene.pro            # qmake 配置
│   ├── CMakeLists.txt              # CMake 配置
│   └── 其他 .pro.user 文件         # Qt Creator 设置
│
├── 构建输出
│   └── build/                      # 编译输出目录
│       ├── Desktop_Qt_6_10_1_MinGW_64_bit-Debug/
│       ├── Desktop_Qt_6_7_2_MinGW_64_bit-Debug/
│       └── Desktop_Qt_6_7_2_MinGW_64_bit-release/
│
├── 图片资源
│   └── images/
│       ├── NodesIcon/              # 图形图标
│       └── 其他 UI 图片
│
└── 单元测试（新增）
    └── tests/
        ├── test_diagramscene_simple.cpp   # 19 个单元测试
        ├── test_diagramscene_simple.exe   # 可执行文件
        ├── TEST_SUMMARY.md                 # 测试报告
        ├── TEST_COVERAGE.md                # 覆盖率文档
        └── QUICKREF.md                     # 快速参考
```

---

## 🔄 类关系和继承链

### 继承树

```
QObject
├── QApplication (Qt)
├── QMainWindow
│   └── MainWindow                          【1,500行】
│       └── 管理 DiagramScene 和 UI 控件
│
└── QGraphicsScene
    └── DiagramScene                        【490行】
        ├── 创建 DiagramItem 实例
        ├── 创建 DiagramTextItem 实例
        ├── 管理 Arrow 连接
        ├── 管理 DiagramPath 连接
        └── 处理鼠标/键盘事件

QGraphicsItem (基类)
├── QGraphicsPixmapItem
│   └── DiagramItem                         【550行】
│       ├── 支持 20+ 种图形类型
│       ├── 有 Arrow 列表
│       ├── 有 DiagramPath 列表
│       └── 有文本项
│
├── QGraphicsTextItem
│   └── DiagramTextItem                     【30行】
│
├── QGraphicsLineItem
│   └── Arrow                               【60行】
│
└── QGraphicsPathItem
    └── DiagramPath                         【90行】

QGraphicsView
└── MyGraphicsView                          【自定义视图】

QDialog
└── FindReplaceDialog                       【查找替换对话框】

QUndoCommand (撤销/重做)
├── DeleteCommand                           【删除命令】
└── SnapshotCommand                         【快照命令】

QFileSystemWatcher 等
└── MainWindow 使用（文件监控）
```

---

## 📊 核心流程图

### 应用启动流程

```
main()
  │
  ├─ new QApplication(argc, argv)
  │
  ├─ new MainWindow()
  │   ├─ 初始化 UI 控件
  │   ├─ 创建 DiagramScene
  │   ├─ 设置 QGraphicsView
  │   ├─ 连接所有信号/槽
  │   └─ 初始化 QUndoStack
  │
  ├─ mainWindow.showMaximized()
  │
  └─ return app.exec()
       └─ 进入事件循环
```

### 绘制图形流程

```
用户选择图形类型（如 "Step"）
  │
  └─ UI 按钮信号 → MainWindow::buttonGroupClicked()
      │
      └─ diagramScene->setItemType(DiagramItem::Step)
          │
          └─ 用户在场景中点击
              │
              └─ DiagramScene::mousePressEvent()
                  │
                  ├─ 判断 myMode == InsertItem
                  │
                  ├─ item = new DiagramItem(Step, contextMenu)
                  │   │
                  │   ├─ 绘制矩形图形
                  │   ├─ 添加文本项 ("请输入")
                  │   ├─ 设置变换点和连接点
                  │   └─ 设置事件处理
                  │
                  ├─ addItem(item)
                  │
                  └─ emit itemInserted(item)
                      │
                      └─ MainWindow::itemInserted()
                          └─ 更新 UI（属性编辑等）
```

### 编辑文本流程

```
用户双击图形
  │
  └─ DiagramItem::mouseDoubleClickEvent() (继承自 QGraphicsItem)
      │
      ├─ 创建 DiagramTextItem（或激活现有的）
      │
      ├─ textItem->setTextInteractionFlags(Qt::TextEditorInteraction)
      │
      └─ textItem->setFocus()  # 进入编辑模式
          │
          ├─ 用户输入文本
          │
          └─ 用户按 Esc 或点击其他地方
              │
              └─ DiagramTextItem::focusOutEvent()
                  │
                  └─ emit lostFocus(this)
                      │
                      └─ DiagramScene::editorLostFocus()
                          │
                          ├─ 如果文本为空：removeItem() 并 deleteLater()
                          └─ 如果文本非空：保留项，更新显示
```

### 撤销/重做流程

```
用户删除图形
  │
  ├─ UI "Delete" 按钮 → MainWindow::deleteItem()
  │
  └─ 遍历 selectedItems()
      │
      ├─ new DeleteCommand(item, scene)
      │
      ├─ undoStack->push(command)  # 压入栈
      │   │
      │   └─ DeleteCommand::redo()
      │       └─ scene->removeItem(item)
      │
      └─ 启用 "Undo" 按钮
          │
          ├─ 用户点击 "Undo"
          │
          └─ undoStack->undo()
              │
              └─ DeleteCommand::undo()
                  └─ scene->addItem(item)  # 恢复
```

### 文件保存/加载流程

```
用户按 Ctrl+S
  │
  ├─ MainWindow::savefile()
  │
  ├─ getStructList() 遍历所有 DiagramItem
  │   ├─ 提取每个项的属性：类型、位置、大小、颜色、角度、文本
  │   └─ 存入 WriteDiagramItem 结构
  │
  ├─ getStructList1() 遍历所有连接
  │   └─ 存入 WriteDiagramPath 结构
  │
  ├─ 序列化为文本格式 (.fcproj)
  │
  └─ 保存到文件
      └─ [scene_name].fcproj

用户按 Ctrl+O
  │
  ├─ MainWindow::loadfile()
  │
  ├─ 打开 .fcproj 文件
  │
  ├─ 解析文本格式
  │
  ├─ 遍历每个 WriteDiagramItem
  │   └─ new DiagramItem() 恢复项
  │
  ├─ 遍历每个 WriteDiagramPath
  │   └─ new DiagramPath() 恢复连接
  │
  └─ 绘制到场景
```

---

## 🎯 关键方法调用关系

### 颜色设置流程

```
MainWindow
  └─ 颜色按钮被点击
      │
      ├─ MainWindow::fillButtonTriggered()  【填充颜色】
      │   └─ diagramScene->setItemColor(color)
      │       └─ 遍历 selectedItems()
      │           └─ DiagramItem::setBrush(color)
      │               └─ painter->setBrush(color) 在 paint() 中
      │
      ├─ MainWindow::lineButtonTriggered()  【线条颜色】
      │   └─ diagramScene->setLineColor(color)
      │       ├─ 如果选中 Arrow：Arrow::setColor(color)
      │       └─ 更新显示
      │
      └─ MainWindow::handleFontChange()     【字体颜色】
          └─ diagramScene->setTextColor(color)
              ├─ 如果选中 DiagramTextItem：setDefaultTextColor(color)
              └─ 更新显示
```

### 连接创建流程

```
MainWindow
  └─ 设置 myMode = InsertPath
      │
      └─ 用户点击起点图形
          │
          └─ DiagramScene::mousePressEvent()
              ├─ item = selectedItems().first()
              └─ startItem = (DiagramItem*)item
                  │
                  └─ 用户移动到终点，再点击
                      │
                      └─ DiagramScene::mousePressEvent() again
                          ├─ endItem = selectedItems().first()
                          │
                          ├─ new DiagramPath(startItem, endItem, startState, endState)
                          │   └─ path->drawZig() 绘制曲线
                          │
                          ├─ addItem(path)
                          │
                          ├─ startItem->addPathes(path)
                          ├─ endItem->addPathes(path)
                          │
                          └─ emit pathInserted(path)
```

### 旋转操作流程

```
用户选中图形
  │
  ├─ 按 R 键（右旋）或 L 键（左旋）
  │
  └─ DiagramScene::keyPressEvent()
      │
      ├─ 获取选中项：selectedItems().first()
      │
      ├─ dynamic_cast<DiagramItem*>(item)
      │
      └─ case Qt::Key_R:
          ├─ diagramItem->setRotationAngle(angle + 5)
          │   │
          │   ├─ m_rotationAngle = newAngle
          │   │
          │   ├─ prepareGeometryChange()  # 通知变化
          │   │
          │   └─ update()  # 触发重绘
          │       │
          │       └─ paint()
          │           └─ painter->rotate(m_rotationAngle)
          │
          └─ updatePathes()  # 更新所有连接位置
```

---

## 🔗 信号/槽关系 (核心)

### MainWindow 连接的关键信号

```cpp
// 场景事件
connect(diagramScene, &DiagramScene::itemInserted,
        this, &MainWindow::itemInserted);
connect(diagramScene, &DiagramScene::textInserted,
        this, &MainWindow::textInserted);
connect(diagramScene, &DiagramScene::itemSelected,
        this, &MainWindow::itemSelected);
connect(diagramScene, &DiagramScene::pathInserted,
        this, &MainWindow::pathInserted);

// 项目事件
connect(item, &DiagramItem::selected,      // 需要在 DiagramItem 中发出
        this, &MainWindow::itemSelected);

// UI 按钮
connect(buttonGroup, QOverload<QAbstractButton*>::of(&QButtonGroup::buttonClicked),
        this, &MainWindow::buttonGroupClicked);
connect(deleteButton, &QPushButton::clicked,
        this, &MainWindow::deleteItem);

// 撤销/重做
connect(undoStack, &QUndoStack::canUndoChanged,
        undoAction, &QAction::setEnabled);
connect(undoStack, &QUndoStack::canRedoChanged,
        redoAction, &QAction::setEnabled);

// 查找替换
connect(findReplaceDialog, &FindReplaceDialog::findText,
        this, &MainWindow::handleFindText);
```

---

## 📈 复杂度分析

### 圈复杂度 (CC) 估计

| 文件 | 函数 | CC | 难度 |
|------|------|----|----|
| mainwindow.cpp | MainWindow() | 15 | 🔴 高 |
| mainwindow.cpp | 各槽函数 | 3-8 | 🟡 中 |
| diagramitem.cpp | paint() | 20+ | 🔴 高 |
| diagramitem.cpp | mouseMoveEvent() | 8 | 🟡 中 |
| diagramscene.cpp | mousePressEvent() | 10 | 🟡 中 |
| diagrampath.cpp | drawZig() | 12 | 🟡 高 |

### 耦合度分析

```
高耦合 (强依赖):
  MainWindow ←→ DiagramScene  (通过信号/槽)
  DiagramScene ←→ DiagramItem  (创建和管理)
  DiagramItem ←→ DiagramPath   (连接管理)
  DiagramItem ←→ Arrow         (连接管理)

中耦合:
  MainWindow ← DeleteCommand  (命令模式)
  MainWindow ← FindReplaceDialog  (对话框)

低耦合:
  MyGraphicsView → DiagramScene  (视图)
  DiagramTextItem → DiagramItem  (内嵌)
```

---

## 🧪 单元测试覆盖

### 测试与源代码对应关系

```
tests/test_diagramscene_simple.cpp
│
├─ test_constructor_defaults()
│   └─ DiagramScene::DiagramScene() 构造函数
│
├─ test_createItem_and_type()
│   └─ DiagramScene::createItem() 方法
│
├─ test_createTextItem()
│   └─ DiagramScene::createTextItem() 方法
│
├─ test_setItemColor_default()
│   └─ DiagramScene::setItemColor() 方法
│
├─ test_setTextColor()
│   └─ DiagramScene::setTextColor() 方法
│
├─ test_setLineColor()
│   └─ DiagramScene::setLineColor() 方法
│
├─ test_editorLostFocus_*()
│   └─ DiagramScene::editorLostFocus() 方法
│
├─ test_rotationAngle_*()
│   └─ DiagramItem::setRotationAngle() 方法
│
├─ test_setLinkVisible_*()
│   └─ DiagramScene::setLinkVisible() 方法
│
├─ test_setMode()
│   └─ DiagramScene::setMode() 方法
│
├─ test_setItemType()
│   └─ DiagramScene::setItemType() 方法
│
└─ ... (19 个测试)
```

---

## 📦 依赖链

### 编译依赖

```
CMakeLists.txt
  └─ find_package(Qt6 ... Widgets)
      ├─ Qt6::Core
      ├─ Qt6::Gui
      └─ Qt6::Widgets

diagramscene.pro (qmake)
  └─ QT += core gui widgets
```

### 运行时依赖

```
应用需要:
  ├─ Qt 运行库 (*.dll)
  ├─ 图形驱动
  ├─ 系统资源 (字体、图片)
  └─ 文件系统 (.fcproj 文件格式)
```

---

## 🎨 UI 控件布局

### MainWindow 的主要 UI 组成

```
┌─────────────────────────────────────────────┐
│ 菜单栏 (File, Edit, View, Help)             │
├─────────────────────────────────────────────┤
│ 工具栏 1: 标准工具 (新建、打开、保存)       │
│ 工具栏 2: 绘制工具 (图形选择)               │
├────────────────┬──────────────────────────────┤
│ 左侧面板       │                            │
│ - 图形工具     │   QGraphicsView            │
│ - 颜色选择器   │   (DiagramScene)           │
│ - 属性编辑     │                            │
│ - 图层面板     │                            │
├────────────────┴──────────────────────────────┤
│ 状态栏 (鼠标位置、缩放比例)                  │
│ 标签页 (多个场景)                            │
└──────────────────────────────────────────────┘
```

---

## 🔐 内存管理

### 关键对象生命周期

```
MainWindow
  ├─ owns DiagramScene (QObject parent)
  │   ├─ owns DiagramItem(s)
  │   │   ├─ owns DiagramTextItem (子项)
  │   │   ├─ references Arrow(s)
  │   │   └─ references DiagramPath(s)
  │   └─ owns QGraphicsView (MyGraphicsView)
  │
  ├─ owns QUndoStack
  │   └─ owns DeleteCommand, SnapshotCommand
  │
  ├─ owns UI Widgets (QToolBox, QComboBox, etc)
  │
  └─ owns Dialog (FindReplaceDialog)

垃圾回收:
  - QObjects: 通过 parent 自动回收
  - QGraphicsItem: 通过 scene 管理
  - Command: 通过 undoStack 管理
  - 手动 deleteLater(): 文本项空时
```

---

## 🎓 代码阅读建议

### 推荐阅读顺序

1. **main.cpp** (3 行) - 理解入口点
2. **diagramscene.h** (30 行) - 了解核心接口
3. **diagramitem.h** (40 行) - 理解图形元素
4. **mainwindow.h** (60 行) - 了解 UI 结构
5. **diagramscene.cpp** (100 行) - 学习事件处理
6. **diagramitem.cpp** (100 行) - 学习绘制逻辑
7. **mainwindow.cpp** (300 行) - 理解完整流程
8. **arrow.cpp**, **diagrampath.cpp** - 特殊连接逻辑

### 重点学习模块

- ✅ **Qt Graphics View 框架** - 高效的图形编辑基础
- ✅ **事件处理** - mousePressEvent, keyPressEvent 等
- ✅ **信号/槽机制** - Qt 的核心编程模型
- ✅ **命令模式** - 撤销/重做的实现
- ✅ **坐标变换** - 旋转、缩放的数学计算

---

## 📊 文件大小分布

```
mainwindow.cpp      76 KB   ███████████████████████████████ 44%
diagramitem.cpp     28 KB   ██████████████                   16%
diagramscene.cpp    25 KB   ████████████                    14%
diagramitemgroup.cpp 9 KB   ████                             5%
diagrampath.cpp      5 KB   ██                               3%
其他 cpp 文件        ~8 KB   ███                              5%
───────────────────────────────────────────────────────────
合计                173 KB   (包括头文件 ~18 KB)
```

---

本文档提供了 FreeCharts 项目的完整架构分析，包括：
- 目录结构和文件关系
- 类继承和关系图
- 核心流程和调用链
- 信号/槽连接
- 测试覆盖
- 依赖关系
- 性能和复杂度分析

