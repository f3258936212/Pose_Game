# Pose Quest 姿态闯关

这是一个 Android 原生小游戏项目，主题来自 `项目1.pdf` 中的“AI 姿态模仿闯关”。App 使用手机摄像头识别人体姿态，用户模仿指定动作后进行本地 AI 推理和评分，分数达标即可通关、解锁下一关和贴纸奖励。

## 最终版功能

- 首页：显示项目名称、已解锁关卡、奖励数量和累计星星。
- 首次标准动作录入：第一次打开时会逐关拍照录入标准姿态点，后续评分以用户录入的标准动作为基准。
- 关卡地图：展示 15 个动作关卡，区分“已通关 / 挑战中 / 未解锁”状态。
- 挑战页：CameraX 前置摄像头预览、姿态骨架叠加、动作提示、目标分数、历史最高分、拍照评分按钮。
- 本地 AI 推理：使用 MediaPipe Pose Landmarker Full 模型，在 Android 端直接输出人体关键点。
- 本地评分：对标准姿态点和用户姿态点做髋中心归一化、肩宽尺度归一化，然后计算相似度分数。
- 结果弹窗：显示通关/失败、分数、星级、奖励贴纸，并支持下一关或返回地图。
- 奖励收藏：展示已获得和未解锁的贴纸奖励。
- 本地进度保存：使用 SharedPreferences 保存解锁进度、最高分、星级和奖励。

## 技术栈

- Android 原生 Java
- XML Layout
- Material Components
- RecyclerView
- CameraX
- MediaPipe Tasks Vision
- MediaPipe Pose Landmarker Full
- SharedPreferences
- Gradle / Android Gradle Plugin
- JUnit 单元测试

## 核心文件

- 主入口：[app/src/main/java/com/posegame/MainActivity.java](app/src/main/java/com/posegame/MainActivity.java)
- CameraX 控制：[app/src/main/java/com/posegame/CameraXController.java](app/src/main/java/com/posegame/CameraXController.java)
- 姿态推理：[app/src/main/java/com/posegame/pose/MediaPipePoseEstimator.java](app/src/main/java/com/posegame/pose/MediaPipePoseEstimator.java)
- 姿态评分：[app/src/main/java/com/posegame/pose/PoseScoreCalculator.java](app/src/main/java/com/posegame/pose/PoseScoreCalculator.java)
- 归一化：[app/src/main/java/com/posegame/pose/PoseNormalizer.java](app/src/main/java/com/posegame/pose/PoseNormalizer.java)
- 标准姿态保存：[app/src/main/java/com/posegame/pose/PoseCalibrationStore.java](app/src/main/java/com/posegame/pose/PoseCalibrationStore.java)
- 游戏进度保存：[app/src/main/java/com/posegame/game/GameProgressManager.java](app/src/main/java/com/posegame/game/GameProgressManager.java)
- 关卡列表：[app/src/main/java/com/posegame/ui/LevelAdapter.java](app/src/main/java/com/posegame/ui/LevelAdapter.java)
- 奖励列表：[app/src/main/java/com/posegame/ui/RewardAdapter.java](app/src/main/java/com/posegame/ui/RewardAdapter.java)

## 页面布局

- 首页：[app/src/main/res/layout/activity_main.xml](app/src/main/res/layout/activity_main.xml)
- 关卡地图：[app/src/main/res/layout/layout_level_map.xml](app/src/main/res/layout/layout_level_map.xml)
- 挑战页：[app/src/main/res/layout/layout_challenge.xml](app/src/main/res/layout/layout_challenge.xml)
- 评分结果弹窗：[app/src/main/res/layout/dialog_score_result.xml](app/src/main/res/layout/dialog_score_result.xml)
- 奖励页：[app/src/main/res/layout/layout_rewards.xml](app/src/main/res/layout/layout_rewards.xml)
- 关卡卡片：[app/src/main/res/layout/item_level.xml](app/src/main/res/layout/item_level.xml)
- 奖励卡片：[app/src/main/res/layout/item_reward.xml](app/src/main/res/layout/item_reward.xml)

## 模型文件

模型文件位于：

```text
app/src/main/assets/pose_landmarker_full.task
```

推理代码通过 `setModelAssetPath("pose_landmarker_full.task")` 加载模型。当前版本不需要把图片上传到后端，推理和评分都在 Android 本地完成。

## 评分规则

MediaPipe 模型只输出人体关键点，不直接给动作分数。App 自己实现评分：

1. 拍照获得当前画面。
2. MediaPipe Pose Landmarker Full 输出人体关键点。
3. 读取该关已录入的标准姿态点。
4. 使用左右髋部中点作为人体中心。
5. 使用左右肩距离作为人体尺度。
6. 对标准姿态和用户姿态做归一化。
7. 计算关键点平均距离。
8. 转换为 0-100 分。

星级规则：

- `score >= 88`：三星
- `score >= 77`：二星
- `score >= pass_score`：一星
- `score < pass_score`：零星

通关后会保存最高分、星级、奖励贴纸，并解锁下一关。

## 构建

PowerShell：

```powershell
$env:JAVA_HOME='C:\Program Files\Android\Android Studio\jbr'
.\gradlew.bat --no-daemon :app:testDebugUnitTest :app:assembleDebug
```

APK 输出路径：

```text
D:\pose\app\build\outputs\apk\debug\app-debug.apk
```

## 真机调试

1. 手机打开“开发者选项”。
2. 打开“USB 调试”。
3. 用 USB 连接电脑并允许调试授权。
4. 安装 APK：

```powershell
C:\Users\32589\AppData\Local\Android\Sdk\platform-tools\adb.exe install -r D:\pose\app\build\outputs\apk\debug\app-debug.apk
```

5. 打开 App 并允许相机权限。
6. 首次使用按提示逐关录入标准动作。
7. 录入完成后进入关卡地图，开始正式闯关。

## 说明

`项目1.pdf` 中提到后端可用于接收图片、转发 AI 推理请求和保存记录。本最终版为了真机调试和课程展示更稳定，采用 Android 本地推理方案：后端不是必需模块，所有关键流程都能离线在手机端完成。
