# Android 课程表 App 设计方案

## 1. 产品目标
做一个**学生日常可直接使用**的课程表 App，核心目标：
- 快速查看“今天有什么课”。
- 一眼看到每周课表。
- 课前自动提醒，避免迟到。
- 能记录作业/考试等课程任务。

目标用户：大学生（优先），兼容中学课程场景。

## 2. MVP（第一版）功能范围

### 2.1 必做功能
1. **学期管理**：新增学期（开始日期、总周数）。
2. **课程管理**：课程名、老师、教室、上课星期、节次范围、起止周、单双周。
3. **课程表视图**：
   - 今日视图（按时间排序）
   - 周视图（周一~周日网格）
4. **提醒通知**：课程开始前 10/20/30 分钟可配置提醒。
5. **任务管理**：可关联课程，记录截止时间（作业/考试/实验）。
6. **本地持久化**：离线可用（Room 数据库）。

### 2.2 第二阶段功能（可选）
- 导入课表（教务系统 ICS/截图 OCR）。
- 云同步（Firebase/Supabase）。
- 桌面小组件（Widget）。
- 课表分享图导出。

## 3. 信息架构（页面）
- `首页（今日）`：今天课程卡片 + 最近任务。
- `周课表`：网格展示所有课程块，支持左右切周。
- `任务`：按“未完成/已完成”分组。
- `我的`：学期、提醒偏好、主题设置。

底部导航建议 4 Tab：今日 / 周课表 / 任务 / 我的。

## 4. 数据模型设计

### 4.1 实体
- `Term`
  - `id: Long`
  - `name: String`
  - `startDate: LocalDate`
  - `weekCount: Int`
- `Course`
  - `id: Long`
  - `termId: Long`
  - `name: String`
  - `teacher: String?`
  - `room: String?`
  - `dayOfWeek: Int` (1~7)
  - `startSection: Int`
  - `endSection: Int`
  - `startWeek: Int`
  - `endWeek: Int`
  - `weekType: Int` (0=每周,1=单周,2=双周)
  - `color: String`
- `Task`
  - `id: Long`
  - `courseId: Long?`
  - `title: String`
  - `deadline: LocalDateTime`
  - `status: Int` (0=未完成,1=已完成)

### 4.2 关键规则
- 通过 `term.startDate` + 当前日期计算当前周次。
- 展示课程时，需匹配：
  1) 星期一致；
  2) 当前周在 `startWeek..endWeek`；
  3) 满足单双周条件。

## 5. 技术方案（Android）
- 语言：`Kotlin`
- UI：`Jetpack Compose + Material 3`
- 架构：`MVVM + Repository`
- 数据：`Room + DataStore`
- 异步：`Kotlin Coroutines + Flow`
- 依赖注入：`Hilt`
- 通知：`WorkManager + AlarmManager`（精确提醒可按系统版本降级处理）

推荐模块：
- `app`（入口）
- `core-ui`（主题、通用组件）
- `feature-schedule`（今日/周课表）
- `feature-task`（任务）
- `feature-settings`（设置）
- `data`（db、dao、repository）

## 6. 交互与 UI 设计要点
1. **颜色编码课程**：同课程固定颜色，提高识别效率。
2. **周课表网格**：
   - 左侧节次；顶部星期。
   - 课程块显示：课程名 + 教室。
   - 点击弹出详情（老师、周次、提醒开关）。
3. **今日卡片**：
   - 距离下一节课倒计时。
   - 已结束课程置灰。
4. **深色模式**：默认跟随系统。
5. **无障碍**：字体缩放适配，保证对比度。

## 7. 非功能需求
- 冷启动 < 2 秒（中端机）。
- 本地查询 1 周课表 < 100ms。
- 崩溃率控制（接入 Firebase Crashlytics 可选）。
- 兼容 Android 8.0+

## 8. 开发排期（示例 2 周）
- Day 1~2：项目脚手架、数据库与实体。
- Day 3~5：课程 CRUD + 今日视图。
- Day 6~8：周课表网格 + 周次计算。
- Day 9~10：提醒通知 + 设置。
- Day 11~12：任务模块。
- Day 13~14：联调、测试、发布准备。

## 9. 测试清单
- 单元测试：
  - 周次计算（跨学期边界）
  - 单双周过滤
  - 课程冲突检测
- UI 测试：
  - 周课表渲染
  - 添加课程流程
  - 提醒配置流程
- 手工测试：
  - 切换深色模式
  - 系统时间变更
  - 重启后提醒是否生效

## 10. 未来可扩展方向
- AI 智能建议：根据任务截止时间推荐复习计划。
- 课堂签到与统计。
- 与校园地图联动（下节课导航）。

---

如果你愿意，我下一步可以直接给你生成一个**可运行的 Android Studio 项目骨架**（Compose + Room + Hilt），你拉下来就能编译运行。
