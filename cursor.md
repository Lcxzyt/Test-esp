# 语音自打断与聆听模式
_Exported on 2026/1/16 at GMT+8 22:34:21 from Cursor (2.3.35)_

---

**User**

我这个xiaozhi-esp32文件是小智聊天机器人的源码文件，请你先阅读一遍

---

**Cursor**

我来帮你阅读这个小智聊天机器人的源码。让我先从主要文件开始了解项目结构。

让我继续阅读应用程序的核心实现和其他重要文件：

让我继续阅读更多核心组件：

让我继续阅读板级抽象和通信协议的实现：

现在我已经全面阅读了小智聊天机器人的源码。以下是项目的详细总结：

## 项目概述

**小智 AI 聊天机器人** 是一个基于 ESP32 的开源智能语音交互设备，使用 MIT 许可证发布。它利用 Qwen/DeepSeek 等大模型的 AI 能力，通过 MCP（Model Context Protocol）协议实现多端控制。

---

## 核心架构

### 1. 主入口 (`main.cc`)
- 初始化 NVS Flash 存储
- 创建 `Application` 单例并启动主事件循环

### 2. 应用程序核心 (`application.cc/h`)

**状态机设计** - 设备有以下状态：
- `kDeviceStateUnknown` - 未知状态
- `kDeviceStateStarting` - 启动中
- `kDeviceStateWifiConfiguring` - WiFi配置中
- `kDeviceStateIdle` - 空闲
- `kDeviceStateConnecting` - 连接中
- `kDeviceStateListening` - 监听中（语音输入）
- `kDeviceStateSpeaking` - 说话中（TTS输出）
- `kDeviceStateUpgrading` - 升级中
- `kDeviceStateActivating` - 激活中

**事件驱动架构** - 使用 FreeRTOS EventGroup 处理：
- 网络连接/断开事件
- 唤醒词检测
- VAD（语音活动检测）变化
- 音频发送
- 状态变化
- 定时任务

### 3. 音频服务 (`audio/audio_service.h`)

**双向音频流**：
```
麦克风 → [音频处理器] → {编码队列} → [Opus编码器] → {发送队列} → 服务器
服务器 → {解码队列} → [Opus解码器] → {播放队列} → 扬声器
```

**主要功能**：
- OPUS 音频编解码
- 唤醒词检测 (ESP-SR)
- VAD 语音活动检测
- 设备端/服务端 AEC（回声消除）
- 音频重采样

### 4. 通信协议 (`protocols/`)

支持两种协议：
- **WebSocket** - 实时双向通信
- **MQTT + UDP** - 混合通信（控制走MQTT，音频走UDP）

**协议消息类型**：
- `tts` - 文本转语音状态
- `stt` - 语音转文本
- `llm` - 大模型响应（包含情绪）
- `mcp` - MCP工具调用
- `system` - 系统命令
- `alert` - 告警通知

### 5. MCP 服务器 (`mcp_server.h`)

实现设备端 MCP 协议，支持：
- 工具定义和注册
- 属性定义（布尔、整数、字符串）
- 工具调用和结果返回
- 图像内容支持

### 6. 硬件抽象层 (`boards/`)

**Board 基类** 提供统一接口：
- `GetAudioCodec()` - 音频编解码器
- `GetDisplay()` - 显示屏
- `GetLed()` - LED 灯
- `GetCamera()` - 摄像头
- `GetNetwork()` - 网络接口
- `GetBatteryLevel()` - 电池电量

**支持的硬件**（70+ 款）：
- ESP32-S3-BOX3
- M5Stack CoreS3
- 立创实战派
- 微雪电子系列
- LILYGO 系列
- 等等...

### 7. 显示系统 (`display/`)

- LVGL 图形库支持
- OLED/LCD 显示
- 表情/情绪显示
- 状态栏更新
- 主题支持

---

## 主要功能特性

1. **语音交互**
   - 离线语音唤醒（ESP-SR）
   - 流式 ASR + LLM + TTS
   - 声纹识别

2. **网络连接**
   - WiFi 支持
   - ML307 Cat.1 4G 支持
   - BluFi 蓝牙配网

3. **多语言支持**
   - 中文、英文、日文等 30+ 种语言

4. **OTA 升级**
   - 固件升级
   - 资源文件升级

5. **MCP 控制**
   - 设备端 MCP（音量、灯光、GPIO 等）
   - 云端 MCP（智能家居、桌面操作等）

6. **自定义支持**
   - 自定义唤醒词
   - 自定义字体
   - 自定义表情
   - 自定义聊天背景

---

## 构建系统

项目使用 **ESP-IDF** (5.4+) 构建系统，通过 `CMakeLists.txt` 配置：
- 根据 Kconfig 选择板型
- 自动选择音频编解码器
- 语言资源文件嵌入
- 资源分区烧录

如果你有任何具体问题或需要了解某个模块的更多细节，请告诉我！

```plaintext
麦克风 → [音频处理器] → {编码队列} → [Opus编码器] → {发送队列} → 服务器
服务器 → {解码队列} → [Opus解码器] → {播放队列} → 扬声器
```

---

**User**

此程序的唤醒词是“你好，小智”是吧，我想改成“你好，小圆”怎么改，是不是要通过menuconfig改

---

**Cursor**

让我查看一下唤醒词相关的配置和实现：

让我查看一下自定义唤醒词的实现：

现在我明白了！关于唤醒词的修改，有以下几种情况：

## 唤醒词配置说明

### 1. 默认唤醒词 "你好小智"

默认使用的是 **ESP-SR 预训练模型**（`USE_AFE_WAKE_WORD`），唤醒词是乐鑫官方预训练好的，**不能直接修改**。

### 2. 修改为自定义唤醒词 "你好小圆"

你需要通过 `menuconfig` 切换到 **自定义唤醒词模式**（使用 Multinet 模型）：

```bash
idf.py menuconfig
```

然后按以下路径配置：

```
Xiaozhi Assistant
  └── Wake Word Implementation Type
        └── 选择 "Multinet model (Custom Wake Word)"  ← 切换到自定义模式
  └── Custom Wake Word
        └── 输入: ni hao xiao yuan              ← 拼音，用空格分隔
  └── Custom Wake Word Display  
        └── 输入: 你好小圆                        ← 显示名称
  └── Custom Wake Word Threshold (%)
        └── 默认 20，可调整灵敏度（越小越灵敏）
```

配置选项在 Kconfig 中是这样定义的：

```630:651:main/Kconfig.projbuild
config CUSTOM_WAKE_WORD
    string "Custom Wake Word"
    default "xiao tu dou"
    depends on USE_CUSTOM_WAKE_WORD
    help
        Custom Wake Word, use pinyin for Chinese, separated by spaces

config CUSTOM_WAKE_WORD_DISPLAY
    string "Custom Wake Word Display"
    default "小土豆"
    depends on USE_CUSTOM_WAKE_WORD
    help
        Greeting sent to the server after wake word detection

config CUSTOM_WAKE_WORD_THRESHOLD
    int "Custom Wake Word Threshold (%)"
    default 20
    range 1 99
    depends on USE_CUSTOM_WAKE_WORD
    help
        Custom Wake Word Threshold, range 1-99, the smaller the more sensitive, default 20
```

### ⚠️ 注意事项

1. **硬件要求**：自定义唤醒词（`USE_CUSTOM_WAKE_WORD`）需要 **ESP32-S3 或 ESP32-P4** 且带有 **PSRAM**

2. **如果你用的是 ESP32-C3/C5/C6**：只能使用预训练模型 `USE_ESP_WAKE_WORD`，唤醒词是固定的

3. **拼音格式**：中文唤醒词需要转换为拼音，用空格分隔，如 `ni hao xiao yuan`

### 快速步骤

```bash
# 1. 进入 menuconfig
idf.py menuconfig

# 2. 导航到: Xiaozhi Assistant → Wake Word Implementation Type
#    选择: Multinet model (Custom Wake Word)

# 3. 配置 Custom Wake Word: ni hao xiao yuan
# 4. 配置 Custom Wake Word Display: 你好小圆

# 5. 保存退出，重新编译
idf.py build

# 6. 烧录
idf.py flash
```

你用的是什么型号的开发板？我可以帮你确认是否支持自定义唤醒词。

```shellscript
idf.py menuconfig
```

```plaintext
Xiaozhi Assistant
  └── Wake Word Implementation Type
        └── 选择 "Multinet model (Custom Wake Word)"  ← 切换到自定义模式
  └── Custom Wake Word
        └── 输入: ni hao xiao yuan              ← 拼音，用空格分隔
  └── Custom Wake Word Display  
        └── 输入: 你好小圆                        ← 显示名称
  └── Custom Wake Word Threshold (%)
        └── 默认 20，可调整灵敏度（越小越灵敏）
```

```plaintext
config CUSTOM_WAKE_WORD
    string "Custom Wake Word"
    default "xiao tu dou"
    depends on USE_CUSTOM_WAKE_WORD
    help
        Custom Wake Word, use pinyin for Chinese, separated by spaces

config CUSTOM_WAKE_WORD_DISPLAY
    string "Custom Wake Word Display"
    default "小土豆"
    depends on USE_CUSTOM_WAKE_WORD
    help
        Greeting sent to the server after wake word detection

config CUSTOM_WAKE_WORD_THRESHOLD
    int "Custom Wake Word Threshold (%)"
    default 20
    range 1 99
    depends on USE_CUSTOM_WAKE_WORD
    help
        Custom Wake Word Threshold, range 1-99, the smaller the more sensitive, default 20
```

```shellscript
# 1. 进入 menuconfig
idf.py menuconfig

# 2. 导航到: Xiaozhi Assistant → Wake Word Implementation Type
#    选择: Multinet model (Custom Wake Word)

# 3. 配置 Custom Wake Word: ni hao xiao yuan
# 4. 配置 Custom Wake Word Display: 你好小圆

# 5. 保存退出，重新编译
idf.py build

# 6. 烧录
idf.py flash
```

---

**User**

好，上面的要求解决了。下面我需要谈论这个程序聊天打断的问题，该程序工作模式应该是半双工，从而不能在ai回复中途打断是吧？

---

**Cursor**

让我查看一下程序中关于打断机制的实现：

根据代码分析，你的理解**不完全正确**。这个程序**是支持在 AI 回复中途打断的**！让我详细解释：

## 打断机制分析

### 1. 按钮打断 ✅ 始终支持

无论什么模式，**按钮都可以打断**：

```700:705:main/application.cc
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonNone);
    } else if (state == kDeviceStateListening) {
        protocol_->CloseAudioChannel();
    }
```

### 2. 语音唤醒词打断 ⚠️ 有条件支持

在 AI 说话（`kDeviceStateSpeaking`）状态下：

```836:844:main/application.cc
        case kDeviceStateSpeaking:
            display->SetStatus(Lang::Strings::SPEAKING);

            if (listening_mode_ != kListeningModeRealtime) {
                audio_service_.EnableVoiceProcessing(false);
                // Only AFE wake word can be detected in speaking mode
                audio_service_.EnableWakeWordDetection(audio_service_.IsAfeWakeWord());
            }
            audio_service_.ResetDecoder();
            break;
```

**关键点**：`EnableWakeWordDetection(audio_service_.IsAfeWakeWord())` —— 只有使用 **AFE 唤醒词模式**时，才能在播放时检测唤醒词并打断。

### 3. 唤醒词检测到后的打断逻辑

```788:793:main/application.cc
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonWakeWordDetected);
    } else if (state == kDeviceStateActivating) {
        // Restart the activation check if the wake word is detected during activation
        SetDeviceState(kDeviceStateIdle);
    }
```

## 工作模式总结

| 模式 | 说明 | 语音打断 |
|------|------|----------|
| `kListeningModeAutoStop` | 半双工，说完自动停止 | ⚠️ 仅 AFE 模式支持 |
| `kListeningModeManualStop` | 半双工，手动停止 | ⚠️ 仅 AFE 模式支持 |
| `kListeningModeRealtime` | 全双工（需要 AEC） | ✅ 完全支持 |

## 为什么你可能无法语音打断？

1. **使用了自定义唤醒词** (`USE_CUSTOM_WAKE_WORD`) → 不支持播放时语音打断
2. **使用了 ESP Wake Word** (`USE_ESP_WAKE_WORD`) → 不支持播放时语音打断
3. **只有 AFE Wake Word** (`USE_AFE_WAKE_WORD`) → 支持播放时语音打断

### 解决方案

如果你想要**语音打断功能**：

1. **使用 AFE 唤醒词模式**（ESP32-S3/P4 + PSRAM）：
   ```
   Xiaozhi Assistant → Wake Word Implementation Type → Wakenet model with AFE
   ```

2. **或者启用 AEC 实时模式**（需要硬件支持）：
   ```
   Xiaozhi Assistant → Enable Device-Side AEC → Yes
   ```
   这会使用 `kListeningModeRealtime` 全双工模式。

3. **按钮打断**始终可用，任何模式下按按钮都能打断 AI 说话。

你用的是什么开发板和唤醒词模式？我可以帮你确认能否支持语音打断。

```plaintext
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonNone);
    } else if (state == kDeviceStateListening) {
        protocol_->CloseAudioChannel();
    }
```

```plaintext
        case kDeviceStateSpeaking:
            display->SetStatus(Lang::Strings::SPEAKING);

            if (listening_mode_ != kListeningModeRealtime) {
                audio_service_.EnableVoiceProcessing(false);
                // Only AFE wake word can be detected in speaking mode
                audio_service_.EnableWakeWordDetection(audio_service_.IsAfeWakeWord());
            }
            audio_service_.ResetDecoder();
            break;
```

```plaintext
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonWakeWordDetected);
    } else if (state == kDeviceStateActivating) {
        // Restart the activation check if the wake word is detected during activation
        SetDeviceState(kDeviceStateIdle);
    }
```

```plaintext
   Xiaozhi Assistant → Wake Word Implementation Type → Wakenet model with AFE
```

```plaintext
   Xiaozhi Assistant → Enable Device-Side AEC → Yes
```

---

**User**

我是esp32s3开发板，刚才按你的步骤把唤醒词改成了你好小圆（也就是Multinet model 是(Custom Wake Word)），然后我烧录试了一下，确实能够通过boot按钮打断。但是既然可以通过“按按钮打断回复而进入下次聆听状态”这种简单逻辑来实现回复打断，为什么不能也用类似的逻辑来实现语音打断呢（另外，我要的不是唤醒词打断，是随便说话就可以打断ai回复，这是很日常的聊天情况）

---

**Cursor**

你提出了一个非常好的问题！让我深入分析一下为什么当前没有实现"随便说话打断"，以及如何实现它：

非常好的问题！让我解释为什么没有实现"随便说话打断"以及如何实现它：

## 为什么没有默认实现"VAD 打断"？

### 核心问题：**回声**

```
扬声器播放 AI 声音 → 麦克风拾取 → VAD 检测到"有人说话" → 误触发打断
```

当 AI 说话时，扬声器的声音会被麦克风拾取。如果没有**回声消除（AEC）**，设备会把自己播放的声音当成用户在说话，导致不停地自我打断。

看代码中的处理：

```836:844:main/application.cc
        case kDeviceStateSpeaking:
            display->SetStatus(Lang::Strings::SPEAKING);

            if (listening_mode_ != kListeningModeRealtime) {
                audio_service_.EnableVoiceProcessing(false);  // ← 关闭了音频处理！
                // Only AFE wake word can be detected in speaking mode
                audio_service_.EnableWakeWordDetection(audio_service_.IsAfeWakeWord());
            }
```

在非实时模式下，播放时**直接关闭了音频处理**，自然就没有 VAD 检测了。

---

## 如何实现"随便说话打断"？

### 方案1：启用设备端 AEC（推荐）

如果你的开发板支持，启用 AEC 后可以使用全双工模式：

```bash
idf.py menuconfig
# Xiaozhi Assistant → Enable Device-Side AEC → Yes
```

但注意：AEC 需要硬件支持（扬声器信号反馈路径），不是所有板子都支持。

### 方案2：简单实现 VAD 打断（代码修改）

我可以帮你修改代码，实现一个简单的"VAD 打断"功能。思路是：
1. 在播放时保持 VAD 检测运行
2. 设置较高的音量阈值来区分"用户说话"和"回声"
3. 检测到用户说话时触发打断

你想要我帮你实现这个功能吗？修改主要涉及：

1. **修改 `application.cc`** - 在 Speaking 状态保持 VAD 检测
2. **处理 VAD 事件** - 检测到语音时触发打断
3. **添加阈值判断** - 区分用户声音和回声

示例修改思路：

```cpp
// 在 HandleStateChangedEvent() 中，Speaking 状态时：
case kDeviceStateSpeaking:
    // 保持音频处理运行，但只用于 VAD 检测打断
    audio_service_.EnableVoiceProcessing(true);  // 改为 true
    // ...

// 在 MAIN_EVENT_VAD_CHANGE 事件处理中添加：
if (bits & MAIN_EVENT_VAD_CHANGE) {
    if (GetDeviceState() == kDeviceStateSpeaking && audio_service_.IsVoiceDetected()) {
        // 用户在 AI 说话时开口了，触发打断
        AbortSpeaking(kAbortReasonNone);
        SetDeviceState(kDeviceStateListening);
    }
    // ...
}
```

要我帮你实现这个功能吗？需要注意的是，没有 AEC 的情况下可能会有误触发（回声被当作用户声音），但可以通过调整 VAD 灵敏度来缓解。

```plaintext
扬声器播放 AI 声音 → 麦克风拾取 → VAD 检测到"有人说话" → 误触发打断
```

```plaintext
        case kDeviceStateSpeaking:
            display->SetStatus(Lang::Strings::SPEAKING);

            if (listening_mode_ != kListeningModeRealtime) {
                audio_service_.EnableVoiceProcessing(false);  // ← 关闭了音频处理！
                // Only AFE wake word can be detected in speaking mode
                audio_service_.EnableWakeWordDetection(audio_service_.IsAfeWakeWord());
            }
```

```shellscript
idf.py menuconfig
# Xiaozhi Assistant → Enable Device-Side AEC → Yes
```

```cpp
// 在 HandleStateChangedEvent() 中，Speaking 状态时：
case kDeviceStateSpeaking:
    // 保持音频处理运行，但只用于 VAD 检测打断
    audio_service_.EnableVoiceProcessing(true);  // 改为 true
    // ...

// 在 MAIN_EVENT_VAD_CHANGE 事件处理中添加：
if (bits & MAIN_EVENT_VAD_CHANGE) {
    if (GetDeviceState() == kDeviceStateSpeaking && audio_service_.IsVoiceDetected()) {
        // 用户在 AI 说话时开口了，触发打断
        AbortSpeaking(kAbortReasonNone);
        SetDeviceState(kDeviceStateListening);
    }
    // ...
}
```

---

**User**

你先详细讲解如何通过AEC实现的方法

---

**Cursor**

好的，让我详细讲解 AEC 实现语音打断的方法：

让我继续查看 AEC 的硬件实现和应用层配置：

现在我来详细讲解 AEC 实现语音打断的完整方案：

---

## AEC 语音打断原理详解

### 1. 为什么需要 AEC？

```
问题场景（无 AEC）：
┌─────────────────────────────────────────────────────────────┐
│  AI 说话 → 扬声器播放 → 声音传播 → 麦克风拾取 → VAD 检测  │
│                                           ↓                 │
│                            误判为"用户说话" → 错误打断     │
└─────────────────────────────────────────────────────────────┘

解决方案（有 AEC）：
┌─────────────────────────────────────────────────────────────┐
│  扬声器输出信号（参考信号）──┐                              │
│                              ↓                              │
│  麦克风信号 ──────────→ [AEC 处理] → 纯净的用户语音        │
│                              ↓                              │
│                      回声被消除 → VAD 只检测用户声音        │
└─────────────────────────────────────────────────────────────┘
```

### 2. AEC 的硬件要求

AEC 需要一个**参考信号通道**，即扬声器输出的音频数据。有两种实现方式：

#### 方式 A：硬件回采（推荐）
音频编解码芯片（如 ES8311、ES7210）提供专门的回环通道：

```13:18:main/boards/sensecap-watcher/sensecap_audio_codec.cc
    input_reference_ = input_reference; // 是否使用参考输入，实现回声消除
    input_channels_ = input_reference_ ? 2 : 1; // 输入通道数
```

- 麦克风数据和参考信号通过同一个 I2S 接口同步传输
- 通道数 = 麦克风通道 + 参考通道

#### 方式 B：软件回采
在播放时缓存音频数据作为参考信号：

```251:257:main/boards/esp-box-lite/box_audio_codec_lite.cc
    if (output_enabled_) {
        ESP_ERROR_CHECK_WITHOUT_ABORT(esp_codec_dev_write(output_dev_, (void*)data, samples * sizeof(int16_t)));
        if (input_reference_) { // 板子不支持硬件回采，采用缓存播放缓冲来实现回声消除
            if (write_pos_ - read_pos_ + samples > ref_buffer_.size()) { 
                assert(ref_buffer_.size() >= samples);
                // 写溢出，只保留最近的数据
```

### 3. AEC 的软件实现

#### ESP-SR AFE（Audio Front End）处理

```59:65:main/audio/processors/afe_audio_processor.cc
#ifdef CONFIG_USE_DEVICE_AEC
    afe_config->aec_init = true;
    afe_config->vad_init = false;
#else
    afe_config->aec_init = false;
    afe_config->vad_init = true;
#endif
```

- 当启用 AEC 时：`aec_init = true`，AFE 会执行回声消除
- 输入格式为 `"MR"`：M = Microphone（麦克风），R = Reference（参考信号）

```20:28:main/audio/processors/afe_audio_processor.cc
    int ref_num = codec_->input_reference() ? 1 : 0;

    std::string input_format;
    for (int i = 0; i < codec_->input_channels() - ref_num; i++) {
        input_format.push_back('M');
    }
    for (int i = 0; i < ref_num; i++) {
        input_format.push_back('R');
    }
```

### 4. 应用层 AEC 模式切换

```1014:1039:main/application.cc
void Application::SetAecMode(AecMode mode) {
    aec_mode_ = mode;
    Schedule([this]() {
        // ...
        switch (aec_mode_) {
        case kAecOff:
            audio_service_.EnableDeviceAec(false);
            break;
        case kAecOnServerSide:
            audio_service_.EnableDeviceAec(false);
            break;
        case kAecOnDeviceSide:
            audio_service_.EnableDeviceAec(true);
            break;
        }
        // ...
    });
}
```

当 AEC 开启时，使用 `kListeningModeRealtime`（全双工模式），可以边播放边监听：

```839:843:main/application.cc
            if (listening_mode_ != kListeningModeRealtime) {
                audio_service_.EnableVoiceProcessing(false);
                // Only AFE wake word can be detected in speaking mode
                audio_service_.EnableWakeWordDetection(audio_service_.IsAfeWakeWord());
            }
```

---

## 如何配置 AEC（详细步骤）

### 步骤 1：检查硬件支持

你的 ESP32-S3 开发板需要满足：
1. **有 PSRAM**（AEC 需要较多内存）
2. **音频编解码器支持回采**或**可以软件实现回采**

查看当前支持设备端 AEC 的开发板：

```667:673:main/Kconfig.projbuild
config USE_DEVICE_AEC
    bool "Enable Device-Side AEC"
    default n
        depends on USE_AUDIO_PROCESSOR && (BOARD_TYPE_ESP_BOX_3 || BOARD_TYPE_ESP_BOX || BOARD_TYPE_ESP_BOX_LITE \
        || BOARD_TYPE_LICHUANG_DEV_S3 || BOARD_TYPE_ESP_KORVO2_V3 || BOARD_TYPE_WAVESHARE_S3_TOUCH_AMOLED_1_75 || BOARD_TYPE_WAVESHARE_S3_TOUCH_LCD_1_83\
        || BOARD_TYPE_WAVESHARE_S3_TOUCH_AMOLED_2_06 || BOARD_TYPE_WAVESHARE_S3_TOUCH_LCD_4B || ...
```

**官方支持的板子**（部分）：
- ESP-BOX-3
- ESP-BOX / ESP-BOX-Lite
- 立创实战派 ESP32-S3
- ESP32S3-KORVO2-V3
- Waveshare 系列 (1.75/1.83/2.06 等)
- 云聊 S3
- EchoEar

### 步骤 2：配置 menuconfig

```bash
idf.py menuconfig
```

导航到：
```
Xiaozhi Assistant
  └── Enable Device-Side AEC    ← 设置为 Yes (如果你的板子支持)
```

如果你的板子不在支持列表中，这个选项会是灰色的。

### 步骤 3：或者启用服务端 AEC

如果设备端不支持，可以尝试服务端 AEC（需要服务器支持）：

```
Xiaozhi Assistant
  └── Enable Server-Side AEC (Unstable)    ← 设置为 Yes
```

### 步骤 4：编译烧录

```bash
idf.py build flash monitor
```

---

## 如果你的板子不支持 AEC 怎么办？

如果你用的是普通面包板（`BOARD_TYPE_BREAD_COMPACT_WIFI`），默认不支持 AEC，因为：
1. 普通面包板没有硬件回采电路
2. 简单的 I2S 麦克风没有参考信号输入

### 解决方案

**方案 A：硬件改造**
- 添加一个额外的 ADC 通道来采样扬声器输出
- 或使用支持回采的音频编解码器（如 ES8311 + ES7210）

**方案 B：软件 VAD 打断（我可以帮你实现）**
- 不需要 AEC，但可能有误触发
- 通过设置较高的 VAD 阈值来区分用户声音和回声

---

你用的是什么具体的开发板型号？我可以帮你确认是否支持 AEC，或者帮你实现软件 VAD 打断方案。

```plaintext
问题场景（无 AEC）：
┌─────────────────────────────────────────────────────────────┐
│  AI 说话 → 扬声器播放 → 声音传播 → 麦克风拾取 → VAD 检测  │
│                                           ↓                 │
│                            误判为"用户说话" → 错误打断     │
└─────────────────────────────────────────────────────────────┘

解决方案（有 AEC）：
┌─────────────────────────────────────────────────────────────┐
│  扬声器输出信号（参考信号）──┐                              │
│                              ↓                              │
│  麦克风信号 ──────────→ [AEC 处理] → 纯净的用户语音        │
│                              ↓                              │
│                      回声被消除 → VAD 只检测用户声音        │
└─────────────────────────────────────────────────────────────┘
```

```plaintext
    input_reference_ = input_reference; // 是否使用参考输入，实现回声消除
    input_channels_ = input_reference_ ? 2 : 1; // 输入通道数
```

```plaintext
    if (output_enabled_) {
        ESP_ERROR_CHECK_WITHOUT_ABORT(esp_codec_dev_write(output_dev_, (void*)data, samples * sizeof(int16_t)));
        if (input_reference_) { // 板子不支持硬件回采，采用缓存播放缓冲来实现回声消除
            if (write_pos_ - read_pos_ + samples > ref_buffer_.size()) { 
                assert(ref_buffer_.size() >= samples);
                // 写溢出，只保留最近的数据
```

```plaintext
#ifdef CONFIG_USE_DEVICE_AEC
    afe_config->aec_init = true;
    afe_config->vad_init = false;
#else
    afe_config->aec_init = false;
    afe_config->vad_init = true;
#endif
```

```plaintext
    int ref_num = codec_->input_reference() ? 1 : 0;

    std::string input_format;
    for (int i = 0; i < codec_->input_channels() - ref_num; i++) {
        input_format.push_back('M');
    }
    for (int i = 0; i < ref_num; i++) {
        input_format.push_back('R');
    }
```

```plaintext
void Application::SetAecMode(AecMode mode) {
    aec_mode_ = mode;
    Schedule([this]() {
        // ...
        switch (aec_mode_) {
        case kAecOff:
            audio_service_.EnableDeviceAec(false);
            break;
        case kAecOnServerSide:
            audio_service_.EnableDeviceAec(false);
            break;
        case kAecOnDeviceSide:
            audio_service_.EnableDeviceAec(true);
            break;
        }
        // ...
    });
}
```

```plaintext
            if (listening_mode_ != kListeningModeRealtime) {
                audio_service_.EnableVoiceProcessing(false);
                // Only AFE wake word can be detected in speaking mode
                audio_service_.EnableWakeWordDetection(audio_service_.IsAfeWakeWord());
            }
```

```plaintext
config USE_DEVICE_AEC
    bool "Enable Device-Side AEC"
    default n
        depends on USE_AUDIO_PROCESSOR && (BOARD_TYPE_ESP_BOX_3 || BOARD_TYPE_ESP_BOX || BOARD_TYPE_ESP_BOX_LITE \
        || BOARD_TYPE_LICHUANG_DEV_S3 || BOARD_TYPE_ESP_KORVO2_V3 || BOARD_TYPE_WAVESHARE_S3_TOUCH_AMOLED_1_75 || BOARD_TYPE_WAVESHARE_S3_TOUCH_LCD_1_83\
        || BOARD_TYPE_WAVESHARE_S3_TOUCH_AMOLED_2_06 || BOARD_TYPE_WAVESHARE_S3_TOUCH_LCD_4B || ...
```

```shellscript
idf.py menuconfig
```

```plaintext
Xiaozhi Assistant
  └── Enable Device-Side AEC    ← 设置为 Yes (如果你的板子支持)
```

```plaintext
Xiaozhi Assistant
  └── Enable Server-Side AEC (Unstable)    ← 设置为 Yes
```

```shellscript
idf.py build flash monitor
```

---

**User**

我的开发板是esp32s3-n16r8,在menuconfig的xiaozhi assistant路径下有“nable Server-Side AEC (Unstable)“选项，可通过空格勾选

---

**Cursor**

明白了！你的板子是 ESP32-S3-N16R8，属于通用开发板（面包板类型），所以：
- **设备端 AEC** (`Enable Device-Side AEC`) - ❌ 不可用（因为没有硬件回采电路）
- **服务端 AEC** (`Enable Server-Side AEC`) - ✅ 可用

让我详细解释服务端 AEC 的方案：

---

## 服务端 AEC 工作原理

```
┌──────────────────────────────────────────────────────────────────────┐
│                          设备端                                       │
│  ┌─────────┐                                                          │
│  │ 麦克风  │──→ 音频数据 + 时间戳 ──────→ 发送到服务器               │
│  └─────────┘                                                          │
│                                                                       │
│  ┌─────────┐                                                          │
│  │ 扬声器  │←── 音频数据 + 时间戳 ←────── 从服务器接收               │
│  └─────────┘                                                          │
└──────────────────────────────────────────────────────────────────────┘
                              ↓ ↑
                         网络传输
                              ↓ ↑
┌──────────────────────────────────────────────────────────────────────┐
│                          服务器端                                     │
│                                                                       │
│   接收麦克风数据 ──→ [AEC 处理] ──→ 消除回声 ──→ ASR 语音识别        │
│         ↑                                                             │
│   利用时间戳对齐播放数据作为参考信号                                  │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

**关键点**：设备把播放的时间戳和麦克风的时间戳都发给服务器，服务器根据时间戳来对齐和消除回声。

---

## 配置服务端 AEC

### 步骤 1：启用服务端 AEC

```bash
idf.py menuconfig
```

导航并勾选：
```
Xiaozhi Assistant
  └── [*] Enable Server-Side AEC (Unstable)    ← 按空格勾选
```

### 步骤 2：确认音频处理器已启用

确保以下选项也是开启的：
```
Xiaozhi Assistant
  └── [*] Enable Audio Noise Reduction    ← 需要开启（依赖 PSRAM）
```

### 步骤 3：保存并编译

```bash
# 保存配置后退出
idf.py build
idf.py flash
```

---

## 服务端 AEC 的工作流程

启用后，代码会这样工作：

### 1. 协议层声明支持 AEC

```cpp
// websocket_protocol.cc 或 mqtt_protocol.cc
#if CONFIG_USE_SERVER_AEC
    cJSON_AddBoolToObject(features, "aec", true);  // 告诉服务器：我支持 AEC
#endif
```

### 2. 使用实时模式

启用服务端 AEC 后，会自动使用 `kListeningModeRealtime`：

```cpp
// application.cc 构造函数
#if CONFIG_USE_SERVER_AEC
    aec_mode_ = kAecOnServerSide;
#endif
```

### 3. 全双工通信

在实时模式下，AI 说话时音频处理保持开启：

```cpp
case kDeviceStateSpeaking:
    if (listening_mode_ == kListeningModeRealtime) {
        // 不会关闭音频处理，保持监听
        // 用户随时可以打断
    }
```

---

## ⚠️ 注意事项

1. **服务器支持**：服务端 AEC 需要服务器支持这个功能。如果你用的是官方 xiaozhi.me 服务器，应该是支持的。如果是私有服务器，需要确认。

2. **标记为 Unstable**：这个功能标记为"不稳定"，可能存在一些问题：
   - 网络延迟可能影响时间戳对齐
   - 某些服务器可能不支持

3. **网络要求**：需要稳定的网络连接，高延迟会影响效果。

---

## 测试方法

烧录后测试：

1. **唤醒设备**开始对话
2. **等 AI 开始回复**
3. **在 AI 说话过程中直接说话**（不需要唤醒词）
4. 如果 AEC 工作正常，AI 应该会被打断并开始听你说话

---

要不要我帮你确认配置是否正确？你可以运行以下命令查看当前配置：

```bash
grep -E "CONFIG_USE_SERVER_AEC|CONFIG_USE_AUDIO_PROCESSOR" sdkconfig
```

如果输出显示：
```
CONFIG_USE_AUDIO_PROCESSOR=y
CONFIG_USE_SERVER_AEC=y
```

就说明配置正确了。试试看能不能实现语音打断？

```plaintext
┌──────────────────────────────────────────────────────────────────────┐
│                          设备端                                       │
│  ┌─────────┐                                                          │
│  │ 麦克风  │──→ 音频数据 + 时间戳 ──────→ 发送到服务器               │
│  └─────────┘                                                          │
│                                                                       │
│  ┌─────────┐                                                          │
│  │ 扬声器  │←── 音频数据 + 时间戳 ←────── 从服务器接收               │
│  └─────────┘                                                          │
└──────────────────────────────────────────────────────────────────────┘
                              ↓ ↑
                         网络传输
                              ↓ ↑
┌──────────────────────────────────────────────────────────────────────┐
│                          服务器端                                     │
│                                                                       │
│   接收麦克风数据 ──→ [AEC 处理] ──→ 消除回声 ──→ ASR 语音识别        │
│         ↑                                                             │
│   利用时间戳对齐播放数据作为参考信号                                  │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

```shellscript
idf.py menuconfig
```

```plaintext
Xiaozhi Assistant
  └── [*] Enable Server-Side AEC (Unstable)    ← 按空格勾选
```

```plaintext
Xiaozhi Assistant
  └── [*] Enable Audio Noise Reduction    ← 需要开启（依赖 PSRAM）
```

```shellscript
# 保存配置后退出
idf.py build
idf.py flash
```

```cpp
// websocket_protocol.cc 或 mqtt_protocol.cc
#if CONFIG_USE_SERVER_AEC
    cJSON_AddBoolToObject(features, "aec", true);  // 告诉服务器：我支持 AEC
#endif
```

```cpp
// application.cc 构造函数
#if CONFIG_USE_SERVER_AEC
    aec_mode_ = kAecOnServerSide;
#endif
```

```cpp
case kDeviceStateSpeaking:
    if (listening_mode_ == kListeningModeRealtime) {
        // 不会关闭音频处理，保持监听
        // 用户随时可以打断
    }
```

```shellscript
grep -E "CONFIG_USE_SERVER_AEC|CONFIG_USE_AUDIO_PROCESSOR" sdkconfig
```

```plaintext
CONFIG_USE_AUDIO_PROCESSOR=y
CONFIG_USE_SERVER_AEC=y
```

---

**User**

我勾选上并保存了Enable Server-Side AEC (Unstable)和Enable Audio Noise Reduction的修改，接下来请你帮我修改代码以实现语音打断的功能

---

**Cursor**

好的，让我先查看当前代码的状态，确认需要修改哪些部分：

我分析了代码，发现问题所在：

**当前状态**：
- 启用服务端 AEC 后，`aec_mode_` = `kAecOnServerSide` ✅
- 在 Speaking 状态时，如果是实时模式，音频处理保持开启 ✅
- **但是**：VAD 变化事件只在 Listening 状态处理，**没有在 Speaking 状态触发打断** ❌

我来帮你修改代码，添加语音打断功能：

```cpp
#include "application.h"
#include "board.h"
#include "display.h"
#include "system_info.h"
#include "audio_codec.h"
#include "mqtt_protocol.h"
#include "websocket_protocol.h"
#include "assets/lang_config.h"
#include "mcp_server.h"
#include "assets.h"
#include "settings.h"

#include <cstring>
#include <esp_log.h>
#include <cJSON.h>
#include <driver/gpio.h>
#include <arpa/inet.h>
#include <font_awesome.h>

#define TAG "Application"


Application::Application() {
    event_group_ = xEventGroupCreate();

#if CONFIG_USE_DEVICE_AEC && CONFIG_USE_SERVER_AEC
#error "CONFIG_USE_DEVICE_AEC and CONFIG_USE_SERVER_AEC cannot be enabled at the same time"
#elif CONFIG_USE_DEVICE_AEC
    aec_mode_ = kAecOnDeviceSide;
#elif CONFIG_USE_SERVER_AEC
    aec_mode_ = kAecOnServerSide;
#else
    aec_mode_ = kAecOff;
#endif

    esp_timer_create_args_t clock_timer_args = {
        .callback = [](void* arg) {
            Application* app = (Application*)arg;
            xEventGroupSetBits(app->event_group_, MAIN_EVENT_CLOCK_TICK);
        },
        .arg = this,
        .dispatch_method = ESP_TIMER_TASK,
        .name = "clock_timer",
        .skip_unhandled_events = true
    };
    esp_timer_create(&clock_timer_args, &clock_timer_handle_);
}

Application::~Application() {
    if (clock_timer_handle_ != nullptr) {
        esp_timer_stop(clock_timer_handle_);
        esp_timer_delete(clock_timer_handle_);
    }
    vEventGroupDelete(event_group_);
}

bool Application::SetDeviceState(DeviceState state) {
    return state_machine_.TransitionTo(state);
}

void Application::Initialize() {
    auto& board = Board::GetInstance();
    SetDeviceState(kDeviceStateStarting);

    // Setup the display
    auto display = board.GetDisplay();

    // Print board name/version info
    display->SetChatMessage("system", SystemInfo::GetUserAgent().c_str());

    // Setup the audio service
    auto codec = board.GetAudioCodec();
    audio_service_.Initialize(codec);
    audio_service_.Start();

    AudioServiceCallbacks callbacks;
    callbacks.on_send_queue_available = [this]() {
        xEventGroupSetBits(event_group_, MAIN_EVENT_SEND_AUDIO);
    };
    callbacks.on_wake_word_detected = [this](const std::string& wake_word) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_WAKE_WORD_DETECTED);
    };
    callbacks.on_vad_change = [this](bool speaking) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_VAD_CHANGE);
    };
    audio_service_.SetCallbacks(callbacks);

    // Add state change listeners
    state_machine_.AddStateChangeListener([this](DeviceState old_state, DeviceState new_state) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_STATE_CHANGED);
    });

    // Start the clock timer to update the status bar
    esp_timer_start_periodic(clock_timer_handle_, 1000000);

    // Add MCP common tools (only once during initialization)
    auto& mcp_server = McpServer::GetInstance();
    mcp_server.AddCommonTools();
    mcp_server.AddUserOnlyTools();

    // Set network event callback for UI updates and network state handling
    board.SetNetworkEventCallback([this](NetworkEvent event, const std::string& data) {
        auto display = Board::GetInstance().GetDisplay();
        
        switch (event) {
            case NetworkEvent::Scanning:
                display->ShowNotification(Lang::Strings::SCANNING_WIFI, 30000);
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_DISCONNECTED);
                break;
            case NetworkEvent::Connecting: {
                if (data.empty()) {
                    // Cellular network - registering without carrier info yet
                    display->SetStatus(Lang::Strings::REGISTERING_NETWORK);
                } else {
                    // WiFi or cellular with carrier info
                    std::string msg = Lang::Strings::CONNECT_TO;
                    msg += data;
                    msg += "...";
                    display->ShowNotification(msg.c_str(), 30000);
                }
                break;
            }
            case NetworkEvent::Connected: {
                std::string msg = Lang::Strings::CONNECTED_TO;
                msg += data;
                display->ShowNotification(msg.c_str(), 30000);
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_CONNECTED);
                break;
            }
            case NetworkEvent::Disconnected:
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_DISCONNECTED);
                break;
            case NetworkEvent::WifiConfigModeEnter:
                // WiFi config mode enter is handled by WifiBoard internally
                break;
            case NetworkEvent::WifiConfigModeExit:
                // WiFi config mode exit is handled by WifiBoard internally
                break;
            // Cellular modem specific events
            case NetworkEvent::ModemDetecting:
                display->SetStatus(Lang::Strings::DETECTING_MODULE);
                break;
            case NetworkEvent::ModemErrorNoSim:
                Alert(Lang::Strings::ERROR, Lang::Strings::PIN_ERROR, "triangle_exclamation", Lang::Sounds::OGG_ERR_PIN);
                break;
            case NetworkEvent::ModemErrorRegDenied:
                Alert(Lang::Strings::ERROR, Lang::Strings::REG_ERROR, "triangle_exclamation", Lang::Sounds::OGG_ERR_REG);
                break;
            case NetworkEvent::ModemErrorInitFailed:
                display->SetStatus(Lang::Strings::DETECTING_MODULE);
                display->SetChatMessage("system", Lang::Strings::DETECTING_MODULE);
                break;
            case NetworkEvent::ModemErrorTimeout:
                display->SetStatus(Lang::Strings::REGISTERING_NETWORK);
                break;
        }
    });

    // Start network asynchronously
    board.StartNetwork();

    // Update the status bar immediately to show the network state
    display->UpdateStatusBar(true);
}

void Application::Run() {
    const EventBits_t ALL_EVENTS = 
        MAIN_EVENT_SCHEDULE |
        MAIN_EVENT_SEND_AUDIO |
        MAIN_EVENT_WAKE_WORD_DETECTED |
        MAIN_EVENT_VAD_CHANGE |
        MAIN_EVENT_CLOCK_TICK |
        MAIN_EVENT_ERROR |
        MAIN_EVENT_NETWORK_CONNECTED |
        MAIN_EVENT_NETWORK_DISCONNECTED |
        MAIN_EVENT_TOGGLE_CHAT |
        MAIN_EVENT_START_LISTENING |
        MAIN_EVENT_STOP_LISTENING |
        MAIN_EVENT_ACTIVATION_DONE |
        MAIN_EVENT_STATE_CHANGED;

    while (true) {
        auto bits = xEventGroupWaitBits(event_group_, ALL_EVENTS, pdTRUE, pdFALSE, portMAX_DELAY);

        if (bits & MAIN_EVENT_ERROR) {
            SetDeviceState(kDeviceStateIdle);
            Alert(Lang::Strings::ERROR, last_error_message_.c_str(), "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
        }

        if (bits & MAIN_EVENT_NETWORK_CONNECTED) {
            HandleNetworkConnectedEvent();
        }

        if (bits & MAIN_EVENT_NETWORK_DISCONNECTED) {
            HandleNetworkDisconnectedEvent();
        }

        if (bits & MAIN_EVENT_ACTIVATION_DONE) {
            HandleActivationDoneEvent();
        }

        if (bits & MAIN_EVENT_STATE_CHANGED) {
            HandleStateChangedEvent();
        }

        if (bits & MAIN_EVENT_TOGGLE_CHAT) {
            HandleToggleChatEvent();
        }

        if (bits & MAIN_EVENT_START_LISTENING) {
            HandleStartListeningEvent();
        }

        if (bits & MAIN_EVENT_STOP_LISTENING) {
            HandleStopListeningEvent();
        }

        if (bits & MAIN_EVENT_SEND_AUDIO) {
            while (auto packet = audio_service_.PopPacketFromSendQueue()) {
                if (protocol_ && !protocol_->SendAudio(std::move(packet))) {
                    break;
                }
            }
        }

        if (bits & MAIN_EVENT_WAKE_WORD_DETECTED) {
            HandleWakeWordDetectedEvent();
        }

        if (bits & MAIN_EVENT_VAD_CHANGE) {
            auto state = GetDeviceState();
            if (state == kDeviceStateListening) {
                auto led = Board::GetInstance().GetLed();
                led->OnStateChanged();
            } else if (state == kDeviceStateSpeaking && listening_mode_ == kListeningModeRealtime) {
                // 在实时模式下，如果 AI 正在说话时检测到用户说话，触发打断
                if (audio_service_.IsVoiceDetected()) {
                    ESP_LOGI(TAG, "Voice detected during speaking, aborting...");
                    AbortSpeaking(kAbortReasonNone);
                }
            }
        }

        if (bits & MAIN_EVENT_SCHEDULE) {
            std::unique_lock<std::mutex> lock(mutex_);
            auto tasks = std::move(main_tasks_);
            lock.unlock();
            for (auto& task : tasks) {
                task();
            }
        }

        if (bits & MAIN_EVENT_CLOCK_TICK) {
            clock_ticks_++;
            auto display = Board::GetInstance().GetDisplay();
            display->UpdateStatusBar();
        
            // Print debug info every 10 seconds
            if (clock_ticks_ % 10 == 0) {
                SystemInfo::PrintHeapStats();
            }
        }
    }
}

void Application::HandleNetworkConnectedEvent() {
    ESP_LOGI(TAG, "Network connected");
    auto state = GetDeviceState();

    if (state == kDeviceStateStarting || state == kDeviceStateWifiConfiguring) {
        // Network is ready, start activation
        SetDeviceState(kDeviceStateActivating);
        if (activation_task_handle_ != nullptr) {
            ESP_LOGW(TAG, "Activation task already running");
            return;
        }

        xTaskCreate([](void* arg) {
            Application* app = static_cast<Application*>(arg);
            app->ActivationTask();
            app->activation_task_handle_ = nullptr;
            vTaskDelete(NULL);
        }, "activation", 4096 * 2, this, 2, &activation_task_handle_);
    }

    // Update the status bar immediately to show the network state
    auto display = Board::GetInstance().GetDisplay();
    display->UpdateStatusBar(true);
}

void Application::HandleNetworkDisconnectedEvent() {
    // Close current conversation when network disconnected
    auto state = GetDeviceState();
    if (state == kDeviceStateConnecting || state == kDeviceStateListening || state == kDeviceStateSpeaking) {
        ESP_LOGI(TAG, "Closing audio channel due to network disconnection");
        protocol_->CloseAudioChannel();
    }

    // Update the status bar immediately to show the network state
    auto display = Board::GetInstance().GetDisplay();
    display->UpdateStatusBar(true);
}

void Application::HandleActivationDoneEvent() {
    ESP_LOGI(TAG, "Activation done");

    SystemInfo::PrintHeapStats();
    SetDeviceState(kDeviceStateIdle);

    has_server_time_ = ota_->HasServerTime();

    auto display = Board::GetInstance().GetDisplay();
    std::string message = std::string(Lang::Strings::VERSION) + ota_->GetCurrentVersion();
    display->ShowNotification(message.c_str());
    display->SetChatMessage("system", "");

    // Play the success sound to indicate the device is ready
    audio_service_.PlaySound(Lang::Sounds::OGG_SUCCESS);

    // Release OTA object after activation is complete
    ota_.reset();
    auto& board = Board::GetInstance();
    board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
}

void Application::ActivationTask() {
    // Create OTA object for activation process
    ota_ = std::make_unique<Ota>();

    // Check for new assets version
    CheckAssetsVersion();

    // Check for new firmware version
    CheckNewVersion();

    // Initialize the protocol
    InitializeProtocol();

    // Signal completion to main loop
    xEventGroupSetBits(event_group_, MAIN_EVENT_ACTIVATION_DONE);
}

void Application::CheckAssetsVersion() {
    // Only allow CheckAssetsVersion to be called once
    if (assets_version_checked_) {
        return;
    }
    assets_version_checked_ = true;

    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto& assets = Assets::GetInstance();

    if (!assets.partition_valid()) {
        ESP_LOGW(TAG, "Assets partition is disabled for board %s", BOARD_NAME);
        return;
    }
    
    Settings settings("assets", true);
    // Check if there is a new assets need to be downloaded
    std::string download_url = settings.GetString("download_url");

    if (!download_url.empty()) {
        settings.EraseKey("download_url");

        char message[256];
        snprintf(message, sizeof(message), Lang::Strings::FOUND_NEW_ASSETS, download_url.c_str());
        Alert(Lang::Strings::LOADING_ASSETS, message, "cloud_arrow_down", Lang::Sounds::OGG_UPGRADE);
        
        // Wait for the audio service to be idle for 3 seconds
        vTaskDelay(pdMS_TO_TICKS(3000));
        SetDeviceState(kDeviceStateUpgrading);
        board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
        display->SetChatMessage("system", Lang::Strings::PLEASE_WAIT);

        bool success = assets.Download(download_url, [display](int progress, size_t speed) -> void {
            std::thread([display, progress, speed]() {
                char buffer[32];
                snprintf(buffer, sizeof(buffer), "%d%% %uKB/s", progress, speed / 1024);
                display->SetChatMessage("system", buffer);
            }).detach();
        });

        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
        vTaskDelay(pdMS_TO_TICKS(1000));

        if (!success) {
            Alert(Lang::Strings::ERROR, Lang::Strings::DOWNLOAD_ASSETS_FAILED, "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
            vTaskDelay(pdMS_TO_TICKS(2000));
            SetDeviceState(kDeviceStateActivating);
            return;
        }
    }

    // Apply assets
    assets.Apply();
    display->SetChatMessage("system", "");
    display->SetEmotion("microchip_ai");
}

void Application::CheckNewVersion() {
    const int MAX_RETRY = 10;
    int retry_count = 0;
    int retry_delay = 10; // Initial retry delay in seconds

    auto& board = Board::GetInstance();
    while (true) {
        auto display = board.GetDisplay();
        display->SetStatus(Lang::Strings::CHECKING_NEW_VERSION);

        esp_err_t err = ota_->CheckVersion();
        if (err != ESP_OK) {
            retry_count++;
            if (retry_count >= MAX_RETRY) {
                ESP_LOGE(TAG, "Too many retries, exit version check");
                return;
            }

            char error_message[128];
            snprintf(error_message, sizeof(error_message), "code=%d, url=%s", err, ota_->GetCheckVersionUrl().c_str());
            char buffer[256];
            snprintf(buffer, sizeof(buffer), Lang::Strings::CHECK_NEW_VERSION_FAILED, retry_delay, error_message);
            Alert(Lang::Strings::ERROR, buffer, "cloud_slash", Lang::Sounds::OGG_EXCLAMATION);

            ESP_LOGW(TAG, "Check new version failed, retry in %d seconds (%d/%d)", retry_delay, retry_count, MAX_RETRY);
            for (int i = 0; i < retry_delay; i++) {
                vTaskDelay(pdMS_TO_TICKS(1000));
                if (GetDeviceState() == kDeviceStateIdle) {
                    break;
                }
            }
            retry_delay *= 2; // Double the retry delay
            continue;
        }
        retry_count = 0;
        retry_delay = 10; // Reset retry delay

        if (ota_->HasNewVersion()) {
            if (UpgradeFirmware(ota_->GetFirmwareUrl(), ota_->GetFirmwareVersion())) {
                return; // This line will never be reached after reboot
            }
            // If upgrade failed, continue to normal operation
        }

        // No new version, mark the current version as valid
        ota_->MarkCurrentVersionValid();
        if (!ota_->HasActivationCode() && !ota_->HasActivationChallenge()) {
            // Exit the loop if done checking new version
            break;
        }

        display->SetStatus(Lang::Strings::ACTIVATION);
        // Activation code is shown to the user and waiting for the user to input
        if (ota_->HasActivationCode()) {
            ShowActivationCode(ota_->GetActivationCode(), ota_->GetActivationMessage());
        }

        // This will block the loop until the activation is done or timeout
        for (int i = 0; i < 10; ++i) {
            ESP_LOGI(TAG, "Activating... %d/%d", i + 1, 10);
            esp_err_t err = ota_->Activate();
            if (err == ESP_OK) {
                break;
            } else if (err == ESP_ERR_TIMEOUT) {
                vTaskDelay(pdMS_TO_TICKS(3000));
            } else {
                vTaskDelay(pdMS_TO_TICKS(10000));
            }
            if (GetDeviceState() == kDeviceStateIdle) {
                break;
            }
        }
    }
}

void Application::InitializeProtocol() {
    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto codec = board.GetAudioCodec();

    display->SetStatus(Lang::Strings::LOADING_PROTOCOL);

    if (ota_->HasMqttConfig()) {
        protocol_ = std::make_unique<MqttProtocol>();
    } else if (ota_->HasWebsocketConfig()) {
        protocol_ = std::make_unique<WebsocketProtocol>();
    } else {
        ESP_LOGW(TAG, "No protocol specified in the OTA config, using MQTT");
        protocol_ = std::make_unique<MqttProtocol>();
    }

    protocol_->OnConnected([this]() {
        DismissAlert();
    });

    protocol_->OnNetworkError([this](const std::string& message) {
        last_error_message_ = message;
        xEventGroupSetBits(event_group_, MAIN_EVENT_ERROR);
    });
    
    protocol_->OnIncomingAudio([this](std::unique_ptr<AudioStreamPacket> packet) {
        if (GetDeviceState() == kDeviceStateSpeaking) {
            audio_service_.PushPacketToDecodeQueue(std::move(packet));
        }
    });
    
    protocol_->OnAudioChannelOpened([this, codec, &board]() {
        board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
        if (protocol_->server_sample_rate() != codec->output_sample_rate()) {
            ESP_LOGW(TAG, "Server sample rate %d does not match device output sample rate %d, resampling may cause distortion",
                protocol_->server_sample_rate(), codec->output_sample_rate());
        }
    });
    
    protocol_->OnAudioChannelClosed([this, &board]() {
        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
        Schedule([this]() {
            auto display = Board::GetInstance().GetDisplay();
            display->SetChatMessage("system", "");
            SetDeviceState(kDeviceStateIdle);
        });
    });
    
    protocol_->OnIncomingJson([this, display](const cJSON* root) {
        // Parse JSON data
        auto type = cJSON_GetObjectItem(root, "type");
        if (strcmp(type->valuestring, "tts") == 0) {
            auto state = cJSON_GetObjectItem(root, "state");
            if (strcmp(state->valuestring, "start") == 0) {
                Schedule([this]() {
                    aborted_ = false;
                    SetDeviceState(kDeviceStateSpeaking);
                });
            } else if (strcmp(state->valuestring, "stop") == 0) {
                Schedule([this]() {
                    if (GetDeviceState() == kDeviceStateSpeaking) {
                        if (listening_mode_ == kListeningModeManualStop) {
                            SetDeviceState(kDeviceStateIdle);
                        } else {
                            SetDeviceState(kDeviceStateListening);
                        }
                    }
                });
            } else if (strcmp(state->valuestring, "sentence_start") == 0) {
                auto text = cJSON_GetObjectItem(root, "text");
                if (cJSON_IsString(text)) {
                    ESP_LOGI(TAG, "<< %s", text->valuestring);
                    Schedule([this, display, message = std::string(text->valuestring)]() {
                        display->SetChatMessage("assistant", message.c_str());
                    });
                }
            }
        } else if (strcmp(type->valuestring, "stt") == 0) {
            auto text = cJSON_GetObjectItem(root, "text");
            if (cJSON_IsString(text)) {
                ESP_LOGI(TAG, ">> %s", text->valuestring);
                Schedule([this, display, message = std::string(text->valuestring)]() {
                    display->SetChatMessage("user", message.c_str());
                });
            }
        } else if (strcmp(type->valuestring, "llm") == 0) {
            auto emotion = cJSON_GetObjectItem(root, "emotion");
            if (cJSON_IsString(emotion)) {
                Schedule([this, display, emotion_str = std::string(emotion->valuestring)]() {
                    display->SetEmotion(emotion_str.c_str());
                });
            }
        } else if (strcmp(type->valuestring, "mcp") == 0) {
            auto payload = cJSON_GetObjectItem(root, "payload");
            if (cJSON_IsObject(payload)) {
                McpServer::GetInstance().ParseMessage(payload);
            }
        } else if (strcmp(type->valuestring, "system") == 0) {
            auto command = cJSON_GetObjectItem(root, "command");
            if (cJSON_IsString(command)) {
                ESP_LOGI(TAG, "System command: %s", command->valuestring);
                if (strcmp(command->valuestring, "reboot") == 0) {
                    // Do a reboot if user requests a OTA update
                    Schedule([this]() {
                        Reboot();
                    });
                } else {
                    ESP_LOGW(TAG, "Unknown system command: %s", command->valuestring);
                }
            }
        } else if (strcmp(type->valuestring, "alert") == 0) {
            auto status = cJSON_GetObjectItem(root, "status");
            auto message = cJSON_GetObjectItem(root, "message");
            auto emotion = cJSON_GetObjectItem(root, "emotion");
            if (cJSON_IsString(status) && cJSON_IsString(message) && cJSON_IsString(emotion)) {
                Alert(status->valuestring, message->valuestring, emotion->valuestring, Lang::Sounds::OGG_VIBRATION);
            } else {
                ESP_LOGW(TAG, "Alert command requires status, message and emotion");
            }
#if CONFIG_RECEIVE_CUSTOM_MESSAGE
        } else if (strcmp(type->valuestring, "custom") == 0) {
            auto payload = cJSON_GetObjectItem(root, "payload");
            ESP_LOGI(TAG, "Received custom message: %s", cJSON_PrintUnformatted(root));
            if (cJSON_IsObject(payload)) {
                Schedule([this, display, payload_str = std::string(cJSON_PrintUnformatted(payload))]() {
                    display->SetChatMessage("system", payload_str.c_str());
                });
            } else {
                ESP_LOGW(TAG, "Invalid custom message format: missing payload");
            }
#endif
        } else {
            ESP_LOGW(TAG, "Unknown message type: %s", type->valuestring);
        }
    });
    
    protocol_->Start();
}

void Application::ShowActivationCode(const std::string& code, const std::string& message) {
    struct digit_sound {
        char digit;
        const std::string_view& sound;
    };
    static const std::array<digit_sound, 10> digit_sounds{{
        digit_sound{'0', Lang::Sounds::OGG_0},
        digit_sound{'1', Lang::Sounds::OGG_1}, 
        digit_sound{'2', Lang::Sounds::OGG_2},
        digit_sound{'3', Lang::Sounds::OGG_3},
        digit_sound{'4', Lang::Sounds::OGG_4},
        digit_sound{'5', Lang::Sounds::OGG_5},
        digit_sound{'6', Lang::Sounds::OGG_6},
        digit_sound{'7', Lang::Sounds::OGG_7},
        digit_sound{'8', Lang::Sounds::OGG_8},
        digit_sound{'9', Lang::Sounds::OGG_9}
    }};

    // This sentence uses 9KB of SRAM, so we need to wait for it to finish
    Alert(Lang::Strings::ACTIVATION, message.c_str(), "link", Lang::Sounds::OGG_ACTIVATION);

    for (const auto& digit : code) {
        auto it = std::find_if(digit_sounds.begin(), digit_sounds.end(),
            [digit](const digit_sound& ds) { return ds.digit == digit; });
        if (it != digit_sounds.end()) {
            audio_service_.PlaySound(it->sound);
        }
    }
}

void Application::Alert(const char* status, const char* message, const char* emotion, const std::string_view& sound) {
    ESP_LOGW(TAG, "Alert [%s] %s: %s", emotion, status, message);
    auto display = Board::GetInstance().GetDisplay();
    display->SetStatus(status);
    display->SetEmotion(emotion);
    display->SetChatMessage("system", message);
    if (!sound.empty()) {
        audio_service_.PlaySound(sound);
    }
}

void Application::DismissAlert() {
    if (GetDeviceState() == kDeviceStateIdle) {
        auto display = Board::GetInstance().GetDisplay();
        display->SetStatus(Lang::Strings::STANDBY);
        display->SetEmotion("neutral");
        display->SetChatMessage("system", "");
    }
}

void Application::ToggleChatState() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_TOGGLE_CHAT);
}

void Application::StartListening() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_START_LISTENING);
}

void Application::StopListening() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_STOP_LISTENING);
}

void Application::HandleToggleChatEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateActivating) {
        SetDeviceState(kDeviceStateIdle);
        return;
    } else if (state == kDeviceStateWifiConfiguring) {
        audio_service_.EnableAudioTesting(true);
        SetDeviceState(kDeviceStateAudioTesting);
        return;
    } else if (state == kDeviceStateAudioTesting) {
        audio_service_.EnableAudioTesting(false);
        SetDeviceState(kDeviceStateWifiConfiguring);
        return;
    }

    if (!protocol_) {
        ESP_LOGE(TAG, "Protocol not initialized");
        return;
    }

    if (state == kDeviceStateIdle) {
        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                return;
            }
        }

        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonNone);
    } else if (state == kDeviceStateListening) {
        protocol_->CloseAudioChannel();
    }
}

void Application::HandleStartListeningEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateActivating) {
        SetDeviceState(kDeviceStateIdle);
        return;
    } else if (state == kDeviceStateWifiConfiguring) {
        audio_service_.EnableAudioTesting(true);
        SetDeviceState(kDeviceStateAudioTesting);
        return;
    }

    if (!protocol_) {
        ESP_LOGE(TAG, "Protocol not initialized");
        return;
    }
    
    if (state == kDeviceStateIdle) {
        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                return;
            }
        }

        SetListeningMode(kListeningModeManualStop);
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonNone);
        SetListeningMode(kListeningModeManualStop);
    }
}

void Application::HandleStopListeningEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateAudioTesting) {
        audio_service_.EnableAudioTesting(false);
        SetDeviceState(kDeviceStateWifiConfiguring);
        return;
    } else if (state == kDeviceStateListening) {
        if (protocol_) {
            protocol_->SendStopListening();
        }
        SetDeviceState(kDeviceStateIdle);
    }
}

void Application::HandleWakeWordDetectedEvent() {
    if (!protocol_) {
        return;
    }

    auto state = GetDeviceState();
    
    if (state == kDeviceStateIdle) {
        audio_service_.EncodeWakeWord();

        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                audio_service_.EnableWakeWordDetection(true);
                return;
            }
        }

        auto wake_word = audio_service_.GetLastWakeWord();
        ESP_LOGI(TAG, "Wake word detected: %s", wake_word.c_str());
#if CONFIG_SEND_WAKE_WORD_DATA
        // Encode and send the wake word data to the server
        while (auto packet = audio_service_.PopWakeWordPacket()) {
            protocol_->SendAudio(std::move(packet));
        }
        // Set the chat state to wake word detected
        protocol_->SendWakeWordDetected(wake_word);
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#else
        // Set flag to play popup sound after state changes to listening
        // (PlaySound here would be cleared by ResetDecoder in EnableVoiceProcessing)
        play_popup_on_listening_ = true;
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#endif
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonWakeWordDetected);
    } else if (state == kDeviceStateActivating) {
        // Restart the activation check if the wake word is detected during activation
        SetDeviceState(kDeviceStateIdle);
    }
}

void Application::HandleStateChangedEvent() {
    DeviceState new_state = state_machine_.GetState();
    clock_ticks_ = 0;

    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto led = board.GetLed();
    led->OnStateChanged();
    
    switch (new_state) {
        case kDeviceStateUnknown:
        case kDeviceStateIdle:
            display->SetStatus(Lang::Strings::STANDBY);
            display->SetEmotion("neutral");
            audio_service_.EnableVoiceProcessing(false);
            audio_service_.EnableWakeWordDetection(true);
            break;
        case kDeviceStateConnecting:
            display->SetStatus(Lang::Strings::CONNECTING);
            display->SetEmotion("neutral");
            display->SetChatMessage("system", "");
            break;
        case kDeviceStateListening:
            display->SetStatus(Lang::Strings::LISTENING);
            display->SetEmotion("neutral");

            // Make sure the audio processor is running
            if (!audio_service_.IsAudioProcessorRunning()) {
                // Send the start listening command
                protocol_->SendStartListening(listening_mode_);
                audio_service_.EnableVoiceProcessing(true);
                audio_service_.EnableWakeWordDetection(false);
            }

            // Play popup sound after ResetDecoder (in EnableVoiceProcessing) has been called
            if (play_popup_on_listening_) {
                play_popup_on_listening_ = false;
                audio_service_.PlaySound(Lang::Sounds::OGG_POPUP);
            }
            break;
        case kDeviceStateSpeaking:
            display->SetStatus(Lang::Strings::SPEAKING);

            if (listening_mode_ != kListeningModeRealtime) {
                audio_service_.EnableVoiceProcessing(false);
                // Only AFE wake word can be detected in speaking mode
                audio_service_.EnableWakeWordDetection(audio_service_.IsAfeWakeWord());
            }
            audio_service_.ResetDecoder();
            break;
        case kDeviceStateWifiConfiguring:
            audio_service_.EnableVoiceProcessing(false);
            audio_service_.EnableWakeWordDetection(false);
            break;
        default:
            // Do nothing
            break;
    }
}

void Application::Schedule(std::function<void()>&& callback) {
    {
        std::lock_guard<std::mutex> lock(mutex_);
        main_tasks_.push_back(std::move(callback));
    }
    xEventGroupSetBits(event_group_, MAIN_EVENT_SCHEDULE);
}

void Application::AbortSpeaking(AbortReason reason) {
    ESP_LOGI(TAG, "Abort speaking");
    aborted_ = true;
    if (protocol_) {
        protocol_->SendAbortSpeaking(reason);
    }
}

void Application::SetListeningMode(ListeningMode mode) {
    listening_mode_ = mode;
    SetDeviceState(kDeviceStateListening);
}

void Application::Reboot() {
    ESP_LOGI(TAG, "Rebooting...");
    // Disconnect the audio channel
    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        protocol_->CloseAudioChannel();
    }
    protocol_.reset();
    audio_service_.Stop();

    vTaskDelay(pdMS_TO_TICKS(1000));
    esp_restart();
}

bool Application::UpgradeFirmware(const std::string& url, const std::string& version) {
    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();

    std::string upgrade_url = url;
    std::string version_info = version.empty() ? "(Manual upgrade)" : version;

    // Close audio channel if it's open
    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        ESP_LOGI(TAG, "Closing audio channel before firmware upgrade");
        protocol_->CloseAudioChannel();
    }
    ESP_LOGI(TAG, "Starting firmware upgrade from URL: %s", upgrade_url.c_str());

    Alert(Lang::Strings::OTA_UPGRADE, Lang::Strings::UPGRADING, "download", Lang::Sounds::OGG_UPGRADE);
    vTaskDelay(pdMS_TO_TICKS(3000));

    SetDeviceState(kDeviceStateUpgrading);

    std::string message = std::string(Lang::Strings::NEW_VERSION) + version_info;
    display->SetChatMessage("system", message.c_str());

    board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
    audio_service_.Stop();
    vTaskDelay(pdMS_TO_TICKS(1000));

    bool upgrade_success = Ota::Upgrade(upgrade_url, [display](int progress, size_t speed) {
        std::thread([display, progress, speed]() {
            char buffer[32];
            snprintf(buffer, sizeof(buffer), "%d%% %uKB/s", progress, speed / 1024);
            display->SetChatMessage("system", buffer);
        }).detach();
    });

    if (!upgrade_success) {
        // Upgrade failed, restart audio service and continue running
        ESP_LOGE(TAG, "Firmware upgrade failed, restarting audio service and continuing operation...");
        audio_service_.Start(); // Restart audio service
        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER); // Restore power save level
        Alert(Lang::Strings::ERROR, Lang::Strings::UPGRADE_FAILED, "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
        vTaskDelay(pdMS_TO_TICKS(3000));
        return false;
    } else {
        // Upgrade success, reboot immediately
        ESP_LOGI(TAG, "Firmware upgrade successful, rebooting...");
        display->SetChatMessage("system", "Upgrade successful, rebooting...");
        vTaskDelay(pdMS_TO_TICKS(1000)); // Brief pause to show message
        Reboot();
        return true;
    }
}

void Application::WakeWordInvoke(const std::string& wake_word) {
    if (!protocol_) {
        return;
    }

    auto state = GetDeviceState();
    
    if (state == kDeviceStateIdle) {
        audio_service_.EncodeWakeWord();

        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                audio_service_.EnableWakeWordDetection(true);
                return;
            }
        }

        ESP_LOGI(TAG, "Wake word detected: %s", wake_word.c_str());
#if CONFIG_USE_AFE_WAKE_WORD || CONFIG_USE_CUSTOM_WAKE_WORD
        // Encode and send the wake word data to the server
        while (auto packet = audio_service_.PopWakeWordPacket()) {
            protocol_->SendAudio(std::move(packet));
        }
        // Set the chat state to wake word detected
        protocol_->SendWakeWordDetected(wake_word);
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#else
        // Set flag to play popup sound after state changes to listening
        // (PlaySound here would be cleared by ResetDecoder in EnableVoiceProcessing)
        play_popup_on_listening_ = true;
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#endif
    } else if (state == kDeviceStateSpeaking) {
        Schedule([this]() {
            AbortSpeaking(kAbortReasonNone);
        });
    } else if (state == kDeviceStateListening) {   
        Schedule([this]() {
            if (protocol_) {
                protocol_->CloseAudioChannel();
            }
        });
    }
}

bool Application::CanEnterSleepMode() {
    if (GetDeviceState() != kDeviceStateIdle) {
        return false;
    }

    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        return false;
    }

    if (!audio_service_.IsIdle()) {
        return false;
    }

    // Now it is safe to enter sleep mode
    return true;
}

void Application::SendMcpMessage(const std::string& payload) {
    // Always schedule to run in main task for thread safety
    Schedule([this, payload = std::move(payload)]() {
        if (protocol_) {
            protocol_->SendMcpMessage(payload);
        }
    });
}

void Application::SetAecMode(AecMode mode) {
    aec_mode_ = mode;
    Schedule([this]() {
        auto& board = Board::GetInstance();
        auto display = board.GetDisplay();
        switch (aec_mode_) {
        case kAecOff:
            audio_service_.EnableDeviceAec(false);
            display->ShowNotification(Lang::Strings::RTC_MODE_OFF);
            break;
        case kAecOnServerSide:
            audio_service_.EnableDeviceAec(false);
            display->ShowNotification(Lang::Strings::RTC_MODE_ON);
            break;
        case kAecOnDeviceSide:
            audio_service_.EnableDeviceAec(true);
            display->ShowNotification(Lang::Strings::RTC_MODE_ON);
            break;
        }

        // If the AEC mode is changed, close the audio channel
        if (protocol_ && protocol_->IsAudioChannelOpened()) {
            protocol_->CloseAudioChannel();
        }
    });
}

void Application::PlaySound(const std::string_view& sound) {
    audio_service_.PlaySound(sound);
}

void Application::ResetProtocol() {
    Schedule([this]() {
        // Close audio channel if opened
        if (protocol_ && protocol_->IsAudioChannelOpened()) {
            protocol_->CloseAudioChannel();
        }
        // Reset protocol
        protocol_.reset();
    });
}


```

修改成功！但我需要再检查一下，确保在 Speaking 状态下音频处理器确实保持运行。让我查看完整的状态变化处理：

代码逻辑看起来正确。但是让我再确认一下打断后的流程是否完整——打断后应该自动进入监听状态：

让我再检查一下 `AbortSpeaking` 的实现，确保流程正确：

很好！代码修改完成。整个流程是这样的：

---

## 修改总结

我在 `application.cc` 中添加了语音打断逻辑：

```230:242:main/application.cc
        if (bits & MAIN_EVENT_VAD_CHANGE) {
            auto state = GetDeviceState();
            if (state == kDeviceStateListening) {
                auto led = Board::GetInstance().GetLed();
                led->OnStateChanged();
            } else if (state == kDeviceStateSpeaking && listening_mode_ == kListeningModeRealtime) {
                // 在实时模式下，如果 AI 正在说话时检测到用户说话，触发打断
                if (audio_service_.IsVoiceDetected()) {
                    ESP_LOGI(TAG, "Voice detected during speaking, aborting...");
                    AbortSpeaking(kAbortReasonNone);
                }
            }
        }
```

---

## 工作流程

```
┌─────────────────────────────────────────────────────────────────────┐
│  1. 用户唤醒设备（"你好小圆"）                                      │
│     ↓                                                               │
│  2. 设备进入 Listening 状态，使用 kListeningModeRealtime 模式       │
│     ↓                                                               │
│  3. 用户说话 → 发送到服务器 → AI 回复                               │
│     ↓                                                               │
│  4. 设备进入 Speaking 状态，但音频处理保持开启（因为是实时模式）   │
│     ↓                                                               │
│  5. 如果用户在 AI 说话时说话：                                      │
│     - VAD 检测到语音活动                                            │
│     - 触发 MAIN_EVENT_VAD_CHANGE                                    │
│     - 检测到 IsVoiceDetected() == true                              │
│     - 调用 AbortSpeaking() 发送打断消息给服务器                     │
│     ↓                                                               │
│  6. 服务器停止 TTS，发送 "tts stop"                                 │
│     ↓                                                               │
│  7. 设备进入 Listening 状态，继续听用户说话                         │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 编译和测试

现在编译并烧录：

```bash
cd /home/lei/xiaozhi-esp32
idf.py build
idf.py flash monitor
```

### 测试步骤：

1. **唤醒设备**：说 "你好小圆"
2. **提问**：说一句话，等待 AI 开始回复
3. **打断测试**：在 AI 说话过程中，**随便说点什么**（不需要唤醒词）
4. **观察日志**：如果打断成功，串口会显示：
   ```
   I (xxxxx) Application: Voice detected during speaking, aborting...
   I (xxxxx) Application: Abort speaking
   ```
5. AI 应该停止说话，设备进入监听状态等待你继续说

---

## 如果效果不好

如果遇到问题（比如容易误触发或不灵敏），可能需要调整：

1. **VAD 灵敏度**：可以在 AFE 配置中调整 `vad_mode` 或 `vad_min_noise_ms`
2. **添加防抖动**：避免短暂的噪声触发打断

需要我帮你进一步优化吗？先试试看效果怎么样！

```plaintext
        if (bits & MAIN_EVENT_VAD_CHANGE) {
            auto state = GetDeviceState();
            if (state == kDeviceStateListening) {
                auto led = Board::GetInstance().GetLed();
                led->OnStateChanged();
            } else if (state == kDeviceStateSpeaking && listening_mode_ == kListeningModeRealtime) {
                // 在实时模式下，如果 AI 正在说话时检测到用户说话，触发打断
                if (audio_service_.IsVoiceDetected()) {
                    ESP_LOGI(TAG, "Voice detected during speaking, aborting...");
                    AbortSpeaking(kAbortReasonNone);
                }
            }
        }
```

```plaintext
┌─────────────────────────────────────────────────────────────────────┐
│  1. 用户唤醒设备（"你好小圆"）                                      │
│     ↓                                                               │
│  2. 设备进入 Listening 状态，使用 kListeningModeRealtime 模式       │
│     ↓                                                               │
│  3. 用户说话 → 发送到服务器 → AI 回复                               │
│     ↓                                                               │
│  4. 设备进入 Speaking 状态，但音频处理保持开启（因为是实时模式）   │
│     ↓                                                               │
│  5. 如果用户在 AI 说话时说话：                                      │
│     - VAD 检测到语音活动                                            │
│     - 触发 MAIN_EVENT_VAD_CHANGE                                    │
│     - 检测到 IsVoiceDetected() == true                              │
│     - 调用 AbortSpeaking() 发送打断消息给服务器                     │
│     ↓                                                               │
│  6. 服务器停止 TTS，发送 "tts stop"                                 │
│     ↓                                                               │
│  7. 设备进入 Listening 状态，继续听用户说话                         │
└─────────────────────────────────────────────────────────────────────┘
```

```shellscript
cd /home/lei/xiaozhi-esp32
idf.py build
idf.py flash monitor
```

```plaintext
   I (xxxxx) Application: Voice detected during speaking, aborting...
   I (xxxxx) Application: Abort speaking
```

---

**User**

为什么是VAD，我们不是要用AEC吗

---

**Cursor**

很好的问题！让我解释清楚 **AEC 和 VAD 的关系**：

---

## AEC 和 VAD 是配合工作的

它们**不是替代关系**，而是**协作关系**：

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        音频处理流程                                      │
│                                                                          │
│   麦克风信号 ───┐                                                        │
│  （含 AI 回声）  │                                                        │
│                 ↓                                                        │
│            ┌─────────┐      ┌─────────┐      ┌─────────┐                │
│            │   AEC   │ ───→ │   VAD   │ ───→ │  打断   │                │
│            │ 回声消除 │      │语音检测 │      │  决策   │                │
│            └─────────┘      └─────────┘      └─────────┘                │
│                 ↑                                                        │
│   扬声器信号 ───┘                                                        │
│  （参考信号）                                                             │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 各自的作用

| 组件 | 作用 | 
|------|------|
| **AEC** | 从麦克风信号中**消除扬声器的回声**，得到"纯净的用户声音" |
| **VAD** | 在纯净信号上**检测是否有人说话** |
| **打断逻辑** | 根据 VAD 结果**决定是否打断** AI |

---

## 为什么两者都需要？

### 没有 AEC 的情况

```
AI 说话 → 扬声器播放 → 麦克风拾取 → VAD 检测到"有声音"
                                            ↓
                                     误判为用户说话
                                            ↓
                                       错误打断！
```

### 有 AEC 的情况

```
AI 说话 → 扬声器播放 → 麦克风拾取 ─┐
                                   ↓
          扬声器信号（参考）────→ AEC 处理 → 消除 AI 的声音
                                            ↓
                                     纯净信号（只有用户声音）
                                            ↓
                                      VAD 检测
                                            ↓
                              用户说话？→ Yes → 打断
                                        → No  → 不打断
```

---

## 在代码中的体现

### AEC 处理在 AFE 音频处理器中

```59:65:main/audio/processors/afe_audio_processor.cc
#ifdef CONFIG_USE_DEVICE_AEC
    afe_config->aec_init = true;   // 启用 AEC
    afe_config->vad_init = false;
#else
    afe_config->aec_init = false;
    afe_config->vad_init = true;   // 启用 VAD
#endif
```

对于服务端 AEC：
- 设备把麦克风数据 + 时间戳发给服务器
- **服务器执行 AEC 处理**
- 但设备端仍然需要 VAD 来检测用户是否说话

### VAD 检测触发回调

```141:150:main/audio/processors/afe_audio_processor.cc
        // VAD state change
        if (vad_state_change_callback_) {
            if (res->vad_state == VAD_SPEECH && !is_speaking_) {
                is_speaking_ = true;
                vad_state_change_callback_(true);   // 检测到开始说话
            } else if (res->vad_state == VAD_SILENCE && is_speaking_) {
                is_speaking_ = false;
                vad_state_change_callback_(false);  // 检测到停止说话
            }
        }
```

### 我添加的打断逻辑使用 VAD 结果

```235:241:main/application.cc
            } else if (state == kDeviceStateSpeaking && listening_mode_ == kListeningModeRealtime) {
                // 在实时模式下，如果 AI 正在说话时检测到用户说话，触发打断
                if (audio_service_.IsVoiceDetected()) {  // ← 这是 VAD 的结果
                    ESP_LOGI(TAG, "Voice detected during speaking, aborting...");
                    AbortSpeaking(kAbortReasonNone);
                }
            }
```

---

## 总结

```
AEC = 清理工具（去除 AI 的回声，让信号变干净）
VAD = 判断工具（判断用户是否在说话）
打断 = 动作（根据 VAD 判断结果执行）

完整流程：AEC 清理 → VAD 判断 → 执行打断
```

**你启用的服务端 AEC**：
- 服务器负责 AEC 处理（消除回声）
- 设备端的 VAD 在"相对干净"的信号上检测用户语音
- 我添加的代码根据 VAD 结果触发打断

这样理解了吗？现在可以编译测试了！

```plaintext
┌─────────────────────────────────────────────────────────────────────────┐
│                        音频处理流程                                      │
│                                                                          │
│   麦克风信号 ───┐                                                        │
│  （含 AI 回声）  │                                                        │
│                 ↓                                                        │
│            ┌─────────┐      ┌─────────┐      ┌─────────┐                │
│            │   AEC   │ ───→ │   VAD   │ ───→ │  打断   │                │
│            │ 回声消除 │      │语音检测 │      │  决策   │                │
│            └─────────┘      └─────────┘      └─────────┘                │
│                 ↑                                                        │
│   扬声器信号 ───┘                                                        │
│  （参考信号）                                                             │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

```plaintext
AI 说话 → 扬声器播放 → 麦克风拾取 → VAD 检测到"有声音"
                                            ↓
                                     误判为用户说话
                                            ↓
                                       错误打断！
```

```plaintext
AI 说话 → 扬声器播放 → 麦克风拾取 ─┐
                                   ↓
          扬声器信号（参考）────→ AEC 处理 → 消除 AI 的声音
                                            ↓
                                     纯净信号（只有用户声音）
                                            ↓
                                      VAD 检测
                                            ↓
                              用户说话？→ Yes → 打断
                                        → No  → 不打断
```

```plaintext
#ifdef CONFIG_USE_DEVICE_AEC
    afe_config->aec_init = true;   // 启用 AEC
    afe_config->vad_init = false;
#else
    afe_config->aec_init = false;
    afe_config->vad_init = true;   // 启用 VAD
#endif
```

```plaintext
        // VAD state change
        if (vad_state_change_callback_) {
            if (res->vad_state == VAD_SPEECH && !is_speaking_) {
                is_speaking_ = true;
                vad_state_change_callback_(true);   // 检测到开始说话
            } else if (res->vad_state == VAD_SILENCE && is_speaking_) {
                is_speaking_ = false;
                vad_state_change_callback_(false);  // 检测到停止说话
            }
        }
```

```plaintext
            } else if (state == kDeviceStateSpeaking && listening_mode_ == kListeningModeRealtime) {
                // 在实时模式下，如果 AI 正在说话时检测到用户说话，触发打断
                if (audio_service_.IsVoiceDetected()) {  // ← 这是 VAD 的结果
                    ESP_LOGI(TAG, "Voice detected during speaking, aborting...");
                    AbortSpeaking(kAbortReasonNone);
                }
            }
```

```plaintext
AEC = 清理工具（去除 AI 的回声，让信号变干净）
VAD = 判断工具（判断用户是否在说话）
打断 = 动作（根据 VAD 判断结果执行）

完整流程：AEC 清理 → VAD 判断 → 执行打断
```

---

**User**

好的，既然你代码修改好了，那需要我修改硬件方面吗

---

**Cursor**

好问题！让我分析一下你的情况：

## 服务端 AEC 的硬件要求

**服务端 AEC 不需要额外的硬件修改**。因为：

- 服务端 AEC 是在**服务器上**执行回声消除
- 设备只需要发送麦克风数据 + 时间戳
- 不需要硬件回采电路

---

## ⚠️ 但是有一个问题！

我刚才意识到一个重要的问题：

```
服务端 AEC 模式下的实际流程：

设备端：
  麦克风信号（含回声）→ VAD 检测 → 可能被回声误触发！
                        ↓
              发送到服务器 + 时间戳

服务器端：
  接收数据 → AEC 处理 → 纯净信号 → ASR 识别
```

**问题所在**：
- 服务端 AEC 是为了让**服务器能正确识别用户语音**
- 但设备端的 **VAD 检测仍然是在未经 AEC 处理的信号上运行的**
- 这意味着扬声器的回声**仍然可能触发 VAD**，导致误打断！

---

## 解决方案

### 方案 1：测试一下效果

先编译烧录试试看：

```bash
idf.py build flash monitor
```

有些情况下，如果：
- 扬声器音量不是很大
- 麦克风和扬声器有一定距离
- AFE 的降噪（NS）功能可以过滤部分回声

可能效果还可以。

### 方案 2：如果误触发严重，添加防抖动/阈值判断

我可以修改代码，添加一些保护措施：
- 只在持续检测到语音一段时间后才触发打断
- 或者检测音量是否超过一定阈值

### 方案 3：硬件升级（最佳效果）

如果效果不好，可以考虑升级到支持设备端 AEC 的开发板，比如：
- **立创实战派 ESP32-S3**
- **ESP-BOX-3**
- **微雪 ESP32-S3-Touch-AMOLED-1.8** 等

这些板子有专门的音频编解码器芯片（如 ES8311），支持硬件回采。

---

## 建议

**先试试当前的配置**，看看效果如何：

```bash
cd /home/lei/xiaozhi-esp32
idf.py build flash monitor
```

测试时注意观察：
1. AI 说话时，是否会自动打断（误触发）
2. 你说话时，是否能成功打断

如果误触发严重，我可以帮你添加防抖动逻辑或音量阈值判断。你想先测试一下吗？

```plaintext
服务端 AEC 模式下的实际流程：

设备端：
  麦克风信号（含回声）→ VAD 检测 → 可能被回声误触发！
                        ↓
              发送到服务器 + 时间戳

服务器端：
  接收数据 → AEC 处理 → 纯净信号 → ASR 识别
```

```shellscript
idf.py build flash monitor
```

```shellscript
cd /home/lei/xiaozhi-esp32
idf.py build flash monitor
```

---

**User**

好的我尝试了，现在的情况是ai语音一回复就会触发打断

---

**Cursor**

这正是我担心的问题！VAD 把扬声器的声音当作用户说话了。让我添加**防抖动保护**来解决这个问题：

```cpp
#ifndef _APPLICATION_H_
#define _APPLICATION_H_

#include <freertos/FreeRTOS.h>
#include <freertos/event_groups.h>
#include <freertos/task.h>
#include <esp_timer.h>

#include <string>
#include <mutex>
#include <deque>
#include <memory>

#include "protocol.h"
#include "ota.h"
#include "audio_service.h"
#include "device_state.h"
#include "device_state_machine.h"

// Main event bits
#define MAIN_EVENT_SCHEDULE             (1 << 0)
#define MAIN_EVENT_SEND_AUDIO           (1 << 1)
#define MAIN_EVENT_WAKE_WORD_DETECTED   (1 << 2)
#define MAIN_EVENT_VAD_CHANGE           (1 << 3)
#define MAIN_EVENT_ERROR                (1 << 4)
#define MAIN_EVENT_ACTIVATION_DONE      (1 << 5)
#define MAIN_EVENT_CLOCK_TICK           (1 << 6)
#define MAIN_EVENT_NETWORK_CONNECTED    (1 << 7)
#define MAIN_EVENT_NETWORK_DISCONNECTED (1 << 8)
#define MAIN_EVENT_TOGGLE_CHAT          (1 << 9)
#define MAIN_EVENT_START_LISTENING      (1 << 10)
#define MAIN_EVENT_STOP_LISTENING       (1 << 11)
#define MAIN_EVENT_STATE_CHANGED        (1 << 12)


enum AecMode {
    kAecOff,
    kAecOnDeviceSide,
    kAecOnServerSide,
};

class Application {
public:
    static Application& GetInstance() {
        static Application instance;
        return instance;
    }
    // Delete copy constructor and assignment operator
    Application(const Application&) = delete;
    Application& operator=(const Application&) = delete;

    /**
     * Initialize the application
     * This sets up display, audio, network callbacks, etc.
     * Network connection starts asynchronously.
     */
    void Initialize();

    /**
     * Run the main event loop
     * This function runs in the main task and never returns.
     * It handles all events including network, state changes, and user interactions.
     */
    void Run();

    DeviceState GetDeviceState() const { return state_machine_.GetState(); }
    bool IsVoiceDetected() const { return audio_service_.IsVoiceDetected(); }
    
    /**
     * Request state transition
     * Returns true if transition was successful
     */
    bool SetDeviceState(DeviceState state);

    /**
     * Schedule a callback to be executed in the main task
     */
    void Schedule(std::function<void()>&& callback);

    /**
     * Alert with status, message, emotion and optional sound
     */
    void Alert(const char* status, const char* message, const char* emotion = "", const std::string_view& sound = "");
    void DismissAlert();

    void AbortSpeaking(AbortReason reason);

    /**
     * Toggle chat state (event-based, thread-safe)
     * Sends MAIN_EVENT_TOGGLE_CHAT to be handled in Run()
     */
    void ToggleChatState();

    /**
     * Start listening (event-based, thread-safe)
     * Sends MAIN_EVENT_START_LISTENING to be handled in Run()
     */
    void StartListening();

    /**
     * Stop listening (event-based, thread-safe)
     * Sends MAIN_EVENT_STOP_LISTENING to be handled in Run()
     */
    void StopListening();

    void Reboot();
    void WakeWordInvoke(const std::string& wake_word);
    bool UpgradeFirmware(const std::string& url, const std::string& version = "");
    bool CanEnterSleepMode();
    void SendMcpMessage(const std::string& payload);
    void SetAecMode(AecMode mode);
    AecMode GetAecMode() const { return aec_mode_; }
    void PlaySound(const std::string_view& sound);
    AudioService& GetAudioService() { return audio_service_; }
    
    /**
     * Reset protocol resources (thread-safe)
     * Can be called from any task to release resources allocated after network connected
     * This includes closing audio channel, resetting protocol and ota objects
     */
    void ResetProtocol();

private:
    Application();
    ~Application();

    std::mutex mutex_;
    std::deque<std::function<void()>> main_tasks_;
    std::unique_ptr<Protocol> protocol_;
    EventGroupHandle_t event_group_ = nullptr;
    esp_timer_handle_t clock_timer_handle_ = nullptr;
    DeviceStateMachine state_machine_;
    ListeningMode listening_mode_ = kListeningModeAutoStop;
    AecMode aec_mode_ = kAecOff;
    std::string last_error_message_;
    AudioService audio_service_;
    std::unique_ptr<Ota> ota_;

    bool has_server_time_ = false;
    bool aborted_ = false;
    bool assets_version_checked_ = false;
    bool play_popup_on_listening_ = false;  // Flag to play popup sound after state changes to listening
    int clock_ticks_ = 0;
    TaskHandle_t activation_task_handle_ = nullptr;

    // Voice interrupt protection (防止 VAD 误触发打断)
    int64_t speaking_start_time_ = 0;           // AI 开始说话的时间
    int64_t voice_detected_start_time_ = 0;     // 检测到用户声音的起始时间
    bool voice_interrupt_pending_ = false;       // 是否有待处理的打断


    // Event handlers
    void HandleStateChangedEvent();
    void HandleToggleChatEvent();
    void HandleStartListeningEvent();
    void HandleStopListeningEvent();
    void HandleNetworkConnectedEvent();
    void HandleNetworkDisconnectedEvent();
    void HandleActivationDoneEvent();
    void HandleWakeWordDetectedEvent();

    // Activation task (runs in background)
    void ActivationTask();

    // Helper methods
    void CheckAssetsVersion();
    void CheckNewVersion();
    void InitializeProtocol();
    void ShowActivationCode(const std::string& code, const std::string& message);
    void SetListeningMode(ListeningMode mode);
    
    // State change handler called by state machine
    void OnStateChanged(DeviceState old_state, DeviceState new_state);
};


class TaskPriorityReset {
public:
    TaskPriorityReset(BaseType_t priority) {
        original_priority_ = uxTaskPriorityGet(NULL);
        vTaskPrioritySet(NULL, priority);
    }
    ~TaskPriorityReset() {
        vTaskPrioritySet(NULL, original_priority_);
    }

private:
    BaseType_t original_priority_;
};

#endif // _APPLICATION_H_

```

```cpp
#include "application.h"
#include "board.h"
#include "display.h"
#include "system_info.h"
#include "audio_codec.h"
#include "mqtt_protocol.h"
#include "websocket_protocol.h"
#include "assets/lang_config.h"
#include "mcp_server.h"
#include "assets.h"
#include "settings.h"

#include <cstring>
#include <esp_log.h>
#include <cJSON.h>
#include <driver/gpio.h>
#include <arpa/inet.h>
#include <font_awesome.h>

#define TAG "Application"


Application::Application() {
    event_group_ = xEventGroupCreate();

#if CONFIG_USE_DEVICE_AEC && CONFIG_USE_SERVER_AEC
#error "CONFIG_USE_DEVICE_AEC and CONFIG_USE_SERVER_AEC cannot be enabled at the same time"
#elif CONFIG_USE_DEVICE_AEC
    aec_mode_ = kAecOnDeviceSide;
#elif CONFIG_USE_SERVER_AEC
    aec_mode_ = kAecOnServerSide;
#else
    aec_mode_ = kAecOff;
#endif

    esp_timer_create_args_t clock_timer_args = {
        .callback = [](void* arg) {
            Application* app = (Application*)arg;
            xEventGroupSetBits(app->event_group_, MAIN_EVENT_CLOCK_TICK);
        },
        .arg = this,
        .dispatch_method = ESP_TIMER_TASK,
        .name = "clock_timer",
        .skip_unhandled_events = true
    };
    esp_timer_create(&clock_timer_args, &clock_timer_handle_);
}

Application::~Application() {
    if (clock_timer_handle_ != nullptr) {
        esp_timer_stop(clock_timer_handle_);
        esp_timer_delete(clock_timer_handle_);
    }
    vEventGroupDelete(event_group_);
}

bool Application::SetDeviceState(DeviceState state) {
    return state_machine_.TransitionTo(state);
}

void Application::Initialize() {
    auto& board = Board::GetInstance();
    SetDeviceState(kDeviceStateStarting);

    // Setup the display
    auto display = board.GetDisplay();

    // Print board name/version info
    display->SetChatMessage("system", SystemInfo::GetUserAgent().c_str());

    // Setup the audio service
    auto codec = board.GetAudioCodec();
    audio_service_.Initialize(codec);
    audio_service_.Start();

    AudioServiceCallbacks callbacks;
    callbacks.on_send_queue_available = [this]() {
        xEventGroupSetBits(event_group_, MAIN_EVENT_SEND_AUDIO);
    };
    callbacks.on_wake_word_detected = [this](const std::string& wake_word) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_WAKE_WORD_DETECTED);
    };
    callbacks.on_vad_change = [this](bool speaking) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_VAD_CHANGE);
    };
    audio_service_.SetCallbacks(callbacks);

    // Add state change listeners
    state_machine_.AddStateChangeListener([this](DeviceState old_state, DeviceState new_state) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_STATE_CHANGED);
    });

    // Start the clock timer to update the status bar
    esp_timer_start_periodic(clock_timer_handle_, 1000000);

    // Add MCP common tools (only once during initialization)
    auto& mcp_server = McpServer::GetInstance();
    mcp_server.AddCommonTools();
    mcp_server.AddUserOnlyTools();

    // Set network event callback for UI updates and network state handling
    board.SetNetworkEventCallback([this](NetworkEvent event, const std::string& data) {
        auto display = Board::GetInstance().GetDisplay();
        
        switch (event) {
            case NetworkEvent::Scanning:
                display->ShowNotification(Lang::Strings::SCANNING_WIFI, 30000);
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_DISCONNECTED);
                break;
            case NetworkEvent::Connecting: {
                if (data.empty()) {
                    // Cellular network - registering without carrier info yet
                    display->SetStatus(Lang::Strings::REGISTERING_NETWORK);
                } else {
                    // WiFi or cellular with carrier info
                    std::string msg = Lang::Strings::CONNECT_TO;
                    msg += data;
                    msg += "...";
                    display->ShowNotification(msg.c_str(), 30000);
                }
                break;
            }
            case NetworkEvent::Connected: {
                std::string msg = Lang::Strings::CONNECTED_TO;
                msg += data;
                display->ShowNotification(msg.c_str(), 30000);
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_CONNECTED);
                break;
            }
            case NetworkEvent::Disconnected:
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_DISCONNECTED);
                break;
            case NetworkEvent::WifiConfigModeEnter:
                // WiFi config mode enter is handled by WifiBoard internally
                break;
            case NetworkEvent::WifiConfigModeExit:
                // WiFi config mode exit is handled by WifiBoard internally
                break;
            // Cellular modem specific events
            case NetworkEvent::ModemDetecting:
                display->SetStatus(Lang::Strings::DETECTING_MODULE);
                break;
            case NetworkEvent::ModemErrorNoSim:
                Alert(Lang::Strings::ERROR, Lang::Strings::PIN_ERROR, "triangle_exclamation", Lang::Sounds::OGG_ERR_PIN);
                break;
            case NetworkEvent::ModemErrorRegDenied:
                Alert(Lang::Strings::ERROR, Lang::Strings::REG_ERROR, "triangle_exclamation", Lang::Sounds::OGG_ERR_REG);
                break;
            case NetworkEvent::ModemErrorInitFailed:
                display->SetStatus(Lang::Strings::DETECTING_MODULE);
                display->SetChatMessage("system", Lang::Strings::DETECTING_MODULE);
                break;
            case NetworkEvent::ModemErrorTimeout:
                display->SetStatus(Lang::Strings::REGISTERING_NETWORK);
                break;
        }
    });

    // Start network asynchronously
    board.StartNetwork();

    // Update the status bar immediately to show the network state
    display->UpdateStatusBar(true);
}

void Application::Run() {
    const EventBits_t ALL_EVENTS = 
        MAIN_EVENT_SCHEDULE |
        MAIN_EVENT_SEND_AUDIO |
        MAIN_EVENT_WAKE_WORD_DETECTED |
        MAIN_EVENT_VAD_CHANGE |
        MAIN_EVENT_CLOCK_TICK |
        MAIN_EVENT_ERROR |
        MAIN_EVENT_NETWORK_CONNECTED |
        MAIN_EVENT_NETWORK_DISCONNECTED |
        MAIN_EVENT_TOGGLE_CHAT |
        MAIN_EVENT_START_LISTENING |
        MAIN_EVENT_STOP_LISTENING |
        MAIN_EVENT_ACTIVATION_DONE |
        MAIN_EVENT_STATE_CHANGED;

    while (true) {
        auto bits = xEventGroupWaitBits(event_group_, ALL_EVENTS, pdTRUE, pdFALSE, portMAX_DELAY);

        if (bits & MAIN_EVENT_ERROR) {
            SetDeviceState(kDeviceStateIdle);
            Alert(Lang::Strings::ERROR, last_error_message_.c_str(), "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
        }

        if (bits & MAIN_EVENT_NETWORK_CONNECTED) {
            HandleNetworkConnectedEvent();
        }

        if (bits & MAIN_EVENT_NETWORK_DISCONNECTED) {
            HandleNetworkDisconnectedEvent();
        }

        if (bits & MAIN_EVENT_ACTIVATION_DONE) {
            HandleActivationDoneEvent();
        }

        if (bits & MAIN_EVENT_STATE_CHANGED) {
            HandleStateChangedEvent();
        }

        if (bits & MAIN_EVENT_TOGGLE_CHAT) {
            HandleToggleChatEvent();
        }

        if (bits & MAIN_EVENT_START_LISTENING) {
            HandleStartListeningEvent();
        }

        if (bits & MAIN_EVENT_STOP_LISTENING) {
            HandleStopListeningEvent();
        }

        if (bits & MAIN_EVENT_SEND_AUDIO) {
            while (auto packet = audio_service_.PopPacketFromSendQueue()) {
                if (protocol_ && !protocol_->SendAudio(std::move(packet))) {
                    break;
                }
            }
        }

        if (bits & MAIN_EVENT_WAKE_WORD_DETECTED) {
            HandleWakeWordDetectedEvent();
        }

        if (bits & MAIN_EVENT_VAD_CHANGE) {
            auto state = GetDeviceState();
            if (state == kDeviceStateListening) {
                auto led = Board::GetInstance().GetLed();
                led->OnStateChanged();
            } else if (state == kDeviceStateSpeaking && listening_mode_ == kListeningModeRealtime) {
                // 在实时模式下，检测用户语音打断
                // 防抖动保护：需要满足以下条件才触发打断
                // 1. AI 说话已超过 1 秒（避免刚开始说话就被打断）
                // 2. 持续检测到用户声音超过 500ms（避免短暂噪声触发）
                
                const int64_t SPEAKING_PROTECTION_MS = 1000;  // AI 说话保护期
                const int64_t VOICE_SUSTAIN_MS = 500;         // 用户声音持续时间要求
                
                int64_t now = esp_timer_get_time() / 1000;  // 转换为毫秒
                
                if (audio_service_.IsVoiceDetected()) {
                    // 检测到声音
                    if (voice_detected_start_time_ == 0) {
                        voice_detected_start_time_ = now;
                        ESP_LOGD(TAG, "Voice detected, starting timer...");
                    }
                    
                    int64_t speaking_duration = now - speaking_start_time_;
                    int64_t voice_duration = now - voice_detected_start_time_;
                    
                    // 检查是否满足打断条件
                    if (speaking_duration > SPEAKING_PROTECTION_MS && 
                        voice_duration > VOICE_SUSTAIN_MS) {
                        ESP_LOGI(TAG, "Voice interrupt triggered (speaking: %lldms, voice: %lldms)", 
                                 speaking_duration, voice_duration);
                        voice_detected_start_time_ = 0;  // 重置
                        AbortSpeaking(kAbortReasonNone);
                    }
                } else {
                    // 声音停止，重置计时器
                    if (voice_detected_start_time_ != 0) {
                        ESP_LOGD(TAG, "Voice stopped, resetting timer");
                        voice_detected_start_time_ = 0;
                    }
                }
            }
        }

        if (bits & MAIN_EVENT_SCHEDULE) {
            std::unique_lock<std::mutex> lock(mutex_);
            auto tasks = std::move(main_tasks_);
            lock.unlock();
            for (auto& task : tasks) {
                task();
            }
        }

        if (bits & MAIN_EVENT_CLOCK_TICK) {
            clock_ticks_++;
            auto display = Board::GetInstance().GetDisplay();
            display->UpdateStatusBar();
        
            // Print debug info every 10 seconds
            if (clock_ticks_ % 10 == 0) {
                SystemInfo::PrintHeapStats();
            }
        }
    }
}

void Application::HandleNetworkConnectedEvent() {
    ESP_LOGI(TAG, "Network connected");
    auto state = GetDeviceState();

    if (state == kDeviceStateStarting || state == kDeviceStateWifiConfiguring) {
        // Network is ready, start activation
        SetDeviceState(kDeviceStateActivating);
        if (activation_task_handle_ != nullptr) {
            ESP_LOGW(TAG, "Activation task already running");
            return;
        }

        xTaskCreate([](void* arg) {
            Application* app = static_cast<Application*>(arg);
            app->ActivationTask();
            app->activation_task_handle_ = nullptr;
            vTaskDelete(NULL);
        }, "activation", 4096 * 2, this, 2, &activation_task_handle_);
    }

    // Update the status bar immediately to show the network state
    auto display = Board::GetInstance().GetDisplay();
    display->UpdateStatusBar(true);
}

void Application::HandleNetworkDisconnectedEvent() {
    // Close current conversation when network disconnected
    auto state = GetDeviceState();
    if (state == kDeviceStateConnecting || state == kDeviceStateListening || state == kDeviceStateSpeaking) {
        ESP_LOGI(TAG, "Closing audio channel due to network disconnection");
        protocol_->CloseAudioChannel();
    }

    // Update the status bar immediately to show the network state
    auto display = Board::GetInstance().GetDisplay();
    display->UpdateStatusBar(true);
}

void Application::HandleActivationDoneEvent() {
    ESP_LOGI(TAG, "Activation done");

    SystemInfo::PrintHeapStats();
    SetDeviceState(kDeviceStateIdle);

    has_server_time_ = ota_->HasServerTime();

    auto display = Board::GetInstance().GetDisplay();
    std::string message = std::string(Lang::Strings::VERSION) + ota_->GetCurrentVersion();
    display->ShowNotification(message.c_str());
    display->SetChatMessage("system", "");

    // Play the success sound to indicate the device is ready
    audio_service_.PlaySound(Lang::Sounds::OGG_SUCCESS);

    // Release OTA object after activation is complete
    ota_.reset();
    auto& board = Board::GetInstance();
    board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
}

void Application::ActivationTask() {
    // Create OTA object for activation process
    ota_ = std::make_unique<Ota>();

    // Check for new assets version
    CheckAssetsVersion();

    // Check for new firmware version
    CheckNewVersion();

    // Initialize the protocol
    InitializeProtocol();

    // Signal completion to main loop
    xEventGroupSetBits(event_group_, MAIN_EVENT_ACTIVATION_DONE);
}

void Application::CheckAssetsVersion() {
    // Only allow CheckAssetsVersion to be called once
    if (assets_version_checked_) {
        return;
    }
    assets_version_checked_ = true;

    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto& assets = Assets::GetInstance();

    if (!assets.partition_valid()) {
        ESP_LOGW(TAG, "Assets partition is disabled for board %s", BOARD_NAME);
        return;
    }
    
    Settings settings("assets", true);
    // Check if there is a new assets need to be downloaded
    std::string download_url = settings.GetString("download_url");

    if (!download_url.empty()) {
        settings.EraseKey("download_url");

        char message[256];
        snprintf(message, sizeof(message), Lang::Strings::FOUND_NEW_ASSETS, download_url.c_str());
        Alert(Lang::Strings::LOADING_ASSETS, message, "cloud_arrow_down", Lang::Sounds::OGG_UPGRADE);
        
        // Wait for the audio service to be idle for 3 seconds
        vTaskDelay(pdMS_TO_TICKS(3000));
        SetDeviceState(kDeviceStateUpgrading);
        board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
        display->SetChatMessage("system", Lang::Strings::PLEASE_WAIT);

        bool success = assets.Download(download_url, [display](int progress, size_t speed) -> void {
            std::thread([display, progress, speed]() {
                char buffer[32];
                snprintf(buffer, sizeof(buffer), "%d%% %uKB/s", progress, speed / 1024);
                display->SetChatMessage("system", buffer);
            }).detach();
        });

        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
        vTaskDelay(pdMS_TO_TICKS(1000));

        if (!success) {
            Alert(Lang::Strings::ERROR, Lang::Strings::DOWNLOAD_ASSETS_FAILED, "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
            vTaskDelay(pdMS_TO_TICKS(2000));
            SetDeviceState(kDeviceStateActivating);
            return;
        }
    }

    // Apply assets
    assets.Apply();
    display->SetChatMessage("system", "");
    display->SetEmotion("microchip_ai");
}

void Application::CheckNewVersion() {
    const int MAX_RETRY = 10;
    int retry_count = 0;
    int retry_delay = 10; // Initial retry delay in seconds

    auto& board = Board::GetInstance();
    while (true) {
        auto display = board.GetDisplay();
        display->SetStatus(Lang::Strings::CHECKING_NEW_VERSION);

        esp_err_t err = ota_->CheckVersion();
        if (err != ESP_OK) {
            retry_count++;
            if (retry_count >= MAX_RETRY) {
                ESP_LOGE(TAG, "Too many retries, exit version check");
                return;
            }

            char error_message[128];
            snprintf(error_message, sizeof(error_message), "code=%d, url=%s", err, ota_->GetCheckVersionUrl().c_str());
            char buffer[256];
            snprintf(buffer, sizeof(buffer), Lang::Strings::CHECK_NEW_VERSION_FAILED, retry_delay, error_message);
            Alert(Lang::Strings::ERROR, buffer, "cloud_slash", Lang::Sounds::OGG_EXCLAMATION);

            ESP_LOGW(TAG, "Check new version failed, retry in %d seconds (%d/%d)", retry_delay, retry_count, MAX_RETRY);
            for (int i = 0; i < retry_delay; i++) {
                vTaskDelay(pdMS_TO_TICKS(1000));
                if (GetDeviceState() == kDeviceStateIdle) {
                    break;
                }
            }
            retry_delay *= 2; // Double the retry delay
            continue;
        }
        retry_count = 0;
        retry_delay = 10; // Reset retry delay

        if (ota_->HasNewVersion()) {
            if (UpgradeFirmware(ota_->GetFirmwareUrl(), ota_->GetFirmwareVersion())) {
                return; // This line will never be reached after reboot
            }
            // If upgrade failed, continue to normal operation
        }

        // No new version, mark the current version as valid
        ota_->MarkCurrentVersionValid();
        if (!ota_->HasActivationCode() && !ota_->HasActivationChallenge()) {
            // Exit the loop if done checking new version
            break;
        }

        display->SetStatus(Lang::Strings::ACTIVATION);
        // Activation code is shown to the user and waiting for the user to input
        if (ota_->HasActivationCode()) {
            ShowActivationCode(ota_->GetActivationCode(), ota_->GetActivationMessage());
        }

        // This will block the loop until the activation is done or timeout
        for (int i = 0; i < 10; ++i) {
            ESP_LOGI(TAG, "Activating... %d/%d", i + 1, 10);
            esp_err_t err = ota_->Activate();
            if (err == ESP_OK) {
                break;
            } else if (err == ESP_ERR_TIMEOUT) {
                vTaskDelay(pdMS_TO_TICKS(3000));
            } else {
                vTaskDelay(pdMS_TO_TICKS(10000));
            }
            if (GetDeviceState() == kDeviceStateIdle) {
                break;
            }
        }
    }
}

void Application::InitializeProtocol() {
    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto codec = board.GetAudioCodec();

    display->SetStatus(Lang::Strings::LOADING_PROTOCOL);

    if (ota_->HasMqttConfig()) {
        protocol_ = std::make_unique<MqttProtocol>();
    } else if (ota_->HasWebsocketConfig()) {
        protocol_ = std::make_unique<WebsocketProtocol>();
    } else {
        ESP_LOGW(TAG, "No protocol specified in the OTA config, using MQTT");
        protocol_ = std::make_unique<MqttProtocol>();
    }

    protocol_->OnConnected([this]() {
        DismissAlert();
    });

    protocol_->OnNetworkError([this](const std::string& message) {
        last_error_message_ = message;
        xEventGroupSetBits(event_group_, MAIN_EVENT_ERROR);
    });
    
    protocol_->OnIncomingAudio([this](std::unique_ptr<AudioStreamPacket> packet) {
        if (GetDeviceState() == kDeviceStateSpeaking) {
            audio_service_.PushPacketToDecodeQueue(std::move(packet));
        }
    });
    
    protocol_->OnAudioChannelOpened([this, codec, &board]() {
        board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
        if (protocol_->server_sample_rate() != codec->output_sample_rate()) {
            ESP_LOGW(TAG, "Server sample rate %d does not match device output sample rate %d, resampling may cause distortion",
                protocol_->server_sample_rate(), codec->output_sample_rate());
        }
    });
    
    protocol_->OnAudioChannelClosed([this, &board]() {
        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
        Schedule([this]() {
            auto display = Board::GetInstance().GetDisplay();
            display->SetChatMessage("system", "");
            SetDeviceState(kDeviceStateIdle);
        });
    });
    
    protocol_->OnIncomingJson([this, display](const cJSON* root) {
        // Parse JSON data
        auto type = cJSON_GetObjectItem(root, "type");
        if (strcmp(type->valuestring, "tts") == 0) {
            auto state = cJSON_GetObjectItem(root, "state");
            if (strcmp(state->valuestring, "start") == 0) {
                Schedule([this]() {
                    aborted_ = false;
                    SetDeviceState(kDeviceStateSpeaking);
                });
            } else if (strcmp(state->valuestring, "stop") == 0) {
                Schedule([this]() {
                    if (GetDeviceState() == kDeviceStateSpeaking) {
                        if (listening_mode_ == kListeningModeManualStop) {
                            SetDeviceState(kDeviceStateIdle);
                        } else {
                            SetDeviceState(kDeviceStateListening);
                        }
                    }
                });
            } else if (strcmp(state->valuestring, "sentence_start") == 0) {
                auto text = cJSON_GetObjectItem(root, "text");
                if (cJSON_IsString(text)) {
                    ESP_LOGI(TAG, "<< %s", text->valuestring);
                    Schedule([this, display, message = std::string(text->valuestring)]() {
                        display->SetChatMessage("assistant", message.c_str());
                    });
                }
            }
        } else if (strcmp(type->valuestring, "stt") == 0) {
            auto text = cJSON_GetObjectItem(root, "text");
            if (cJSON_IsString(text)) {
                ESP_LOGI(TAG, ">> %s", text->valuestring);
                Schedule([this, display, message = std::string(text->valuestring)]() {
                    display->SetChatMessage("user", message.c_str());
                });
            }
        } else if (strcmp(type->valuestring, "llm") == 0) {
            auto emotion = cJSON_GetObjectItem(root, "emotion");
            if (cJSON_IsString(emotion)) {
                Schedule([this, display, emotion_str = std::string(emotion->valuestring)]() {
                    display->SetEmotion(emotion_str.c_str());
                });
            }
        } else if (strcmp(type->valuestring, "mcp") == 0) {
            auto payload = cJSON_GetObjectItem(root, "payload");
            if (cJSON_IsObject(payload)) {
                McpServer::GetInstance().ParseMessage(payload);
            }
        } else if (strcmp(type->valuestring, "system") == 0) {
            auto command = cJSON_GetObjectItem(root, "command");
            if (cJSON_IsString(command)) {
                ESP_LOGI(TAG, "System command: %s", command->valuestring);
                if (strcmp(command->valuestring, "reboot") == 0) {
                    // Do a reboot if user requests a OTA update
                    Schedule([this]() {
                        Reboot();
                    });
                } else {
                    ESP_LOGW(TAG, "Unknown system command: %s", command->valuestring);
                }
            }
        } else if (strcmp(type->valuestring, "alert") == 0) {
            auto status = cJSON_GetObjectItem(root, "status");
            auto message = cJSON_GetObjectItem(root, "message");
            auto emotion = cJSON_GetObjectItem(root, "emotion");
            if (cJSON_IsString(status) && cJSON_IsString(message) && cJSON_IsString(emotion)) {
                Alert(status->valuestring, message->valuestring, emotion->valuestring, Lang::Sounds::OGG_VIBRATION);
            } else {
                ESP_LOGW(TAG, "Alert command requires status, message and emotion");
            }
#if CONFIG_RECEIVE_CUSTOM_MESSAGE
        } else if (strcmp(type->valuestring, "custom") == 0) {
            auto payload = cJSON_GetObjectItem(root, "payload");
            ESP_LOGI(TAG, "Received custom message: %s", cJSON_PrintUnformatted(root));
            if (cJSON_IsObject(payload)) {
                Schedule([this, display, payload_str = std::string(cJSON_PrintUnformatted(payload))]() {
                    display->SetChatMessage("system", payload_str.c_str());
                });
            } else {
                ESP_LOGW(TAG, "Invalid custom message format: missing payload");
            }
#endif
        } else {
            ESP_LOGW(TAG, "Unknown message type: %s", type->valuestring);
        }
    });
    
    protocol_->Start();
}

void Application::ShowActivationCode(const std::string& code, const std::string& message) {
    struct digit_sound {
        char digit;
        const std::string_view& sound;
    };
    static const std::array<digit_sound, 10> digit_sounds{{
        digit_sound{'0', Lang::Sounds::OGG_0},
        digit_sound{'1', Lang::Sounds::OGG_1}, 
        digit_sound{'2', Lang::Sounds::OGG_2},
        digit_sound{'3', Lang::Sounds::OGG_3},
        digit_sound{'4', Lang::Sounds::OGG_4},
        digit_sound{'5', Lang::Sounds::OGG_5},
        digit_sound{'6', Lang::Sounds::OGG_6},
        digit_sound{'7', Lang::Sounds::OGG_7},
        digit_sound{'8', Lang::Sounds::OGG_8},
        digit_sound{'9', Lang::Sounds::OGG_9}
    }};

    // This sentence uses 9KB of SRAM, so we need to wait for it to finish
    Alert(Lang::Strings::ACTIVATION, message.c_str(), "link", Lang::Sounds::OGG_ACTIVATION);

    for (const auto& digit : code) {
        auto it = std::find_if(digit_sounds.begin(), digit_sounds.end(),
            [digit](const digit_sound& ds) { return ds.digit == digit; });
        if (it != digit_sounds.end()) {
            audio_service_.PlaySound(it->sound);
        }
    }
}

void Application::Alert(const char* status, const char* message, const char* emotion, const std::string_view& sound) {
    ESP_LOGW(TAG, "Alert [%s] %s: %s", emotion, status, message);
    auto display = Board::GetInstance().GetDisplay();
    display->SetStatus(status);
    display->SetEmotion(emotion);
    display->SetChatMessage("system", message);
    if (!sound.empty()) {
        audio_service_.PlaySound(sound);
    }
}

void Application::DismissAlert() {
    if (GetDeviceState() == kDeviceStateIdle) {
        auto display = Board::GetInstance().GetDisplay();
        display->SetStatus(Lang::Strings::STANDBY);
        display->SetEmotion("neutral");
        display->SetChatMessage("system", "");
    }
}

void Application::ToggleChatState() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_TOGGLE_CHAT);
}

void Application::StartListening() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_START_LISTENING);
}

void Application::StopListening() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_STOP_LISTENING);
}

void Application::HandleToggleChatEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateActivating) {
        SetDeviceState(kDeviceStateIdle);
        return;
    } else if (state == kDeviceStateWifiConfiguring) {
        audio_service_.EnableAudioTesting(true);
        SetDeviceState(kDeviceStateAudioTesting);
        return;
    } else if (state == kDeviceStateAudioTesting) {
        audio_service_.EnableAudioTesting(false);
        SetDeviceState(kDeviceStateWifiConfiguring);
        return;
    }

    if (!protocol_) {
        ESP_LOGE(TAG, "Protocol not initialized");
        return;
    }

    if (state == kDeviceStateIdle) {
        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                return;
            }
        }

        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonNone);
    } else if (state == kDeviceStateListening) {
        protocol_->CloseAudioChannel();
    }
}

void Application::HandleStartListeningEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateActivating) {
        SetDeviceState(kDeviceStateIdle);
        return;
    } else if (state == kDeviceStateWifiConfiguring) {
        audio_service_.EnableAudioTesting(true);
        SetDeviceState(kDeviceStateAudioTesting);
        return;
    }

    if (!protocol_) {
        ESP_LOGE(TAG, "Protocol not initialized");
        return;
    }
    
    if (state == kDeviceStateIdle) {
        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                return;
            }
        }

        SetListeningMode(kListeningModeManualStop);
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonNone);
        SetListeningMode(kListeningModeManualStop);
    }
}

void Application::HandleStopListeningEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateAudioTesting) {
        audio_service_.EnableAudioTesting(false);
        SetDeviceState(kDeviceStateWifiConfiguring);
        return;
    } else if (state == kDeviceStateListening) {
        if (protocol_) {
            protocol_->SendStopListening();
        }
        SetDeviceState(kDeviceStateIdle);
    }
}

void Application::HandleWakeWordDetectedEvent() {
    if (!protocol_) {
        return;
    }

    auto state = GetDeviceState();
    
    if (state == kDeviceStateIdle) {
        audio_service_.EncodeWakeWord();

        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                audio_service_.EnableWakeWordDetection(true);
                return;
            }
        }

        auto wake_word = audio_service_.GetLastWakeWord();
        ESP_LOGI(TAG, "Wake word detected: %s", wake_word.c_str());
#if CONFIG_SEND_WAKE_WORD_DATA
        // Encode and send the wake word data to the server
        while (auto packet = audio_service_.PopWakeWordPacket()) {
            protocol_->SendAudio(std::move(packet));
        }
        // Set the chat state to wake word detected
        protocol_->SendWakeWordDetected(wake_word);
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#else
        // Set flag to play popup sound after state changes to listening
        // (PlaySound here would be cleared by ResetDecoder in EnableVoiceProcessing)
        play_popup_on_listening_ = true;
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#endif
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonWakeWordDetected);
    } else if (state == kDeviceStateActivating) {
        // Restart the activation check if the wake word is detected during activation
        SetDeviceState(kDeviceStateIdle);
    }
}

void Application::HandleStateChangedEvent() {
    DeviceState new_state = state_machine_.GetState();
    clock_ticks_ = 0;

    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto led = board.GetLed();
    led->OnStateChanged();
    
    switch (new_state) {
        case kDeviceStateUnknown:
        case kDeviceStateIdle:
            display->SetStatus(Lang::Strings::STANDBY);
            display->SetEmotion("neutral");
            audio_service_.EnableVoiceProcessing(false);
            audio_service_.EnableWakeWordDetection(true);
            break;
        case kDeviceStateConnecting:
            display->SetStatus(Lang::Strings::CONNECTING);
            display->SetEmotion("neutral");
            display->SetChatMessage("system", "");
            break;
        case kDeviceStateListening:
            display->SetStatus(Lang::Strings::LISTENING);
            display->SetEmotion("neutral");

            // Make sure the audio processor is running
            if (!audio_service_.IsAudioProcessorRunning()) {
                // Send the start listening command
                protocol_->SendStartListening(listening_mode_);
                audio_service_.EnableVoiceProcessing(true);
                audio_service_.EnableWakeWordDetection(false);
            }

            // Play popup sound after ResetDecoder (in EnableVoiceProcessing) has been called
            if (play_popup_on_listening_) {
                play_popup_on_listening_ = false;
                audio_service_.PlaySound(Lang::Sounds::OGG_POPUP);
            }
            break;
        case kDeviceStateSpeaking:
            display->SetStatus(Lang::Strings::SPEAKING);

            if (listening_mode_ != kListeningModeRealtime) {
                audio_service_.EnableVoiceProcessing(false);
                // Only AFE wake word can be detected in speaking mode
                audio_service_.EnableWakeWordDetection(audio_service_.IsAfeWakeWord());
            }
            audio_service_.ResetDecoder();
            break;
        case kDeviceStateWifiConfiguring:
            audio_service_.EnableVoiceProcessing(false);
            audio_service_.EnableWakeWordDetection(false);
            break;
        default:
            // Do nothing
            break;
    }
}

void Application::Schedule(std::function<void()>&& callback) {
    {
        std::lock_guard<std::mutex> lock(mutex_);
        main_tasks_.push_back(std::move(callback));
    }
    xEventGroupSetBits(event_group_, MAIN_EVENT_SCHEDULE);
}

void Application::AbortSpeaking(AbortReason reason) {
    ESP_LOGI(TAG, "Abort speaking");
    aborted_ = true;
    if (protocol_) {
        protocol_->SendAbortSpeaking(reason);
    }
}

void Application::SetListeningMode(ListeningMode mode) {
    listening_mode_ = mode;
    SetDeviceState(kDeviceStateListening);
}

void Application::Reboot() {
    ESP_LOGI(TAG, "Rebooting...");
    // Disconnect the audio channel
    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        protocol_->CloseAudioChannel();
    }
    protocol_.reset();
    audio_service_.Stop();

    vTaskDelay(pdMS_TO_TICKS(1000));
    esp_restart();
}

bool Application::UpgradeFirmware(const std::string& url, const std::string& version) {
    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();

    std::string upgrade_url = url;
    std::string version_info = version.empty() ? "(Manual upgrade)" : version;

    // Close audio channel if it's open
    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        ESP_LOGI(TAG, "Closing audio channel before firmware upgrade");
        protocol_->CloseAudioChannel();
    }
    ESP_LOGI(TAG, "Starting firmware upgrade from URL: %s", upgrade_url.c_str());

    Alert(Lang::Strings::OTA_UPGRADE, Lang::Strings::UPGRADING, "download", Lang::Sounds::OGG_UPGRADE);
    vTaskDelay(pdMS_TO_TICKS(3000));

    SetDeviceState(kDeviceStateUpgrading);

    std::string message = std::string(Lang::Strings::NEW_VERSION) + version_info;
    display->SetChatMessage("system", message.c_str());

    board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
    audio_service_.Stop();
    vTaskDelay(pdMS_TO_TICKS(1000));

    bool upgrade_success = Ota::Upgrade(upgrade_url, [display](int progress, size_t speed) {
        std::thread([display, progress, speed]() {
            char buffer[32];
            snprintf(buffer, sizeof(buffer), "%d%% %uKB/s", progress, speed / 1024);
            display->SetChatMessage("system", buffer);
        }).detach();
    });

    if (!upgrade_success) {
        // Upgrade failed, restart audio service and continue running
        ESP_LOGE(TAG, "Firmware upgrade failed, restarting audio service and continuing operation...");
        audio_service_.Start(); // Restart audio service
        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER); // Restore power save level
        Alert(Lang::Strings::ERROR, Lang::Strings::UPGRADE_FAILED, "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
        vTaskDelay(pdMS_TO_TICKS(3000));
        return false;
    } else {
        // Upgrade success, reboot immediately
        ESP_LOGI(TAG, "Firmware upgrade successful, rebooting...");
        display->SetChatMessage("system", "Upgrade successful, rebooting...");
        vTaskDelay(pdMS_TO_TICKS(1000)); // Brief pause to show message
        Reboot();
        return true;
    }
}

void Application::WakeWordInvoke(const std::string& wake_word) {
    if (!protocol_) {
        return;
    }

    auto state = GetDeviceState();
    
    if (state == kDeviceStateIdle) {
        audio_service_.EncodeWakeWord();

        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                audio_service_.EnableWakeWordDetection(true);
                return;
            }
        }

        ESP_LOGI(TAG, "Wake word detected: %s", wake_word.c_str());
#if CONFIG_USE_AFE_WAKE_WORD || CONFIG_USE_CUSTOM_WAKE_WORD
        // Encode and send the wake word data to the server
        while (auto packet = audio_service_.PopWakeWordPacket()) {
            protocol_->SendAudio(std::move(packet));
        }
        // Set the chat state to wake word detected
        protocol_->SendWakeWordDetected(wake_word);
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#else
        // Set flag to play popup sound after state changes to listening
        // (PlaySound here would be cleared by ResetDecoder in EnableVoiceProcessing)
        play_popup_on_listening_ = true;
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#endif
    } else if (state == kDeviceStateSpeaking) {
        Schedule([this]() {
            AbortSpeaking(kAbortReasonNone);
        });
    } else if (state == kDeviceStateListening) {   
        Schedule([this]() {
            if (protocol_) {
                protocol_->CloseAudioChannel();
            }
        });
    }
}

bool Application::CanEnterSleepMode() {
    if (GetDeviceState() != kDeviceStateIdle) {
        return false;
    }

    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        return false;
    }

    if (!audio_service_.IsIdle()) {
        return false;
    }

    // Now it is safe to enter sleep mode
    return true;
}

void Application::SendMcpMessage(const std::string& payload) {
    // Always schedule to run in main task for thread safety
    Schedule([this, payload = std::move(payload)]() {
        if (protocol_) {
            protocol_->SendMcpMessage(payload);
        }
    });
}

void Application::SetAecMode(AecMode mode) {
    aec_mode_ = mode;
    Schedule([this]() {
        auto& board = Board::GetInstance();
        auto display = board.GetDisplay();
        switch (aec_mode_) {
        case kAecOff:
            audio_service_.EnableDeviceAec(false);
            display->ShowNotification(Lang::Strings::RTC_MODE_OFF);
            break;
        case kAecOnServerSide:
            audio_service_.EnableDeviceAec(false);
            display->ShowNotification(Lang::Strings::RTC_MODE_ON);
            break;
        case kAecOnDeviceSide:
            audio_service_.EnableDeviceAec(true);
            display->ShowNotification(Lang::Strings::RTC_MODE_ON);
            break;
        }

        // If the AEC mode is changed, close the audio channel
        if (protocol_ && protocol_->IsAudioChannelOpened()) {
            protocol_->CloseAudioChannel();
        }
    });
}

void Application::PlaySound(const std::string_view& sound) {
    audio_service_.PlaySound(sound);
}

void Application::ResetProtocol() {
    Schedule([this]() {
        // Close audio channel if opened
        if (protocol_ && protocol_->IsAudioChannelOpened()) {
            protocol_->CloseAudioChannel();
        }
        // Reset protocol
        protocol_.reset();
    });
}


```

现在在进入 Speaking 状态时记录开始时间：

```cpp
#include "application.h"
#include "board.h"
#include "display.h"
#include "system_info.h"
#include "audio_codec.h"
#include "mqtt_protocol.h"
#include "websocket_protocol.h"
#include "assets/lang_config.h"
#include "mcp_server.h"
#include "assets.h"
#include "settings.h"

#include <cstring>
#include <esp_log.h>
#include <cJSON.h>
#include <driver/gpio.h>
#include <arpa/inet.h>
#include <font_awesome.h>

#define TAG "Application"


Application::Application() {
    event_group_ = xEventGroupCreate();

#if CONFIG_USE_DEVICE_AEC && CONFIG_USE_SERVER_AEC
#error "CONFIG_USE_DEVICE_AEC and CONFIG_USE_SERVER_AEC cannot be enabled at the same time"
#elif CONFIG_USE_DEVICE_AEC
    aec_mode_ = kAecOnDeviceSide;
#elif CONFIG_USE_SERVER_AEC
    aec_mode_ = kAecOnServerSide;
#else
    aec_mode_ = kAecOff;
#endif

    esp_timer_create_args_t clock_timer_args = {
        .callback = [](void* arg) {
            Application* app = (Application*)arg;
            xEventGroupSetBits(app->event_group_, MAIN_EVENT_CLOCK_TICK);
        },
        .arg = this,
        .dispatch_method = ESP_TIMER_TASK,
        .name = "clock_timer",
        .skip_unhandled_events = true
    };
    esp_timer_create(&clock_timer_args, &clock_timer_handle_);
}

Application::~Application() {
    if (clock_timer_handle_ != nullptr) {
        esp_timer_stop(clock_timer_handle_);
        esp_timer_delete(clock_timer_handle_);
    }
    vEventGroupDelete(event_group_);
}

bool Application::SetDeviceState(DeviceState state) {
    return state_machine_.TransitionTo(state);
}

void Application::Initialize() {
    auto& board = Board::GetInstance();
    SetDeviceState(kDeviceStateStarting);

    // Setup the display
    auto display = board.GetDisplay();

    // Print board name/version info
    display->SetChatMessage("system", SystemInfo::GetUserAgent().c_str());

    // Setup the audio service
    auto codec = board.GetAudioCodec();
    audio_service_.Initialize(codec);
    audio_service_.Start();

    AudioServiceCallbacks callbacks;
    callbacks.on_send_queue_available = [this]() {
        xEventGroupSetBits(event_group_, MAIN_EVENT_SEND_AUDIO);
    };
    callbacks.on_wake_word_detected = [this](const std::string& wake_word) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_WAKE_WORD_DETECTED);
    };
    callbacks.on_vad_change = [this](bool speaking) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_VAD_CHANGE);
    };
    audio_service_.SetCallbacks(callbacks);

    // Add state change listeners
    state_machine_.AddStateChangeListener([this](DeviceState old_state, DeviceState new_state) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_STATE_CHANGED);
    });

    // Start the clock timer to update the status bar
    esp_timer_start_periodic(clock_timer_handle_, 1000000);

    // Add MCP common tools (only once during initialization)
    auto& mcp_server = McpServer::GetInstance();
    mcp_server.AddCommonTools();
    mcp_server.AddUserOnlyTools();

    // Set network event callback for UI updates and network state handling
    board.SetNetworkEventCallback([this](NetworkEvent event, const std::string& data) {
        auto display = Board::GetInstance().GetDisplay();
        
        switch (event) {
            case NetworkEvent::Scanning:
                display->ShowNotification(Lang::Strings::SCANNING_WIFI, 30000);
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_DISCONNECTED);
                break;
            case NetworkEvent::Connecting: {
                if (data.empty()) {
                    // Cellular network - registering without carrier info yet
                    display->SetStatus(Lang::Strings::REGISTERING_NETWORK);
                } else {
                    // WiFi or cellular with carrier info
                    std::string msg = Lang::Strings::CONNECT_TO;
                    msg += data;
                    msg += "...";
                    display->ShowNotification(msg.c_str(), 30000);
                }
                break;
            }
            case NetworkEvent::Connected: {
                std::string msg = Lang::Strings::CONNECTED_TO;
                msg += data;
                display->ShowNotification(msg.c_str(), 30000);
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_CONNECTED);
                break;
            }
            case NetworkEvent::Disconnected:
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_DISCONNECTED);
                break;
            case NetworkEvent::WifiConfigModeEnter:
                // WiFi config mode enter is handled by WifiBoard internally
                break;
            case NetworkEvent::WifiConfigModeExit:
                // WiFi config mode exit is handled by WifiBoard internally
                break;
            // Cellular modem specific events
            case NetworkEvent::ModemDetecting:
                display->SetStatus(Lang::Strings::DETECTING_MODULE);
                break;
            case NetworkEvent::ModemErrorNoSim:
                Alert(Lang::Strings::ERROR, Lang::Strings::PIN_ERROR, "triangle_exclamation", Lang::Sounds::OGG_ERR_PIN);
                break;
            case NetworkEvent::ModemErrorRegDenied:
                Alert(Lang::Strings::ERROR, Lang::Strings::REG_ERROR, "triangle_exclamation", Lang::Sounds::OGG_ERR_REG);
                break;
            case NetworkEvent::ModemErrorInitFailed:
                display->SetStatus(Lang::Strings::DETECTING_MODULE);
                display->SetChatMessage("system", Lang::Strings::DETECTING_MODULE);
                break;
            case NetworkEvent::ModemErrorTimeout:
                display->SetStatus(Lang::Strings::REGISTERING_NETWORK);
                break;
        }
    });

    // Start network asynchronously
    board.StartNetwork();

    // Update the status bar immediately to show the network state
    display->UpdateStatusBar(true);
}

void Application::Run() {
    const EventBits_t ALL_EVENTS = 
        MAIN_EVENT_SCHEDULE |
        MAIN_EVENT_SEND_AUDIO |
        MAIN_EVENT_WAKE_WORD_DETECTED |
        MAIN_EVENT_VAD_CHANGE |
        MAIN_EVENT_CLOCK_TICK |
        MAIN_EVENT_ERROR |
        MAIN_EVENT_NETWORK_CONNECTED |
        MAIN_EVENT_NETWORK_DISCONNECTED |
        MAIN_EVENT_TOGGLE_CHAT |
        MAIN_EVENT_START_LISTENING |
        MAIN_EVENT_STOP_LISTENING |
        MAIN_EVENT_ACTIVATION_DONE |
        MAIN_EVENT_STATE_CHANGED;

    while (true) {
        auto bits = xEventGroupWaitBits(event_group_, ALL_EVENTS, pdTRUE, pdFALSE, portMAX_DELAY);

        if (bits & MAIN_EVENT_ERROR) {
            SetDeviceState(kDeviceStateIdle);
            Alert(Lang::Strings::ERROR, last_error_message_.c_str(), "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
        }

        if (bits & MAIN_EVENT_NETWORK_CONNECTED) {
            HandleNetworkConnectedEvent();
        }

        if (bits & MAIN_EVENT_NETWORK_DISCONNECTED) {
            HandleNetworkDisconnectedEvent();
        }

        if (bits & MAIN_EVENT_ACTIVATION_DONE) {
            HandleActivationDoneEvent();
        }

        if (bits & MAIN_EVENT_STATE_CHANGED) {
            HandleStateChangedEvent();
        }

        if (bits & MAIN_EVENT_TOGGLE_CHAT) {
            HandleToggleChatEvent();
        }

        if (bits & MAIN_EVENT_START_LISTENING) {
            HandleStartListeningEvent();
        }

        if (bits & MAIN_EVENT_STOP_LISTENING) {
            HandleStopListeningEvent();
        }

        if (bits & MAIN_EVENT_SEND_AUDIO) {
            while (auto packet = audio_service_.PopPacketFromSendQueue()) {
                if (protocol_ && !protocol_->SendAudio(std::move(packet))) {
                    break;
                }
            }
        }

        if (bits & MAIN_EVENT_WAKE_WORD_DETECTED) {
            HandleWakeWordDetectedEvent();
        }

        if (bits & MAIN_EVENT_VAD_CHANGE) {
            auto state = GetDeviceState();
            if (state == kDeviceStateListening) {
                auto led = Board::GetInstance().GetLed();
                led->OnStateChanged();
            } else if (state == kDeviceStateSpeaking && listening_mode_ == kListeningModeRealtime) {
                // 在实时模式下，检测用户语音打断
                // 防抖动保护：需要满足以下条件才触发打断
                // 1. AI 说话已超过 1 秒（避免刚开始说话就被打断）
                // 2. 持续检测到用户声音超过 500ms（避免短暂噪声触发）
                
                const int64_t SPEAKING_PROTECTION_MS = 1000;  // AI 说话保护期
                const int64_t VOICE_SUSTAIN_MS = 500;         // 用户声音持续时间要求
                
                int64_t now = esp_timer_get_time() / 1000;  // 转换为毫秒
                
                if (audio_service_.IsVoiceDetected()) {
                    // 检测到声音
                    if (voice_detected_start_time_ == 0) {
                        voice_detected_start_time_ = now;
                        ESP_LOGD(TAG, "Voice detected, starting timer...");
                    }
                    
                    int64_t speaking_duration = now - speaking_start_time_;
                    int64_t voice_duration = now - voice_detected_start_time_;
                    
                    // 检查是否满足打断条件
                    if (speaking_duration > SPEAKING_PROTECTION_MS && 
                        voice_duration > VOICE_SUSTAIN_MS) {
                        ESP_LOGI(TAG, "Voice interrupt triggered (speaking: %lldms, voice: %lldms)", 
                                 speaking_duration, voice_duration);
                        voice_detected_start_time_ = 0;  // 重置
                        AbortSpeaking(kAbortReasonNone);
                    }
                } else {
                    // 声音停止，重置计时器
                    if (voice_detected_start_time_ != 0) {
                        ESP_LOGD(TAG, "Voice stopped, resetting timer");
                        voice_detected_start_time_ = 0;
                    }
                }
            }
        }

        if (bits & MAIN_EVENT_SCHEDULE) {
            std::unique_lock<std::mutex> lock(mutex_);
            auto tasks = std::move(main_tasks_);
            lock.unlock();
            for (auto& task : tasks) {
                task();
            }
        }

        if (bits & MAIN_EVENT_CLOCK_TICK) {
            clock_ticks_++;
            auto display = Board::GetInstance().GetDisplay();
            display->UpdateStatusBar();
        
            // Print debug info every 10 seconds
            if (clock_ticks_ % 10 == 0) {
                SystemInfo::PrintHeapStats();
            }
        }
    }
}

void Application::HandleNetworkConnectedEvent() {
    ESP_LOGI(TAG, "Network connected");
    auto state = GetDeviceState();

    if (state == kDeviceStateStarting || state == kDeviceStateWifiConfiguring) {
        // Network is ready, start activation
        SetDeviceState(kDeviceStateActivating);
        if (activation_task_handle_ != nullptr) {
            ESP_LOGW(TAG, "Activation task already running");
            return;
        }

        xTaskCreate([](void* arg) {
            Application* app = static_cast<Application*>(arg);
            app->ActivationTask();
            app->activation_task_handle_ = nullptr;
            vTaskDelete(NULL);
        }, "activation", 4096 * 2, this, 2, &activation_task_handle_);
    }

    // Update the status bar immediately to show the network state
    auto display = Board::GetInstance().GetDisplay();
    display->UpdateStatusBar(true);
}

void Application::HandleNetworkDisconnectedEvent() {
    // Close current conversation when network disconnected
    auto state = GetDeviceState();
    if (state == kDeviceStateConnecting || state == kDeviceStateListening || state == kDeviceStateSpeaking) {
        ESP_LOGI(TAG, "Closing audio channel due to network disconnection");
        protocol_->CloseAudioChannel();
    }

    // Update the status bar immediately to show the network state
    auto display = Board::GetInstance().GetDisplay();
    display->UpdateStatusBar(true);
}

void Application::HandleActivationDoneEvent() {
    ESP_LOGI(TAG, "Activation done");

    SystemInfo::PrintHeapStats();
    SetDeviceState(kDeviceStateIdle);

    has_server_time_ = ota_->HasServerTime();

    auto display = Board::GetInstance().GetDisplay();
    std::string message = std::string(Lang::Strings::VERSION) + ota_->GetCurrentVersion();
    display->ShowNotification(message.c_str());
    display->SetChatMessage("system", "");

    // Play the success sound to indicate the device is ready
    audio_service_.PlaySound(Lang::Sounds::OGG_SUCCESS);

    // Release OTA object after activation is complete
    ota_.reset();
    auto& board = Board::GetInstance();
    board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
}

void Application::ActivationTask() {
    // Create OTA object for activation process
    ota_ = std::make_unique<Ota>();

    // Check for new assets version
    CheckAssetsVersion();

    // Check for new firmware version
    CheckNewVersion();

    // Initialize the protocol
    InitializeProtocol();

    // Signal completion to main loop
    xEventGroupSetBits(event_group_, MAIN_EVENT_ACTIVATION_DONE);
}

void Application::CheckAssetsVersion() {
    // Only allow CheckAssetsVersion to be called once
    if (assets_version_checked_) {
        return;
    }
    assets_version_checked_ = true;

    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto& assets = Assets::GetInstance();

    if (!assets.partition_valid()) {
        ESP_LOGW(TAG, "Assets partition is disabled for board %s", BOARD_NAME);
        return;
    }
    
    Settings settings("assets", true);
    // Check if there is a new assets need to be downloaded
    std::string download_url = settings.GetString("download_url");

    if (!download_url.empty()) {
        settings.EraseKey("download_url");

        char message[256];
        snprintf(message, sizeof(message), Lang::Strings::FOUND_NEW_ASSETS, download_url.c_str());
        Alert(Lang::Strings::LOADING_ASSETS, message, "cloud_arrow_down", Lang::Sounds::OGG_UPGRADE);
        
        // Wait for the audio service to be idle for 3 seconds
        vTaskDelay(pdMS_TO_TICKS(3000));
        SetDeviceState(kDeviceStateUpgrading);
        board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
        display->SetChatMessage("system", Lang::Strings::PLEASE_WAIT);

        bool success = assets.Download(download_url, [display](int progress, size_t speed) -> void {
            std::thread([display, progress, speed]() {
                char buffer[32];
                snprintf(buffer, sizeof(buffer), "%d%% %uKB/s", progress, speed / 1024);
                display->SetChatMessage("system", buffer);
            }).detach();
        });

        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
        vTaskDelay(pdMS_TO_TICKS(1000));

        if (!success) {
            Alert(Lang::Strings::ERROR, Lang::Strings::DOWNLOAD_ASSETS_FAILED, "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
            vTaskDelay(pdMS_TO_TICKS(2000));
            SetDeviceState(kDeviceStateActivating);
            return;
        }
    }

    // Apply assets
    assets.Apply();
    display->SetChatMessage("system", "");
    display->SetEmotion("microchip_ai");
}

void Application::CheckNewVersion() {
    const int MAX_RETRY = 10;
    int retry_count = 0;
    int retry_delay = 10; // Initial retry delay in seconds

    auto& board = Board::GetInstance();
    while (true) {
        auto display = board.GetDisplay();
        display->SetStatus(Lang::Strings::CHECKING_NEW_VERSION);

        esp_err_t err = ota_->CheckVersion();
        if (err != ESP_OK) {
            retry_count++;
            if (retry_count >= MAX_RETRY) {
                ESP_LOGE(TAG, "Too many retries, exit version check");
                return;
            }

            char error_message[128];
            snprintf(error_message, sizeof(error_message), "code=%d, url=%s", err, ota_->GetCheckVersionUrl().c_str());
            char buffer[256];
            snprintf(buffer, sizeof(buffer), Lang::Strings::CHECK_NEW_VERSION_FAILED, retry_delay, error_message);
            Alert(Lang::Strings::ERROR, buffer, "cloud_slash", Lang::Sounds::OGG_EXCLAMATION);

            ESP_LOGW(TAG, "Check new version failed, retry in %d seconds (%d/%d)", retry_delay, retry_count, MAX_RETRY);
            for (int i = 0; i < retry_delay; i++) {
                vTaskDelay(pdMS_TO_TICKS(1000));
                if (GetDeviceState() == kDeviceStateIdle) {
                    break;
                }
            }
            retry_delay *= 2; // Double the retry delay
            continue;
        }
        retry_count = 0;
        retry_delay = 10; // Reset retry delay

        if (ota_->HasNewVersion()) {
            if (UpgradeFirmware(ota_->GetFirmwareUrl(), ota_->GetFirmwareVersion())) {
                return; // This line will never be reached after reboot
            }
            // If upgrade failed, continue to normal operation
        }

        // No new version, mark the current version as valid
        ota_->MarkCurrentVersionValid();
        if (!ota_->HasActivationCode() && !ota_->HasActivationChallenge()) {
            // Exit the loop if done checking new version
            break;
        }

        display->SetStatus(Lang::Strings::ACTIVATION);
        // Activation code is shown to the user and waiting for the user to input
        if (ota_->HasActivationCode()) {
            ShowActivationCode(ota_->GetActivationCode(), ota_->GetActivationMessage());
        }

        // This will block the loop until the activation is done or timeout
        for (int i = 0; i < 10; ++i) {
            ESP_LOGI(TAG, "Activating... %d/%d", i + 1, 10);
            esp_err_t err = ota_->Activate();
            if (err == ESP_OK) {
                break;
            } else if (err == ESP_ERR_TIMEOUT) {
                vTaskDelay(pdMS_TO_TICKS(3000));
            } else {
                vTaskDelay(pdMS_TO_TICKS(10000));
            }
            if (GetDeviceState() == kDeviceStateIdle) {
                break;
            }
        }
    }
}

void Application::InitializeProtocol() {
    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto codec = board.GetAudioCodec();

    display->SetStatus(Lang::Strings::LOADING_PROTOCOL);

    if (ota_->HasMqttConfig()) {
        protocol_ = std::make_unique<MqttProtocol>();
    } else if (ota_->HasWebsocketConfig()) {
        protocol_ = std::make_unique<WebsocketProtocol>();
    } else {
        ESP_LOGW(TAG, "No protocol specified in the OTA config, using MQTT");
        protocol_ = std::make_unique<MqttProtocol>();
    }

    protocol_->OnConnected([this]() {
        DismissAlert();
    });

    protocol_->OnNetworkError([this](const std::string& message) {
        last_error_message_ = message;
        xEventGroupSetBits(event_group_, MAIN_EVENT_ERROR);
    });
    
    protocol_->OnIncomingAudio([this](std::unique_ptr<AudioStreamPacket> packet) {
        if (GetDeviceState() == kDeviceStateSpeaking) {
            audio_service_.PushPacketToDecodeQueue(std::move(packet));
        }
    });
    
    protocol_->OnAudioChannelOpened([this, codec, &board]() {
        board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
        if (protocol_->server_sample_rate() != codec->output_sample_rate()) {
            ESP_LOGW(TAG, "Server sample rate %d does not match device output sample rate %d, resampling may cause distortion",
                protocol_->server_sample_rate(), codec->output_sample_rate());
        }
    });
    
    protocol_->OnAudioChannelClosed([this, &board]() {
        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
        Schedule([this]() {
            auto display = Board::GetInstance().GetDisplay();
            display->SetChatMessage("system", "");
            SetDeviceState(kDeviceStateIdle);
        });
    });
    
    protocol_->OnIncomingJson([this, display](const cJSON* root) {
        // Parse JSON data
        auto type = cJSON_GetObjectItem(root, "type");
        if (strcmp(type->valuestring, "tts") == 0) {
            auto state = cJSON_GetObjectItem(root, "state");
            if (strcmp(state->valuestring, "start") == 0) {
                Schedule([this]() {
                    aborted_ = false;
                    SetDeviceState(kDeviceStateSpeaking);
                });
            } else if (strcmp(state->valuestring, "stop") == 0) {
                Schedule([this]() {
                    if (GetDeviceState() == kDeviceStateSpeaking) {
                        if (listening_mode_ == kListeningModeManualStop) {
                            SetDeviceState(kDeviceStateIdle);
                        } else {
                            SetDeviceState(kDeviceStateListening);
                        }
                    }
                });
            } else if (strcmp(state->valuestring, "sentence_start") == 0) {
                auto text = cJSON_GetObjectItem(root, "text");
                if (cJSON_IsString(text)) {
                    ESP_LOGI(TAG, "<< %s", text->valuestring);
                    Schedule([this, display, message = std::string(text->valuestring)]() {
                        display->SetChatMessage("assistant", message.c_str());
                    });
                }
            }
        } else if (strcmp(type->valuestring, "stt") == 0) {
            auto text = cJSON_GetObjectItem(root, "text");
            if (cJSON_IsString(text)) {
                ESP_LOGI(TAG, ">> %s", text->valuestring);
                Schedule([this, display, message = std::string(text->valuestring)]() {
                    display->SetChatMessage("user", message.c_str());
                });
            }
        } else if (strcmp(type->valuestring, "llm") == 0) {
            auto emotion = cJSON_GetObjectItem(root, "emotion");
            if (cJSON_IsString(emotion)) {
                Schedule([this, display, emotion_str = std::string(emotion->valuestring)]() {
                    display->SetEmotion(emotion_str.c_str());
                });
            }
        } else if (strcmp(type->valuestring, "mcp") == 0) {
            auto payload = cJSON_GetObjectItem(root, "payload");
            if (cJSON_IsObject(payload)) {
                McpServer::GetInstance().ParseMessage(payload);
            }
        } else if (strcmp(type->valuestring, "system") == 0) {
            auto command = cJSON_GetObjectItem(root, "command");
            if (cJSON_IsString(command)) {
                ESP_LOGI(TAG, "System command: %s", command->valuestring);
                if (strcmp(command->valuestring, "reboot") == 0) {
                    // Do a reboot if user requests a OTA update
                    Schedule([this]() {
                        Reboot();
                    });
                } else {
                    ESP_LOGW(TAG, "Unknown system command: %s", command->valuestring);
                }
            }
        } else if (strcmp(type->valuestring, "alert") == 0) {
            auto status = cJSON_GetObjectItem(root, "status");
            auto message = cJSON_GetObjectItem(root, "message");
            auto emotion = cJSON_GetObjectItem(root, "emotion");
            if (cJSON_IsString(status) && cJSON_IsString(message) && cJSON_IsString(emotion)) {
                Alert(status->valuestring, message->valuestring, emotion->valuestring, Lang::Sounds::OGG_VIBRATION);
            } else {
                ESP_LOGW(TAG, "Alert command requires status, message and emotion");
            }
#if CONFIG_RECEIVE_CUSTOM_MESSAGE
        } else if (strcmp(type->valuestring, "custom") == 0) {
            auto payload = cJSON_GetObjectItem(root, "payload");
            ESP_LOGI(TAG, "Received custom message: %s", cJSON_PrintUnformatted(root));
            if (cJSON_IsObject(payload)) {
                Schedule([this, display, payload_str = std::string(cJSON_PrintUnformatted(payload))]() {
                    display->SetChatMessage("system", payload_str.c_str());
                });
            } else {
                ESP_LOGW(TAG, "Invalid custom message format: missing payload");
            }
#endif
        } else {
            ESP_LOGW(TAG, "Unknown message type: %s", type->valuestring);
        }
    });
    
    protocol_->Start();
}

void Application::ShowActivationCode(const std::string& code, const std::string& message) {
    struct digit_sound {
        char digit;
        const std::string_view& sound;
    };
    static const std::array<digit_sound, 10> digit_sounds{{
        digit_sound{'0', Lang::Sounds::OGG_0},
        digit_sound{'1', Lang::Sounds::OGG_1}, 
        digit_sound{'2', Lang::Sounds::OGG_2},
        digit_sound{'3', Lang::Sounds::OGG_3},
        digit_sound{'4', Lang::Sounds::OGG_4},
        digit_sound{'5', Lang::Sounds::OGG_5},
        digit_sound{'6', Lang::Sounds::OGG_6},
        digit_sound{'7', Lang::Sounds::OGG_7},
        digit_sound{'8', Lang::Sounds::OGG_8},
        digit_sound{'9', Lang::Sounds::OGG_9}
    }};

    // This sentence uses 9KB of SRAM, so we need to wait for it to finish
    Alert(Lang::Strings::ACTIVATION, message.c_str(), "link", Lang::Sounds::OGG_ACTIVATION);

    for (const auto& digit : code) {
        auto it = std::find_if(digit_sounds.begin(), digit_sounds.end(),
            [digit](const digit_sound& ds) { return ds.digit == digit; });
        if (it != digit_sounds.end()) {
            audio_service_.PlaySound(it->sound);
        }
    }
}

void Application::Alert(const char* status, const char* message, const char* emotion, const std::string_view& sound) {
    ESP_LOGW(TAG, "Alert [%s] %s: %s", emotion, status, message);
    auto display = Board::GetInstance().GetDisplay();
    display->SetStatus(status);
    display->SetEmotion(emotion);
    display->SetChatMessage("system", message);
    if (!sound.empty()) {
        audio_service_.PlaySound(sound);
    }
}

void Application::DismissAlert() {
    if (GetDeviceState() == kDeviceStateIdle) {
        auto display = Board::GetInstance().GetDisplay();
        display->SetStatus(Lang::Strings::STANDBY);
        display->SetEmotion("neutral");
        display->SetChatMessage("system", "");
    }
}

void Application::ToggleChatState() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_TOGGLE_CHAT);
}

void Application::StartListening() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_START_LISTENING);
}

void Application::StopListening() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_STOP_LISTENING);
}

void Application::HandleToggleChatEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateActivating) {
        SetDeviceState(kDeviceStateIdle);
        return;
    } else if (state == kDeviceStateWifiConfiguring) {
        audio_service_.EnableAudioTesting(true);
        SetDeviceState(kDeviceStateAudioTesting);
        return;
    } else if (state == kDeviceStateAudioTesting) {
        audio_service_.EnableAudioTesting(false);
        SetDeviceState(kDeviceStateWifiConfiguring);
        return;
    }

    if (!protocol_) {
        ESP_LOGE(TAG, "Protocol not initialized");
        return;
    }

    if (state == kDeviceStateIdle) {
        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                return;
            }
        }

        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonNone);
    } else if (state == kDeviceStateListening) {
        protocol_->CloseAudioChannel();
    }
}

void Application::HandleStartListeningEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateActivating) {
        SetDeviceState(kDeviceStateIdle);
        return;
    } else if (state == kDeviceStateWifiConfiguring) {
        audio_service_.EnableAudioTesting(true);
        SetDeviceState(kDeviceStateAudioTesting);
        return;
    }

    if (!protocol_) {
        ESP_LOGE(TAG, "Protocol not initialized");
        return;
    }
    
    if (state == kDeviceStateIdle) {
        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                return;
            }
        }

        SetListeningMode(kListeningModeManualStop);
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonNone);
        SetListeningMode(kListeningModeManualStop);
    }
}

void Application::HandleStopListeningEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateAudioTesting) {
        audio_service_.EnableAudioTesting(false);
        SetDeviceState(kDeviceStateWifiConfiguring);
        return;
    } else if (state == kDeviceStateListening) {
        if (protocol_) {
            protocol_->SendStopListening();
        }
        SetDeviceState(kDeviceStateIdle);
    }
}

void Application::HandleWakeWordDetectedEvent() {
    if (!protocol_) {
        return;
    }

    auto state = GetDeviceState();
    
    if (state == kDeviceStateIdle) {
        audio_service_.EncodeWakeWord();

        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                audio_service_.EnableWakeWordDetection(true);
                return;
            }
        }

        auto wake_word = audio_service_.GetLastWakeWord();
        ESP_LOGI(TAG, "Wake word detected: %s", wake_word.c_str());
#if CONFIG_SEND_WAKE_WORD_DATA
        // Encode and send the wake word data to the server
        while (auto packet = audio_service_.PopWakeWordPacket()) {
            protocol_->SendAudio(std::move(packet));
        }
        // Set the chat state to wake word detected
        protocol_->SendWakeWordDetected(wake_word);
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#else
        // Set flag to play popup sound after state changes to listening
        // (PlaySound here would be cleared by ResetDecoder in EnableVoiceProcessing)
        play_popup_on_listening_ = true;
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#endif
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonWakeWordDetected);
    } else if (state == kDeviceStateActivating) {
        // Restart the activation check if the wake word is detected during activation
        SetDeviceState(kDeviceStateIdle);
    }
}

void Application::HandleStateChangedEvent() {
    DeviceState new_state = state_machine_.GetState();
    clock_ticks_ = 0;

    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto led = board.GetLed();
    led->OnStateChanged();
    
    switch (new_state) {
        case kDeviceStateUnknown:
        case kDeviceStateIdle:
            display->SetStatus(Lang::Strings::STANDBY);
            display->SetEmotion("neutral");
            audio_service_.EnableVoiceProcessing(false);
            audio_service_.EnableWakeWordDetection(true);
            break;
        case kDeviceStateConnecting:
            display->SetStatus(Lang::Strings::CONNECTING);
            display->SetEmotion("neutral");
            display->SetChatMessage("system", "");
            break;
        case kDeviceStateListening:
            display->SetStatus(Lang::Strings::LISTENING);
            display->SetEmotion("neutral");

            // Make sure the audio processor is running
            if (!audio_service_.IsAudioProcessorRunning()) {
                // Send the start listening command
                protocol_->SendStartListening(listening_mode_);
                audio_service_.EnableVoiceProcessing(true);
                audio_service_.EnableWakeWordDetection(false);
            }

            // Play popup sound after ResetDecoder (in EnableVoiceProcessing) has been called
            if (play_popup_on_listening_) {
                play_popup_on_listening_ = false;
                audio_service_.PlaySound(Lang::Sounds::OGG_POPUP);
            }
            break;
        case kDeviceStateSpeaking:
            display->SetStatus(Lang::Strings::SPEAKING);

            // 记录 AI 开始说话的时间，用于语音打断保护
            speaking_start_time_ = esp_timer_get_time() / 1000;
            voice_detected_start_time_ = 0;  // 重置用户声音检测计时

            if (listening_mode_ != kListeningModeRealtime) {
                audio_service_.EnableVoiceProcessing(false);
                // Only AFE wake word can be detected in speaking mode
                audio_service_.EnableWakeWordDetection(audio_service_.IsAfeWakeWord());
            }
            audio_service_.ResetDecoder();
            break;
        case kDeviceStateWifiConfiguring:
            audio_service_.EnableVoiceProcessing(false);
            audio_service_.EnableWakeWordDetection(false);
            break;
        default:
            // Do nothing
            break;
    }
}

void Application::Schedule(std::function<void()>&& callback) {
    {
        std::lock_guard<std::mutex> lock(mutex_);
        main_tasks_.push_back(std::move(callback));
    }
    xEventGroupSetBits(event_group_, MAIN_EVENT_SCHEDULE);
}

void Application::AbortSpeaking(AbortReason reason) {
    ESP_LOGI(TAG, "Abort speaking");
    aborted_ = true;
    if (protocol_) {
        protocol_->SendAbortSpeaking(reason);
    }
}

void Application::SetListeningMode(ListeningMode mode) {
    listening_mode_ = mode;
    SetDeviceState(kDeviceStateListening);
}

void Application::Reboot() {
    ESP_LOGI(TAG, "Rebooting...");
    // Disconnect the audio channel
    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        protocol_->CloseAudioChannel();
    }
    protocol_.reset();
    audio_service_.Stop();

    vTaskDelay(pdMS_TO_TICKS(1000));
    esp_restart();
}

bool Application::UpgradeFirmware(const std::string& url, const std::string& version) {
    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();

    std::string upgrade_url = url;
    std::string version_info = version.empty() ? "(Manual upgrade)" : version;

    // Close audio channel if it's open
    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        ESP_LOGI(TAG, "Closing audio channel before firmware upgrade");
        protocol_->CloseAudioChannel();
    }
    ESP_LOGI(TAG, "Starting firmware upgrade from URL: %s", upgrade_url.c_str());

    Alert(Lang::Strings::OTA_UPGRADE, Lang::Strings::UPGRADING, "download", Lang::Sounds::OGG_UPGRADE);
    vTaskDelay(pdMS_TO_TICKS(3000));

    SetDeviceState(kDeviceStateUpgrading);

    std::string message = std::string(Lang::Strings::NEW_VERSION) + version_info;
    display->SetChatMessage("system", message.c_str());

    board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
    audio_service_.Stop();
    vTaskDelay(pdMS_TO_TICKS(1000));

    bool upgrade_success = Ota::Upgrade(upgrade_url, [display](int progress, size_t speed) {
        std::thread([display, progress, speed]() {
            char buffer[32];
            snprintf(buffer, sizeof(buffer), "%d%% %uKB/s", progress, speed / 1024);
            display->SetChatMessage("system", buffer);
        }).detach();
    });

    if (!upgrade_success) {
        // Upgrade failed, restart audio service and continue running
        ESP_LOGE(TAG, "Firmware upgrade failed, restarting audio service and continuing operation...");
        audio_service_.Start(); // Restart audio service
        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER); // Restore power save level
        Alert(Lang::Strings::ERROR, Lang::Strings::UPGRADE_FAILED, "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
        vTaskDelay(pdMS_TO_TICKS(3000));
        return false;
    } else {
        // Upgrade success, reboot immediately
        ESP_LOGI(TAG, "Firmware upgrade successful, rebooting...");
        display->SetChatMessage("system", "Upgrade successful, rebooting...");
        vTaskDelay(pdMS_TO_TICKS(1000)); // Brief pause to show message
        Reboot();
        return true;
    }
}

void Application::WakeWordInvoke(const std::string& wake_word) {
    if (!protocol_) {
        return;
    }

    auto state = GetDeviceState();
    
    if (state == kDeviceStateIdle) {
        audio_service_.EncodeWakeWord();

        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                audio_service_.EnableWakeWordDetection(true);
                return;
            }
        }

        ESP_LOGI(TAG, "Wake word detected: %s", wake_word.c_str());
#if CONFIG_USE_AFE_WAKE_WORD || CONFIG_USE_CUSTOM_WAKE_WORD
        // Encode and send the wake word data to the server
        while (auto packet = audio_service_.PopWakeWordPacket()) {
            protocol_->SendAudio(std::move(packet));
        }
        // Set the chat state to wake word detected
        protocol_->SendWakeWordDetected(wake_word);
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#else
        // Set flag to play popup sound after state changes to listening
        // (PlaySound here would be cleared by ResetDecoder in EnableVoiceProcessing)
        play_popup_on_listening_ = true;
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#endif
    } else if (state == kDeviceStateSpeaking) {
        Schedule([this]() {
            AbortSpeaking(kAbortReasonNone);
        });
    } else if (state == kDeviceStateListening) {   
        Schedule([this]() {
            if (protocol_) {
                protocol_->CloseAudioChannel();
            }
        });
    }
}

bool Application::CanEnterSleepMode() {
    if (GetDeviceState() != kDeviceStateIdle) {
        return false;
    }

    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        return false;
    }

    if (!audio_service_.IsIdle()) {
        return false;
    }

    // Now it is safe to enter sleep mode
    return true;
}

void Application::SendMcpMessage(const std::string& payload) {
    // Always schedule to run in main task for thread safety
    Schedule([this, payload = std::move(payload)]() {
        if (protocol_) {
            protocol_->SendMcpMessage(payload);
        }
    });
}

void Application::SetAecMode(AecMode mode) {
    aec_mode_ = mode;
    Schedule([this]() {
        auto& board = Board::GetInstance();
        auto display = board.GetDisplay();
        switch (aec_mode_) {
        case kAecOff:
            audio_service_.EnableDeviceAec(false);
            display->ShowNotification(Lang::Strings::RTC_MODE_OFF);
            break;
        case kAecOnServerSide:
            audio_service_.EnableDeviceAec(false);
            display->ShowNotification(Lang::Strings::RTC_MODE_ON);
            break;
        case kAecOnDeviceSide:
            audio_service_.EnableDeviceAec(true);
            display->ShowNotification(Lang::Strings::RTC_MODE_ON);
            break;
        }

        // If the AEC mode is changed, close the audio channel
        if (protocol_ && protocol_->IsAudioChannelOpened()) {
            protocol_->CloseAudioChannel();
        }
    });
}

void Application::PlaySound(const std::string_view& sound) {
    audio_service_.PlaySound(sound);
}

void Application::ResetProtocol() {
    Schedule([this]() {
        // Close audio channel if opened
        if (protocol_ && protocol_->IsAudioChannelOpened()) {
            protocol_->CloseAudioChannel();
        }
        // Reset protocol
        protocol_.reset();
    });
}


```

代码修改完成！让我展示一下修改的内容：

---

## 修改完成！添加了防抖动保护

### 修改内容

**1. 新增变量** (`application.h`)：
- `speaking_start_time_` - 记录 AI 开始说话的时间
- `voice_detected_start_time_` - 记录检测到用户声音的起始时间

**2. 防抖动逻辑** (`application.cc`)：

打断需要**同时满足两个条件**：

| 条件 | 参数 | 说明 |
|------|------|------|
| AI 说话保护期 | 1000ms | AI 开始说话后 1 秒内不会被打断 |
| 声音持续时间 | 500ms | 用户需要持续说话超过 0.5 秒才触发打断 |

这样可以过滤掉：
- ✅ AI 刚开始说话时的回声
- ✅ 短暂的噪声/环境声
- ✅ 偶然的声音波动

---

## 重新编译测试

```bash
cd /home/lei/xiaozhi-esp32
idf.py build flash monitor
```

### 测试时观察

1. **AI 开始回复** - 前 1 秒不应该被打断
2. **持续说话 0.5 秒以上** - 应该触发打断
3. **短暂的声音** - 不应该触发打断

### 日志观察

如果打断成功触发，会看到：
```
I (xxxxx) Application: Voice interrupt triggered (speaking: 1500ms, voice: 600ms)
I (xxxxx) Application: Abort speaking
```

---

## 如果效果仍不好

可以调整两个参数：

```cpp
const int64_t SPEAKING_PROTECTION_MS = 1000;  // 增大这个值，AI 说话保护期更长
const int64_t VOICE_SUSTAIN_MS = 500;         // 增大这个值，需要更长时间的持续说话才触发
```

比如改成 `2000` 和 `800`，会更不容易误触发。

试试看效果怎么样？

```shellscript
cd /home/lei/xiaozhi-esp32
idf.py build flash monitor
```

```plaintext
I (xxxxx) Application: Voice interrupt triggered (speaking: 1500ms, voice: 600ms)
I (xxxxx) Application: Abort speaking
```

```cpp
const int64_t SPEAKING_PROTECTION_MS = 1000;  // 增大这个值，AI 说话保护期更长
const int64_t VOICE_SUSTAIN_MS = 500;         // 增大这个值，需要更长时间的持续说话才触发
```

---

**User**

等一下，这个aec的逻辑不是vad检测音频帧与参考音频进行比对吗，为啥会误触发，你这次优化修改的原理又是什么

---

**Cursor**

非常好的问题！让我解释清楚这个关键的误解：

---

## 服务端 AEC vs 设备端 AEC 的区别

### 你启用的是"服务端 AEC"

当你勾选 `Enable Server-Side AEC` 时，代码中的情况是：

```cpp
// afe_audio_processor.cc
#ifdef CONFIG_USE_DEVICE_AEC      // ← 你没有启用这个！
    afe_config->aec_init = true;   // 设备端做 AEC
    afe_config->vad_init = false;
#else                              // ← 你走的是这个分支
    afe_config->aec_init = false;  // 设备端不做 AEC！
    afe_config->vad_init = true;   // 设备端只做 VAD
#endif
```

**关键点**：服务端 AEC 意味着 **AEC 在服务器上执行，不是在设备上执行！**

---

## 两种 AEC 的工作流程对比

### 设备端 AEC（你的硬件不支持）

```
┌─────────────────────────────────────────────────────────────┐
│ 麦克风信号 ──┐                                              │
│              ↓                                              │
│         [设备端 AEC] ←── 扬声器参考信号                    │
│              ↓                                              │
│        纯净的用户声音                                       │
│              ↓                                              │
│         [设备端 VAD] ── 检测用户是否说话 ── 准确！        │
└─────────────────────────────────────────────────────────────┘
```

### 服务端 AEC（你启用的）

```
┌─────────────────────────────────────────────────────────────┐
│ 麦克风信号（含回声）──┬──→ [设备端 VAD] ── 会被回声干扰！  │
│                       │                                      │
│                       └──→ 发送到服务器 + 时间戳            │
│                                   ↓                          │
│                            [服务器端 AEC]                    │
│                                   ↓                          │
│                         纯净信号 → ASR 识别                  │
└─────────────────────────────────────────────────────────────┘
```

**服务端 AEC 的目的**：让服务器能正确识别你说的话（语音转文字）

**服务端 AEC 不能做的事**：让设备端的 VAD 不被回声干扰

---

## 为什么会误触发？

因为在服务端 AEC 模式下：
- **设备端不做 AEC**（`aec_init = false`）
- **设备端的 VAD 运行在未经处理的麦克风信号上**
- 麦克风信号里包含扬声器的回声
- VAD 把回声当作"有人说话"→ 误触发打断

---

## 我的修改原理

由于设备端没有真正的 AEC，我只能用**时间保护**来减少误触发：

```
不是真正过滤回声，而是假设：
1. 回声只会在 AI 刚开始说话时被检测到（前 1 秒保护）
2. 回声导致的 VAD 检测是不稳定的、断断续续的（需要持续 500ms）
3. 用户真正说话会比较稳定、持续
```

这是一个 **workaround（变通方案）**，不是真正的 AEC 解决方案。

---

## 真正的解决方案

### 方案 1：使用设备端 AEC（需要硬件支持）

需要的硬件条件：
- 音频编解码器支持回采（如 ES8311 + ES7210）
- 或者有软件回采的实现

你的通用面包板不满足这个条件。

### 方案 2：基于音量阈值的检测

假设用户打断时会说得比回声更大声：

```cpp
// 检测音量是否超过阈值
if (audio_volume > INTERRUPT_THRESHOLD) {
    // 触发打断
}
```

---

## 你想怎么做？

1. **继续用当前的时间保护方案** - 调整参数看效果
2. **我帮你实现音量阈值检测** - 可能效果更好
3. **升级硬件** - 使用支持设备端 AEC 的开发板

你倾向于哪个方案？

```cpp
// afe_audio_processor.cc
#ifdef CONFIG_USE_DEVICE_AEC      // ← 你没有启用这个！
    afe_config->aec_init = true;   // 设备端做 AEC
    afe_config->vad_init = false;
#else                              // ← 你走的是这个分支
    afe_config->aec_init = false;  // 设备端不做 AEC！
    afe_config->vad_init = true;   // 设备端只做 VAD
#endif
```

```plaintext
┌─────────────────────────────────────────────────────────────┐
│ 麦克风信号 ──┐                                              │
│              ↓                                              │
│         [设备端 AEC] ←── 扬声器参考信号                    │
│              ↓                                              │
│        纯净的用户声音                                       │
│              ↓                                              │
│         [设备端 VAD] ── 检测用户是否说话 ── 准确！        │
└─────────────────────────────────────────────────────────────┘
```

```plaintext
┌─────────────────────────────────────────────────────────────┐
│ 麦克风信号（含回声）──┬──→ [设备端 VAD] ── 会被回声干扰！  │
│                       │                                      │
│                       └──→ 发送到服务器 + 时间戳            │
│                                   ↓                          │
│                            [服务器端 AEC]                    │
│                                   ↓                          │
│                         纯净信号 → ASR 识别                  │
└─────────────────────────────────────────────────────────────┘
```

```plaintext
不是真正过滤回声，而是假设：
1. 回声只会在 AI 刚开始说话时被检测到（前 1 秒保护）
2. 回声导致的 VAD 检测是不稳定的、断断续续的（需要持续 500ms）
3. 用户真正说话会比较稳定、持续
```

```cpp
// 检测音量是否超过阈值
if (audio_volume > INTERRUPT_THRESHOLD) {
    // 触发打断
}
```

---

**User**

好的，我已经尝试，请你看monitor监控窗口结果@ESP-IDF Monitor  ，有打断效果，但是误判率还是有一点点高，你注意到没

---

**Cursor**

我看到日志了！确实有一些误判的情况。让我分析一下：

## 日志分析

### 可能的误判例子：

1. **第 337-339 行**：
   ```
   I (36785) Application: << 我找到几首好听的歌...
   I (37245) Application: >> 好的我。   ← 这像是被误触发的识别
   I (37495) MQTT: Received goodbye message
   ```

2. **第 371-372 行**：
   ```
   I (94145) StateMachine: State: listening -> speaking
   I (94155) Application: >> 我的是。   ← 不完整的话，可能是误触发
   ```

3. **第 375-376 行**：
   ```
   I (96545) StateMachine: State: listening -> speaking
   I (96545) Application: >> 您说的是。  ← 同样不完整
   ```

这些 `>> xxx` 是服务端 ASR 识别到的内容，说明回声被当作用户语音发送到服务端了。

---

## 优化方案：增加保护参数

让我调整防抖动参数，让打断更不容易被误触发：

```plaintext
   I (36785) Application: << 我找到几首好听的歌...
   I (37245) Application: >> 好的我。   ← 这像是被误触发的识别
   I (37495) MQTT: Received goodbye message
```

```plaintext
   I (94145) StateMachine: State: listening -> speaking
   I (94155) Application: >> 我的是。   ← 不完整的话，可能是误触发
```

```plaintext
   I (96545) StateMachine: State: listening -> speaking
   I (96545) Application: >> 您说的是。  ← 同样不完整
```

我注意到日志中**没有看到** `"Voice interrupt triggered"` 的日志，这说明实际的打断可能不是来自我添加的代码，而是**服务端主动检测**的！

在实时模式下，服务端会持续接收音频并做 ASR，当服务端检测到"声音"（包括回声）时，会停止 TTS。

不过，让我增加保护参数，减少误触发：

```cpp
#include "application.h"
#include "board.h"
#include "display.h"
#include "system_info.h"
#include "audio_codec.h"
#include "mqtt_protocol.h"
#include "websocket_protocol.h"
#include "assets/lang_config.h"
#include "mcp_server.h"
#include "assets.h"
#include "settings.h"

#include <cstring>
#include <esp_log.h>
#include <cJSON.h>
#include <driver/gpio.h>
#include <arpa/inet.h>
#include <font_awesome.h>

#define TAG "Application"


Application::Application() {
    event_group_ = xEventGroupCreate();

#if CONFIG_USE_DEVICE_AEC && CONFIG_USE_SERVER_AEC
#error "CONFIG_USE_DEVICE_AEC and CONFIG_USE_SERVER_AEC cannot be enabled at the same time"
#elif CONFIG_USE_DEVICE_AEC
    aec_mode_ = kAecOnDeviceSide;
#elif CONFIG_USE_SERVER_AEC
    aec_mode_ = kAecOnServerSide;
#else
    aec_mode_ = kAecOff;
#endif

    esp_timer_create_args_t clock_timer_args = {
        .callback = [](void* arg) {
            Application* app = (Application*)arg;
            xEventGroupSetBits(app->event_group_, MAIN_EVENT_CLOCK_TICK);
        },
        .arg = this,
        .dispatch_method = ESP_TIMER_TASK,
        .name = "clock_timer",
        .skip_unhandled_events = true
    };
    esp_timer_create(&clock_timer_args, &clock_timer_handle_);
}

Application::~Application() {
    if (clock_timer_handle_ != nullptr) {
        esp_timer_stop(clock_timer_handle_);
        esp_timer_delete(clock_timer_handle_);
    }
    vEventGroupDelete(event_group_);
}

bool Application::SetDeviceState(DeviceState state) {
    return state_machine_.TransitionTo(state);
}

void Application::Initialize() {
    auto& board = Board::GetInstance();
    SetDeviceState(kDeviceStateStarting);

    // Setup the display
    auto display = board.GetDisplay();

    // Print board name/version info
    display->SetChatMessage("system", SystemInfo::GetUserAgent().c_str());

    // Setup the audio service
    auto codec = board.GetAudioCodec();
    audio_service_.Initialize(codec);
    audio_service_.Start();

    AudioServiceCallbacks callbacks;
    callbacks.on_send_queue_available = [this]() {
        xEventGroupSetBits(event_group_, MAIN_EVENT_SEND_AUDIO);
    };
    callbacks.on_wake_word_detected = [this](const std::string& wake_word) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_WAKE_WORD_DETECTED);
    };
    callbacks.on_vad_change = [this](bool speaking) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_VAD_CHANGE);
    };
    audio_service_.SetCallbacks(callbacks);

    // Add state change listeners
    state_machine_.AddStateChangeListener([this](DeviceState old_state, DeviceState new_state) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_STATE_CHANGED);
    });

    // Start the clock timer to update the status bar
    esp_timer_start_periodic(clock_timer_handle_, 1000000);

    // Add MCP common tools (only once during initialization)
    auto& mcp_server = McpServer::GetInstance();
    mcp_server.AddCommonTools();
    mcp_server.AddUserOnlyTools();

    // Set network event callback for UI updates and network state handling
    board.SetNetworkEventCallback([this](NetworkEvent event, const std::string& data) {
        auto display = Board::GetInstance().GetDisplay();
        
        switch (event) {
            case NetworkEvent::Scanning:
                display->ShowNotification(Lang::Strings::SCANNING_WIFI, 30000);
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_DISCONNECTED);
                break;
            case NetworkEvent::Connecting: {
                if (data.empty()) {
                    // Cellular network - registering without carrier info yet
                    display->SetStatus(Lang::Strings::REGISTERING_NETWORK);
                } else {
                    // WiFi or cellular with carrier info
                    std::string msg = Lang::Strings::CONNECT_TO;
                    msg += data;
                    msg += "...";
                    display->ShowNotification(msg.c_str(), 30000);
                }
                break;
            }
            case NetworkEvent::Connected: {
                std::string msg = Lang::Strings::CONNECTED_TO;
                msg += data;
                display->ShowNotification(msg.c_str(), 30000);
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_CONNECTED);
                break;
            }
            case NetworkEvent::Disconnected:
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_DISCONNECTED);
                break;
            case NetworkEvent::WifiConfigModeEnter:
                // WiFi config mode enter is handled by WifiBoard internally
                break;
            case NetworkEvent::WifiConfigModeExit:
                // WiFi config mode exit is handled by WifiBoard internally
                break;
            // Cellular modem specific events
            case NetworkEvent::ModemDetecting:
                display->SetStatus(Lang::Strings::DETECTING_MODULE);
                break;
            case NetworkEvent::ModemErrorNoSim:
                Alert(Lang::Strings::ERROR, Lang::Strings::PIN_ERROR, "triangle_exclamation", Lang::Sounds::OGG_ERR_PIN);
                break;
            case NetworkEvent::ModemErrorRegDenied:
                Alert(Lang::Strings::ERROR, Lang::Strings::REG_ERROR, "triangle_exclamation", Lang::Sounds::OGG_ERR_REG);
                break;
            case NetworkEvent::ModemErrorInitFailed:
                display->SetStatus(Lang::Strings::DETECTING_MODULE);
                display->SetChatMessage("system", Lang::Strings::DETECTING_MODULE);
                break;
            case NetworkEvent::ModemErrorTimeout:
                display->SetStatus(Lang::Strings::REGISTERING_NETWORK);
                break;
        }
    });

    // Start network asynchronously
    board.StartNetwork();

    // Update the status bar immediately to show the network state
    display->UpdateStatusBar(true);
}

void Application::Run() {
    const EventBits_t ALL_EVENTS = 
        MAIN_EVENT_SCHEDULE |
        MAIN_EVENT_SEND_AUDIO |
        MAIN_EVENT_WAKE_WORD_DETECTED |
        MAIN_EVENT_VAD_CHANGE |
        MAIN_EVENT_CLOCK_TICK |
        MAIN_EVENT_ERROR |
        MAIN_EVENT_NETWORK_CONNECTED |
        MAIN_EVENT_NETWORK_DISCONNECTED |
        MAIN_EVENT_TOGGLE_CHAT |
        MAIN_EVENT_START_LISTENING |
        MAIN_EVENT_STOP_LISTENING |
        MAIN_EVENT_ACTIVATION_DONE |
        MAIN_EVENT_STATE_CHANGED;

    while (true) {
        auto bits = xEventGroupWaitBits(event_group_, ALL_EVENTS, pdTRUE, pdFALSE, portMAX_DELAY);

        if (bits & MAIN_EVENT_ERROR) {
            SetDeviceState(kDeviceStateIdle);
            Alert(Lang::Strings::ERROR, last_error_message_.c_str(), "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
        }

        if (bits & MAIN_EVENT_NETWORK_CONNECTED) {
            HandleNetworkConnectedEvent();
        }

        if (bits & MAIN_EVENT_NETWORK_DISCONNECTED) {
            HandleNetworkDisconnectedEvent();
        }

        if (bits & MAIN_EVENT_ACTIVATION_DONE) {
            HandleActivationDoneEvent();
        }

        if (bits & MAIN_EVENT_STATE_CHANGED) {
            HandleStateChangedEvent();
        }

        if (bits & MAIN_EVENT_TOGGLE_CHAT) {
            HandleToggleChatEvent();
        }

        if (bits & MAIN_EVENT_START_LISTENING) {
            HandleStartListeningEvent();
        }

        if (bits & MAIN_EVENT_STOP_LISTENING) {
            HandleStopListeningEvent();
        }

        if (bits & MAIN_EVENT_SEND_AUDIO) {
            while (auto packet = audio_service_.PopPacketFromSendQueue()) {
                if (protocol_ && !protocol_->SendAudio(std::move(packet))) {
                    break;
                }
            }
        }

        if (bits & MAIN_EVENT_WAKE_WORD_DETECTED) {
            HandleWakeWordDetectedEvent();
        }

        if (bits & MAIN_EVENT_VAD_CHANGE) {
            auto state = GetDeviceState();
            if (state == kDeviceStateListening) {
                auto led = Board::GetInstance().GetLed();
                led->OnStateChanged();
            } else if (state == kDeviceStateSpeaking && listening_mode_ == kListeningModeRealtime) {
                // 在实时模式下，检测用户语音打断
                // 防抖动保护：需要满足以下条件才触发打断
                // 1. AI 说话已超过一定时间（避免刚开始说话就被打断）
                // 2. 持续检测到用户声音超过一定时间（避免短暂噪声/回声触发）
                
                const int64_t SPEAKING_PROTECTION_MS = 1500;  // AI 说话保护期（增加到1.5秒）
                const int64_t VOICE_SUSTAIN_MS = 800;         // 用户声音持续时间要求（增加到0.8秒）
                
                int64_t now = esp_timer_get_time() / 1000;  // 转换为毫秒
                
                if (audio_service_.IsVoiceDetected()) {
                    // 检测到声音
                    if (voice_detected_start_time_ == 0) {
                        voice_detected_start_time_ = now;
                        ESP_LOGD(TAG, "Voice detected, starting timer...");
                    }
                    
                    int64_t speaking_duration = now - speaking_start_time_;
                    int64_t voice_duration = now - voice_detected_start_time_;
                    
                    // 检查是否满足打断条件
                    if (speaking_duration > SPEAKING_PROTECTION_MS && 
                        voice_duration > VOICE_SUSTAIN_MS) {
                        ESP_LOGI(TAG, "Voice interrupt triggered (speaking: %lldms, voice: %lldms)", 
                                 speaking_duration, voice_duration);
                        voice_detected_start_time_ = 0;  // 重置
                        AbortSpeaking(kAbortReasonNone);
                    }
                } else {
                    // 声音停止，重置计时器
                    if (voice_detected_start_time_ != 0) {
                        ESP_LOGD(TAG, "Voice stopped, resetting timer");
                        voice_detected_start_time_ = 0;
                    }
                }
            }
        }

        if (bits & MAIN_EVENT_SCHEDULE) {
            std::unique_lock<std::mutex> lock(mutex_);
            auto tasks = std::move(main_tasks_);
            lock.unlock();
            for (auto& task : tasks) {
                task();
            }
        }

        if (bits & MAIN_EVENT_CLOCK_TICK) {
            clock_ticks_++;
            auto display = Board::GetInstance().GetDisplay();
            display->UpdateStatusBar();
        
            // Print debug info every 10 seconds
            if (clock_ticks_ % 10 == 0) {
                SystemInfo::PrintHeapStats();
            }
        }
    }
}

void Application::HandleNetworkConnectedEvent() {
    ESP_LOGI(TAG, "Network connected");
    auto state = GetDeviceState();

    if (state == kDeviceStateStarting || state == kDeviceStateWifiConfiguring) {
        // Network is ready, start activation
        SetDeviceState(kDeviceStateActivating);
        if (activation_task_handle_ != nullptr) {
            ESP_LOGW(TAG, "Activation task already running");
            return;
        }

        xTaskCreate([](void* arg) {
            Application* app = static_cast<Application*>(arg);
            app->ActivationTask();
            app->activation_task_handle_ = nullptr;
            vTaskDelete(NULL);
        }, "activation", 4096 * 2, this, 2, &activation_task_handle_);
    }

    // Update the status bar immediately to show the network state
    auto display = Board::GetInstance().GetDisplay();
    display->UpdateStatusBar(true);
}

void Application::HandleNetworkDisconnectedEvent() {
    // Close current conversation when network disconnected
    auto state = GetDeviceState();
    if (state == kDeviceStateConnecting || state == kDeviceStateListening || state == kDeviceStateSpeaking) {
        ESP_LOGI(TAG, "Closing audio channel due to network disconnection");
        protocol_->CloseAudioChannel();
    }

    // Update the status bar immediately to show the network state
    auto display = Board::GetInstance().GetDisplay();
    display->UpdateStatusBar(true);
}

void Application::HandleActivationDoneEvent() {
    ESP_LOGI(TAG, "Activation done");

    SystemInfo::PrintHeapStats();
    SetDeviceState(kDeviceStateIdle);

    has_server_time_ = ota_->HasServerTime();

    auto display = Board::GetInstance().GetDisplay();
    std::string message = std::string(Lang::Strings::VERSION) + ota_->GetCurrentVersion();
    display->ShowNotification(message.c_str());
    display->SetChatMessage("system", "");

    // Play the success sound to indicate the device is ready
    audio_service_.PlaySound(Lang::Sounds::OGG_SUCCESS);

    // Release OTA object after activation is complete
    ota_.reset();
    auto& board = Board::GetInstance();
    board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
}

void Application::ActivationTask() {
    // Create OTA object for activation process
    ota_ = std::make_unique<Ota>();

    // Check for new assets version
    CheckAssetsVersion();

    // Check for new firmware version
    CheckNewVersion();

    // Initialize the protocol
    InitializeProtocol();

    // Signal completion to main loop
    xEventGroupSetBits(event_group_, MAIN_EVENT_ACTIVATION_DONE);
}

void Application::CheckAssetsVersion() {
    // Only allow CheckAssetsVersion to be called once
    if (assets_version_checked_) {
        return;
    }
    assets_version_checked_ = true;

    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto& assets = Assets::GetInstance();

    if (!assets.partition_valid()) {
        ESP_LOGW(TAG, "Assets partition is disabled for board %s", BOARD_NAME);
        return;
    }
    
    Settings settings("assets", true);
    // Check if there is a new assets need to be downloaded
    std::string download_url = settings.GetString("download_url");

    if (!download_url.empty()) {
        settings.EraseKey("download_url");

        char message[256];
        snprintf(message, sizeof(message), Lang::Strings::FOUND_NEW_ASSETS, download_url.c_str());
        Alert(Lang::Strings::LOADING_ASSETS, message, "cloud_arrow_down", Lang::Sounds::OGG_UPGRADE);
        
        // Wait for the audio service to be idle for 3 seconds
        vTaskDelay(pdMS_TO_TICKS(3000));
        SetDeviceState(kDeviceStateUpgrading);
        board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
        display->SetChatMessage("system", Lang::Strings::PLEASE_WAIT);

        bool success = assets.Download(download_url, [display](int progress, size_t speed) -> void {
            std::thread([display, progress, speed]() {
                char buffer[32];
                snprintf(buffer, sizeof(buffer), "%d%% %uKB/s", progress, speed / 1024);
                display->SetChatMessage("system", buffer);
            }).detach();
        });

        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
        vTaskDelay(pdMS_TO_TICKS(1000));

        if (!success) {
            Alert(Lang::Strings::ERROR, Lang::Strings::DOWNLOAD_ASSETS_FAILED, "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
            vTaskDelay(pdMS_TO_TICKS(2000));
            SetDeviceState(kDeviceStateActivating);
            return;
        }
    }

    // Apply assets
    assets.Apply();
    display->SetChatMessage("system", "");
    display->SetEmotion("microchip_ai");
}

void Application::CheckNewVersion() {
    const int MAX_RETRY = 10;
    int retry_count = 0;
    int retry_delay = 10; // Initial retry delay in seconds

    auto& board = Board::GetInstance();
    while (true) {
        auto display = board.GetDisplay();
        display->SetStatus(Lang::Strings::CHECKING_NEW_VERSION);

        esp_err_t err = ota_->CheckVersion();
        if (err != ESP_OK) {
            retry_count++;
            if (retry_count >= MAX_RETRY) {
                ESP_LOGE(TAG, "Too many retries, exit version check");
                return;
            }

            char error_message[128];
            snprintf(error_message, sizeof(error_message), "code=%d, url=%s", err, ota_->GetCheckVersionUrl().c_str());
            char buffer[256];
            snprintf(buffer, sizeof(buffer), Lang::Strings::CHECK_NEW_VERSION_FAILED, retry_delay, error_message);
            Alert(Lang::Strings::ERROR, buffer, "cloud_slash", Lang::Sounds::OGG_EXCLAMATION);

            ESP_LOGW(TAG, "Check new version failed, retry in %d seconds (%d/%d)", retry_delay, retry_count, MAX_RETRY);
            for (int i = 0; i < retry_delay; i++) {
                vTaskDelay(pdMS_TO_TICKS(1000));
                if (GetDeviceState() == kDeviceStateIdle) {
                    break;
                }
            }
            retry_delay *= 2; // Double the retry delay
            continue;
        }
        retry_count = 0;
        retry_delay = 10; // Reset retry delay

        if (ota_->HasNewVersion()) {
            if (UpgradeFirmware(ota_->GetFirmwareUrl(), ota_->GetFirmwareVersion())) {
                return; // This line will never be reached after reboot
            }
            // If upgrade failed, continue to normal operation
        }

        // No new version, mark the current version as valid
        ota_->MarkCurrentVersionValid();
        if (!ota_->HasActivationCode() && !ota_->HasActivationChallenge()) {
            // Exit the loop if done checking new version
            break;
        }

        display->SetStatus(Lang::Strings::ACTIVATION);
        // Activation code is shown to the user and waiting for the user to input
        if (ota_->HasActivationCode()) {
            ShowActivationCode(ota_->GetActivationCode(), ota_->GetActivationMessage());
        }

        // This will block the loop until the activation is done or timeout
        for (int i = 0; i < 10; ++i) {
            ESP_LOGI(TAG, "Activating... %d/%d", i + 1, 10);
            esp_err_t err = ota_->Activate();
            if (err == ESP_OK) {
                break;
            } else if (err == ESP_ERR_TIMEOUT) {
                vTaskDelay(pdMS_TO_TICKS(3000));
            } else {
                vTaskDelay(pdMS_TO_TICKS(10000));
            }
            if (GetDeviceState() == kDeviceStateIdle) {
                break;
            }
        }
    }
}

void Application::InitializeProtocol() {
    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto codec = board.GetAudioCodec();

    display->SetStatus(Lang::Strings::LOADING_PROTOCOL);

    if (ota_->HasMqttConfig()) {
        protocol_ = std::make_unique<MqttProtocol>();
    } else if (ota_->HasWebsocketConfig()) {
        protocol_ = std::make_unique<WebsocketProtocol>();
    } else {
        ESP_LOGW(TAG, "No protocol specified in the OTA config, using MQTT");
        protocol_ = std::make_unique<MqttProtocol>();
    }

    protocol_->OnConnected([this]() {
        DismissAlert();
    });

    protocol_->OnNetworkError([this](const std::string& message) {
        last_error_message_ = message;
        xEventGroupSetBits(event_group_, MAIN_EVENT_ERROR);
    });
    
    protocol_->OnIncomingAudio([this](std::unique_ptr<AudioStreamPacket> packet) {
        if (GetDeviceState() == kDeviceStateSpeaking) {
            audio_service_.PushPacketToDecodeQueue(std::move(packet));
        }
    });
    
    protocol_->OnAudioChannelOpened([this, codec, &board]() {
        board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
        if (protocol_->server_sample_rate() != codec->output_sample_rate()) {
            ESP_LOGW(TAG, "Server sample rate %d does not match device output sample rate %d, resampling may cause distortion",
                protocol_->server_sample_rate(), codec->output_sample_rate());
        }
    });
    
    protocol_->OnAudioChannelClosed([this, &board]() {
        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
        Schedule([this]() {
            auto display = Board::GetInstance().GetDisplay();
            display->SetChatMessage("system", "");
            SetDeviceState(kDeviceStateIdle);
        });
    });
    
    protocol_->OnIncomingJson([this, display](const cJSON* root) {
        // Parse JSON data
        auto type = cJSON_GetObjectItem(root, "type");
        if (strcmp(type->valuestring, "tts") == 0) {
            auto state = cJSON_GetObjectItem(root, "state");
            if (strcmp(state->valuestring, "start") == 0) {
                Schedule([this]() {
                    aborted_ = false;
                    SetDeviceState(kDeviceStateSpeaking);
                });
            } else if (strcmp(state->valuestring, "stop") == 0) {
                Schedule([this]() {
                    if (GetDeviceState() == kDeviceStateSpeaking) {
                        if (listening_mode_ == kListeningModeManualStop) {
                            SetDeviceState(kDeviceStateIdle);
                        } else {
                            SetDeviceState(kDeviceStateListening);
                        }
                    }
                });
            } else if (strcmp(state->valuestring, "sentence_start") == 0) {
                auto text = cJSON_GetObjectItem(root, "text");
                if (cJSON_IsString(text)) {
                    ESP_LOGI(TAG, "<< %s", text->valuestring);
                    Schedule([this, display, message = std::string(text->valuestring)]() {
                        display->SetChatMessage("assistant", message.c_str());
                    });
                }
            }
        } else if (strcmp(type->valuestring, "stt") == 0) {
            auto text = cJSON_GetObjectItem(root, "text");
            if (cJSON_IsString(text)) {
                ESP_LOGI(TAG, ">> %s", text->valuestring);
                Schedule([this, display, message = std::string(text->valuestring)]() {
                    display->SetChatMessage("user", message.c_str());
                });
            }
        } else if (strcmp(type->valuestring, "llm") == 0) {
            auto emotion = cJSON_GetObjectItem(root, "emotion");
            if (cJSON_IsString(emotion)) {
                Schedule([this, display, emotion_str = std::string(emotion->valuestring)]() {
                    display->SetEmotion(emotion_str.c_str());
                });
            }
        } else if (strcmp(type->valuestring, "mcp") == 0) {
            auto payload = cJSON_GetObjectItem(root, "payload");
            if (cJSON_IsObject(payload)) {
                McpServer::GetInstance().ParseMessage(payload);
            }
        } else if (strcmp(type->valuestring, "system") == 0) {
            auto command = cJSON_GetObjectItem(root, "command");
            if (cJSON_IsString(command)) {
                ESP_LOGI(TAG, "System command: %s", command->valuestring);
                if (strcmp(command->valuestring, "reboot") == 0) {
                    // Do a reboot if user requests a OTA update
                    Schedule([this]() {
                        Reboot();
                    });
                } else {
                    ESP_LOGW(TAG, "Unknown system command: %s", command->valuestring);
                }
            }
        } else if (strcmp(type->valuestring, "alert") == 0) {
            auto status = cJSON_GetObjectItem(root, "status");
            auto message = cJSON_GetObjectItem(root, "message");
            auto emotion = cJSON_GetObjectItem(root, "emotion");
            if (cJSON_IsString(status) && cJSON_IsString(message) && cJSON_IsString(emotion)) {
                Alert(status->valuestring, message->valuestring, emotion->valuestring, Lang::Sounds::OGG_VIBRATION);
            } else {
                ESP_LOGW(TAG, "Alert command requires status, message and emotion");
            }
#if CONFIG_RECEIVE_CUSTOM_MESSAGE
        } else if (strcmp(type->valuestring, "custom") == 0) {
            auto payload = cJSON_GetObjectItem(root, "payload");
            ESP_LOGI(TAG, "Received custom message: %s", cJSON_PrintUnformatted(root));
            if (cJSON_IsObject(payload)) {
                Schedule([this, display, payload_str = std::string(cJSON_PrintUnformatted(payload))]() {
                    display->SetChatMessage("system", payload_str.c_str());
                });
            } else {
                ESP_LOGW(TAG, "Invalid custom message format: missing payload");
            }
#endif
        } else {
            ESP_LOGW(TAG, "Unknown message type: %s", type->valuestring);
        }
    });
    
    protocol_->Start();
}

void Application::ShowActivationCode(const std::string& code, const std::string& message) {
    struct digit_sound {
        char digit;
        const std::string_view& sound;
    };
    static const std::array<digit_sound, 10> digit_sounds{{
        digit_sound{'0', Lang::Sounds::OGG_0},
        digit_sound{'1', Lang::Sounds::OGG_1}, 
        digit_sound{'2', Lang::Sounds::OGG_2},
        digit_sound{'3', Lang::Sounds::OGG_3},
        digit_sound{'4', Lang::Sounds::OGG_4},
        digit_sound{'5', Lang::Sounds::OGG_5},
        digit_sound{'6', Lang::Sounds::OGG_6},
        digit_sound{'7', Lang::Sounds::OGG_7},
        digit_sound{'8', Lang::Sounds::OGG_8},
        digit_sound{'9', Lang::Sounds::OGG_9}
    }};

    // This sentence uses 9KB of SRAM, so we need to wait for it to finish
    Alert(Lang::Strings::ACTIVATION, message.c_str(), "link", Lang::Sounds::OGG_ACTIVATION);

    for (const auto& digit : code) {
        auto it = std::find_if(digit_sounds.begin(), digit_sounds.end(),
            [digit](const digit_sound& ds) { return ds.digit == digit; });
        if (it != digit_sounds.end()) {
            audio_service_.PlaySound(it->sound);
        }
    }
}

void Application::Alert(const char* status, const char* message, const char* emotion, const std::string_view& sound) {
    ESP_LOGW(TAG, "Alert [%s] %s: %s", emotion, status, message);
    auto display = Board::GetInstance().GetDisplay();
    display->SetStatus(status);
    display->SetEmotion(emotion);
    display->SetChatMessage("system", message);
    if (!sound.empty()) {
        audio_service_.PlaySound(sound);
    }
}

void Application::DismissAlert() {
    if (GetDeviceState() == kDeviceStateIdle) {
        auto display = Board::GetInstance().GetDisplay();
        display->SetStatus(Lang::Strings::STANDBY);
        display->SetEmotion("neutral");
        display->SetChatMessage("system", "");
    }
}

void Application::ToggleChatState() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_TOGGLE_CHAT);
}

void Application::StartListening() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_START_LISTENING);
}

void Application::StopListening() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_STOP_LISTENING);
}

void Application::HandleToggleChatEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateActivating) {
        SetDeviceState(kDeviceStateIdle);
        return;
    } else if (state == kDeviceStateWifiConfiguring) {
        audio_service_.EnableAudioTesting(true);
        SetDeviceState(kDeviceStateAudioTesting);
        return;
    } else if (state == kDeviceStateAudioTesting) {
        audio_service_.EnableAudioTesting(false);
        SetDeviceState(kDeviceStateWifiConfiguring);
        return;
    }

    if (!protocol_) {
        ESP_LOGE(TAG, "Protocol not initialized");
        return;
    }

    if (state == kDeviceStateIdle) {
        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                return;
            }
        }

        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonNone);
    } else if (state == kDeviceStateListening) {
        protocol_->CloseAudioChannel();
    }
}

void Application::HandleStartListeningEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateActivating) {
        SetDeviceState(kDeviceStateIdle);
        return;
    } else if (state == kDeviceStateWifiConfiguring) {
        audio_service_.EnableAudioTesting(true);
        SetDeviceState(kDeviceStateAudioTesting);
        return;
    }

    if (!protocol_) {
        ESP_LOGE(TAG, "Protocol not initialized");
        return;
    }
    
    if (state == kDeviceStateIdle) {
        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                return;
            }
        }

        SetListeningMode(kListeningModeManualStop);
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonNone);
        SetListeningMode(kListeningModeManualStop);
    }
}

void Application::HandleStopListeningEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateAudioTesting) {
        audio_service_.EnableAudioTesting(false);
        SetDeviceState(kDeviceStateWifiConfiguring);
        return;
    } else if (state == kDeviceStateListening) {
        if (protocol_) {
            protocol_->SendStopListening();
        }
        SetDeviceState(kDeviceStateIdle);
    }
}

void Application::HandleWakeWordDetectedEvent() {
    if (!protocol_) {
        return;
    }

    auto state = GetDeviceState();
    
    if (state == kDeviceStateIdle) {
        audio_service_.EncodeWakeWord();

        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                audio_service_.EnableWakeWordDetection(true);
                return;
            }
        }

        auto wake_word = audio_service_.GetLastWakeWord();
        ESP_LOGI(TAG, "Wake word detected: %s", wake_word.c_str());
#if CONFIG_SEND_WAKE_WORD_DATA
        // Encode and send the wake word data to the server
        while (auto packet = audio_service_.PopWakeWordPacket()) {
            protocol_->SendAudio(std::move(packet));
        }
        // Set the chat state to wake word detected
        protocol_->SendWakeWordDetected(wake_word);
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#else
        // Set flag to play popup sound after state changes to listening
        // (PlaySound here would be cleared by ResetDecoder in EnableVoiceProcessing)
        play_popup_on_listening_ = true;
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#endif
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonWakeWordDetected);
    } else if (state == kDeviceStateActivating) {
        // Restart the activation check if the wake word is detected during activation
        SetDeviceState(kDeviceStateIdle);
    }
}

void Application::HandleStateChangedEvent() {
    DeviceState new_state = state_machine_.GetState();
    clock_ticks_ = 0;

    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto led = board.GetLed();
    led->OnStateChanged();
    
    switch (new_state) {
        case kDeviceStateUnknown:
        case kDeviceStateIdle:
            display->SetStatus(Lang::Strings::STANDBY);
            display->SetEmotion("neutral");
            audio_service_.EnableVoiceProcessing(false);
            audio_service_.EnableWakeWordDetection(true);
            break;
        case kDeviceStateConnecting:
            display->SetStatus(Lang::Strings::CONNECTING);
            display->SetEmotion("neutral");
            display->SetChatMessage("system", "");
            break;
        case kDeviceStateListening:
            display->SetStatus(Lang::Strings::LISTENING);
            display->SetEmotion("neutral");

            // Make sure the audio processor is running
            if (!audio_service_.IsAudioProcessorRunning()) {
                // Send the start listening command
                protocol_->SendStartListening(listening_mode_);
                audio_service_.EnableVoiceProcessing(true);
                audio_service_.EnableWakeWordDetection(false);
            }

            // Play popup sound after ResetDecoder (in EnableVoiceProcessing) has been called
            if (play_popup_on_listening_) {
                play_popup_on_listening_ = false;
                audio_service_.PlaySound(Lang::Sounds::OGG_POPUP);
            }
            break;
        case kDeviceStateSpeaking:
            display->SetStatus(Lang::Strings::SPEAKING);

            // 记录 AI 开始说话的时间，用于语音打断保护
            speaking_start_time_ = esp_timer_get_time() / 1000;
            voice_detected_start_time_ = 0;  // 重置用户声音检测计时

            if (listening_mode_ != kListeningModeRealtime) {
                audio_service_.EnableVoiceProcessing(false);
                // Only AFE wake word can be detected in speaking mode
                audio_service_.EnableWakeWordDetection(audio_service_.IsAfeWakeWord());
            }
            audio_service_.ResetDecoder();
            break;
        case kDeviceStateWifiConfiguring:
            audio_service_.EnableVoiceProcessing(false);
            audio_service_.EnableWakeWordDetection(false);
            break;
        default:
            // Do nothing
            break;
    }
}

void Application::Schedule(std::function<void()>&& callback) {
    {
        std::lock_guard<std::mutex> lock(mutex_);
        main_tasks_.push_back(std::move(callback));
    }
    xEventGroupSetBits(event_group_, MAIN_EVENT_SCHEDULE);
}

void Application::AbortSpeaking(AbortReason reason) {
    ESP_LOGI(TAG, "Abort speaking");
    aborted_ = true;
    if (protocol_) {
        protocol_->SendAbortSpeaking(reason);
    }
}

void Application::SetListeningMode(ListeningMode mode) {
    listening_mode_ = mode;
    SetDeviceState(kDeviceStateListening);
}

void Application::Reboot() {
    ESP_LOGI(TAG, "Rebooting...");
    // Disconnect the audio channel
    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        protocol_->CloseAudioChannel();
    }
    protocol_.reset();
    audio_service_.Stop();

    vTaskDelay(pdMS_TO_TICKS(1000));
    esp_restart();
}

bool Application::UpgradeFirmware(const std::string& url, const std::string& version) {
    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();

    std::string upgrade_url = url;
    std::string version_info = version.empty() ? "(Manual upgrade)" : version;

    // Close audio channel if it's open
    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        ESP_LOGI(TAG, "Closing audio channel before firmware upgrade");
        protocol_->CloseAudioChannel();
    }
    ESP_LOGI(TAG, "Starting firmware upgrade from URL: %s", upgrade_url.c_str());

    Alert(Lang::Strings::OTA_UPGRADE, Lang::Strings::UPGRADING, "download", Lang::Sounds::OGG_UPGRADE);
    vTaskDelay(pdMS_TO_TICKS(3000));

    SetDeviceState(kDeviceStateUpgrading);

    std::string message = std::string(Lang::Strings::NEW_VERSION) + version_info;
    display->SetChatMessage("system", message.c_str());

    board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
    audio_service_.Stop();
    vTaskDelay(pdMS_TO_TICKS(1000));

    bool upgrade_success = Ota::Upgrade(upgrade_url, [display](int progress, size_t speed) {
        std::thread([display, progress, speed]() {
            char buffer[32];
            snprintf(buffer, sizeof(buffer), "%d%% %uKB/s", progress, speed / 1024);
            display->SetChatMessage("system", buffer);
        }).detach();
    });

    if (!upgrade_success) {
        // Upgrade failed, restart audio service and continue running
        ESP_LOGE(TAG, "Firmware upgrade failed, restarting audio service and continuing operation...");
        audio_service_.Start(); // Restart audio service
        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER); // Restore power save level
        Alert(Lang::Strings::ERROR, Lang::Strings::UPGRADE_FAILED, "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
        vTaskDelay(pdMS_TO_TICKS(3000));
        return false;
    } else {
        // Upgrade success, reboot immediately
        ESP_LOGI(TAG, "Firmware upgrade successful, rebooting...");
        display->SetChatMessage("system", "Upgrade successful, rebooting...");
        vTaskDelay(pdMS_TO_TICKS(1000)); // Brief pause to show message
        Reboot();
        return true;
    }
}

void Application::WakeWordInvoke(const std::string& wake_word) {
    if (!protocol_) {
        return;
    }

    auto state = GetDeviceState();
    
    if (state == kDeviceStateIdle) {
        audio_service_.EncodeWakeWord();

        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                audio_service_.EnableWakeWordDetection(true);
                return;
            }
        }

        ESP_LOGI(TAG, "Wake word detected: %s", wake_word.c_str());
#if CONFIG_USE_AFE_WAKE_WORD || CONFIG_USE_CUSTOM_WAKE_WORD
        // Encode and send the wake word data to the server
        while (auto packet = audio_service_.PopWakeWordPacket()) {
            protocol_->SendAudio(std::move(packet));
        }
        // Set the chat state to wake word detected
        protocol_->SendWakeWordDetected(wake_word);
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#else
        // Set flag to play popup sound after state changes to listening
        // (PlaySound here would be cleared by ResetDecoder in EnableVoiceProcessing)
        play_popup_on_listening_ = true;
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#endif
    } else if (state == kDeviceStateSpeaking) {
        Schedule([this]() {
            AbortSpeaking(kAbortReasonNone);
        });
    } else if (state == kDeviceStateListening) {   
        Schedule([this]() {
            if (protocol_) {
                protocol_->CloseAudioChannel();
            }
        });
    }
}

bool Application::CanEnterSleepMode() {
    if (GetDeviceState() != kDeviceStateIdle) {
        return false;
    }

    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        return false;
    }

    if (!audio_service_.IsIdle()) {
        return false;
    }

    // Now it is safe to enter sleep mode
    return true;
}

void Application::SendMcpMessage(const std::string& payload) {
    // Always schedule to run in main task for thread safety
    Schedule([this, payload = std::move(payload)]() {
        if (protocol_) {
            protocol_->SendMcpMessage(payload);
        }
    });
}

void Application::SetAecMode(AecMode mode) {
    aec_mode_ = mode;
    Schedule([this]() {
        auto& board = Board::GetInstance();
        auto display = board.GetDisplay();
        switch (aec_mode_) {
        case kAecOff:
            audio_service_.EnableDeviceAec(false);
            display->ShowNotification(Lang::Strings::RTC_MODE_OFF);
            break;
        case kAecOnServerSide:
            audio_service_.EnableDeviceAec(false);
            display->ShowNotification(Lang::Strings::RTC_MODE_ON);
            break;
        case kAecOnDeviceSide:
            audio_service_.EnableDeviceAec(true);
            display->ShowNotification(Lang::Strings::RTC_MODE_ON);
            break;
        }

        // If the AEC mode is changed, close the audio channel
        if (protocol_ && protocol_->IsAudioChannelOpened()) {
            protocol_->CloseAudioChannel();
        }
    });
}

void Application::PlaySound(const std::string_view& sound) {
    audio_service_.PlaySound(sound);
}

void Application::ResetProtocol() {
    Schedule([this]() {
        // Close audio channel if opened
        if (protocol_ && protocol_->IsAudioChannelOpened()) {
            protocol_->CloseAudioChannel();
        }
        // Reset protocol
        protocol_.reset();
    });
}


```

等等，我需要更仔细分析一下这个情况：

## 重要发现

从日志来看，**没有出现** `"Voice interrupt triggered"` 的日志，这意味着：

### 实际发生的情况

```
┌──────────────────────────────────────────────────────────────────────┐
│ 实时模式下的流程：                                                    │
│                                                                       │
│ 1. 设备持续发送麦克风音频给服务器（包含 AI 的回声）                  │
│                                                                       │
│ 2. 服务器端的 ASR 检测到"声音"（回声被误识别）                       │
│    → 识别出 "好的我"、"我的是" 等不完整的词                          │
│                                                                       │
│ 3. 服务器认为用户在说话，停止 TTS，发送识别结果                      │
│                                                                       │
│ 4. 这就是为什么会有那些奇怪的识别结果                                 │
└──────────────────────────────────────────────────────────────────────┘
```

**问题不在设备端，而在服务端！** 服务端的 AEC 效果可能不够好，回声没有被完全消除，导致 ASR 误识别。

---

## 解决方案

### 方案 1：降低实时模式的使用（推荐测试）

既然服务端 AEC 效果不理想，可以考虑**不使用实时模式**，改用普通的半双工模式：

```plaintext
┌──────────────────────────────────────────────────────────────────────┐
│ 实时模式下的流程：                                                    │
│                                                                       │
│ 1. 设备持续发送麦克风音频给服务器（包含 AI 的回声）                  │
│                                                                       │
│ 2. 服务器端的 ASR 检测到"声音"（回声被误识别）                       │
│    → 识别出 "好的我"、"我的是" 等不完整的词                          │
│                                                                       │
│ 3. 服务器认为用户在说话，停止 TTS，发送识别结果                      │
│                                                                       │
│ 4. 这就是为什么会有那些奇怪的识别结果                                 │
└──────────────────────────────────────────────────────────────────────┘
```

找到问题根源了！看这行代码：

```818:823:main/application.cc
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
```

当启用服务端 AEC 时，使用 `kListeningModeRealtime`，**设备会在 AI 说话时持续发送音频给服务器**，服务器的 ASR 会检测到回声并误识别。

---

## 你现在有两个选择

### 选择 A：放弃实时模式，使用按钮打断

修改代码，即使启用了服务端 AEC，也使用 `AutoStop` 模式。这样：
- ✅ 不会有回声误触发
- ❌ 不能语音打断，只能按钮打断

### 选择 B：保持实时模式，但用设备端 VAD 控制打断

修改代码：
1. 在 AI 说话时**暂停发送音频给服务器**
2. 只在设备端做 VAD 检测
3. 当设备端 VAD 检测到用户持续说话时，才恢复发送音频并打断

这样：
- ✅ 设备端控制打断时机
- ✅ 减少回声误触发
- ⚠️ 可能仍有误触发（因为设备端 VAD 没有 AEC）

---

## 你想选择哪个方案？

- **选择 A**：简单可靠，但没有语音打断
- **选择 B**：保留语音打断功能，但需要更多调试

告诉我你的选择，我来帮你实现！

```plaintext
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
```

---

**User**

我想要在我目前这个esp32s3-n16r8设备上实现语音打断，你能做到吗

---

**Cursor**

能做到！让我实现一个更好的方案：

## 方案：设备端 VAD 控制打断

**核心思路**：
1. AI 说话时**暂停发送音频给服务器**（避免回声被 ASR 误识别）
2. 但保持设备端 VAD 检测运行
3. 当设备端检测到用户持续说话时，触发打断并恢复发送音频

```cpp
#include "application.h"
#include "board.h"
#include "display.h"
#include "system_info.h"
#include "audio_codec.h"
#include "mqtt_protocol.h"
#include "websocket_protocol.h"
#include "assets/lang_config.h"
#include "mcp_server.h"
#include "assets.h"
#include "settings.h"

#include <cstring>
#include <esp_log.h>
#include <cJSON.h>
#include <driver/gpio.h>
#include <arpa/inet.h>
#include <font_awesome.h>

#define TAG "Application"


Application::Application() {
    event_group_ = xEventGroupCreate();

#if CONFIG_USE_DEVICE_AEC && CONFIG_USE_SERVER_AEC
#error "CONFIG_USE_DEVICE_AEC and CONFIG_USE_SERVER_AEC cannot be enabled at the same time"
#elif CONFIG_USE_DEVICE_AEC
    aec_mode_ = kAecOnDeviceSide;
#elif CONFIG_USE_SERVER_AEC
    aec_mode_ = kAecOnServerSide;
#else
    aec_mode_ = kAecOff;
#endif

    esp_timer_create_args_t clock_timer_args = {
        .callback = [](void* arg) {
            Application* app = (Application*)arg;
            xEventGroupSetBits(app->event_group_, MAIN_EVENT_CLOCK_TICK);
        },
        .arg = this,
        .dispatch_method = ESP_TIMER_TASK,
        .name = "clock_timer",
        .skip_unhandled_events = true
    };
    esp_timer_create(&clock_timer_args, &clock_timer_handle_);
}

Application::~Application() {
    if (clock_timer_handle_ != nullptr) {
        esp_timer_stop(clock_timer_handle_);
        esp_timer_delete(clock_timer_handle_);
    }
    vEventGroupDelete(event_group_);
}

bool Application::SetDeviceState(DeviceState state) {
    return state_machine_.TransitionTo(state);
}

void Application::Initialize() {
    auto& board = Board::GetInstance();
    SetDeviceState(kDeviceStateStarting);

    // Setup the display
    auto display = board.GetDisplay();

    // Print board name/version info
    display->SetChatMessage("system", SystemInfo::GetUserAgent().c_str());

    // Setup the audio service
    auto codec = board.GetAudioCodec();
    audio_service_.Initialize(codec);
    audio_service_.Start();

    AudioServiceCallbacks callbacks;
    callbacks.on_send_queue_available = [this]() {
        xEventGroupSetBits(event_group_, MAIN_EVENT_SEND_AUDIO);
    };
    callbacks.on_wake_word_detected = [this](const std::string& wake_word) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_WAKE_WORD_DETECTED);
    };
    callbacks.on_vad_change = [this](bool speaking) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_VAD_CHANGE);
    };
    audio_service_.SetCallbacks(callbacks);

    // Add state change listeners
    state_machine_.AddStateChangeListener([this](DeviceState old_state, DeviceState new_state) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_STATE_CHANGED);
    });

    // Start the clock timer to update the status bar
    esp_timer_start_periodic(clock_timer_handle_, 1000000);

    // Add MCP common tools (only once during initialization)
    auto& mcp_server = McpServer::GetInstance();
    mcp_server.AddCommonTools();
    mcp_server.AddUserOnlyTools();

    // Set network event callback for UI updates and network state handling
    board.SetNetworkEventCallback([this](NetworkEvent event, const std::string& data) {
        auto display = Board::GetInstance().GetDisplay();
        
        switch (event) {
            case NetworkEvent::Scanning:
                display->ShowNotification(Lang::Strings::SCANNING_WIFI, 30000);
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_DISCONNECTED);
                break;
            case NetworkEvent::Connecting: {
                if (data.empty()) {
                    // Cellular network - registering without carrier info yet
                    display->SetStatus(Lang::Strings::REGISTERING_NETWORK);
                } else {
                    // WiFi or cellular with carrier info
                    std::string msg = Lang::Strings::CONNECT_TO;
                    msg += data;
                    msg += "...";
                    display->ShowNotification(msg.c_str(), 30000);
                }
                break;
            }
            case NetworkEvent::Connected: {
                std::string msg = Lang::Strings::CONNECTED_TO;
                msg += data;
                display->ShowNotification(msg.c_str(), 30000);
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_CONNECTED);
                break;
            }
            case NetworkEvent::Disconnected:
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_DISCONNECTED);
                break;
            case NetworkEvent::WifiConfigModeEnter:
                // WiFi config mode enter is handled by WifiBoard internally
                break;
            case NetworkEvent::WifiConfigModeExit:
                // WiFi config mode exit is handled by WifiBoard internally
                break;
            // Cellular modem specific events
            case NetworkEvent::ModemDetecting:
                display->SetStatus(Lang::Strings::DETECTING_MODULE);
                break;
            case NetworkEvent::ModemErrorNoSim:
                Alert(Lang::Strings::ERROR, Lang::Strings::PIN_ERROR, "triangle_exclamation", Lang::Sounds::OGG_ERR_PIN);
                break;
            case NetworkEvent::ModemErrorRegDenied:
                Alert(Lang::Strings::ERROR, Lang::Strings::REG_ERROR, "triangle_exclamation", Lang::Sounds::OGG_ERR_REG);
                break;
            case NetworkEvent::ModemErrorInitFailed:
                display->SetStatus(Lang::Strings::DETECTING_MODULE);
                display->SetChatMessage("system", Lang::Strings::DETECTING_MODULE);
                break;
            case NetworkEvent::ModemErrorTimeout:
                display->SetStatus(Lang::Strings::REGISTERING_NETWORK);
                break;
        }
    });

    // Start network asynchronously
    board.StartNetwork();

    // Update the status bar immediately to show the network state
    display->UpdateStatusBar(true);
}

void Application::Run() {
    const EventBits_t ALL_EVENTS = 
        MAIN_EVENT_SCHEDULE |
        MAIN_EVENT_SEND_AUDIO |
        MAIN_EVENT_WAKE_WORD_DETECTED |
        MAIN_EVENT_VAD_CHANGE |
        MAIN_EVENT_CLOCK_TICK |
        MAIN_EVENT_ERROR |
        MAIN_EVENT_NETWORK_CONNECTED |
        MAIN_EVENT_NETWORK_DISCONNECTED |
        MAIN_EVENT_TOGGLE_CHAT |
        MAIN_EVENT_START_LISTENING |
        MAIN_EVENT_STOP_LISTENING |
        MAIN_EVENT_ACTIVATION_DONE |
        MAIN_EVENT_STATE_CHANGED;

    while (true) {
        auto bits = xEventGroupWaitBits(event_group_, ALL_EVENTS, pdTRUE, pdFALSE, portMAX_DELAY);

        if (bits & MAIN_EVENT_ERROR) {
            SetDeviceState(kDeviceStateIdle);
            Alert(Lang::Strings::ERROR, last_error_message_.c_str(), "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
        }

        if (bits & MAIN_EVENT_NETWORK_CONNECTED) {
            HandleNetworkConnectedEvent();
        }

        if (bits & MAIN_EVENT_NETWORK_DISCONNECTED) {
            HandleNetworkDisconnectedEvent();
        }

        if (bits & MAIN_EVENT_ACTIVATION_DONE) {
            HandleActivationDoneEvent();
        }

        if (bits & MAIN_EVENT_STATE_CHANGED) {
            HandleStateChangedEvent();
        }

        if (bits & MAIN_EVENT_TOGGLE_CHAT) {
            HandleToggleChatEvent();
        }

        if (bits & MAIN_EVENT_START_LISTENING) {
            HandleStartListeningEvent();
        }

        if (bits & MAIN_EVENT_STOP_LISTENING) {
            HandleStopListeningEvent();
        }

        if (bits & MAIN_EVENT_SEND_AUDIO) {
            // 在 Speaking 状态且是实时模式时，暂停发送音频给服务器
            // 这样可以避免回声被服务器 ASR 误识别
            // 打断由设备端 VAD 控制
            auto current_state = GetDeviceState();
            bool should_send = !(current_state == kDeviceStateSpeaking && 
                                 listening_mode_ == kListeningModeRealtime);
            
            while (auto packet = audio_service_.PopPacketFromSendQueue()) {
                if (should_send && protocol_ && !protocol_->SendAudio(std::move(packet))) {
                    break;
                }
                // 如果不发送，packet 会在作用域结束时被释放
            }
        }

        if (bits & MAIN_EVENT_WAKE_WORD_DETECTED) {
            HandleWakeWordDetectedEvent();
        }

        if (bits & MAIN_EVENT_VAD_CHANGE) {
            auto state = GetDeviceState();
            if (state == kDeviceStateListening) {
                auto led = Board::GetInstance().GetLed();
                led->OnStateChanged();
            } else if (state == kDeviceStateSpeaking && listening_mode_ == kListeningModeRealtime) {
                // 在实时模式下，检测用户语音打断
                // 防抖动保护：需要满足以下条件才触发打断
                // 1. AI 说话已超过一定时间（避免刚开始说话就被打断）
                // 2. 持续检测到用户声音超过一定时间（避免短暂噪声/回声触发）
                
                const int64_t SPEAKING_PROTECTION_MS = 1500;  // AI 说话保护期（增加到1.5秒）
                const int64_t VOICE_SUSTAIN_MS = 800;         // 用户声音持续时间要求（增加到0.8秒）
                
                int64_t now = esp_timer_get_time() / 1000;  // 转换为毫秒
                
                if (audio_service_.IsVoiceDetected()) {
                    // 检测到声音
                    if (voice_detected_start_time_ == 0) {
                        voice_detected_start_time_ = now;
                        ESP_LOGD(TAG, "Voice detected, starting timer...");
                    }
                    
                    int64_t speaking_duration = now - speaking_start_time_;
                    int64_t voice_duration = now - voice_detected_start_time_;
                    
                    // 检查是否满足打断条件
                    if (speaking_duration > SPEAKING_PROTECTION_MS && 
                        voice_duration > VOICE_SUSTAIN_MS) {
                        ESP_LOGI(TAG, "Voice interrupt triggered (speaking: %lldms, voice: %lldms)", 
                                 speaking_duration, voice_duration);
                        voice_detected_start_time_ = 0;  // 重置
                        AbortSpeaking(kAbortReasonNone);
                    }
                } else {
                    // 声音停止，重置计时器
                    if (voice_detected_start_time_ != 0) {
                        ESP_LOGD(TAG, "Voice stopped, resetting timer");
                        voice_detected_start_time_ = 0;
                    }
                }
            }
        }

        if (bits & MAIN_EVENT_SCHEDULE) {
            std::unique_lock<std::mutex> lock(mutex_);
            auto tasks = std::move(main_tasks_);
            lock.unlock();
            for (auto& task : tasks) {
                task();
            }
        }

        if (bits & MAIN_EVENT_CLOCK_TICK) {
            clock_ticks_++;
            auto display = Board::GetInstance().GetDisplay();
            display->UpdateStatusBar();
        
            // Print debug info every 10 seconds
            if (clock_ticks_ % 10 == 0) {
                SystemInfo::PrintHeapStats();
            }
        }
    }
}

void Application::HandleNetworkConnectedEvent() {
    ESP_LOGI(TAG, "Network connected");
    auto state = GetDeviceState();

    if (state == kDeviceStateStarting || state == kDeviceStateWifiConfiguring) {
        // Network is ready, start activation
        SetDeviceState(kDeviceStateActivating);
        if (activation_task_handle_ != nullptr) {
            ESP_LOGW(TAG, "Activation task already running");
            return;
        }

        xTaskCreate([](void* arg) {
            Application* app = static_cast<Application*>(arg);
            app->ActivationTask();
            app->activation_task_handle_ = nullptr;
            vTaskDelete(NULL);
        }, "activation", 4096 * 2, this, 2, &activation_task_handle_);
    }

    // Update the status bar immediately to show the network state
    auto display = Board::GetInstance().GetDisplay();
    display->UpdateStatusBar(true);
}

void Application::HandleNetworkDisconnectedEvent() {
    // Close current conversation when network disconnected
    auto state = GetDeviceState();
    if (state == kDeviceStateConnecting || state == kDeviceStateListening || state == kDeviceStateSpeaking) {
        ESP_LOGI(TAG, "Closing audio channel due to network disconnection");
        protocol_->CloseAudioChannel();
    }

    // Update the status bar immediately to show the network state
    auto display = Board::GetInstance().GetDisplay();
    display->UpdateStatusBar(true);
}

void Application::HandleActivationDoneEvent() {
    ESP_LOGI(TAG, "Activation done");

    SystemInfo::PrintHeapStats();
    SetDeviceState(kDeviceStateIdle);

    has_server_time_ = ota_->HasServerTime();

    auto display = Board::GetInstance().GetDisplay();
    std::string message = std::string(Lang::Strings::VERSION) + ota_->GetCurrentVersion();
    display->ShowNotification(message.c_str());
    display->SetChatMessage("system", "");

    // Play the success sound to indicate the device is ready
    audio_service_.PlaySound(Lang::Sounds::OGG_SUCCESS);

    // Release OTA object after activation is complete
    ota_.reset();
    auto& board = Board::GetInstance();
    board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
}

void Application::ActivationTask() {
    // Create OTA object for activation process
    ota_ = std::make_unique<Ota>();

    // Check for new assets version
    CheckAssetsVersion();

    // Check for new firmware version
    CheckNewVersion();

    // Initialize the protocol
    InitializeProtocol();

    // Signal completion to main loop
    xEventGroupSetBits(event_group_, MAIN_EVENT_ACTIVATION_DONE);
}

void Application::CheckAssetsVersion() {
    // Only allow CheckAssetsVersion to be called once
    if (assets_version_checked_) {
        return;
    }
    assets_version_checked_ = true;

    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto& assets = Assets::GetInstance();

    if (!assets.partition_valid()) {
        ESP_LOGW(TAG, "Assets partition is disabled for board %s", BOARD_NAME);
        return;
    }
    
    Settings settings("assets", true);
    // Check if there is a new assets need to be downloaded
    std::string download_url = settings.GetString("download_url");

    if (!download_url.empty()) {
        settings.EraseKey("download_url");

        char message[256];
        snprintf(message, sizeof(message), Lang::Strings::FOUND_NEW_ASSETS, download_url.c_str());
        Alert(Lang::Strings::LOADING_ASSETS, message, "cloud_arrow_down", Lang::Sounds::OGG_UPGRADE);
        
        // Wait for the audio service to be idle for 3 seconds
        vTaskDelay(pdMS_TO_TICKS(3000));
        SetDeviceState(kDeviceStateUpgrading);
        board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
        display->SetChatMessage("system", Lang::Strings::PLEASE_WAIT);

        bool success = assets.Download(download_url, [display](int progress, size_t speed) -> void {
            std::thread([display, progress, speed]() {
                char buffer[32];
                snprintf(buffer, sizeof(buffer), "%d%% %uKB/s", progress, speed / 1024);
                display->SetChatMessage("system", buffer);
            }).detach();
        });

        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
        vTaskDelay(pdMS_TO_TICKS(1000));

        if (!success) {
            Alert(Lang::Strings::ERROR, Lang::Strings::DOWNLOAD_ASSETS_FAILED, "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
            vTaskDelay(pdMS_TO_TICKS(2000));
            SetDeviceState(kDeviceStateActivating);
            return;
        }
    }

    // Apply assets
    assets.Apply();
    display->SetChatMessage("system", "");
    display->SetEmotion("microchip_ai");
}

void Application::CheckNewVersion() {
    const int MAX_RETRY = 10;
    int retry_count = 0;
    int retry_delay = 10; // Initial retry delay in seconds

    auto& board = Board::GetInstance();
    while (true) {
        auto display = board.GetDisplay();
        display->SetStatus(Lang::Strings::CHECKING_NEW_VERSION);

        esp_err_t err = ota_->CheckVersion();
        if (err != ESP_OK) {
            retry_count++;
            if (retry_count >= MAX_RETRY) {
                ESP_LOGE(TAG, "Too many retries, exit version check");
                return;
            }

            char error_message[128];
            snprintf(error_message, sizeof(error_message), "code=%d, url=%s", err, ota_->GetCheckVersionUrl().c_str());
            char buffer[256];
            snprintf(buffer, sizeof(buffer), Lang::Strings::CHECK_NEW_VERSION_FAILED, retry_delay, error_message);
            Alert(Lang::Strings::ERROR, buffer, "cloud_slash", Lang::Sounds::OGG_EXCLAMATION);

            ESP_LOGW(TAG, "Check new version failed, retry in %d seconds (%d/%d)", retry_delay, retry_count, MAX_RETRY);
            for (int i = 0; i < retry_delay; i++) {
                vTaskDelay(pdMS_TO_TICKS(1000));
                if (GetDeviceState() == kDeviceStateIdle) {
                    break;
                }
            }
            retry_delay *= 2; // Double the retry delay
            continue;
        }
        retry_count = 0;
        retry_delay = 10; // Reset retry delay

        if (ota_->HasNewVersion()) {
            if (UpgradeFirmware(ota_->GetFirmwareUrl(), ota_->GetFirmwareVersion())) {
                return; // This line will never be reached after reboot
            }
            // If upgrade failed, continue to normal operation
        }

        // No new version, mark the current version as valid
        ota_->MarkCurrentVersionValid();
        if (!ota_->HasActivationCode() && !ota_->HasActivationChallenge()) {
            // Exit the loop if done checking new version
            break;
        }

        display->SetStatus(Lang::Strings::ACTIVATION);
        // Activation code is shown to the user and waiting for the user to input
        if (ota_->HasActivationCode()) {
            ShowActivationCode(ota_->GetActivationCode(), ota_->GetActivationMessage());
        }

        // This will block the loop until the activation is done or timeout
        for (int i = 0; i < 10; ++i) {
            ESP_LOGI(TAG, "Activating... %d/%d", i + 1, 10);
            esp_err_t err = ota_->Activate();
            if (err == ESP_OK) {
                break;
            } else if (err == ESP_ERR_TIMEOUT) {
                vTaskDelay(pdMS_TO_TICKS(3000));
            } else {
                vTaskDelay(pdMS_TO_TICKS(10000));
            }
            if (GetDeviceState() == kDeviceStateIdle) {
                break;
            }
        }
    }
}

void Application::InitializeProtocol() {
    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto codec = board.GetAudioCodec();

    display->SetStatus(Lang::Strings::LOADING_PROTOCOL);

    if (ota_->HasMqttConfig()) {
        protocol_ = std::make_unique<MqttProtocol>();
    } else if (ota_->HasWebsocketConfig()) {
        protocol_ = std::make_unique<WebsocketProtocol>();
    } else {
        ESP_LOGW(TAG, "No protocol specified in the OTA config, using MQTT");
        protocol_ = std::make_unique<MqttProtocol>();
    }

    protocol_->OnConnected([this]() {
        DismissAlert();
    });

    protocol_->OnNetworkError([this](const std::string& message) {
        last_error_message_ = message;
        xEventGroupSetBits(event_group_, MAIN_EVENT_ERROR);
    });
    
    protocol_->OnIncomingAudio([this](std::unique_ptr<AudioStreamPacket> packet) {
        if (GetDeviceState() == kDeviceStateSpeaking) {
            audio_service_.PushPacketToDecodeQueue(std::move(packet));
        }
    });
    
    protocol_->OnAudioChannelOpened([this, codec, &board]() {
        board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
        if (protocol_->server_sample_rate() != codec->output_sample_rate()) {
            ESP_LOGW(TAG, "Server sample rate %d does not match device output sample rate %d, resampling may cause distortion",
                protocol_->server_sample_rate(), codec->output_sample_rate());
        }
    });
    
    protocol_->OnAudioChannelClosed([this, &board]() {
        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
        Schedule([this]() {
            auto display = Board::GetInstance().GetDisplay();
            display->SetChatMessage("system", "");
            SetDeviceState(kDeviceStateIdle);
        });
    });
    
    protocol_->OnIncomingJson([this, display](const cJSON* root) {
        // Parse JSON data
        auto type = cJSON_GetObjectItem(root, "type");
        if (strcmp(type->valuestring, "tts") == 0) {
            auto state = cJSON_GetObjectItem(root, "state");
            if (strcmp(state->valuestring, "start") == 0) {
                Schedule([this]() {
                    aborted_ = false;
                    SetDeviceState(kDeviceStateSpeaking);
                });
            } else if (strcmp(state->valuestring, "stop") == 0) {
                Schedule([this]() {
                    if (GetDeviceState() == kDeviceStateSpeaking) {
                        if (listening_mode_ == kListeningModeManualStop) {
                            SetDeviceState(kDeviceStateIdle);
                        } else {
                            SetDeviceState(kDeviceStateListening);
                        }
                    }
                });
            } else if (strcmp(state->valuestring, "sentence_start") == 0) {
                auto text = cJSON_GetObjectItem(root, "text");
                if (cJSON_IsString(text)) {
                    ESP_LOGI(TAG, "<< %s", text->valuestring);
                    Schedule([this, display, message = std::string(text->valuestring)]() {
                        display->SetChatMessage("assistant", message.c_str());
                    });
                }
            }
        } else if (strcmp(type->valuestring, "stt") == 0) {
            auto text = cJSON_GetObjectItem(root, "text");
            if (cJSON_IsString(text)) {
                ESP_LOGI(TAG, ">> %s", text->valuestring);
                Schedule([this, display, message = std::string(text->valuestring)]() {
                    display->SetChatMessage("user", message.c_str());
                });
            }
        } else if (strcmp(type->valuestring, "llm") == 0) {
            auto emotion = cJSON_GetObjectItem(root, "emotion");
            if (cJSON_IsString(emotion)) {
                Schedule([this, display, emotion_str = std::string(emotion->valuestring)]() {
                    display->SetEmotion(emotion_str.c_str());
                });
            }
        } else if (strcmp(type->valuestring, "mcp") == 0) {
            auto payload = cJSON_GetObjectItem(root, "payload");
            if (cJSON_IsObject(payload)) {
                McpServer::GetInstance().ParseMessage(payload);
            }
        } else if (strcmp(type->valuestring, "system") == 0) {
            auto command = cJSON_GetObjectItem(root, "command");
            if (cJSON_IsString(command)) {
                ESP_LOGI(TAG, "System command: %s", command->valuestring);
                if (strcmp(command->valuestring, "reboot") == 0) {
                    // Do a reboot if user requests a OTA update
                    Schedule([this]() {
                        Reboot();
                    });
                } else {
                    ESP_LOGW(TAG, "Unknown system command: %s", command->valuestring);
                }
            }
        } else if (strcmp(type->valuestring, "alert") == 0) {
            auto status = cJSON_GetObjectItem(root, "status");
            auto message = cJSON_GetObjectItem(root, "message");
            auto emotion = cJSON_GetObjectItem(root, "emotion");
            if (cJSON_IsString(status) && cJSON_IsString(message) && cJSON_IsString(emotion)) {
                Alert(status->valuestring, message->valuestring, emotion->valuestring, Lang::Sounds::OGG_VIBRATION);
            } else {
                ESP_LOGW(TAG, "Alert command requires status, message and emotion");
            }
#if CONFIG_RECEIVE_CUSTOM_MESSAGE
        } else if (strcmp(type->valuestring, "custom") == 0) {
            auto payload = cJSON_GetObjectItem(root, "payload");
            ESP_LOGI(TAG, "Received custom message: %s", cJSON_PrintUnformatted(root));
            if (cJSON_IsObject(payload)) {
                Schedule([this, display, payload_str = std::string(cJSON_PrintUnformatted(payload))]() {
                    display->SetChatMessage("system", payload_str.c_str());
                });
            } else {
                ESP_LOGW(TAG, "Invalid custom message format: missing payload");
            }
#endif
        } else {
            ESP_LOGW(TAG, "Unknown message type: %s", type->valuestring);
        }
    });
    
    protocol_->Start();
}

void Application::ShowActivationCode(const std::string& code, const std::string& message) {
    struct digit_sound {
        char digit;
        const std::string_view& sound;
    };
    static const std::array<digit_sound, 10> digit_sounds{{
        digit_sound{'0', Lang::Sounds::OGG_0},
        digit_sound{'1', Lang::Sounds::OGG_1}, 
        digit_sound{'2', Lang::Sounds::OGG_2},
        digit_sound{'3', Lang::Sounds::OGG_3},
        digit_sound{'4', Lang::Sounds::OGG_4},
        digit_sound{'5', Lang::Sounds::OGG_5},
        digit_sound{'6', Lang::Sounds::OGG_6},
        digit_sound{'7', Lang::Sounds::OGG_7},
        digit_sound{'8', Lang::Sounds::OGG_8},
        digit_sound{'9', Lang::Sounds::OGG_9}
    }};

    // This sentence uses 9KB of SRAM, so we need to wait for it to finish
    Alert(Lang::Strings::ACTIVATION, message.c_str(), "link", Lang::Sounds::OGG_ACTIVATION);

    for (const auto& digit : code) {
        auto it = std::find_if(digit_sounds.begin(), digit_sounds.end(),
            [digit](const digit_sound& ds) { return ds.digit == digit; });
        if (it != digit_sounds.end()) {
            audio_service_.PlaySound(it->sound);
        }
    }
}

void Application::Alert(const char* status, const char* message, const char* emotion, const std::string_view& sound) {
    ESP_LOGW(TAG, "Alert [%s] %s: %s", emotion, status, message);
    auto display = Board::GetInstance().GetDisplay();
    display->SetStatus(status);
    display->SetEmotion(emotion);
    display->SetChatMessage("system", message);
    if (!sound.empty()) {
        audio_service_.PlaySound(sound);
    }
}

void Application::DismissAlert() {
    if (GetDeviceState() == kDeviceStateIdle) {
        auto display = Board::GetInstance().GetDisplay();
        display->SetStatus(Lang::Strings::STANDBY);
        display->SetEmotion("neutral");
        display->SetChatMessage("system", "");
    }
}

void Application::ToggleChatState() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_TOGGLE_CHAT);
}

void Application::StartListening() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_START_LISTENING);
}

void Application::StopListening() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_STOP_LISTENING);
}

void Application::HandleToggleChatEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateActivating) {
        SetDeviceState(kDeviceStateIdle);
        return;
    } else if (state == kDeviceStateWifiConfiguring) {
        audio_service_.EnableAudioTesting(true);
        SetDeviceState(kDeviceStateAudioTesting);
        return;
    } else if (state == kDeviceStateAudioTesting) {
        audio_service_.EnableAudioTesting(false);
        SetDeviceState(kDeviceStateWifiConfiguring);
        return;
    }

    if (!protocol_) {
        ESP_LOGE(TAG, "Protocol not initialized");
        return;
    }

    if (state == kDeviceStateIdle) {
        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                return;
            }
        }

        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonNone);
    } else if (state == kDeviceStateListening) {
        protocol_->CloseAudioChannel();
    }
}

void Application::HandleStartListeningEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateActivating) {
        SetDeviceState(kDeviceStateIdle);
        return;
    } else if (state == kDeviceStateWifiConfiguring) {
        audio_service_.EnableAudioTesting(true);
        SetDeviceState(kDeviceStateAudioTesting);
        return;
    }

    if (!protocol_) {
        ESP_LOGE(TAG, "Protocol not initialized");
        return;
    }
    
    if (state == kDeviceStateIdle) {
        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                return;
            }
        }

        SetListeningMode(kListeningModeManualStop);
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonNone);
        SetListeningMode(kListeningModeManualStop);
    }
}

void Application::HandleStopListeningEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateAudioTesting) {
        audio_service_.EnableAudioTesting(false);
        SetDeviceState(kDeviceStateWifiConfiguring);
        return;
    } else if (state == kDeviceStateListening) {
        if (protocol_) {
            protocol_->SendStopListening();
        }
        SetDeviceState(kDeviceStateIdle);
    }
}

void Application::HandleWakeWordDetectedEvent() {
    if (!protocol_) {
        return;
    }

    auto state = GetDeviceState();
    
    if (state == kDeviceStateIdle) {
        audio_service_.EncodeWakeWord();

        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                audio_service_.EnableWakeWordDetection(true);
                return;
            }
        }

        auto wake_word = audio_service_.GetLastWakeWord();
        ESP_LOGI(TAG, "Wake word detected: %s", wake_word.c_str());
#if CONFIG_SEND_WAKE_WORD_DATA
        // Encode and send the wake word data to the server
        while (auto packet = audio_service_.PopWakeWordPacket()) {
            protocol_->SendAudio(std::move(packet));
        }
        // Set the chat state to wake word detected
        protocol_->SendWakeWordDetected(wake_word);
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#else
        // Set flag to play popup sound after state changes to listening
        // (PlaySound here would be cleared by ResetDecoder in EnableVoiceProcessing)
        play_popup_on_listening_ = true;
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#endif
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonWakeWordDetected);
    } else if (state == kDeviceStateActivating) {
        // Restart the activation check if the wake word is detected during activation
        SetDeviceState(kDeviceStateIdle);
    }
}

void Application::HandleStateChangedEvent() {
    DeviceState new_state = state_machine_.GetState();
    clock_ticks_ = 0;

    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto led = board.GetLed();
    led->OnStateChanged();
    
    switch (new_state) {
        case kDeviceStateUnknown:
        case kDeviceStateIdle:
            display->SetStatus(Lang::Strings::STANDBY);
            display->SetEmotion("neutral");
            audio_service_.EnableVoiceProcessing(false);
            audio_service_.EnableWakeWordDetection(true);
            break;
        case kDeviceStateConnecting:
            display->SetStatus(Lang::Strings::CONNECTING);
            display->SetEmotion("neutral");
            display->SetChatMessage("system", "");
            break;
        case kDeviceStateListening:
            display->SetStatus(Lang::Strings::LISTENING);
            display->SetEmotion("neutral");

            // Make sure the audio processor is running
            if (!audio_service_.IsAudioProcessorRunning()) {
                // Send the start listening command
                protocol_->SendStartListening(listening_mode_);
                audio_service_.EnableVoiceProcessing(true);
                audio_service_.EnableWakeWordDetection(false);
            }

            // Play popup sound after ResetDecoder (in EnableVoiceProcessing) has been called
            if (play_popup_on_listening_) {
                play_popup_on_listening_ = false;
                audio_service_.PlaySound(Lang::Sounds::OGG_POPUP);
            }
            break;
        case kDeviceStateSpeaking:
            display->SetStatus(Lang::Strings::SPEAKING);

            // 记录 AI 开始说话的时间，用于语音打断保护
            speaking_start_time_ = esp_timer_get_time() / 1000;
            voice_detected_start_time_ = 0;  // 重置用户声音检测计时

            if (listening_mode_ != kListeningModeRealtime) {
                audio_service_.EnableVoiceProcessing(false);
                // Only AFE wake word can be detected in speaking mode
                audio_service_.EnableWakeWordDetection(audio_service_.IsAfeWakeWord());
            }
            audio_service_.ResetDecoder();
            break;
        case kDeviceStateWifiConfiguring:
            audio_service_.EnableVoiceProcessing(false);
            audio_service_.EnableWakeWordDetection(false);
            break;
        default:
            // Do nothing
            break;
    }
}

void Application::Schedule(std::function<void()>&& callback) {
    {
        std::lock_guard<std::mutex> lock(mutex_);
        main_tasks_.push_back(std::move(callback));
    }
    xEventGroupSetBits(event_group_, MAIN_EVENT_SCHEDULE);
}

void Application::AbortSpeaking(AbortReason reason) {
    ESP_LOGI(TAG, "Abort speaking");
    aborted_ = true;
    if (protocol_) {
        protocol_->SendAbortSpeaking(reason);
    }
}

void Application::SetListeningMode(ListeningMode mode) {
    listening_mode_ = mode;
    SetDeviceState(kDeviceStateListening);
}

void Application::Reboot() {
    ESP_LOGI(TAG, "Rebooting...");
    // Disconnect the audio channel
    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        protocol_->CloseAudioChannel();
    }
    protocol_.reset();
    audio_service_.Stop();

    vTaskDelay(pdMS_TO_TICKS(1000));
    esp_restart();
}

bool Application::UpgradeFirmware(const std::string& url, const std::string& version) {
    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();

    std::string upgrade_url = url;
    std::string version_info = version.empty() ? "(Manual upgrade)" : version;

    // Close audio channel if it's open
    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        ESP_LOGI(TAG, "Closing audio channel before firmware upgrade");
        protocol_->CloseAudioChannel();
    }
    ESP_LOGI(TAG, "Starting firmware upgrade from URL: %s", upgrade_url.c_str());

    Alert(Lang::Strings::OTA_UPGRADE, Lang::Strings::UPGRADING, "download", Lang::Sounds::OGG_UPGRADE);
    vTaskDelay(pdMS_TO_TICKS(3000));

    SetDeviceState(kDeviceStateUpgrading);

    std::string message = std::string(Lang::Strings::NEW_VERSION) + version_info;
    display->SetChatMessage("system", message.c_str());

    board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
    audio_service_.Stop();
    vTaskDelay(pdMS_TO_TICKS(1000));

    bool upgrade_success = Ota::Upgrade(upgrade_url, [display](int progress, size_t speed) {
        std::thread([display, progress, speed]() {
            char buffer[32];
            snprintf(buffer, sizeof(buffer), "%d%% %uKB/s", progress, speed / 1024);
            display->SetChatMessage("system", buffer);
        }).detach();
    });

    if (!upgrade_success) {
        // Upgrade failed, restart audio service and continue running
        ESP_LOGE(TAG, "Firmware upgrade failed, restarting audio service and continuing operation...");
        audio_service_.Start(); // Restart audio service
        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER); // Restore power save level
        Alert(Lang::Strings::ERROR, Lang::Strings::UPGRADE_FAILED, "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
        vTaskDelay(pdMS_TO_TICKS(3000));
        return false;
    } else {
        // Upgrade success, reboot immediately
        ESP_LOGI(TAG, "Firmware upgrade successful, rebooting...");
        display->SetChatMessage("system", "Upgrade successful, rebooting...");
        vTaskDelay(pdMS_TO_TICKS(1000)); // Brief pause to show message
        Reboot();
        return true;
    }
}

void Application::WakeWordInvoke(const std::string& wake_word) {
    if (!protocol_) {
        return;
    }

    auto state = GetDeviceState();
    
    if (state == kDeviceStateIdle) {
        audio_service_.EncodeWakeWord();

        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                audio_service_.EnableWakeWordDetection(true);
                return;
            }
        }

        ESP_LOGI(TAG, "Wake word detected: %s", wake_word.c_str());
#if CONFIG_USE_AFE_WAKE_WORD || CONFIG_USE_CUSTOM_WAKE_WORD
        // Encode and send the wake word data to the server
        while (auto packet = audio_service_.PopWakeWordPacket()) {
            protocol_->SendAudio(std::move(packet));
        }
        // Set the chat state to wake word detected
        protocol_->SendWakeWordDetected(wake_word);
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#else
        // Set flag to play popup sound after state changes to listening
        // (PlaySound here would be cleared by ResetDecoder in EnableVoiceProcessing)
        play_popup_on_listening_ = true;
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#endif
    } else if (state == kDeviceStateSpeaking) {
        Schedule([this]() {
            AbortSpeaking(kAbortReasonNone);
        });
    } else if (state == kDeviceStateListening) {   
        Schedule([this]() {
            if (protocol_) {
                protocol_->CloseAudioChannel();
            }
        });
    }
}

bool Application::CanEnterSleepMode() {
    if (GetDeviceState() != kDeviceStateIdle) {
        return false;
    }

    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        return false;
    }

    if (!audio_service_.IsIdle()) {
        return false;
    }

    // Now it is safe to enter sleep mode
    return true;
}

void Application::SendMcpMessage(const std::string& payload) {
    // Always schedule to run in main task for thread safety
    Schedule([this, payload = std::move(payload)]() {
        if (protocol_) {
            protocol_->SendMcpMessage(payload);
        }
    });
}

void Application::SetAecMode(AecMode mode) {
    aec_mode_ = mode;
    Schedule([this]() {
        auto& board = Board::GetInstance();
        auto display = board.GetDisplay();
        switch (aec_mode_) {
        case kAecOff:
            audio_service_.EnableDeviceAec(false);
            display->ShowNotification(Lang::Strings::RTC_MODE_OFF);
            break;
        case kAecOnServerSide:
            audio_service_.EnableDeviceAec(false);
            display->ShowNotification(Lang::Strings::RTC_MODE_ON);
            break;
        case kAecOnDeviceSide:
            audio_service_.EnableDeviceAec(true);
            display->ShowNotification(Lang::Strings::RTC_MODE_ON);
            break;
        }

        // If the AEC mode is changed, close the audio channel
        if (protocol_ && protocol_->IsAudioChannelOpened()) {
            protocol_->CloseAudioChannel();
        }
    });
}

void Application::PlaySound(const std::string_view& sound) {
    audio_service_.PlaySound(sound);
}

void Application::ResetProtocol() {
    Schedule([this]() {
        // Close audio channel if opened
        if (protocol_ && protocol_->IsAudioChannelOpened()) {
            protocol_->CloseAudioChannel();
        }
        // Reset protocol
        protocol_.reset();
    });
}


```

```cpp
#include "application.h"
#include "board.h"
#include "display.h"
#include "system_info.h"
#include "audio_codec.h"
#include "mqtt_protocol.h"
#include "websocket_protocol.h"
#include "assets/lang_config.h"
#include "mcp_server.h"
#include "assets.h"
#include "settings.h"

#include <cstring>
#include <esp_log.h>
#include <cJSON.h>
#include <driver/gpio.h>
#include <arpa/inet.h>
#include <font_awesome.h>

#define TAG "Application"


Application::Application() {
    event_group_ = xEventGroupCreate();

#if CONFIG_USE_DEVICE_AEC && CONFIG_USE_SERVER_AEC
#error "CONFIG_USE_DEVICE_AEC and CONFIG_USE_SERVER_AEC cannot be enabled at the same time"
#elif CONFIG_USE_DEVICE_AEC
    aec_mode_ = kAecOnDeviceSide;
#elif CONFIG_USE_SERVER_AEC
    aec_mode_ = kAecOnServerSide;
#else
    aec_mode_ = kAecOff;
#endif

    esp_timer_create_args_t clock_timer_args = {
        .callback = [](void* arg) {
            Application* app = (Application*)arg;
            xEventGroupSetBits(app->event_group_, MAIN_EVENT_CLOCK_TICK);
        },
        .arg = this,
        .dispatch_method = ESP_TIMER_TASK,
        .name = "clock_timer",
        .skip_unhandled_events = true
    };
    esp_timer_create(&clock_timer_args, &clock_timer_handle_);
}

Application::~Application() {
    if (clock_timer_handle_ != nullptr) {
        esp_timer_stop(clock_timer_handle_);
        esp_timer_delete(clock_timer_handle_);
    }
    vEventGroupDelete(event_group_);
}

bool Application::SetDeviceState(DeviceState state) {
    return state_machine_.TransitionTo(state);
}

void Application::Initialize() {
    auto& board = Board::GetInstance();
    SetDeviceState(kDeviceStateStarting);

    // Setup the display
    auto display = board.GetDisplay();

    // Print board name/version info
    display->SetChatMessage("system", SystemInfo::GetUserAgent().c_str());

    // Setup the audio service
    auto codec = board.GetAudioCodec();
    audio_service_.Initialize(codec);
    audio_service_.Start();

    AudioServiceCallbacks callbacks;
    callbacks.on_send_queue_available = [this]() {
        xEventGroupSetBits(event_group_, MAIN_EVENT_SEND_AUDIO);
    };
    callbacks.on_wake_word_detected = [this](const std::string& wake_word) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_WAKE_WORD_DETECTED);
    };
    callbacks.on_vad_change = [this](bool speaking) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_VAD_CHANGE);
    };
    audio_service_.SetCallbacks(callbacks);

    // Add state change listeners
    state_machine_.AddStateChangeListener([this](DeviceState old_state, DeviceState new_state) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_STATE_CHANGED);
    });

    // Start the clock timer to update the status bar
    esp_timer_start_periodic(clock_timer_handle_, 1000000);

    // Add MCP common tools (only once during initialization)
    auto& mcp_server = McpServer::GetInstance();
    mcp_server.AddCommonTools();
    mcp_server.AddUserOnlyTools();

    // Set network event callback for UI updates and network state handling
    board.SetNetworkEventCallback([this](NetworkEvent event, const std::string& data) {
        auto display = Board::GetInstance().GetDisplay();
        
        switch (event) {
            case NetworkEvent::Scanning:
                display->ShowNotification(Lang::Strings::SCANNING_WIFI, 30000);
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_DISCONNECTED);
                break;
            case NetworkEvent::Connecting: {
                if (data.empty()) {
                    // Cellular network - registering without carrier info yet
                    display->SetStatus(Lang::Strings::REGISTERING_NETWORK);
                } else {
                    // WiFi or cellular with carrier info
                    std::string msg = Lang::Strings::CONNECT_TO;
                    msg += data;
                    msg += "...";
                    display->ShowNotification(msg.c_str(), 30000);
                }
                break;
            }
            case NetworkEvent::Connected: {
                std::string msg = Lang::Strings::CONNECTED_TO;
                msg += data;
                display->ShowNotification(msg.c_str(), 30000);
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_CONNECTED);
                break;
            }
            case NetworkEvent::Disconnected:
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_DISCONNECTED);
                break;
            case NetworkEvent::WifiConfigModeEnter:
                // WiFi config mode enter is handled by WifiBoard internally
                break;
            case NetworkEvent::WifiConfigModeExit:
                // WiFi config mode exit is handled by WifiBoard internally
                break;
            // Cellular modem specific events
            case NetworkEvent::ModemDetecting:
                display->SetStatus(Lang::Strings::DETECTING_MODULE);
                break;
            case NetworkEvent::ModemErrorNoSim:
                Alert(Lang::Strings::ERROR, Lang::Strings::PIN_ERROR, "triangle_exclamation", Lang::Sounds::OGG_ERR_PIN);
                break;
            case NetworkEvent::ModemErrorRegDenied:
                Alert(Lang::Strings::ERROR, Lang::Strings::REG_ERROR, "triangle_exclamation", Lang::Sounds::OGG_ERR_REG);
                break;
            case NetworkEvent::ModemErrorInitFailed:
                display->SetStatus(Lang::Strings::DETECTING_MODULE);
                display->SetChatMessage("system", Lang::Strings::DETECTING_MODULE);
                break;
            case NetworkEvent::ModemErrorTimeout:
                display->SetStatus(Lang::Strings::REGISTERING_NETWORK);
                break;
        }
    });

    // Start network asynchronously
    board.StartNetwork();

    // Update the status bar immediately to show the network state
    display->UpdateStatusBar(true);
}

void Application::Run() {
    const EventBits_t ALL_EVENTS = 
        MAIN_EVENT_SCHEDULE |
        MAIN_EVENT_SEND_AUDIO |
        MAIN_EVENT_WAKE_WORD_DETECTED |
        MAIN_EVENT_VAD_CHANGE |
        MAIN_EVENT_CLOCK_TICK |
        MAIN_EVENT_ERROR |
        MAIN_EVENT_NETWORK_CONNECTED |
        MAIN_EVENT_NETWORK_DISCONNECTED |
        MAIN_EVENT_TOGGLE_CHAT |
        MAIN_EVENT_START_LISTENING |
        MAIN_EVENT_STOP_LISTENING |
        MAIN_EVENT_ACTIVATION_DONE |
        MAIN_EVENT_STATE_CHANGED;

    while (true) {
        auto bits = xEventGroupWaitBits(event_group_, ALL_EVENTS, pdTRUE, pdFALSE, portMAX_DELAY);

        if (bits & MAIN_EVENT_ERROR) {
            SetDeviceState(kDeviceStateIdle);
            Alert(Lang::Strings::ERROR, last_error_message_.c_str(), "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
        }

        if (bits & MAIN_EVENT_NETWORK_CONNECTED) {
            HandleNetworkConnectedEvent();
        }

        if (bits & MAIN_EVENT_NETWORK_DISCONNECTED) {
            HandleNetworkDisconnectedEvent();
        }

        if (bits & MAIN_EVENT_ACTIVATION_DONE) {
            HandleActivationDoneEvent();
        }

        if (bits & MAIN_EVENT_STATE_CHANGED) {
            HandleStateChangedEvent();
        }

        if (bits & MAIN_EVENT_TOGGLE_CHAT) {
            HandleToggleChatEvent();
        }

        if (bits & MAIN_EVENT_START_LISTENING) {
            HandleStartListeningEvent();
        }

        if (bits & MAIN_EVENT_STOP_LISTENING) {
            HandleStopListeningEvent();
        }

        if (bits & MAIN_EVENT_SEND_AUDIO) {
            // 在 Speaking 状态且是实时模式时，暂停发送音频给服务器
            // 这样可以避免回声被服务器 ASR 误识别
            // 打断由设备端 VAD 控制
            auto current_state = GetDeviceState();
            bool should_send = !(current_state == kDeviceStateSpeaking && 
                                 listening_mode_ == kListeningModeRealtime);
            
            while (auto packet = audio_service_.PopPacketFromSendQueue()) {
                if (should_send && protocol_ && !protocol_->SendAudio(std::move(packet))) {
                    break;
                }
                // 如果不发送，packet 会在作用域结束时被释放
            }
        }

        if (bits & MAIN_EVENT_WAKE_WORD_DETECTED) {
            HandleWakeWordDetectedEvent();
        }

        if (bits & MAIN_EVENT_VAD_CHANGE) {
            auto state = GetDeviceState();
            if (state == kDeviceStateListening) {
                auto led = Board::GetInstance().GetLed();
                led->OnStateChanged();
            } else if (state == kDeviceStateSpeaking && listening_mode_ == kListeningModeRealtime) {
                // 在实时模式下，由设备端 VAD 控制语音打断
                // 注意：此时音频不会发送给服务器，避免回声被误识别
                // 防抖动保护条件：
                // 1. AI 说话已超过保护期（避免刚开始说话就被回声打断）
                // 2. 持续检测到声音超过一定时间（过滤短暂噪声）
                
                const int64_t SPEAKING_PROTECTION_MS = 800;   // AI 说话保护期
                const int64_t VOICE_SUSTAIN_MS = 600;         // 声音持续时间要求
                
                int64_t now = esp_timer_get_time() / 1000;  // 转换为毫秒
                
                if (audio_service_.IsVoiceDetected()) {
                    // 检测到声音
                    if (voice_detected_start_time_ == 0) {
                        voice_detected_start_time_ = now;
                        ESP_LOGI(TAG, "Voice detected during speaking, starting timer...");
                    }
                    
                    int64_t speaking_duration = now - speaking_start_time_;
                    int64_t voice_duration = now - voice_detected_start_time_;
                    
                    // 检查是否满足打断条件
                    if (speaking_duration > SPEAKING_PROTECTION_MS && 
                        voice_duration > VOICE_SUSTAIN_MS) {
                        ESP_LOGI(TAG, ">>> Voice interrupt triggered! (speaking: %lldms, voice: %lldms)", 
                                 speaking_duration, voice_duration);
                        voice_detected_start_time_ = 0;  // 重置
                        
                        // 打断 AI 说话，并切换到监听状态
                        AbortSpeaking(kAbortReasonNone);
                        // 状态会在收到服务器的 tts stop 后自动切换到 Listening
                    }
                } else {
                    // 声音停止，重置计时器
                    if (voice_detected_start_time_ != 0) {
                        ESP_LOGD(TAG, "Voice stopped, resetting timer");
                        voice_detected_start_time_ = 0;
                    }
                }
            }
        }

        if (bits & MAIN_EVENT_SCHEDULE) {
            std::unique_lock<std::mutex> lock(mutex_);
            auto tasks = std::move(main_tasks_);
            lock.unlock();
            for (auto& task : tasks) {
                task();
            }
        }

        if (bits & MAIN_EVENT_CLOCK_TICK) {
            clock_ticks_++;
            auto display = Board::GetInstance().GetDisplay();
            display->UpdateStatusBar();
        
            // Print debug info every 10 seconds
            if (clock_ticks_ % 10 == 0) {
                SystemInfo::PrintHeapStats();
            }
        }
    }
}

void Application::HandleNetworkConnectedEvent() {
    ESP_LOGI(TAG, "Network connected");
    auto state = GetDeviceState();

    if (state == kDeviceStateStarting || state == kDeviceStateWifiConfiguring) {
        // Network is ready, start activation
        SetDeviceState(kDeviceStateActivating);
        if (activation_task_handle_ != nullptr) {
            ESP_LOGW(TAG, "Activation task already running");
            return;
        }

        xTaskCreate([](void* arg) {
            Application* app = static_cast<Application*>(arg);
            app->ActivationTask();
            app->activation_task_handle_ = nullptr;
            vTaskDelete(NULL);
        }, "activation", 4096 * 2, this, 2, &activation_task_handle_);
    }

    // Update the status bar immediately to show the network state
    auto display = Board::GetInstance().GetDisplay();
    display->UpdateStatusBar(true);
}

void Application::HandleNetworkDisconnectedEvent() {
    // Close current conversation when network disconnected
    auto state = GetDeviceState();
    if (state == kDeviceStateConnecting || state == kDeviceStateListening || state == kDeviceStateSpeaking) {
        ESP_LOGI(TAG, "Closing audio channel due to network disconnection");
        protocol_->CloseAudioChannel();
    }

    // Update the status bar immediately to show the network state
    auto display = Board::GetInstance().GetDisplay();
    display->UpdateStatusBar(true);
}

void Application::HandleActivationDoneEvent() {
    ESP_LOGI(TAG, "Activation done");

    SystemInfo::PrintHeapStats();
    SetDeviceState(kDeviceStateIdle);

    has_server_time_ = ota_->HasServerTime();

    auto display = Board::GetInstance().GetDisplay();
    std::string message = std::string(Lang::Strings::VERSION) + ota_->GetCurrentVersion();
    display->ShowNotification(message.c_str());
    display->SetChatMessage("system", "");

    // Play the success sound to indicate the device is ready
    audio_service_.PlaySound(Lang::Sounds::OGG_SUCCESS);

    // Release OTA object after activation is complete
    ota_.reset();
    auto& board = Board::GetInstance();
    board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
}

void Application::ActivationTask() {
    // Create OTA object for activation process
    ota_ = std::make_unique<Ota>();

    // Check for new assets version
    CheckAssetsVersion();

    // Check for new firmware version
    CheckNewVersion();

    // Initialize the protocol
    InitializeProtocol();

    // Signal completion to main loop
    xEventGroupSetBits(event_group_, MAIN_EVENT_ACTIVATION_DONE);
}

void Application::CheckAssetsVersion() {
    // Only allow CheckAssetsVersion to be called once
    if (assets_version_checked_) {
        return;
    }
    assets_version_checked_ = true;

    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto& assets = Assets::GetInstance();

    if (!assets.partition_valid()) {
        ESP_LOGW(TAG, "Assets partition is disabled for board %s", BOARD_NAME);
        return;
    }
    
    Settings settings("assets", true);
    // Check if there is a new assets need to be downloaded
    std::string download_url = settings.GetString("download_url");

    if (!download_url.empty()) {
        settings.EraseKey("download_url");

        char message[256];
        snprintf(message, sizeof(message), Lang::Strings::FOUND_NEW_ASSETS, download_url.c_str());
        Alert(Lang::Strings::LOADING_ASSETS, message, "cloud_arrow_down", Lang::Sounds::OGG_UPGRADE);
        
        // Wait for the audio service to be idle for 3 seconds
        vTaskDelay(pdMS_TO_TICKS(3000));
        SetDeviceState(kDeviceStateUpgrading);
        board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
        display->SetChatMessage("system", Lang::Strings::PLEASE_WAIT);

        bool success = assets.Download(download_url, [display](int progress, size_t speed) -> void {
            std::thread([display, progress, speed]() {
                char buffer[32];
                snprintf(buffer, sizeof(buffer), "%d%% %uKB/s", progress, speed / 1024);
                display->SetChatMessage("system", buffer);
            }).detach();
        });

        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
        vTaskDelay(pdMS_TO_TICKS(1000));

        if (!success) {
            Alert(Lang::Strings::ERROR, Lang::Strings::DOWNLOAD_ASSETS_FAILED, "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
            vTaskDelay(pdMS_TO_TICKS(2000));
            SetDeviceState(kDeviceStateActivating);
            return;
        }
    }

    // Apply assets
    assets.Apply();
    display->SetChatMessage("system", "");
    display->SetEmotion("microchip_ai");
}

void Application::CheckNewVersion() {
    const int MAX_RETRY = 10;
    int retry_count = 0;
    int retry_delay = 10; // Initial retry delay in seconds

    auto& board = Board::GetInstance();
    while (true) {
        auto display = board.GetDisplay();
        display->SetStatus(Lang::Strings::CHECKING_NEW_VERSION);

        esp_err_t err = ota_->CheckVersion();
        if (err != ESP_OK) {
            retry_count++;
            if (retry_count >= MAX_RETRY) {
                ESP_LOGE(TAG, "Too many retries, exit version check");
                return;
            }

            char error_message[128];
            snprintf(error_message, sizeof(error_message), "code=%d, url=%s", err, ota_->GetCheckVersionUrl().c_str());
            char buffer[256];
            snprintf(buffer, sizeof(buffer), Lang::Strings::CHECK_NEW_VERSION_FAILED, retry_delay, error_message);
            Alert(Lang::Strings::ERROR, buffer, "cloud_slash", Lang::Sounds::OGG_EXCLAMATION);

            ESP_LOGW(TAG, "Check new version failed, retry in %d seconds (%d/%d)", retry_delay, retry_count, MAX_RETRY);
            for (int i = 0; i < retry_delay; i++) {
                vTaskDelay(pdMS_TO_TICKS(1000));
                if (GetDeviceState() == kDeviceStateIdle) {
                    break;
                }
            }
            retry_delay *= 2; // Double the retry delay
            continue;
        }
        retry_count = 0;
        retry_delay = 10; // Reset retry delay

        if (ota_->HasNewVersion()) {
            if (UpgradeFirmware(ota_->GetFirmwareUrl(), ota_->GetFirmwareVersion())) {
                return; // This line will never be reached after reboot
            }
            // If upgrade failed, continue to normal operation
        }

        // No new version, mark the current version as valid
        ota_->MarkCurrentVersionValid();
        if (!ota_->HasActivationCode() && !ota_->HasActivationChallenge()) {
            // Exit the loop if done checking new version
            break;
        }

        display->SetStatus(Lang::Strings::ACTIVATION);
        // Activation code is shown to the user and waiting for the user to input
        if (ota_->HasActivationCode()) {
            ShowActivationCode(ota_->GetActivationCode(), ota_->GetActivationMessage());
        }

        // This will block the loop until the activation is done or timeout
        for (int i = 0; i < 10; ++i) {
            ESP_LOGI(TAG, "Activating... %d/%d", i + 1, 10);
            esp_err_t err = ota_->Activate();
            if (err == ESP_OK) {
                break;
            } else if (err == ESP_ERR_TIMEOUT) {
                vTaskDelay(pdMS_TO_TICKS(3000));
            } else {
                vTaskDelay(pdMS_TO_TICKS(10000));
            }
            if (GetDeviceState() == kDeviceStateIdle) {
                break;
            }
        }
    }
}

void Application::InitializeProtocol() {
    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto codec = board.GetAudioCodec();

    display->SetStatus(Lang::Strings::LOADING_PROTOCOL);

    if (ota_->HasMqttConfig()) {
        protocol_ = std::make_unique<MqttProtocol>();
    } else if (ota_->HasWebsocketConfig()) {
        protocol_ = std::make_unique<WebsocketProtocol>();
    } else {
        ESP_LOGW(TAG, "No protocol specified in the OTA config, using MQTT");
        protocol_ = std::make_unique<MqttProtocol>();
    }

    protocol_->OnConnected([this]() {
        DismissAlert();
    });

    protocol_->OnNetworkError([this](const std::string& message) {
        last_error_message_ = message;
        xEventGroupSetBits(event_group_, MAIN_EVENT_ERROR);
    });
    
    protocol_->OnIncomingAudio([this](std::unique_ptr<AudioStreamPacket> packet) {
        if (GetDeviceState() == kDeviceStateSpeaking) {
            audio_service_.PushPacketToDecodeQueue(std::move(packet));
        }
    });
    
    protocol_->OnAudioChannelOpened([this, codec, &board]() {
        board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
        if (protocol_->server_sample_rate() != codec->output_sample_rate()) {
            ESP_LOGW(TAG, "Server sample rate %d does not match device output sample rate %d, resampling may cause distortion",
                protocol_->server_sample_rate(), codec->output_sample_rate());
        }
    });
    
    protocol_->OnAudioChannelClosed([this, &board]() {
        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
        Schedule([this]() {
            auto display = Board::GetInstance().GetDisplay();
            display->SetChatMessage("system", "");
            SetDeviceState(kDeviceStateIdle);
        });
    });
    
    protocol_->OnIncomingJson([this, display](const cJSON* root) {
        // Parse JSON data
        auto type = cJSON_GetObjectItem(root, "type");
        if (strcmp(type->valuestring, "tts") == 0) {
            auto state = cJSON_GetObjectItem(root, "state");
            if (strcmp(state->valuestring, "start") == 0) {
                Schedule([this]() {
                    aborted_ = false;
                    SetDeviceState(kDeviceStateSpeaking);
                });
            } else if (strcmp(state->valuestring, "stop") == 0) {
                Schedule([this]() {
                    if (GetDeviceState() == kDeviceStateSpeaking) {
                        if (listening_mode_ == kListeningModeManualStop) {
                            SetDeviceState(kDeviceStateIdle);
                        } else {
                            SetDeviceState(kDeviceStateListening);
                        }
                    }
                });
            } else if (strcmp(state->valuestring, "sentence_start") == 0) {
                auto text = cJSON_GetObjectItem(root, "text");
                if (cJSON_IsString(text)) {
                    ESP_LOGI(TAG, "<< %s", text->valuestring);
                    Schedule([this, display, message = std::string(text->valuestring)]() {
                        display->SetChatMessage("assistant", message.c_str());
                    });
                }
            }
        } else if (strcmp(type->valuestring, "stt") == 0) {
            auto text = cJSON_GetObjectItem(root, "text");
            if (cJSON_IsString(text)) {
                ESP_LOGI(TAG, ">> %s", text->valuestring);
                Schedule([this, display, message = std::string(text->valuestring)]() {
                    display->SetChatMessage("user", message.c_str());
                });
            }
        } else if (strcmp(type->valuestring, "llm") == 0) {
            auto emotion = cJSON_GetObjectItem(root, "emotion");
            if (cJSON_IsString(emotion)) {
                Schedule([this, display, emotion_str = std::string(emotion->valuestring)]() {
                    display->SetEmotion(emotion_str.c_str());
                });
            }
        } else if (strcmp(type->valuestring, "mcp") == 0) {
            auto payload = cJSON_GetObjectItem(root, "payload");
            if (cJSON_IsObject(payload)) {
                McpServer::GetInstance().ParseMessage(payload);
            }
        } else if (strcmp(type->valuestring, "system") == 0) {
            auto command = cJSON_GetObjectItem(root, "command");
            if (cJSON_IsString(command)) {
                ESP_LOGI(TAG, "System command: %s", command->valuestring);
                if (strcmp(command->valuestring, "reboot") == 0) {
                    // Do a reboot if user requests a OTA update
                    Schedule([this]() {
                        Reboot();
                    });
                } else {
                    ESP_LOGW(TAG, "Unknown system command: %s", command->valuestring);
                }
            }
        } else if (strcmp(type->valuestring, "alert") == 0) {
            auto status = cJSON_GetObjectItem(root, "status");
            auto message = cJSON_GetObjectItem(root, "message");
            auto emotion = cJSON_GetObjectItem(root, "emotion");
            if (cJSON_IsString(status) && cJSON_IsString(message) && cJSON_IsString(emotion)) {
                Alert(status->valuestring, message->valuestring, emotion->valuestring, Lang::Sounds::OGG_VIBRATION);
            } else {
                ESP_LOGW(TAG, "Alert command requires status, message and emotion");
            }
#if CONFIG_RECEIVE_CUSTOM_MESSAGE
        } else if (strcmp(type->valuestring, "custom") == 0) {
            auto payload = cJSON_GetObjectItem(root, "payload");
            ESP_LOGI(TAG, "Received custom message: %s", cJSON_PrintUnformatted(root));
            if (cJSON_IsObject(payload)) {
                Schedule([this, display, payload_str = std::string(cJSON_PrintUnformatted(payload))]() {
                    display->SetChatMessage("system", payload_str.c_str());
                });
            } else {
                ESP_LOGW(TAG, "Invalid custom message format: missing payload");
            }
#endif
        } else {
            ESP_LOGW(TAG, "Unknown message type: %s", type->valuestring);
        }
    });
    
    protocol_->Start();
}

void Application::ShowActivationCode(const std::string& code, const std::string& message) {
    struct digit_sound {
        char digit;
        const std::string_view& sound;
    };
    static const std::array<digit_sound, 10> digit_sounds{{
        digit_sound{'0', Lang::Sounds::OGG_0},
        digit_sound{'1', Lang::Sounds::OGG_1}, 
        digit_sound{'2', Lang::Sounds::OGG_2},
        digit_sound{'3', Lang::Sounds::OGG_3},
        digit_sound{'4', Lang::Sounds::OGG_4},
        digit_sound{'5', Lang::Sounds::OGG_5},
        digit_sound{'6', Lang::Sounds::OGG_6},
        digit_sound{'7', Lang::Sounds::OGG_7},
        digit_sound{'8', Lang::Sounds::OGG_8},
        digit_sound{'9', Lang::Sounds::OGG_9}
    }};

    // This sentence uses 9KB of SRAM, so we need to wait for it to finish
    Alert(Lang::Strings::ACTIVATION, message.c_str(), "link", Lang::Sounds::OGG_ACTIVATION);

    for (const auto& digit : code) {
        auto it = std::find_if(digit_sounds.begin(), digit_sounds.end(),
            [digit](const digit_sound& ds) { return ds.digit == digit; });
        if (it != digit_sounds.end()) {
            audio_service_.PlaySound(it->sound);
        }
    }
}

void Application::Alert(const char* status, const char* message, const char* emotion, const std::string_view& sound) {
    ESP_LOGW(TAG, "Alert [%s] %s: %s", emotion, status, message);
    auto display = Board::GetInstance().GetDisplay();
    display->SetStatus(status);
    display->SetEmotion(emotion);
    display->SetChatMessage("system", message);
    if (!sound.empty()) {
        audio_service_.PlaySound(sound);
    }
}

void Application::DismissAlert() {
    if (GetDeviceState() == kDeviceStateIdle) {
        auto display = Board::GetInstance().GetDisplay();
        display->SetStatus(Lang::Strings::STANDBY);
        display->SetEmotion("neutral");
        display->SetChatMessage("system", "");
    }
}

void Application::ToggleChatState() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_TOGGLE_CHAT);
}

void Application::StartListening() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_START_LISTENING);
}

void Application::StopListening() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_STOP_LISTENING);
}

void Application::HandleToggleChatEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateActivating) {
        SetDeviceState(kDeviceStateIdle);
        return;
    } else if (state == kDeviceStateWifiConfiguring) {
        audio_service_.EnableAudioTesting(true);
        SetDeviceState(kDeviceStateAudioTesting);
        return;
    } else if (state == kDeviceStateAudioTesting) {
        audio_service_.EnableAudioTesting(false);
        SetDeviceState(kDeviceStateWifiConfiguring);
        return;
    }

    if (!protocol_) {
        ESP_LOGE(TAG, "Protocol not initialized");
        return;
    }

    if (state == kDeviceStateIdle) {
        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                return;
            }
        }

        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonNone);
    } else if (state == kDeviceStateListening) {
        protocol_->CloseAudioChannel();
    }
}

void Application::HandleStartListeningEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateActivating) {
        SetDeviceState(kDeviceStateIdle);
        return;
    } else if (state == kDeviceStateWifiConfiguring) {
        audio_service_.EnableAudioTesting(true);
        SetDeviceState(kDeviceStateAudioTesting);
        return;
    }

    if (!protocol_) {
        ESP_LOGE(TAG, "Protocol not initialized");
        return;
    }
    
    if (state == kDeviceStateIdle) {
        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                return;
            }
        }

        SetListeningMode(kListeningModeManualStop);
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonNone);
        SetListeningMode(kListeningModeManualStop);
    }
}

void Application::HandleStopListeningEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateAudioTesting) {
        audio_service_.EnableAudioTesting(false);
        SetDeviceState(kDeviceStateWifiConfiguring);
        return;
    } else if (state == kDeviceStateListening) {
        if (protocol_) {
            protocol_->SendStopListening();
        }
        SetDeviceState(kDeviceStateIdle);
    }
}

void Application::HandleWakeWordDetectedEvent() {
    if (!protocol_) {
        return;
    }

    auto state = GetDeviceState();
    
    if (state == kDeviceStateIdle) {
        audio_service_.EncodeWakeWord();

        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                audio_service_.EnableWakeWordDetection(true);
                return;
            }
        }

        auto wake_word = audio_service_.GetLastWakeWord();
        ESP_LOGI(TAG, "Wake word detected: %s", wake_word.c_str());
#if CONFIG_SEND_WAKE_WORD_DATA
        // Encode and send the wake word data to the server
        while (auto packet = audio_service_.PopWakeWordPacket()) {
            protocol_->SendAudio(std::move(packet));
        }
        // Set the chat state to wake word detected
        protocol_->SendWakeWordDetected(wake_word);
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#else
        // Set flag to play popup sound after state changes to listening
        // (PlaySound here would be cleared by ResetDecoder in EnableVoiceProcessing)
        play_popup_on_listening_ = true;
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#endif
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonWakeWordDetected);
    } else if (state == kDeviceStateActivating) {
        // Restart the activation check if the wake word is detected during activation
        SetDeviceState(kDeviceStateIdle);
    }
}

void Application::HandleStateChangedEvent() {
    DeviceState new_state = state_machine_.GetState();
    clock_ticks_ = 0;

    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto led = board.GetLed();
    led->OnStateChanged();
    
    switch (new_state) {
        case kDeviceStateUnknown:
        case kDeviceStateIdle:
            display->SetStatus(Lang::Strings::STANDBY);
            display->SetEmotion("neutral");
            audio_service_.EnableVoiceProcessing(false);
            audio_service_.EnableWakeWordDetection(true);
            break;
        case kDeviceStateConnecting:
            display->SetStatus(Lang::Strings::CONNECTING);
            display->SetEmotion("neutral");
            display->SetChatMessage("system", "");
            break;
        case kDeviceStateListening:
            display->SetStatus(Lang::Strings::LISTENING);
            display->SetEmotion("neutral");

            // Make sure the audio processor is running
            if (!audio_service_.IsAudioProcessorRunning()) {
                // Send the start listening command
                protocol_->SendStartListening(listening_mode_);
                audio_service_.EnableVoiceProcessing(true);
                audio_service_.EnableWakeWordDetection(false);
            }

            // Play popup sound after ResetDecoder (in EnableVoiceProcessing) has been called
            if (play_popup_on_listening_) {
                play_popup_on_listening_ = false;
                audio_service_.PlaySound(Lang::Sounds::OGG_POPUP);
            }
            break;
        case kDeviceStateSpeaking:
            display->SetStatus(Lang::Strings::SPEAKING);

            // 记录 AI 开始说话的时间，用于语音打断保护
            speaking_start_time_ = esp_timer_get_time() / 1000;
            voice_detected_start_time_ = 0;  // 重置用户声音检测计时

            if (listening_mode_ != kListeningModeRealtime) {
                audio_service_.EnableVoiceProcessing(false);
                // Only AFE wake word can be detected in speaking mode
                audio_service_.EnableWakeWordDetection(audio_service_.IsAfeWakeWord());
            }
            audio_service_.ResetDecoder();
            break;
        case kDeviceStateWifiConfiguring:
            audio_service_.EnableVoiceProcessing(false);
            audio_service_.EnableWakeWordDetection(false);
            break;
        default:
            // Do nothing
            break;
    }
}

void Application::Schedule(std::function<void()>&& callback) {
    {
        std::lock_guard<std::mutex> lock(mutex_);
        main_tasks_.push_back(std::move(callback));
    }
    xEventGroupSetBits(event_group_, MAIN_EVENT_SCHEDULE);
}

void Application::AbortSpeaking(AbortReason reason) {
    ESP_LOGI(TAG, "Abort speaking");
    aborted_ = true;
    if (protocol_) {
        protocol_->SendAbortSpeaking(reason);
    }
}

void Application::SetListeningMode(ListeningMode mode) {
    listening_mode_ = mode;
    SetDeviceState(kDeviceStateListening);
}

void Application::Reboot() {
    ESP_LOGI(TAG, "Rebooting...");
    // Disconnect the audio channel
    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        protocol_->CloseAudioChannel();
    }
    protocol_.reset();
    audio_service_.Stop();

    vTaskDelay(pdMS_TO_TICKS(1000));
    esp_restart();
}

bool Application::UpgradeFirmware(const std::string& url, const std::string& version) {
    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();

    std::string upgrade_url = url;
    std::string version_info = version.empty() ? "(Manual upgrade)" : version;

    // Close audio channel if it's open
    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        ESP_LOGI(TAG, "Closing audio channel before firmware upgrade");
        protocol_->CloseAudioChannel();
    }
    ESP_LOGI(TAG, "Starting firmware upgrade from URL: %s", upgrade_url.c_str());

    Alert(Lang::Strings::OTA_UPGRADE, Lang::Strings::UPGRADING, "download", Lang::Sounds::OGG_UPGRADE);
    vTaskDelay(pdMS_TO_TICKS(3000));

    SetDeviceState(kDeviceStateUpgrading);

    std::string message = std::string(Lang::Strings::NEW_VERSION) + version_info;
    display->SetChatMessage("system", message.c_str());

    board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
    audio_service_.Stop();
    vTaskDelay(pdMS_TO_TICKS(1000));

    bool upgrade_success = Ota::Upgrade(upgrade_url, [display](int progress, size_t speed) {
        std::thread([display, progress, speed]() {
            char buffer[32];
            snprintf(buffer, sizeof(buffer), "%d%% %uKB/s", progress, speed / 1024);
            display->SetChatMessage("system", buffer);
        }).detach();
    });

    if (!upgrade_success) {
        // Upgrade failed, restart audio service and continue running
        ESP_LOGE(TAG, "Firmware upgrade failed, restarting audio service and continuing operation...");
        audio_service_.Start(); // Restart audio service
        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER); // Restore power save level
        Alert(Lang::Strings::ERROR, Lang::Strings::UPGRADE_FAILED, "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
        vTaskDelay(pdMS_TO_TICKS(3000));
        return false;
    } else {
        // Upgrade success, reboot immediately
        ESP_LOGI(TAG, "Firmware upgrade successful, rebooting...");
        display->SetChatMessage("system", "Upgrade successful, rebooting...");
        vTaskDelay(pdMS_TO_TICKS(1000)); // Brief pause to show message
        Reboot();
        return true;
    }
}

void Application::WakeWordInvoke(const std::string& wake_word) {
    if (!protocol_) {
        return;
    }

    auto state = GetDeviceState();
    
    if (state == kDeviceStateIdle) {
        audio_service_.EncodeWakeWord();

        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                audio_service_.EnableWakeWordDetection(true);
                return;
            }
        }

        ESP_LOGI(TAG, "Wake word detected: %s", wake_word.c_str());
#if CONFIG_USE_AFE_WAKE_WORD || CONFIG_USE_CUSTOM_WAKE_WORD
        // Encode and send the wake word data to the server
        while (auto packet = audio_service_.PopWakeWordPacket()) {
            protocol_->SendAudio(std::move(packet));
        }
        // Set the chat state to wake word detected
        protocol_->SendWakeWordDetected(wake_word);
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#else
        // Set flag to play popup sound after state changes to listening
        // (PlaySound here would be cleared by ResetDecoder in EnableVoiceProcessing)
        play_popup_on_listening_ = true;
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#endif
    } else if (state == kDeviceStateSpeaking) {
        Schedule([this]() {
            AbortSpeaking(kAbortReasonNone);
        });
    } else if (state == kDeviceStateListening) {   
        Schedule([this]() {
            if (protocol_) {
                protocol_->CloseAudioChannel();
            }
        });
    }
}

bool Application::CanEnterSleepMode() {
    if (GetDeviceState() != kDeviceStateIdle) {
        return false;
    }

    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        return false;
    }

    if (!audio_service_.IsIdle()) {
        return false;
    }

    // Now it is safe to enter sleep mode
    return true;
}

void Application::SendMcpMessage(const std::string& payload) {
    // Always schedule to run in main task for thread safety
    Schedule([this, payload = std::move(payload)]() {
        if (protocol_) {
            protocol_->SendMcpMessage(payload);
        }
    });
}

void Application::SetAecMode(AecMode mode) {
    aec_mode_ = mode;
    Schedule([this]() {
        auto& board = Board::GetInstance();
        auto display = board.GetDisplay();
        switch (aec_mode_) {
        case kAecOff:
            audio_service_.EnableDeviceAec(false);
            display->ShowNotification(Lang::Strings::RTC_MODE_OFF);
            break;
        case kAecOnServerSide:
            audio_service_.EnableDeviceAec(false);
            display->ShowNotification(Lang::Strings::RTC_MODE_ON);
            break;
        case kAecOnDeviceSide:
            audio_service_.EnableDeviceAec(true);
            display->ShowNotification(Lang::Strings::RTC_MODE_ON);
            break;
        }

        // If the AEC mode is changed, close the audio channel
        if (protocol_ && protocol_->IsAudioChannelOpened()) {
            protocol_->CloseAudioChannel();
        }
    });
}

void Application::PlaySound(const std::string_view& sound) {
    audio_service_.PlaySound(sound);
}

void Application::ResetProtocol() {
    Schedule([this]() {
        // Close audio channel if opened
        if (protocol_ && protocol_->IsAudioChannelOpened()) {
            protocol_->CloseAudioChannel();
        }
        // Reset protocol
        protocol_.reset();
    });
}


```

现在重新编译固件：

---

**User**

现在的代码是不是有错啊，build报错了，@ESP-IDF Build 

---

**Cursor**

这个错误**不是我修改的代码导致的**，是你之前的配置问题。

看错误信息：

```
Error: USE_CUSTOM_WAKE_WORD is enabled but no multinet models are selected in sdkconfig
Please select appropriate CONFIG_SR_MN_* options in menuconfig, or disable USE_CUSTOM_WAKE_WORD
```

问题是你启用了自定义唤醒词 (`USE_CUSTOM_WAKE_WORD`)，但没有选择 Multinet 模型。

**解决方法**：

进入 `menuconfig`，导航到：
```
Component config → ESP Speech Recognition → Select Multinet model
```

然后选择一个 Multinet 模型，比如 `Chinese Multinet8q8` 或其他适合你设备的模型。

或者如果你不需要自定义唤醒词（"你好小圆"），可以把唤醒词类型改回 `AFE Wake Word`（这样只能用默认的"你好小智"）。

你想用哪种方式？

1. **保留自定义唤醒词** → 在 menuconfig 中选择 Multinet 模型
2. **使用默认唤醒词** → 把唤醒词类型改回 AFE Wake Word

```plaintext
Error: USE_CUSTOM_WAKE_WORD is enabled but no multinet models are selected in sdkconfig
Please select appropriate CONFIG_SR_MN_* options in menuconfig, or disable USE_CUSTOM_WAKE_WORD
```

```plaintext
Component config → ESP Speech Recognition → Select Multinet model
```

---

**User**

不是按照这样修改就行了吗

---

**Cursor**

你截图里的配置是**唤醒词文本设置**，但还缺少一个关键配置：**Multinet 模型选择**。

这是两个不同的东西：
- ✅ 你已配置：唤醒词文本（"ni hao xiao yuan"）
- ❌ 还需配置：Multinet 语音识别模型

**在 menuconfig 中，还需要去这里选择模型**：

```
Component config
    └── ESP Speech Recognition
        └── Select Multinet model
            └── 选择: Chinese Multinet8q8 (或其他中文模型)
```

Multinet 模型是实际做语音识别的神经网络，没有它就无法识别你说的"你好小圆"。

选好模型后保存退出，再编译就能通过了。

```plaintext
Component config
    └── ESP Speech Recognition
        └── Select Multinet model
            └── 选择: Chinese Multinet8q8 (或其他中文模型)
```

---

**User**

你确定？你最开始都没说需要这样，而且我也没找到Select Multinet model这个选项

---

**Cursor**

抱歉，让我查一下实际的配置选项在哪里：

抱歉之前说错了！正确的路径是：

**在 menuconfig 中配置**：

```
Component config
    └── ESP-SR
        └── Chinese Speech Commands Model
            └── 选择: general chinese recognition (mn7_cn)
                或者: general chinese recognition (mn6_cn)
```

你的设备是 ESP32-S3，所以可以选择：
- `mn7_cn` - 推荐，最新模型
- `mn6_cn` - 也可以

这个选项不叫 "Select Multinet model"，而是叫 **"Chinese Speech Commands Model"**，在 ESP-SR 配置下面。

找到后选一个中文模型，保存退出再编译就行了。

```plaintext
Component config
    └── ESP-SR
        └── Chinese Speech Commands Model
            └── 选择: general chinese recognition (mn7_cn)
                或者: general chinese recognition (mn6_cn)
```

---

**User**

我也没找到啊

---

**Cursor**

让我检查一下你当前的 sdkconfig，然后直接帮你修改：

找到问题了！当前配置是 `CONFIG_SR_MN_CN_NONE=y`（没有选择模型）。

我直接帮你改 sdkconfig 文件：

```kconfig
#
# Automatically generated file. DO NOT EDIT.
# Espressif IoT Development Framework (ESP-IDF) 5.4.3 Project Configuration
#
CONFIG_SOC_MPU_MIN_REGION_SIZE=0x20000000
CONFIG_SOC_MPU_REGIONS_MAX_NUM=8
CONFIG_SOC_ADC_SUPPORTED=y
CONFIG_SOC_UART_SUPPORTED=y
CONFIG_SOC_PCNT_SUPPORTED=y
CONFIG_SOC_PHY_SUPPORTED=y
CONFIG_SOC_WIFI_SUPPORTED=y
CONFIG_SOC_TWAI_SUPPORTED=y
CONFIG_SOC_GDMA_SUPPORTED=y
CONFIG_SOC_UHCI_SUPPORTED=y
CONFIG_SOC_AHB_GDMA_SUPPORTED=y
CONFIG_SOC_GPTIMER_SUPPORTED=y
CONFIG_SOC_LCDCAM_SUPPORTED=y
CONFIG_SOC_LCDCAM_CAM_SUPPORTED=y
CONFIG_SOC_LCDCAM_I80_LCD_SUPPORTED=y
CONFIG_SOC_LCDCAM_RGB_LCD_SUPPORTED=y
CONFIG_SOC_MCPWM_SUPPORTED=y
CONFIG_SOC_DEDICATED_GPIO_SUPPORTED=y
CONFIG_SOC_CACHE_SUPPORT_WRAP=y
CONFIG_SOC_ULP_SUPPORTED=y
CONFIG_SOC_ULP_FSM_SUPPORTED=y
CONFIG_SOC_RISCV_COPROC_SUPPORTED=y
CONFIG_SOC_BT_SUPPORTED=y
CONFIG_SOC_USB_OTG_SUPPORTED=y
CONFIG_SOC_USB_SERIAL_JTAG_SUPPORTED=y
CONFIG_SOC_CCOMP_TIMER_SUPPORTED=y
CONFIG_SOC_ASYNC_MEMCPY_SUPPORTED=y
CONFIG_SOC_SUPPORTS_SECURE_DL_MODE=y
CONFIG_SOC_EFUSE_KEY_PURPOSE_FIELD=y
CONFIG_SOC_EFUSE_SUPPORTED=y
CONFIG_SOC_SDMMC_HOST_SUPPORTED=y
CONFIG_SOC_RTC_FAST_MEM_SUPPORTED=y
CONFIG_SOC_RTC_SLOW_MEM_SUPPORTED=y
CONFIG_SOC_RTC_MEM_SUPPORTED=y
CONFIG_SOC_PSRAM_DMA_CAPABLE=y
CONFIG_SOC_XT_WDT_SUPPORTED=y
CONFIG_SOC_I2S_SUPPORTED=y
CONFIG_SOC_RMT_SUPPORTED=y
CONFIG_SOC_SDM_SUPPORTED=y
CONFIG_SOC_GPSPI_SUPPORTED=y
CONFIG_SOC_LEDC_SUPPORTED=y
CONFIG_SOC_I2C_SUPPORTED=y
CONFIG_SOC_SYSTIMER_SUPPORTED=y
CONFIG_SOC_SUPPORT_COEXISTENCE=y
CONFIG_SOC_TEMP_SENSOR_SUPPORTED=y
CONFIG_SOC_AES_SUPPORTED=y
CONFIG_SOC_MPI_SUPPORTED=y
CONFIG_SOC_SHA_SUPPORTED=y
CONFIG_SOC_HMAC_SUPPORTED=y
CONFIG_SOC_DIG_SIGN_SUPPORTED=y
CONFIG_SOC_FLASH_ENC_SUPPORTED=y
CONFIG_SOC_SECURE_BOOT_SUPPORTED=y
CONFIG_SOC_MEMPROT_SUPPORTED=y
CONFIG_SOC_TOUCH_SENSOR_SUPPORTED=y
CONFIG_SOC_BOD_SUPPORTED=y
CONFIG_SOC_CLK_TREE_SUPPORTED=y
CONFIG_SOC_MPU_SUPPORTED=y
CONFIG_SOC_WDT_SUPPORTED=y
CONFIG_SOC_SPI_FLASH_SUPPORTED=y
CONFIG_SOC_RNG_SUPPORTED=y
CONFIG_SOC_LIGHT_SLEEP_SUPPORTED=y
CONFIG_SOC_DEEP_SLEEP_SUPPORTED=y
CONFIG_SOC_LP_PERIPH_SHARE_INTERRUPT=y
CONFIG_SOC_PM_SUPPORTED=y
CONFIG_SOC_SIMD_INSTRUCTION_SUPPORTED=y
CONFIG_SOC_XTAL_SUPPORT_40M=y
CONFIG_SOC_APPCPU_HAS_CLOCK_GATING_BUG=y
CONFIG_SOC_ADC_RTC_CTRL_SUPPORTED=y
CONFIG_SOC_ADC_DIG_CTRL_SUPPORTED=y
CONFIG_SOC_ADC_ARBITER_SUPPORTED=y
CONFIG_SOC_ADC_DIG_IIR_FILTER_SUPPORTED=y
CONFIG_SOC_ADC_MONITOR_SUPPORTED=y
CONFIG_SOC_ADC_DMA_SUPPORTED=y
CONFIG_SOC_ADC_PERIPH_NUM=2
CONFIG_SOC_ADC_MAX_CHANNEL_NUM=10
CONFIG_SOC_ADC_ATTEN_NUM=4
CONFIG_SOC_ADC_DIGI_CONTROLLER_NUM=2
CONFIG_SOC_ADC_PATT_LEN_MAX=24
CONFIG_SOC_ADC_DIGI_MIN_BITWIDTH=12
CONFIG_SOC_ADC_DIGI_MAX_BITWIDTH=12
CONFIG_SOC_ADC_DIGI_RESULT_BYTES=4
CONFIG_SOC_ADC_DIGI_DATA_BYTES_PER_CONV=4
CONFIG_SOC_ADC_DIGI_IIR_FILTER_NUM=2
CONFIG_SOC_ADC_DIGI_MONITOR_NUM=2
CONFIG_SOC_ADC_SAMPLE_FREQ_THRES_HIGH=83333
CONFIG_SOC_ADC_SAMPLE_FREQ_THRES_LOW=611
CONFIG_SOC_ADC_RTC_MIN_BITWIDTH=12
CONFIG_SOC_ADC_RTC_MAX_BITWIDTH=12
CONFIG_SOC_ADC_CALIBRATION_V1_SUPPORTED=y
CONFIG_SOC_ADC_SELF_HW_CALI_SUPPORTED=y
CONFIG_SOC_ADC_SHARED_POWER=y
CONFIG_SOC_APB_BACKUP_DMA=y
CONFIG_SOC_BROWNOUT_RESET_SUPPORTED=y
CONFIG_SOC_CACHE_WRITEBACK_SUPPORTED=y
CONFIG_SOC_CACHE_FREEZE_SUPPORTED=y
CONFIG_SOC_CPU_CORES_NUM=2
CONFIG_SOC_CPU_INTR_NUM=32
CONFIG_SOC_CPU_HAS_FPU=y
CONFIG_SOC_HP_CPU_HAS_MULTIPLE_CORES=y
CONFIG_SOC_CPU_BREAKPOINTS_NUM=2
CONFIG_SOC_CPU_WATCHPOINTS_NUM=2
CONFIG_SOC_CPU_WATCHPOINT_MAX_REGION_SIZE=0x40
CONFIG_SOC_SIMD_PREFERRED_DATA_ALIGNMENT=16
CONFIG_SOC_DS_SIGNATURE_MAX_BIT_LEN=4096
CONFIG_SOC_DS_KEY_PARAM_MD_IV_LENGTH=16
CONFIG_SOC_DS_KEY_CHECK_MAX_WAIT_US=1100
CONFIG_SOC_AHB_GDMA_VERSION=1
CONFIG_SOC_GDMA_NUM_GROUPS_MAX=1
CONFIG_SOC_GDMA_PAIRS_PER_GROUP=5
CONFIG_SOC_GDMA_PAIRS_PER_GROUP_MAX=5
CONFIG_SOC_AHB_GDMA_SUPPORT_PSRAM=y
CONFIG_SOC_GPIO_PORT=1
CONFIG_SOC_GPIO_PIN_COUNT=49
CONFIG_SOC_GPIO_SUPPORT_PIN_GLITCH_FILTER=y
CONFIG_SOC_GPIO_FILTER_CLK_SUPPORT_APB=y
CONFIG_SOC_GPIO_SUPPORT_RTC_INDEPENDENT=y
CONFIG_SOC_GPIO_SUPPORT_FORCE_HOLD=y
CONFIG_SOC_GPIO_VALID_GPIO_MASK=0x1FFFFFFFFFFFF
CONFIG_SOC_GPIO_IN_RANGE_MAX=48
CONFIG_SOC_GPIO_OUT_RANGE_MAX=48
CONFIG_SOC_GPIO_VALID_DIGITAL_IO_PAD_MASK=0x0001FFFFFC000000
CONFIG_SOC_GPIO_CLOCKOUT_BY_IO_MUX=y
CONFIG_SOC_GPIO_CLOCKOUT_CHANNEL_NUM=3
CONFIG_SOC_GPIO_SUPPORT_HOLD_IO_IN_DSLP=y
CONFIG_SOC_DEDIC_GPIO_OUT_CHANNELS_NUM=8
CONFIG_SOC_DEDIC_GPIO_IN_CHANNELS_NUM=8
CONFIG_SOC_DEDIC_GPIO_OUT_AUTO_ENABLE=y
CONFIG_SOC_I2C_NUM=2
CONFIG_SOC_HP_I2C_NUM=2
CONFIG_SOC_I2C_FIFO_LEN=32
CONFIG_SOC_I2C_CMD_REG_NUM=8
CONFIG_SOC_I2C_SUPPORT_SLAVE=y
CONFIG_SOC_I2C_SUPPORT_HW_CLR_BUS=y
CONFIG_SOC_I2C_SUPPORT_XTAL=y
CONFIG_SOC_I2C_SUPPORT_RTC=y
CONFIG_SOC_I2C_SUPPORT_10BIT_ADDR=y
CONFIG_SOC_I2C_SLAVE_SUPPORT_BROADCAST=y
CONFIG_SOC_I2C_SLAVE_SUPPORT_I2CRAM_ACCESS=y
CONFIG_SOC_I2C_SLAVE_CAN_GET_STRETCH_CAUSE=y
CONFIG_SOC_I2S_NUM=2
CONFIG_SOC_I2S_HW_VERSION_2=y
CONFIG_SOC_I2S_SUPPORTS_XTAL=y
CONFIG_SOC_I2S_SUPPORTS_PLL_F160M=y
CONFIG_SOC_I2S_SUPPORTS_PCM=y
CONFIG_SOC_I2S_SUPPORTS_PDM=y
CONFIG_SOC_I2S_SUPPORTS_PDM_TX=y
CONFIG_SOC_I2S_PDM_MAX_TX_LINES=2
CONFIG_SOC_I2S_SUPPORTS_PDM_RX=y
CONFIG_SOC_I2S_PDM_MAX_RX_LINES=4
CONFIG_SOC_I2S_SUPPORTS_TDM=y
CONFIG_SOC_LEDC_SUPPORT_APB_CLOCK=y
CONFIG_SOC_LEDC_SUPPORT_XTAL_CLOCK=y
CONFIG_SOC_LEDC_TIMER_NUM=4
CONFIG_SOC_LEDC_CHANNEL_NUM=8
CONFIG_SOC_LEDC_TIMER_BIT_WIDTH=14
CONFIG_SOC_LEDC_SUPPORT_FADE_STOP=y
CONFIG_SOC_MCPWM_GROUPS=2
CONFIG_SOC_MCPWM_TIMERS_PER_GROUP=3
CONFIG_SOC_MCPWM_OPERATORS_PER_GROUP=3
CONFIG_SOC_MCPWM_COMPARATORS_PER_OPERATOR=2
CONFIG_SOC_MCPWM_GENERATORS_PER_OPERATOR=2
CONFIG_SOC_MCPWM_TRIGGERS_PER_OPERATOR=2
CONFIG_SOC_MCPWM_GPIO_FAULTS_PER_GROUP=3
CONFIG_SOC_MCPWM_CAPTURE_TIMERS_PER_GROUP=y
CONFIG_SOC_MCPWM_CAPTURE_CHANNELS_PER_TIMER=3
CONFIG_SOC_MCPWM_GPIO_SYNCHROS_PER_GROUP=3
CONFIG_SOC_MCPWM_SWSYNC_CAN_PROPAGATE=y
CONFIG_SOC_MMU_LINEAR_ADDRESS_REGION_NUM=1
CONFIG_SOC_MMU_PERIPH_NUM=1
CONFIG_SOC_PCNT_GROUPS=1
CONFIG_SOC_PCNT_UNITS_PER_GROUP=4
CONFIG_SOC_PCNT_CHANNELS_PER_UNIT=2
CONFIG_SOC_PCNT_THRES_POINT_PER_UNIT=2
CONFIG_SOC_RMT_GROUPS=1
CONFIG_SOC_RMT_TX_CANDIDATES_PER_GROUP=4
CONFIG_SOC_RMT_RX_CANDIDATES_PER_GROUP=4
CONFIG_SOC_RMT_CHANNELS_PER_GROUP=8
CONFIG_SOC_RMT_MEM_WORDS_PER_CHANNEL=48
CONFIG_SOC_RMT_SUPPORT_RX_PINGPONG=y
CONFIG_SOC_RMT_SUPPORT_RX_DEMODULATION=y
CONFIG_SOC_RMT_SUPPORT_ASYNC_STOP=y
CONFIG_SOC_RMT_SUPPORT_TX_LOOP_COUNT=y
CONFIG_SOC_RMT_SUPPORT_TX_LOOP_AUTO_STOP=y
CONFIG_SOC_RMT_SUPPORT_TX_SYNCHRO=y
CONFIG_SOC_RMT_SUPPORT_TX_CARRIER_DATA_ONLY=y
CONFIG_SOC_RMT_SUPPORT_XTAL=y
CONFIG_SOC_RMT_SUPPORT_RC_FAST=y
CONFIG_SOC_RMT_SUPPORT_APB=y
CONFIG_SOC_RMT_SUPPORT_DMA=y
CONFIG_SOC_LCD_I80_SUPPORTED=y
CONFIG_SOC_LCD_RGB_SUPPORTED=y
CONFIG_SOC_LCD_I80_BUSES=1
CONFIG_SOC_LCD_RGB_PANELS=1
CONFIG_SOC_LCD_I80_BUS_WIDTH=16
CONFIG_SOC_LCD_RGB_DATA_WIDTH=16
CONFIG_SOC_LCD_SUPPORT_RGB_YUV_CONV=y
CONFIG_SOC_LCDCAM_I80_NUM_BUSES=1
CONFIG_SOC_LCDCAM_I80_BUS_WIDTH=16
CONFIG_SOC_LCDCAM_RGB_NUM_PANELS=1
CONFIG_SOC_LCDCAM_RGB_DATA_WIDTH=16
CONFIG_SOC_RTC_CNTL_CPU_PD_DMA_BUS_WIDTH=128
CONFIG_SOC_RTC_CNTL_CPU_PD_REG_FILE_NUM=549
CONFIG_SOC_RTC_CNTL_TAGMEM_PD_DMA_BUS_WIDTH=128
CONFIG_SOC_RTCIO_PIN_COUNT=22
CONFIG_SOC_RTCIO_INPUT_OUTPUT_SUPPORTED=y
CONFIG_SOC_RTCIO_HOLD_SUPPORTED=y
CONFIG_SOC_RTCIO_WAKE_SUPPORTED=y
CONFIG_SOC_LP_IO_CLOCK_IS_INDEPENDENT=y
CONFIG_SOC_SDM_GROUPS=1
CONFIG_SOC_SDM_CHANNELS_PER_GROUP=8
CONFIG_SOC_SDM_CLK_SUPPORT_APB=y
CONFIG_SOC_SPI_PERIPH_NUM=3
CONFIG_SOC_SPI_MAX_CS_NUM=6
CONFIG_SOC_SPI_MAXIMUM_BUFFER_SIZE=64
CONFIG_SOC_SPI_SUPPORT_DDRCLK=y
CONFIG_SOC_SPI_SLAVE_SUPPORT_SEG_TRANS=y
CONFIG_SOC_SPI_SUPPORT_CD_SIG=y
CONFIG_SOC_SPI_SUPPORT_CONTINUOUS_TRANS=y
CONFIG_SOC_SPI_SUPPORT_SLAVE_HD_VER2=y
CONFIG_SOC_SPI_SUPPORT_CLK_APB=y
CONFIG_SOC_SPI_SUPPORT_CLK_XTAL=y
CONFIG_SOC_SPI_PERIPH_SUPPORT_CONTROL_DUMMY_OUT=y
CONFIG_SOC_MEMSPI_IS_INDEPENDENT=y
CONFIG_SOC_SPI_MAX_PRE_DIVIDER=16
CONFIG_SOC_SPI_SUPPORT_OCT=y
CONFIG_SOC_SPI_SCT_SUPPORTED=y
CONFIG_SOC_SPI_SCT_REG_NUM=14
CONFIG_SOC_SPI_SCT_BUFFER_NUM_MAX=y
CONFIG_SOC_SPI_SCT_CONF_BITLEN_MAX=0x3FFFA
CONFIG_SOC_MEMSPI_SRC_FREQ_120M=y
CONFIG_SOC_MEMSPI_SRC_FREQ_80M_SUPPORTED=y
CONFIG_SOC_MEMSPI_SRC_FREQ_40M_SUPPORTED=y
CONFIG_SOC_MEMSPI_SRC_FREQ_20M_SUPPORTED=y
CONFIG_SOC_SPIRAM_SUPPORTED=y
CONFIG_SOC_SPIRAM_XIP_SUPPORTED=y
CONFIG_SOC_SYSTIMER_COUNTER_NUM=2
CONFIG_SOC_SYSTIMER_ALARM_NUM=3
CONFIG_SOC_SYSTIMER_BIT_WIDTH_LO=32
CONFIG_SOC_SYSTIMER_BIT_WIDTH_HI=20
CONFIG_SOC_SYSTIMER_FIXED_DIVIDER=y
CONFIG_SOC_SYSTIMER_INT_LEVEL=y
CONFIG_SOC_SYSTIMER_ALARM_MISS_COMPENSATE=y
CONFIG_SOC_TIMER_GROUPS=2
CONFIG_SOC_TIMER_GROUP_TIMERS_PER_GROUP=2
CONFIG_SOC_TIMER_GROUP_COUNTER_BIT_WIDTH=54
CONFIG_SOC_TIMER_GROUP_SUPPORT_XTAL=y
CONFIG_SOC_TIMER_GROUP_SUPPORT_APB=y
CONFIG_SOC_TIMER_GROUP_TOTAL_TIMERS=4
CONFIG_SOC_LP_TIMER_BIT_WIDTH_LO=32
CONFIG_SOC_LP_TIMER_BIT_WIDTH_HI=16
CONFIG_SOC_TOUCH_SENSOR_VERSION=2
CONFIG_SOC_TOUCH_SENSOR_NUM=15
CONFIG_SOC_TOUCH_SUPPORT_SLEEP_WAKEUP=y
CONFIG_SOC_TOUCH_SUPPORT_WATERPROOF=y
CONFIG_SOC_TOUCH_SUPPORT_PROX_SENSING=y
CONFIG_SOC_TOUCH_PROXIMITY_CHANNEL_NUM=3
CONFIG_SOC_TOUCH_PROXIMITY_MEAS_DONE_SUPPORTED=y
CONFIG_SOC_TOUCH_SAMPLE_CFG_NUM=1
CONFIG_SOC_TWAI_CONTROLLER_NUM=1
CONFIG_SOC_TWAI_CLK_SUPPORT_APB=y
CONFIG_SOC_TWAI_BRP_MIN=2
CONFIG_SOC_TWAI_BRP_MAX=16384
CONFIG_SOC_TWAI_SUPPORTS_RX_STATUS=y
CONFIG_SOC_UART_NUM=3
CONFIG_SOC_UART_HP_NUM=3
CONFIG_SOC_UART_FIFO_LEN=128
CONFIG_SOC_UART_BITRATE_MAX=5000000
CONFIG_SOC_UART_SUPPORT_FSM_TX_WAIT_SEND=y
CONFIG_SOC_UART_SUPPORT_WAKEUP_INT=y
CONFIG_SOC_UART_SUPPORT_APB_CLK=y
CONFIG_SOC_UART_SUPPORT_RTC_CLK=y
CONFIG_SOC_UART_SUPPORT_XTAL_CLK=y
CONFIG_SOC_UHCI_NUM=1
CONFIG_SOC_USB_OTG_PERIPH_NUM=1
CONFIG_SOC_SHA_DMA_MAX_BUFFER_SIZE=3968
CONFIG_SOC_SHA_SUPPORT_DMA=y
CONFIG_SOC_SHA_SUPPORT_RESUME=y
CONFIG_SOC_SHA_GDMA=y
CONFIG_SOC_SHA_SUPPORT_SHA1=y
CONFIG_SOC_SHA_SUPPORT_SHA224=y
CONFIG_SOC_SHA_SUPPORT_SHA256=y
CONFIG_SOC_SHA_SUPPORT_SHA384=y
CONFIG_SOC_SHA_SUPPORT_SHA512=y
CONFIG_SOC_SHA_SUPPORT_SHA512_224=y
CONFIG_SOC_SHA_SUPPORT_SHA512_256=y
CONFIG_SOC_SHA_SUPPORT_SHA512_T=y
CONFIG_SOC_MPI_MEM_BLOCKS_NUM=4
CONFIG_SOC_MPI_OPERATIONS_NUM=3
CONFIG_SOC_RSA_MAX_BIT_LEN=4096
CONFIG_SOC_AES_SUPPORT_DMA=y
CONFIG_SOC_AES_GDMA=y
CONFIG_SOC_AES_SUPPORT_AES_128=y
CONFIG_SOC_AES_SUPPORT_AES_256=y
CONFIG_SOC_PM_SUPPORT_EXT0_WAKEUP=y
CONFIG_SOC_PM_SUPPORT_EXT1_WAKEUP=y
CONFIG_SOC_PM_SUPPORT_EXT_WAKEUP=y
CONFIG_SOC_PM_SUPPORT_WIFI_WAKEUP=y
CONFIG_SOC_PM_SUPPORT_BT_WAKEUP=y
CONFIG_SOC_PM_SUPPORT_TOUCH_SENSOR_WAKEUP=y
CONFIG_SOC_PM_SUPPORT_CPU_PD=y
CONFIG_SOC_PM_SUPPORT_TAGMEM_PD=y
CONFIG_SOC_PM_SUPPORT_RTC_PERIPH_PD=y
CONFIG_SOC_PM_SUPPORT_RC_FAST_PD=y
CONFIG_SOC_PM_SUPPORT_VDDSDIO_PD=y
CONFIG_SOC_PM_SUPPORT_MAC_BB_PD=y
CONFIG_SOC_PM_SUPPORT_MODEM_PD=y
CONFIG_SOC_CONFIGURABLE_VDDSDIO_SUPPORTED=y
CONFIG_SOC_PM_SUPPORT_DEEPSLEEP_CHECK_STUB_ONLY=y
CONFIG_SOC_PM_CPU_RETENTION_BY_RTCCNTL=y
CONFIG_SOC_PM_MODEM_RETENTION_BY_BACKUPDMA=y
CONFIG_SOC_PM_MODEM_PD_BY_SW=y
CONFIG_SOC_CLK_RC_FAST_D256_SUPPORTED=y
CONFIG_SOC_RTC_SLOW_CLK_SUPPORT_RC_FAST_D256=y
CONFIG_SOC_CLK_RC_FAST_SUPPORT_CALIBRATION=y
CONFIG_SOC_CLK_XTAL32K_SUPPORTED=y
CONFIG_SOC_EFUSE_DIS_DOWNLOAD_ICACHE=y
CONFIG_SOC_EFUSE_DIS_DOWNLOAD_DCACHE=y
CONFIG_SOC_EFUSE_HARD_DIS_JTAG=y
CONFIG_SOC_EFUSE_DIS_USB_JTAG=y
CONFIG_SOC_EFUSE_SOFT_DIS_JTAG=y
CONFIG_SOC_EFUSE_DIS_DIRECT_BOOT=y
CONFIG_SOC_EFUSE_DIS_ICACHE=y
CONFIG_SOC_EFUSE_BLOCK9_KEY_PURPOSE_QUIRK=y
CONFIG_SOC_SECURE_BOOT_V2_RSA=y
CONFIG_SOC_EFUSE_SECURE_BOOT_KEY_DIGESTS=3
CONFIG_SOC_EFUSE_REVOKE_BOOT_KEY_DIGESTS=y
CONFIG_SOC_SUPPORT_SECURE_BOOT_REVOKE_KEY=y
CONFIG_SOC_FLASH_ENCRYPTED_XTS_AES_BLOCK_MAX=64
CONFIG_SOC_FLASH_ENCRYPTION_XTS_AES=y
CONFIG_SOC_FLASH_ENCRYPTION_XTS_AES_OPTIONS=y
CONFIG_SOC_FLASH_ENCRYPTION_XTS_AES_128=y
CONFIG_SOC_FLASH_ENCRYPTION_XTS_AES_256=y
CONFIG_SOC_MEMPROT_CPU_PREFETCH_PAD_SIZE=16
CONFIG_SOC_MEMPROT_MEM_ALIGN_SIZE=256
CONFIG_SOC_PHY_DIG_REGS_MEM_SIZE=21
CONFIG_SOC_MAC_BB_PD_MEM_SIZE=192
CONFIG_SOC_WIFI_LIGHT_SLEEP_CLK_WIDTH=12
CONFIG_SOC_SPI_MEM_SUPPORT_AUTO_WAIT_IDLE=y
CONFIG_SOC_SPI_MEM_SUPPORT_AUTO_SUSPEND=y
CONFIG_SOC_SPI_MEM_SUPPORT_AUTO_RESUME=y
CONFIG_SOC_SPI_MEM_SUPPORT_SW_SUSPEND=y
CONFIG_SOC_SPI_MEM_SUPPORT_OPI_MODE=y
CONFIG_SOC_SPI_MEM_SUPPORT_TIMING_TUNING=y
CONFIG_SOC_SPI_MEM_SUPPORT_CONFIG_GPIO_BY_EFUSE=y
CONFIG_SOC_SPI_MEM_SUPPORT_WRAP=y
CONFIG_SOC_MEMSPI_TIMING_TUNING_BY_MSPI_DELAY=y
CONFIG_SOC_MEMSPI_CORE_CLK_SHARED_WITH_PSRAM=y
CONFIG_SOC_SPI_MEM_SUPPORT_CACHE_32BIT_ADDR_MAP=y
CONFIG_SOC_COEX_HW_PTI=y
CONFIG_SOC_EXTERNAL_COEX_LEADER_TX_LINE=y
CONFIG_SOC_SDMMC_USE_GPIO_MATRIX=y
CONFIG_SOC_SDMMC_NUM_SLOTS=2
CONFIG_SOC_SDMMC_SUPPORT_XTAL_CLOCK=y
CONFIG_SOC_SDMMC_DELAY_PHASE_NUM=4
CONFIG_SOC_TEMPERATURE_SENSOR_SUPPORT_FAST_RC=y
CONFIG_SOC_WIFI_HW_TSF=y
CONFIG_SOC_WIFI_FTM_SUPPORT=y
CONFIG_SOC_WIFI_GCMP_SUPPORT=y
CONFIG_SOC_WIFI_WAPI_SUPPORT=y
CONFIG_SOC_WIFI_CSI_SUPPORT=y
CONFIG_SOC_WIFI_MESH_SUPPORT=y
CONFIG_SOC_WIFI_SUPPORT_VARIABLE_BEACON_WINDOW=y
CONFIG_SOC_WIFI_PHY_NEEDS_USB_WORKAROUND=y
CONFIG_SOC_BLE_SUPPORTED=y
CONFIG_SOC_BLE_MESH_SUPPORTED=y
CONFIG_SOC_BLE_50_SUPPORTED=y
CONFIG_SOC_BLE_DEVICE_PRIVACY_SUPPORTED=y
CONFIG_SOC_BLUFI_SUPPORTED=y
CONFIG_SOC_ULP_HAS_ADC=y
CONFIG_SOC_PHY_COMBO_MODULE=y
CONFIG_SOC_LCDCAM_CAM_SUPPORT_RGB_YUV_CONV=y
CONFIG_SOC_LCDCAM_CAM_PERIPH_NUM=1
CONFIG_SOC_LCDCAM_CAM_DATA_WIDTH_MAX=16
CONFIG_IDF_CMAKE=y
CONFIG_IDF_TOOLCHAIN="gcc"
CONFIG_IDF_TOOLCHAIN_GCC=y
CONFIG_IDF_TARGET_ARCH_XTENSA=y
CONFIG_IDF_TARGET_ARCH="xtensa"
CONFIG_IDF_TARGET="esp32s3"
CONFIG_IDF_INIT_VERSION="5.4.3"
CONFIG_IDF_TARGET_ESP32S3=y
CONFIG_IDF_FIRMWARE_CHIP_ID=0x0009

#
# Build type
#
CONFIG_APP_BUILD_TYPE_APP_2NDBOOT=y
# CONFIG_APP_BUILD_TYPE_RAM is not set
CONFIG_APP_BUILD_GENERATE_BINARIES=y
CONFIG_APP_BUILD_BOOTLOADER=y
CONFIG_APP_BUILD_USE_FLASH_SECTIONS=y
# CONFIG_APP_REPRODUCIBLE_BUILD is not set
# CONFIG_APP_NO_BLOBS is not set
# end of Build type

#
# Bootloader config
#

#
# Bootloader manager
#
CONFIG_BOOTLOADER_COMPILE_TIME_DATE=y
CONFIG_BOOTLOADER_PROJECT_VER=1
# end of Bootloader manager

CONFIG_BOOTLOADER_OFFSET_IN_FLASH=0x0
# CONFIG_BOOTLOADER_COMPILER_OPTIMIZATION_SIZE is not set
# CONFIG_BOOTLOADER_COMPILER_OPTIMIZATION_DEBUG is not set
CONFIG_BOOTLOADER_COMPILER_OPTIMIZATION_PERF=y
# CONFIG_BOOTLOADER_COMPILER_OPTIMIZATION_NONE is not set

#
# Log
#
CONFIG_BOOTLOADER_LOG_LEVEL_NONE=y
# CONFIG_BOOTLOADER_LOG_LEVEL_ERROR is not set
# CONFIG_BOOTLOADER_LOG_LEVEL_WARN is not set
# CONFIG_BOOTLOADER_LOG_LEVEL_INFO is not set
# CONFIG_BOOTLOADER_LOG_LEVEL_DEBUG is not set
# CONFIG_BOOTLOADER_LOG_LEVEL_VERBOSE is not set
CONFIG_BOOTLOADER_LOG_LEVEL=0

#
# Format
#
# CONFIG_BOOTLOADER_LOG_COLORS is not set
CONFIG_BOOTLOADER_LOG_TIMESTAMP_SOURCE_CPU_TICKS=y
# end of Format
# end of Log

#
# Serial Flash Configurations
#
# CONFIG_BOOTLOADER_FLASH_DC_AWARE is not set
CONFIG_BOOTLOADER_FLASH_XMC_SUPPORT=y
# end of Serial Flash Configurations

CONFIG_BOOTLOADER_VDDSDIO_BOOST_1_9V=y
# CONFIG_BOOTLOADER_FACTORY_RESET is not set
# CONFIG_BOOTLOADER_APP_TEST is not set
CONFIG_BOOTLOADER_REGION_PROTECTION_ENABLE=y
CONFIG_BOOTLOADER_WDT_ENABLE=y
# CONFIG_BOOTLOADER_WDT_DISABLE_IN_USER_CODE is not set
CONFIG_BOOTLOADER_WDT_TIME_MS=9000
CONFIG_BOOTLOADER_APP_ROLLBACK_ENABLE=y
# CONFIG_BOOTLOADER_APP_ANTI_ROLLBACK is not set
CONFIG_BOOTLOADER_SKIP_VALIDATE_IN_DEEP_SLEEP=y
CONFIG_BOOTLOADER_SKIP_VALIDATE_ON_POWER_ON=y
CONFIG_BOOTLOADER_SKIP_VALIDATE_ALWAYS=y
CONFIG_BOOTLOADER_RESERVE_RTC_SIZE=0x10
# CONFIG_BOOTLOADER_CUSTOM_RESERVE_RTC is not set
CONFIG_BOOTLOADER_RESERVE_RTC_MEM=y
# end of Bootloader config

#
# Security features
#
CONFIG_SECURE_BOOT_V2_RSA_SUPPORTED=y
CONFIG_SECURE_BOOT_V2_PREFERRED=y
# CONFIG_SECURE_SIGNED_APPS_NO_SECURE_BOOT is not set
# CONFIG_SECURE_BOOT is not set
# CONFIG_SECURE_FLASH_ENC_ENABLED is not set
CONFIG_SECURE_ROM_DL_MODE_ENABLED=y
# end of Security features

#
# Application manager
#
CONFIG_APP_COMPILE_TIME_DATE=y
# CONFIG_APP_EXCLUDE_PROJECT_VER_VAR is not set
# CONFIG_APP_EXCLUDE_PROJECT_NAME_VAR is not set
# CONFIG_APP_PROJECT_VER_FROM_CONFIG is not set
CONFIG_APP_RETRIEVE_LEN_ELF_SHA=9
# end of Application manager

CONFIG_ESP_ROM_HAS_CRC_LE=y
CONFIG_ESP_ROM_HAS_CRC_BE=y
CONFIG_ESP_ROM_HAS_MZ_CRC32=y
CONFIG_ESP_ROM_HAS_JPEG_DECODE=y
CONFIG_ESP_ROM_UART_CLK_IS_XTAL=y
CONFIG_ESP_ROM_HAS_RETARGETABLE_LOCKING=y
CONFIG_ESP_ROM_USB_OTG_NUM=3
CONFIG_ESP_ROM_USB_SERIAL_DEVICE_NUM=4
CONFIG_ESP_ROM_HAS_ERASE_0_REGION_BUG=y
CONFIG_ESP_ROM_HAS_ENCRYPTED_WRITES_USING_LEGACY_DRV=y
CONFIG_ESP_ROM_GET_CLK_FREQ=y
CONFIG_ESP_ROM_HAS_HAL_WDT=y
CONFIG_ESP_ROM_NEEDS_SWSETUP_WORKAROUND=y
CONFIG_ESP_ROM_HAS_LAYOUT_TABLE=y
CONFIG_ESP_ROM_HAS_SPI_FLASH=y
CONFIG_ESP_ROM_HAS_ETS_PRINTF_BUG=y
CONFIG_ESP_ROM_HAS_NEWLIB=y
CONFIG_ESP_ROM_HAS_NEWLIB_NANO_FORMAT=y
CONFIG_ESP_ROM_HAS_NEWLIB_32BIT_TIME=y
CONFIG_ESP_ROM_NEEDS_SET_CACHE_MMU_SIZE=y
CONFIG_ESP_ROM_RAM_APP_NEEDS_MMU_INIT=y
CONFIG_ESP_ROM_HAS_FLASH_COUNT_PAGES_BUG=y
CONFIG_ESP_ROM_HAS_CACHE_SUSPEND_WAITI_BUG=y
CONFIG_ESP_ROM_HAS_CACHE_WRITEBACK_BUG=y
CONFIG_ESP_ROM_HAS_SW_FLOAT=y
CONFIG_ESP_ROM_HAS_VERSION=y
CONFIG_ESP_ROM_SUPPORT_DEEP_SLEEP_WAKEUP_STUB=y
CONFIG_ESP_ROM_HAS_OUTPUT_PUTC_FUNC=y
CONFIG_ESP_ROM_CONSOLE_OUTPUT_SECONDARY=y

#
# Boot ROM Behavior
#
CONFIG_BOOT_ROM_LOG_ALWAYS_ON=y
# CONFIG_BOOT_ROM_LOG_ALWAYS_OFF is not set
# CONFIG_BOOT_ROM_LOG_ON_GPIO_HIGH is not set
# CONFIG_BOOT_ROM_LOG_ON_GPIO_LOW is not set
# end of Boot ROM Behavior

#
# Serial flasher config
#
# CONFIG_ESPTOOLPY_NO_STUB is not set
# CONFIG_ESPTOOLPY_OCT_FLASH is not set
CONFIG_ESPTOOLPY_FLASH_MODE_AUTO_DETECT=y
CONFIG_ESPTOOLPY_FLASHMODE_QIO=y
# CONFIG_ESPTOOLPY_FLASHMODE_QOUT is not set
# CONFIG_ESPTOOLPY_FLASHMODE_DIO is not set
# CONFIG_ESPTOOLPY_FLASHMODE_DOUT is not set
CONFIG_ESPTOOLPY_FLASH_SAMPLE_MODE_STR=y
CONFIG_ESPTOOLPY_FLASHMODE="dio"
# CONFIG_ESPTOOLPY_FLASHFREQ_120M is not set
CONFIG_ESPTOOLPY_FLASHFREQ_80M=y
# CONFIG_ESPTOOLPY_FLASHFREQ_40M is not set
# CONFIG_ESPTOOLPY_FLASHFREQ_20M is not set
CONFIG_ESPTOOLPY_FLASHFREQ="80m"
# CONFIG_ESPTOOLPY_FLASHSIZE_1MB is not set
# CONFIG_ESPTOOLPY_FLASHSIZE_2MB is not set
# CONFIG_ESPTOOLPY_FLASHSIZE_4MB is not set
# CONFIG_ESPTOOLPY_FLASHSIZE_8MB is not set
CONFIG_ESPTOOLPY_FLASHSIZE_16MB=y
# CONFIG_ESPTOOLPY_FLASHSIZE_32MB is not set
# CONFIG_ESPTOOLPY_FLASHSIZE_64MB is not set
# CONFIG_ESPTOOLPY_FLASHSIZE_128MB is not set
CONFIG_ESPTOOLPY_FLASHSIZE="16MB"
# CONFIG_ESPTOOLPY_HEADER_FLASHSIZE_UPDATE is not set
CONFIG_ESPTOOLPY_BEFORE_RESET=y
# CONFIG_ESPTOOLPY_BEFORE_NORESET is not set
CONFIG_ESPTOOLPY_BEFORE="default_reset"
CONFIG_ESPTOOLPY_AFTER_RESET=y
# CONFIG_ESPTOOLPY_AFTER_NORESET is not set
CONFIG_ESPTOOLPY_AFTER="hard_reset"
CONFIG_ESPTOOLPY_MONITOR_BAUD=115200
# end of Serial flasher config

#
# Partition Table
#
# CONFIG_PARTITION_TABLE_SINGLE_APP is not set
# CONFIG_PARTITION_TABLE_SINGLE_APP_LARGE is not set
# CONFIG_PARTITION_TABLE_TWO_OTA is not set
# CONFIG_PARTITION_TABLE_TWO_OTA_LARGE is not set
CONFIG_PARTITION_TABLE_CUSTOM=y
CONFIG_PARTITION_TABLE_CUSTOM_FILENAME="partitions/v2/16m.csv"
CONFIG_PARTITION_TABLE_FILENAME="partitions/v2/16m.csv"
CONFIG_PARTITION_TABLE_OFFSET=0x8000
CONFIG_PARTITION_TABLE_MD5=y
# end of Partition Table

#
# Xiaozhi Assistant
#
CONFIG_OTA_URL="https://api.tenclass.net/xiaozhi/ota/"
# CONFIG_FLASH_NONE_ASSETS is not set
CONFIG_FLASH_DEFAULT_ASSETS=y
# CONFIG_FLASH_CUSTOM_ASSETS is not set
CONFIG_LANGUAGE_ZH_CN=y
# CONFIG_LANGUAGE_ZH_TW is not set
# CONFIG_LANGUAGE_EN_US is not set
# CONFIG_LANGUAGE_JA_JP is not set
# CONFIG_LANGUAGE_KO_KR is not set
# CONFIG_LANGUAGE_VI_VN is not set
# CONFIG_LANGUAGE_TH_TH is not set
# CONFIG_LANGUAGE_DE_DE is not set
# CONFIG_LANGUAGE_FR_FR is not set
# CONFIG_LANGUAGE_ES_ES is not set
# CONFIG_LANGUAGE_IT_IT is not set
# CONFIG_LANGUAGE_RU_RU is not set
# CONFIG_LANGUAGE_AR_SA is not set
# CONFIG_LANGUAGE_HI_IN is not set
# CONFIG_LANGUAGE_PT_PT is not set
# CONFIG_LANGUAGE_PL_PL is not set
# CONFIG_LANGUAGE_CS_CZ is not set
# CONFIG_LANGUAGE_FI_FI is not set
# CONFIG_LANGUAGE_TR_TR is not set
# CONFIG_LANGUAGE_ID_ID is not set
# CONFIG_LANGUAGE_UK_UA is not set
# CONFIG_LANGUAGE_RO_RO is not set
# CONFIG_LANGUAGE_BG_BG is not set
# CONFIG_LANGUAGE_CA_ES is not set
# CONFIG_LANGUAGE_DA_DK is not set
# CONFIG_LANGUAGE_EL_GR is not set
# CONFIG_LANGUAGE_FA_IR is not set
# CONFIG_LANGUAGE_FIL_PH is not set
# CONFIG_LANGUAGE_HE_IL is not set
# CONFIG_LANGUAGE_HR_HR is not set
# CONFIG_LANGUAGE_HU_HU is not set
# CONFIG_LANGUAGE_MS_MY is not set
# CONFIG_LANGUAGE_NB_NO is not set
# CONFIG_LANGUAGE_NL_NL is not set
# CONFIG_LANGUAGE_SK_SK is not set
# CONFIG_LANGUAGE_SL_SI is not set
# CONFIG_LANGUAGE_SV_SE is not set
# CONFIG_LANGUAGE_SR_RS is not set
CONFIG_BOARD_TYPE_BREAD_COMPACT_WIFI=y
# CONFIG_BOARD_TYPE_BREAD_COMPACT_WIFI_LCD is not set
# CONFIG_BOARD_TYPE_BREAD_COMPACT_WIFI_CAM is not set
# CONFIG_BOARD_TYPE_BREAD_COMPACT_ML307 is not set
# CONFIG_BOARD_TYPE_ESP_KORVO2_V3 is not set
# CONFIG_BOARD_TYPE_ESP_SPARKBOT is not set
# CONFIG_BOARD_TYPE_ESP_SPOT_S3 is not set
# CONFIG_BOARD_TYPE_ESP_BOX_3 is not set
# CONFIG_BOARD_TYPE_ESP_BOX is not set
# CONFIG_BOARD_TYPE_ESP_BOX_LITE is not set
# CONFIG_BOARD_TYPE_ECHOEAR is not set
# CONFIG_BOARD_TYPE_KEVIN_BOX_2 is not set
# CONFIG_BOARD_TYPE_KEVIN_SP_V3_DEV is not set
# CONFIG_BOARD_TYPE_KEVIN_SP_V4_DEV is not set
# CONFIG_BOARD_TYPE_KEVIN_YUYING_313LCD is not set
# CONFIG_BOARD_TYPE_LICHUANG_DEV_S3 is not set
# CONFIG_BOARD_TYPE_DF_K10 is not set
# CONFIG_BOARD_TYPE_DF_S3_AI_CAM is not set
# CONFIG_BOARD_TYPE_MAGICLICK_S3_2P4 is not set
# CONFIG_BOARD_TYPE_MAGICLICK_S3_2P5 is not set
# CONFIG_BOARD_TYPE_M5STACK_CORE_S3 is not set
# CONFIG_BOARD_TYPE_M5STACK_ATOM_S3_ECHO_BASE is not set
# CONFIG_BOARD_TYPE_M5STACK_ATOM_S3R_ECHO_BASE is not set
# CONFIG_BOARD_TYPE_M5STACK_ATOM_S3R_CAM_M12_ECHO_BASE is not set
# CONFIG_BOARD_TYPE_M5STACK_ATOM_ECHOS3R is not set
# CONFIG_BOARD_TYPE_WAVESHARE_S3_AUDIO_BOARD is not set
# CONFIG_BOARD_TYPE_WAVESHARE_S3_TOUCH_AMOLED_1_8 is not set
# CONFIG_BOARD_TYPE_WAVESHARE_S3_TOUCH_AMOLED_2_06 is not set
# CONFIG_BOARD_TYPE_WAVESHARE_S3_TOUCH_AMOLED_1_75 is not set
# CONFIG_BOARD_TYPE_WAVESHARE_S3_TOUCH_LCD_1_83 is not set
# CONFIG_BOARD_TYPE_WAVESHARE_S3_TOUCH_LCD_4B is not set
# CONFIG_BOARD_TYPE_WAVESHARE_S3_TOUCH_LCD_1_85C is not set
# CONFIG_BOARD_TYPE_WAVESHARE_S3_TOUCH_LCD_1_85 is not set
# CONFIG_BOARD_TYPE_WAVESHARE_S3_TOUCH_LCD_1_46 is not set
# CONFIG_BOARD_TYPE_WAVESHARE_S3_TOUCH_AMOLED_1_32 is not set
# CONFIG_BOARD_TYPE_WAVESHARE_S3_TOUCH_LCD_3_49 is not set
# CONFIG_BOARD_TYPE_WAVESHARE_S3_TOUCH_LCD_3_5 is not set
# CONFIG_BOARD_TYPE_WAVESHARE_S3_ePaper_1_54 is not set
# CONFIG_BOARD_TYPE_WAVESHARE_S3_TOUCH_LCD_3_5B is not set
# CONFIG_BOARD_TYPE_TUDOUZI is not set
# CONFIG_BOARD_TYPE_LILYGO_T_CIRCLE_S3 is not set
# CONFIG_BOARD_TYPE_LILYGO_T_CAMERAPLUS_S3_V1_0_V1_1 is not set
# CONFIG_BOARD_TYPE_LILYGO_T_CAMERAPLUS_S3_V1_2 is not set
# CONFIG_BOARD_TYPE_LILYGO_T_DISPLAY_S3_PRO_MVSRLORA is not set
# CONFIG_BOARD_TYPE_LILYGO_T_DISPLAY_S3_PRO_MVSRLORA_NO_BATTERY is not set
# CONFIG_BOARD_TYPE_MOVECALL_MOJI_ESP32S3 is not set
# CONFIG_BOARD_TYPE_MOVECALL_CUICAN_ESP32S3 is not set
# CONFIG_BOARD_TYPE_ATK_DNESP32S3 is not set
# CONFIG_BOARD_TYPE_ATK_DNESP32S3_BOX is not set
# CONFIG_BOARD_TYPE_ATK_DNESP32S3_BOX0 is not set
# CONFIG_BOARD_TYPE_ATK_DNESP32S3_BOX2_WIFI is not set
# CONFIG_BOARD_TYPE_ATK_DNESP32S3_BOX2_4G is not set
# CONFIG_BOARD_TYPE_ATK_DNESP32S3M_WIFI is not set
# CONFIG_BOARD_TYPE_ATK_DNESP32S3M_4G is not set
# CONFIG_BOARD_TYPE_DU_CHATX is not set
# CONFIG_BOARD_TYPE_TAIJI_PI_S3 is not set
# CONFIG_BOARD_TYPE_XINGZHI_CUBE_0_85TFT_WIFI is not set
# CONFIG_BOARD_TYPE_XINGZHI_CUBE_0_85TFT_ML307 is not set
# CONFIG_BOARD_TYPE_XINGZHI_CUBE_0_96OLED_WIFI is not set
# CONFIG_BOARD_TYPE_XINGZHI_CUBE_0_96OLED_ML307 is not set
# CONFIG_BOARD_TYPE_XINGZHI_CUBE_1_54TFT_WIFI is not set
# CONFIG_BOARD_TYPE_XINGZHI_CUBE_1_54TFT_ML307 is not set
# CONFIG_BOARD_TYPE_XINGZHI_METAL_1_54_WIFI is not set
# CONFIG_BOARD_TYPE_SEEED_STUDIO_SENSECAP_WATCHER is not set
# CONFIG_BOARD_TYPE_DOIT_S3_AIBOX is not set
# CONFIG_BOARD_TYPE_MIXGO_NOVA is not set
# CONFIG_BOARD_TYPE_GENJUTECH_S3_1_54TFT is not set
# CONFIG_BOARD_TYPE_ESP_S3_LCD_EV_Board is not set
# CONFIG_BOARD_TYPE_ESP_S3_LCD_EV_Board_2 is not set
# CONFIG_BOARD_TYPE_ZHENGCHEN_1_54TFT_WIFI is not set
# CONFIG_BOARD_TYPE_ZHENGCHEN_1_54TFT_ML307 is not set
# CONFIG_BOARD_TYPE_MINSI_K08_DUAL is not set
# CONFIG_BOARD_TYPE_SPOTPEAR_ESP32_S3_1_54_MUMA is not set
# CONFIG_BOARD_TYPE_SPOTPEAR_ESP32_S3_1_28_BOX is not set
# CONFIG_BOARD_TYPE_OTTO_ROBOT is not set
# CONFIG_BOARD_TYPE_ELECTRON_BOT is not set
# CONFIG_BOARD_TYPE_JIUCHUAN is not set
# CONFIG_BOARD_TYPE_LABPLUS_MPYTHON_V3 is not set
# CONFIG_BOARD_TYPE_LABPLUS_LEDONG_V2 is not set
# CONFIG_BOARD_TYPE_YUNLIAO_S3 is not set
# CONFIG_BOARD_TYPE_AIPI_LITE is not set
# CONFIG_BOARD_TYPE_HU_087 is not set
CONFIG_OLED_SSD1306_128X32=y
# CONFIG_OLED_SSD1306_128X64 is not set
# CONFIG_OLED_SH1106_128X64 is not set
CONFIG_USE_DEFAULT_MESSAGE_STYLE=y
# CONFIG_USE_WECHAT_MESSAGE_STYLE is not set
# CONFIG_WAKE_WORD_DISABLED is not set
# CONFIG_USE_AFE_WAKE_WORD is not set
CONFIG_USE_CUSTOM_WAKE_WORD=y
CONFIG_CUSTOM_WAKE_WORD="ni hao xiao yuan"
CONFIG_CUSTOM_WAKE_WORD_DISPLAY="你好小圆"
CONFIG_CUSTOM_WAKE_WORD_THRESHOLD=20
CONFIG_SEND_WAKE_WORD_DATA=y
CONFIG_USE_AUDIO_PROCESSOR=y
CONFIG_USE_SERVER_AEC=y
# CONFIG_USE_AUDIO_DEBUGGER is not set

#
# WiFi Configuration Method
#
CONFIG_USE_HOTSPOT_WIFI_PROVISIONING=y
# CONFIG_USE_ACOUSTIC_WIFI_PROVISIONING is not set
# CONFIG_USE_ESP_BLUFI_WIFI_PROVISIONING is not set
# end of WiFi Configuration Method

# CONFIG_RECEIVE_CUSTOM_MESSAGE is not set

#
# Camera Configuration
#

#
# Warning: Please read the help text before modifying these settings.
#
# CONFIG_XIAOZHI_CAMERA_ALLOW_JPEG_INPUT is not set
# CONFIG_XIAOZHI_ENABLE_CAMERA_DEBUG_MODE is not set
# CONFIG_XIAOZHI_ENABLE_CAMERA_ENDIANNESS_SWAP is not set
# CONFIG_XIAOZHI_ENABLE_ROTATE_CAMERA_IMAGE is not set
# end of Camera Configuration
# end of Xiaozhi Assistant

#
# Expression Emote Configuration
#
CONFIG_EMOTE_DEF_SCROLL_SPEED=10
CONFIG_EMOTE_DEF_LABEL_HEIGHT=30
CONFIG_EMOTE_DEF_LABEL_WIDTH=100
CONFIG_EMOTE_DEF_LABEL_Y_OFFSET=20
CONFIG_EMOTE_DEF_ANIMATION_FPS=20
CONFIG_EMOTE_DEF_FONT_COLOR=0xFFFFFF
CONFIG_EMOTE_DEF_BG_COLOR=0x000000
CONFIG_EMOTE_ASSETS_HASH_TABLE_SIZE=32
# end of Expression Emote Configuration

#
# ESP Speech Recognition
#
CONFIG_MODEL_IN_FLASH=y
# CONFIG_MODEL_IN_SDCARD is not set
CONFIG_AFE_INTERFACE_V1=y
CONFIG_SR_NSN_WEBRTC=y
# CONFIG_SR_NSN_NSNET2 is not set
CONFIG_SR_VADN_WEBRTC=y
# CONFIG_SR_VADN_VADNET1_MEDIUM is not set

#
# Load Multiple Wake Words (WakeNet9s)
#
# CONFIG_SR_WN_WN9S_HILEXIN is not set
# CONFIG_SR_WN_WN9S_HIESP is not set
# CONFIG_SR_WN_WN9S_NIHAOXIAOZHI is not set
# CONFIG_SR_WN_WN9S_HIJASON is not set
# end of Load Multiple Wake Words (WakeNet9s)

#
# Load Multiple Wake Words (WakeNet9)
#
# CONFIG_SR_WN_WN9_HILEXIN is not set
# CONFIG_SR_WN_WN9_NIHAOMIAOBAN_TTS2 is not set
# CONFIG_SR_WN_WN9_XIAOAITONGXUE is not set
# CONFIG_SR_WN_WN9L_XIAOAITONGXUE is not set
CONFIG_SR_WN_WN9_NIHAOXIAOZHI_TTS=y
# CONFIG_SR_WN_WN9L_NIHAOXIAOZHI_TTS3 is not set
# CONFIG_SR_WN_WN9_ALEXA is not set
# CONFIG_SR_WN_WN9_HIESP is not set
# CONFIG_SR_WN_WN9_JARVIS_TTS is not set
# CONFIG_SR_WN_WN9_COMPUTER_TTS is not set
# CONFIG_SR_WN_WN9_HEYWILLOW_TTS is not set
# CONFIG_SR_WN_WN9_HIMFIVE is not set
# CONFIG_SR_WN_WN9_SOPHIA_TTS is not set
# CONFIG_SR_WN_WN9_HEYWANDA_TTS is not set
# CONFIG_SR_WN_WN9_HIJOLLY_TTS2 is not set
# CONFIG_SR_WN_WN9_HIFAIRY_TTS2 is not set
# CONFIG_SR_WN_WN9_HEYPRINTER_TTS is not set
# CONFIG_SR_WN_WN9_MYCROFT_TTS is not set
# CONFIG_SR_WN_WN9_HIJOY_TTS is not set
# CONFIG_SR_WN_WN9_HIJASON_TTS2 is not set
# CONFIG_SR_WN_WN9_ASTROLABE_TTS is not set
# CONFIG_SR_WN_WN9_HEYILY_TTS2 is not set
# CONFIG_SR_WN_WN9_BLUECHIP_TTS2 is not set
# CONFIG_SR_WN_WN9_HIANDY_TTS2 is not set
# CONFIG_SR_WN_WN9_HEYIVY_TTS2 is not set
# CONFIG_SR_WN_WN9_HISTACKCHAN_TTS3 is not set
# CONFIG_SR_WN_WN9_XIAOLONGXIAOLONG_TTS is not set
# CONFIG_SR_WN_WN9_HIMIAOMIAO_TTS is not set
# CONFIG_SR_WN_WN9_MIAOMIAOTONGXUE_TTS is not set
# CONFIG_SR_WN_WN9_NIHAOXIAOXIN_TTS is not set
# CONFIG_SR_WN_WN9_XIAOMEITONGXUE_TTS is not set
# CONFIG_SR_WN_WN9_HILILI_TTS is not set
# CONFIG_SR_WN_WN9_HITELLY_TTS is not set
# CONFIG_SR_WN_WN9_XIAOBINXIAOBIN_TTS is not set
# CONFIG_SR_WN_WN9_HAIXIAOWU_TTS is not set
# CONFIG_SR_WN_WN9_XIAOYAXIAOYA_TTS2 is not set
# CONFIG_SR_WN_WN9_LINAIBAN_TTS2 is not set
# CONFIG_SR_WN_WN9_XIAOSUROU_TTS2 is not set
# CONFIG_SR_WN_WN9_XIAOYUTONGXUE_TTS2 is not set
# CONFIG_SR_WN_WN9_XIAOMINGTONGXUE_TTS2 is not set
# CONFIG_SR_WN_WN9_XIAOKANGTONGXUE_TTS2 is not set
# CONFIG_SR_WN_WN9_XIAOJIANXIAOJIAN_TTS2 is not set
# CONFIG_SR_WN_WN9_XIAOTEXIAOTE_TTS2 is not set
# CONFIG_SR_WN_WN9_NIHAOXIAOYI_TTS2 is not set
# CONFIG_SR_WN_WN9_NIHAOBAIYING_TTS2 is not set
# CONFIG_SR_WN_WN9_NIHAODONGDONG_TTS2 is not set
# CONFIG_SR_WN_WN9_HIWALLE_TTS2 is not set
# CONFIG_SR_WN_WN9_XIAOLUXIAOLU_TTS2 is not set
# CONFIG_SR_WN_WN9_NIHAOXIAOAN_TTS2 is not set
# CONFIG_SR_WN_WN9_NI3HAO3XIAO3MAI4_TTS2 is not set
# CONFIG_SR_WN_WN9_NI3HAO3XIAO3RUI4_TTS3 is not set
# CONFIG_SR_WN_WN9_HAI1XIAO3OU1_TTS3 is not set
# CONFIG_SR_WN_WN9_XIAO3JIA1XIAO3JIA1_TTS3 is not set
# CONFIG_SR_WN_WN9_XIAO3FENG1XIAO3FENG1_TTS3 is not set
# end of Load Multiple Wake Words (WakeNet9)

# CONFIG_SR_MN_CN_NONE is not set
# CONFIG_SR_MN_CN_MULTINET5_RECOGNITION_QUANT8 is not set
# CONFIG_SR_MN_CN_MULTINET6_QUANT is not set
# CONFIG_SR_MN_CN_MULTINET6_AC_QUANT is not set
CONFIG_SR_MN_CN_MULTINET7_QUANT=y
# CONFIG_SR_MN_CN_MULTINET7_AC_QUANT is not set
CONFIG_SR_MN_EN_NONE=y
# CONFIG_SR_MN_EN_MULTINET5_SINGLE_RECOGNITION_QUANT8 is not set
# CONFIG_SR_MN_EN_MULTINET6_QUANT is not set
# CONFIG_SR_MN_EN_MULTINET7_QUANT is not set
# end of ESP Speech Recognition

#
# Compiler options
#
# CONFIG_COMPILER_OPTIMIZATION_DEBUG is not set
CONFIG_COMPILER_OPTIMIZATION_SIZE=y
# CONFIG_COMPILER_OPTIMIZATION_PERF is not set
# CONFIG_COMPILER_OPTIMIZATION_NONE is not set
CONFIG_COMPILER_OPTIMIZATION_ASSERTIONS_ENABLE=y
# CONFIG_COMPILER_OPTIMIZATION_ASSERTIONS_SILENT is not set
# CONFIG_COMPILER_OPTIMIZATION_ASSERTIONS_DISABLE is not set
CONFIG_COMPILER_ASSERT_NDEBUG_EVALUATE=y
CONFIG_COMPILER_FLOAT_LIB_FROM_GCCLIB=y
CONFIG_COMPILER_OPTIMIZATION_ASSERTION_LEVEL=2
# CONFIG_COMPILER_OPTIMIZATION_CHECKS_SILENT is not set
CONFIG_COMPILER_HIDE_PATHS_MACROS=y
CONFIG_COMPILER_CXX_EXCEPTIONS=y
CONFIG_COMPILER_CXX_EXCEPTIONS_EMG_POOL_SIZE=1024
CONFIG_COMPILER_CXX_RTTI=y
CONFIG_COMPILER_STACK_CHECK_MODE_NONE=y
# CONFIG_COMPILER_STACK_CHECK_MODE_NORM is not set
# CONFIG_COMPILER_STACK_CHECK_MODE_STRONG is not set
# CONFIG_COMPILER_STACK_CHECK_MODE_ALL is not set
# CONFIG_COMPILER_NO_MERGE_CONSTANTS is not set
# CONFIG_COMPILER_WARN_WRITE_STRINGS is not set
CONFIG_COMPILER_DISABLE_DEFAULT_ERRORS=y
# CONFIG_COMPILER_DISABLE_GCC12_WARNINGS is not set
# CONFIG_COMPILER_DISABLE_GCC13_WARNINGS is not set
# CONFIG_COMPILER_DISABLE_GCC14_WARNINGS is not set
# CONFIG_COMPILER_DUMP_RTL_FILES is not set
CONFIG_COMPILER_RT_LIB_GCCLIB=y
CONFIG_COMPILER_RT_LIB_NAME="gcc"
CONFIG_COMPILER_ORPHAN_SECTIONS_WARNING=y
# CONFIG_COMPILER_ORPHAN_SECTIONS_PLACE is not set
# CONFIG_COMPILER_STATIC_ANALYZER is not set
# end of Compiler options

#
# Component config
#

#
# Application Level Tracing
#
# CONFIG_APPTRACE_DEST_JTAG is not set
CONFIG_APPTRACE_DEST_NONE=y
# CONFIG_APPTRACE_DEST_UART1 is not set
# CONFIG_APPTRACE_DEST_UART2 is not set
# CONFIG_APPTRACE_DEST_USB_CDC is not set
CONFIG_APPTRACE_DEST_UART_NONE=y
CONFIG_APPTRACE_UART_TASK_PRIO=1
CONFIG_APPTRACE_LOCK_ENABLE=y
# end of Application Level Tracing

#
# Bluetooth
#
# CONFIG_BT_ENABLED is not set

#
# Common Options
#

#
# BLE Log
#
# CONFIG_BLE_LOG_ENABLED is not set
# end of BLE Log

# CONFIG_BT_BLE_LOG_SPI_OUT_ENABLED is not set
# CONFIG_BT_BLE_LOG_UHCI_OUT_ENABLED is not set
# end of Common Options
# end of Bluetooth

#
# Console Library
#
# CONFIG_CONSOLE_SORTED_HELP is not set
# end of Console Library

#
# Driver Configurations
#

#
# TWAI Configuration
#
# CONFIG_TWAI_ISR_IN_IRAM is not set
CONFIG_TWAI_ERRATA_FIX_LISTEN_ONLY_DOM=y
# end of TWAI Configuration

#
# Legacy ADC Driver Configuration
#
# CONFIG_ADC_SUPPRESS_DEPRECATE_WARN is not set
# CONFIG_ADC_SKIP_LEGACY_CONFLICT_CHECK is not set

#
# Legacy ADC Calibration Configuration
#
# CONFIG_ADC_CALI_SUPPRESS_DEPRECATE_WARN is not set
# end of Legacy ADC Calibration Configuration
# end of Legacy ADC Driver Configuration

#
# Legacy MCPWM Driver Configurations
#
# CONFIG_MCPWM_SUPPRESS_DEPRECATE_WARN is not set
# CONFIG_MCPWM_SKIP_LEGACY_CONFLICT_CHECK is not set
# end of Legacy MCPWM Driver Configurations

#
# Legacy Timer Group Driver Configurations
#
# CONFIG_GPTIMER_SUPPRESS_DEPRECATE_WARN is not set
# CONFIG_GPTIMER_SKIP_LEGACY_CONFLICT_CHECK is not set
# end of Legacy Timer Group Driver Configurations

#
# Legacy RMT Driver Configurations
#
# CONFIG_RMT_SUPPRESS_DEPRECATE_WARN is not set
# CONFIG_RMT_SKIP_LEGACY_CONFLICT_CHECK is not set
# end of Legacy RMT Driver Configurations

#
# Legacy I2S Driver Configurations
#
# CONFIG_I2S_SUPPRESS_DEPRECATE_WARN is not set
# CONFIG_I2S_SKIP_LEGACY_CONFLICT_CHECK is not set
# end of Legacy I2S Driver Configurations

#
# Legacy I2C Driver Configurations
#
# CONFIG_I2C_SKIP_LEGACY_CONFLICT_CHECK is not set
# end of Legacy I2C Driver Configurations

#
# Legacy PCNT Driver Configurations
#
# CONFIG_PCNT_SUPPRESS_DEPRECATE_WARN is not set
# CONFIG_PCNT_SKIP_LEGACY_CONFLICT_CHECK is not set
# end of Legacy PCNT Driver Configurations

#
# Legacy SDM Driver Configurations
#
# CONFIG_SDM_SUPPRESS_DEPRECATE_WARN is not set
# CONFIG_SDM_SKIP_LEGACY_CONFLICT_CHECK is not set
# end of Legacy SDM Driver Configurations

#
# Legacy Temperature Sensor Driver Configurations
#
# CONFIG_TEMP_SENSOR_SUPPRESS_DEPRECATE_WARN is not set
# CONFIG_TEMP_SENSOR_SKIP_LEGACY_CONFLICT_CHECK is not set
# end of Legacy Temperature Sensor Driver Configurations
# end of Driver Configurations

#
# eFuse Bit Manager
#
# CONFIG_EFUSE_CUSTOM_TABLE is not set
# CONFIG_EFUSE_VIRTUAL is not set
CONFIG_EFUSE_MAX_BLK_LEN=256
# end of eFuse Bit Manager

#
# ESP-TLS
#
CONFIG_ESP_TLS_USING_MBEDTLS=y
CONFIG_ESP_TLS_USE_DS_PERIPHERAL=y
# CONFIG_ESP_TLS_CLIENT_SESSION_TICKETS is not set
# CONFIG_ESP_TLS_SERVER_SESSION_TICKETS is not set
# CONFIG_ESP_TLS_SERVER_CERT_SELECT_HOOK is not set
# CONFIG_ESP_TLS_SERVER_MIN_AUTH_MODE_OPTIONAL is not set
# CONFIG_ESP_TLS_PSK_VERIFICATION is not set
# CONFIG_ESP_TLS_INSECURE is not set
CONFIG_ESP_TLS_DYN_BUF_STRATEGY_SUPPORTED=y
# end of ESP-TLS

#
# ADC and ADC Calibration
#
# CONFIG_ADC_ONESHOT_CTRL_FUNC_IN_IRAM is not set
# CONFIG_ADC_CONTINUOUS_ISR_IRAM_SAFE is not set
# CONFIG_ADC_CONTINUOUS_FORCE_USE_ADC2_ON_C3_S3 is not set
# CONFIG_ADC_ENABLE_DEBUG_LOG is not set
# end of ADC and ADC Calibration

#
# Wireless Coexistence
#
CONFIG_ESP_COEX_ENABLED=y
# CONFIG_ESP_COEX_EXTERNAL_COEXIST_ENABLE is not set
# CONFIG_ESP_COEX_GPIO_DEBUG is not set
# end of Wireless Coexistence

#
# Common ESP-related
#
CONFIG_ESP_ERR_TO_NAME_LOOKUP=y
# end of Common ESP-related

#
# ESP-Driver:Camera Controller Configurations
#
# CONFIG_CAM_CTLR_DVP_CAM_ISR_CACHE_SAFE is not set
# end of ESP-Driver:Camera Controller Configurations

#
# ESP-Driver:GPIO Configurations
#
# CONFIG_GPIO_CTRL_FUNC_IN_IRAM is not set
# end of ESP-Driver:GPIO Configurations

#
# ESP-Driver:GPTimer Configurations
#
CONFIG_GPTIMER_ISR_HANDLER_IN_IRAM=y
# CONFIG_GPTIMER_CTRL_FUNC_IN_IRAM is not set
# CONFIG_GPTIMER_ISR_IRAM_SAFE is not set
CONFIG_GPTIMER_OBJ_CACHE_SAFE=y
# CONFIG_GPTIMER_ENABLE_DEBUG_LOG is not set
# end of ESP-Driver:GPTimer Configurations

#
# ESP-Driver:I2C Configurations
#
# CONFIG_I2C_ISR_IRAM_SAFE is not set
# CONFIG_I2C_ENABLE_DEBUG_LOG is not set
# CONFIG_I2C_ENABLE_SLAVE_DRIVER_VERSION_2 is not set
# end of ESP-Driver:I2C Configurations

#
# ESP-Driver:I2S Configurations
#
# CONFIG_I2S_ISR_IRAM_SAFE is not set
# CONFIG_I2S_ENABLE_DEBUG_LOG is not set
# end of ESP-Driver:I2S Configurations

#
# ESP-Driver:LEDC Configurations
#
# CONFIG_LEDC_CTRL_FUNC_IN_IRAM is not set
# end of ESP-Driver:LEDC Configurations

#
# ESP-Driver:MCPWM Configurations
#
# CONFIG_MCPWM_ISR_IRAM_SAFE is not set
# CONFIG_MCPWM_CTRL_FUNC_IN_IRAM is not set
# CONFIG_MCPWM_ENABLE_DEBUG_LOG is not set
# end of ESP-Driver:MCPWM Configurations

#
# ESP-Driver:PCNT Configurations
#
# CONFIG_PCNT_CTRL_FUNC_IN_IRAM is not set
# CONFIG_PCNT_ISR_IRAM_SAFE is not set
# CONFIG_PCNT_ENABLE_DEBUG_LOG is not set
# end of ESP-Driver:PCNT Configurations

#
# ESP-Driver:RMT Configurations
#
# CONFIG_RMT_ISR_IRAM_SAFE is not set
# CONFIG_RMT_RECV_FUNC_IN_IRAM is not set
# CONFIG_RMT_ENABLE_DEBUG_LOG is not set
# end of ESP-Driver:RMT Configurations

#
# ESP-Driver:Sigma Delta Modulator Configurations
#
# CONFIG_SDM_CTRL_FUNC_IN_IRAM is not set
# CONFIG_SDM_ENABLE_DEBUG_LOG is not set
# end of ESP-Driver:Sigma Delta Modulator Configurations

#
# ESP-Driver:SPI Configurations
#
# CONFIG_SPI_MASTER_IN_IRAM is not set
CONFIG_SPI_MASTER_ISR_IN_IRAM=y
# CONFIG_SPI_SLAVE_IN_IRAM is not set
CONFIG_SPI_SLAVE_ISR_IN_IRAM=y
# end of ESP-Driver:SPI Configurations

#
# ESP-Driver:Touch Sensor Configurations
#
# CONFIG_TOUCH_CTRL_FUNC_IN_IRAM is not set
# CONFIG_TOUCH_ISR_IRAM_SAFE is not set
# CONFIG_TOUCH_ENABLE_DEBUG_LOG is not set
# end of ESP-Driver:Touch Sensor Configurations

#
# ESP-Driver:Temperature Sensor Configurations
#
# CONFIG_TEMP_SENSOR_ENABLE_DEBUG_LOG is not set
# end of ESP-Driver:Temperature Sensor Configurations

#
# ESP-Driver:UART Configurations
#
CONFIG_UART_ISR_IN_IRAM=y
# end of ESP-Driver:UART Configurations

#
# ESP-Driver:UHCI Configurations
#
# CONFIG_UHCI_ISR_HANDLER_IN_IRAM is not set
# CONFIG_UHCI_ISR_CACHE_SAFE is not set
# CONFIG_UHCI_ENABLE_DEBUG_LOG is not set
# end of ESP-Driver:UHCI Configurations

#
# ESP-Driver:USB Serial/JTAG Configuration
#
CONFIG_USJ_ENABLE_USB_SERIAL_JTAG=y
# end of ESP-Driver:USB Serial/JTAG Configuration

#
# Ethernet
#
CONFIG_ETH_ENABLED=y
CONFIG_ETH_USE_SPI_ETHERNET=y
# CONFIG_ETH_SPI_ETHERNET_DM9051 is not set
# CONFIG_ETH_SPI_ETHERNET_W5500 is not set
# CONFIG_ETH_SPI_ETHERNET_KSZ8851SNL is not set
# CONFIG_ETH_USE_OPENETH is not set
# CONFIG_ETH_TRANSMIT_MUTEX is not set
# end of Ethernet

#
# Event Loop Library
#
# CONFIG_ESP_EVENT_LOOP_PROFILING is not set
CONFIG_ESP_EVENT_POST_FROM_ISR=y
CONFIG_ESP_EVENT_POST_FROM_IRAM_ISR=y
# end of Event Loop Library

#
# GDB Stub
#
CONFIG_ESP_GDBSTUB_ENABLED=y
# CONFIG_ESP_SYSTEM_GDBSTUB_RUNTIME is not set
CONFIG_ESP_GDBSTUB_SUPPORT_TASKS=y
CONFIG_ESP_GDBSTUB_MAX_TASKS=32
# end of GDB Stub

#
# ESP HID
#
CONFIG_ESPHID_TASK_SIZE_BT=2048
CONFIG_ESPHID_TASK_SIZE_BLE=4096
# end of ESP HID

#
# ESP HTTP client
#
CONFIG_ESP_HTTP_CLIENT_ENABLE_HTTPS=y
# CONFIG_ESP_HTTP_CLIENT_ENABLE_BASIC_AUTH is not set
# CONFIG_ESP_HTTP_CLIENT_ENABLE_DIGEST_AUTH is not set
# CONFIG_ESP_HTTP_CLIENT_ENABLE_CUSTOM_TRANSPORT is not set
CONFIG_ESP_HTTP_CLIENT_EVENT_POST_TIMEOUT=2000
# end of ESP HTTP client

#
# HTTP Server
#
CONFIG_HTTPD_MAX_REQ_HDR_LEN=2048
CONFIG_HTTPD_MAX_URI_LEN=2048
CONFIG_HTTPD_ERR_RESP_NO_DELAY=y
CONFIG_HTTPD_PURGE_BUF_LEN=32
# CONFIG_HTTPD_LOG_PURGE_DATA is not set
# CONFIG_HTTPD_WS_SUPPORT is not set
# CONFIG_HTTPD_QUEUE_WORK_BLOCKING is not set
CONFIG_HTTPD_SERVER_EVENT_POST_TIMEOUT=2000
# end of HTTP Server

#
# ESP HTTPS OTA
#
# CONFIG_ESP_HTTPS_OTA_DECRYPT_CB is not set
# CONFIG_ESP_HTTPS_OTA_ALLOW_HTTP is not set
CONFIG_ESP_HTTPS_OTA_EVENT_POST_TIMEOUT=2000
# end of ESP HTTPS OTA

#
# ESP HTTPS server
#
# CONFIG_ESP_HTTPS_SERVER_ENABLE is not set
CONFIG_ESP_HTTPS_SERVER_EVENT_POST_TIMEOUT=2000
# end of ESP HTTPS server

#
# Hardware Settings
#

#
# Chip revision
#
CONFIG_ESP32S3_REV_MIN_0=y
# CONFIG_ESP32S3_REV_MIN_1 is not set
# CONFIG_ESP32S3_REV_MIN_2 is not set
CONFIG_ESP32S3_REV_MIN_FULL=0
CONFIG_ESP_REV_MIN_FULL=0

#
# Maximum Supported ESP32-S3 Revision (Rev v0.99)
#
CONFIG_ESP32S3_REV_MAX_FULL=99
CONFIG_ESP_REV_MAX_FULL=99
CONFIG_ESP_EFUSE_BLOCK_REV_MIN_FULL=0
CONFIG_ESP_EFUSE_BLOCK_REV_MAX_FULL=199

#
# Maximum Supported ESP32-S3 eFuse Block Revision (eFuse Block Rev v1.99)
#
# end of Chip revision

#
# MAC Config
#
CONFIG_ESP_MAC_ADDR_UNIVERSE_WIFI_STA=y
CONFIG_ESP_MAC_ADDR_UNIVERSE_WIFI_AP=y
CONFIG_ESP_MAC_ADDR_UNIVERSE_BT=y
CONFIG_ESP_MAC_ADDR_UNIVERSE_ETH=y
CONFIG_ESP_MAC_UNIVERSAL_MAC_ADDRESSES_FOUR=y
CONFIG_ESP_MAC_UNIVERSAL_MAC_ADDRESSES=4
# CONFIG_ESP32S3_UNIVERSAL_MAC_ADDRESSES_TWO is not set
CONFIG_ESP32S3_UNIVERSAL_MAC_ADDRESSES_FOUR=y
CONFIG_ESP32S3_UNIVERSAL_MAC_ADDRESSES=4
# CONFIG_ESP_MAC_USE_CUSTOM_MAC_AS_BASE_MAC is not set
# end of MAC Config

#
# Sleep Config
#
CONFIG_ESP_SLEEP_FLASH_LEAKAGE_WORKAROUND=y
CONFIG_ESP_SLEEP_PSRAM_LEAKAGE_WORKAROUND=y
CONFIG_ESP_SLEEP_MSPI_NEED_ALL_IO_PU=y
CONFIG_ESP_SLEEP_RTC_BUS_ISO_WORKAROUND=y
CONFIG_ESP_SLEEP_GPIO_RESET_WORKAROUND=y
CONFIG_ESP_SLEEP_WAIT_FLASH_READY_EXTRA_DELAY=2000
# CONFIG_ESP_SLEEP_CACHE_SAFE_ASSERTION is not set
# CONFIG_ESP_SLEEP_DEBUG is not set
CONFIG_ESP_SLEEP_GPIO_ENABLE_INTERNAL_RESISTORS=y
# end of Sleep Config

#
# RTC Clock Config
#
CONFIG_RTC_CLK_SRC_INT_RC=y
# CONFIG_RTC_CLK_SRC_EXT_CRYS is not set
# CONFIG_RTC_CLK_SRC_EXT_OSC is not set
# CONFIG_RTC_CLK_SRC_INT_8MD256 is not set
CONFIG_RTC_CLK_CAL_CYCLES=1024
# end of RTC Clock Config

#
# Peripheral Control
#
# CONFIG_PERIPH_CTRL_FUNC_IN_IRAM is not set
# end of Peripheral Control

#
# GDMA Configurations
#
CONFIG_GDMA_CTRL_FUNC_IN_IRAM=y
CONFIG_GDMA_ISR_HANDLER_IN_IRAM=y
CONFIG_GDMA_OBJ_DRAM_SAFE=y
# CONFIG_GDMA_ENABLE_DEBUG_LOG is not set
# CONFIG_GDMA_ISR_IRAM_SAFE is not set
# end of GDMA Configurations

#
# Main XTAL Config
#
CONFIG_XTAL_FREQ_40=y
CONFIG_XTAL_FREQ=40
# end of Main XTAL Config

CONFIG_ESP_SPI_BUS_LOCK_ISR_FUNCS_IN_IRAM=y
# end of Hardware Settings

#
# ESP-Driver:LCD Controller Configurations
#
# CONFIG_LCD_ENABLE_DEBUG_LOG is not set
# CONFIG_LCD_RGB_ISR_IRAM_SAFE is not set
# CONFIG_LCD_RGB_RESTART_IN_VSYNC is not set
# end of ESP-Driver:LCD Controller Configurations

#
# ESP-MM: Memory Management Configurations
#
# CONFIG_ESP_MM_CACHE_MSYNC_C2M_CHUNKED_OPS is not set
# end of ESP-MM: Memory Management Configurations

#
# ESP NETIF Adapter
#
CONFIG_ESP_NETIF_IP_LOST_TIMER_INTERVAL=120
# CONFIG_ESP_NETIF_PROVIDE_CUSTOM_IMPLEMENTATION is not set
CONFIG_ESP_NETIF_TCPIP_LWIP=y
# CONFIG_ESP_NETIF_LOOPBACK is not set
CONFIG_ESP_NETIF_USES_TCPIP_WITH_BSD_API=y
CONFIG_ESP_NETIF_REPORT_DATA_TRAFFIC=y
# CONFIG_ESP_NETIF_RECEIVE_REPORT_ERRORS is not set
# CONFIG_ESP_NETIF_L2_TAP is not set
# CONFIG_ESP_NETIF_BRIDGE_EN is not set
# CONFIG_ESP_NETIF_SET_DNS_PER_DEFAULT_NETIF is not set
# end of ESP NETIF Adapter

#
# Partition API Configuration
#
# end of Partition API Configuration

#
# PHY
#
CONFIG_ESP_PHY_ENABLED=y
CONFIG_ESP_PHY_CALIBRATION_AND_DATA_STORAGE=y
# CONFIG_ESP_PHY_INIT_DATA_IN_PARTITION is not set
CONFIG_ESP_PHY_MAX_WIFI_TX_POWER=20
CONFIG_ESP_PHY_MAX_TX_POWER=20
# CONFIG_ESP_PHY_REDUCE_TX_POWER is not set
CONFIG_ESP_PHY_ENABLE_USB=y
# CONFIG_ESP_PHY_ENABLE_CERT_TEST is not set
CONFIG_ESP_PHY_RF_CAL_PARTIAL=y
# CONFIG_ESP_PHY_RF_CAL_NONE is not set
# CONFIG_ESP_PHY_RF_CAL_FULL is not set
CONFIG_ESP_PHY_CALIBRATION_MODE=0
CONFIG_ESP_PHY_PLL_TRACK_PERIOD_MS=1000
# CONFIG_ESP_PHY_PLL_TRACK_DEBUG is not set
# CONFIG_ESP_PHY_RECORD_USED_TIME is not set
CONFIG_ESP_PHY_IRAM_OPT=y
# CONFIG_ESP_PHY_DEBUG is not set
# end of PHY

#
# Power Management
#
# CONFIG_PM_ENABLE is not set
# CONFIG_PM_SLP_IRAM_OPT is not set
CONFIG_PM_POWER_DOWN_CPU_IN_LIGHT_SLEEP=y
CONFIG_PM_RESTORE_CACHE_TAGMEM_AFTER_LIGHT_SLEEP=y
# end of Power Management

#
# ESP PSRAM
#
CONFIG_SPIRAM=y

#
# SPI RAM config
#
# CONFIG_SPIRAM_MODE_QUAD is not set
CONFIG_SPIRAM_MODE_OCT=y
CONFIG_SPIRAM_TYPE_AUTO=y
# CONFIG_SPIRAM_TYPE_ESPPSRAM64 is not set
CONFIG_SPIRAM_CLK_IO=30
CONFIG_SPIRAM_CS_IO=26
# CONFIG_SPIRAM_XIP_FROM_PSRAM is not set
# CONFIG_SPIRAM_FETCH_INSTRUCTIONS is not set
# CONFIG_SPIRAM_RODATA is not set
CONFIG_SPIRAM_SPEED_80M=y
# CONFIG_SPIRAM_SPEED_40M is not set
CONFIG_SPIRAM_SPEED=80
# CONFIG_SPIRAM_ECC_ENABLE is not set
CONFIG_SPIRAM_BOOT_INIT=y
CONFIG_SPIRAM_PRE_CONFIGURE_MEMORY_PROTECTION=y
# CONFIG_SPIRAM_IGNORE_NOTFOUND is not set
# CONFIG_SPIRAM_USE_MEMMAP is not set
# CONFIG_SPIRAM_USE_CAPS_ALLOC is not set
CONFIG_SPIRAM_USE_MALLOC=y
# CONFIG_SPIRAM_MEMTEST is not set
CONFIG_SPIRAM_MALLOC_ALWAYSINTERNAL=512
# CONFIG_SPIRAM_TRY_ALLOCATE_WIFI_LWIP is not set
CONFIG_SPIRAM_MALLOC_RESERVE_INTERNAL=65536
# CONFIG_SPIRAM_ALLOW_BSS_SEG_EXTERNAL_MEMORY is not set
# CONFIG_SPIRAM_ALLOW_NOINIT_SEG_EXTERNAL_MEMORY is not set
# end of SPI RAM config
# end of ESP PSRAM

#
# ESP Ringbuf
#
# CONFIG_RINGBUF_PLACE_FUNCTIONS_INTO_FLASH is not set
# end of ESP Ringbuf

#
# ESP Security Specific
#
# end of ESP Security Specific

#
# ESP System Settings
#
# CONFIG_ESP_DEFAULT_CPU_FREQ_MHZ_80 is not set
# CONFIG_ESP_DEFAULT_CPU_FREQ_MHZ_160 is not set
CONFIG_ESP_DEFAULT_CPU_FREQ_MHZ_240=y
CONFIG_ESP_DEFAULT_CPU_FREQ_MHZ=240

#
# Cache config
#
# CONFIG_ESP32S3_INSTRUCTION_CACHE_16KB is not set
CONFIG_ESP32S3_INSTRUCTION_CACHE_32KB=y
CONFIG_ESP32S3_INSTRUCTION_CACHE_SIZE=0x8000
# CONFIG_ESP32S3_INSTRUCTION_CACHE_4WAYS is not set
CONFIG_ESP32S3_INSTRUCTION_CACHE_8WAYS=y
CONFIG_ESP32S3_ICACHE_ASSOCIATED_WAYS=8
CONFIG_ESP32S3_INSTRUCTION_CACHE_LINE_32B=y
CONFIG_ESP32S3_INSTRUCTION_CACHE_LINE_SIZE=32
# CONFIG_ESP32S3_DATA_CACHE_16KB is not set
CONFIG_ESP32S3_DATA_CACHE_32KB=y
# CONFIG_ESP32S3_DATA_CACHE_64KB is not set
CONFIG_ESP32S3_DATA_CACHE_SIZE=0x8000
# CONFIG_ESP32S3_DATA_CACHE_4WAYS is not set
CONFIG_ESP32S3_DATA_CACHE_8WAYS=y
CONFIG_ESP32S3_DCACHE_ASSOCIATED_WAYS=8
# CONFIG_ESP32S3_DATA_CACHE_LINE_16B is not set
# CONFIG_ESP32S3_DATA_CACHE_LINE_32B is not set
CONFIG_ESP32S3_DATA_CACHE_LINE_64B=y
CONFIG_ESP32S3_DATA_CACHE_LINE_SIZE=64
# end of Cache config

#
# Memory
#
# CONFIG_ESP32S3_RTCDATA_IN_FAST_MEM is not set
# CONFIG_ESP32S3_USE_FIXED_STATIC_RAM_SIZE is not set
# end of Memory

#
# Trace memory
#
# CONFIG_ESP32S3_TRAX is not set
CONFIG_ESP32S3_TRACEMEM_RESERVE_DRAM=0x0
# end of Trace memory

# CONFIG_ESP_SYSTEM_PANIC_PRINT_HALT is not set
CONFIG_ESP_SYSTEM_PANIC_PRINT_REBOOT=y
# CONFIG_ESP_SYSTEM_PANIC_SILENT_REBOOT is not set
# CONFIG_ESP_SYSTEM_PANIC_GDBSTUB is not set
CONFIG_ESP_SYSTEM_PANIC_REBOOT_DELAY_SECONDS=0
CONFIG_ESP_SYSTEM_RTC_FAST_MEM_AS_HEAP_DEPCHECK=y
CONFIG_ESP_SYSTEM_ALLOW_RTC_FAST_MEM_AS_HEAP=y

#
# Memory protection
#
CONFIG_ESP_SYSTEM_MEMPROT_FEATURE=y
CONFIG_ESP_SYSTEM_MEMPROT_FEATURE_LOCK=y
# end of Memory protection

CONFIG_ESP_SYSTEM_EVENT_QUEUE_SIZE=32
CONFIG_ESP_SYSTEM_EVENT_TASK_STACK_SIZE=4096
CONFIG_ESP_MAIN_TASK_STACK_SIZE=8192
CONFIG_ESP_MAIN_TASK_AFFINITY_CPU0=y
# CONFIG_ESP_MAIN_TASK_AFFINITY_CPU1 is not set
# CONFIG_ESP_MAIN_TASK_AFFINITY_NO_AFFINITY is not set
CONFIG_ESP_MAIN_TASK_AFFINITY=0x0
CONFIG_ESP_MINIMAL_SHARED_STACK_SIZE=2048
CONFIG_ESP_CONSOLE_UART_DEFAULT=y
# CONFIG_ESP_CONSOLE_USB_CDC is not set
# CONFIG_ESP_CONSOLE_USB_SERIAL_JTAG is not set
# CONFIG_ESP_CONSOLE_UART_CUSTOM is not set
# CONFIG_ESP_CONSOLE_NONE is not set
# CONFIG_ESP_CONSOLE_SECONDARY_NONE is not set
CONFIG_ESP_CONSOLE_SECONDARY_USB_SERIAL_JTAG=y
CONFIG_ESP_CONSOLE_USB_SERIAL_JTAG_ENABLED=y
CONFIG_ESP_CONSOLE_UART=y
CONFIG_ESP_CONSOLE_UART_NUM=0
CONFIG_ESP_CONSOLE_ROM_SERIAL_PORT_NUM=0
CONFIG_ESP_CONSOLE_UART_BAUDRATE=115200
CONFIG_ESP_INT_WDT=y
CONFIG_ESP_INT_WDT_TIMEOUT_MS=300
CONFIG_ESP_INT_WDT_CHECK_CPU1=y
CONFIG_ESP_TASK_WDT_EN=y
CONFIG_ESP_TASK_WDT_INIT=y
# CONFIG_ESP_TASK_WDT_PANIC is not set
CONFIG_ESP_TASK_WDT_TIMEOUT_S=10
CONFIG_ESP_TASK_WDT_CHECK_IDLE_TASK_CPU0=y
CONFIG_ESP_TASK_WDT_CHECK_IDLE_TASK_CPU1=y
# CONFIG_ESP_PANIC_HANDLER_IRAM is not set
# CONFIG_ESP_DEBUG_STUBS_ENABLE is not set
CONFIG_ESP_DEBUG_OCDAWARE=y
CONFIG_ESP_SYSTEM_CHECK_INT_LEVEL_4=y

#
# Brownout Detector
#
CONFIG_ESP_BROWNOUT_DET=y
CONFIG_ESP_BROWNOUT_DET_LVL_SEL_7=y
# CONFIG_ESP_BROWNOUT_DET_LVL_SEL_6 is not set
# CONFIG_ESP_BROWNOUT_DET_LVL_SEL_5 is not set
# CONFIG_ESP_BROWNOUT_DET_LVL_SEL_4 is not set
# CONFIG_ESP_BROWNOUT_DET_LVL_SEL_3 is not set
# CONFIG_ESP_BROWNOUT_DET_LVL_SEL_2 is not set
# CONFIG_ESP_BROWNOUT_DET_LVL_SEL_1 is not set
CONFIG_ESP_BROWNOUT_DET_LVL=7
# end of Brownout Detector

CONFIG_ESP_SYSTEM_BROWNOUT_INTR=y
CONFIG_ESP_SYSTEM_BBPLL_RECALIB=y
# end of ESP System Settings

#
# IPC (Inter-Processor Call)
#
CONFIG_ESP_IPC_TASK_STACK_SIZE=1280
CONFIG_ESP_IPC_USES_CALLERS_PRIORITY=y
CONFIG_ESP_IPC_ISR_ENABLE=y
# end of IPC (Inter-Processor Call)

#
# ESP Timer (High Resolution Timer)
#
# CONFIG_ESP_TIMER_PROFILING is not set
CONFIG_ESP_TIME_FUNCS_USE_RTC_TIMER=y
CONFIG_ESP_TIME_FUNCS_USE_ESP_TIMER=y
CONFIG_ESP_TIMER_TASK_STACK_SIZE=3584
CONFIG_ESP_TIMER_INTERRUPT_LEVEL=1
# CONFIG_ESP_TIMER_SHOW_EXPERIMENTAL is not set
CONFIG_ESP_TIMER_TASK_AFFINITY=0x0
CONFIG_ESP_TIMER_TASK_AFFINITY_CPU0=y
CONFIG_ESP_TIMER_ISR_AFFINITY_CPU0=y
# CONFIG_ESP_TIMER_SUPPORTS_ISR_DISPATCH_METHOD is not set
CONFIG_ESP_TIMER_IMPL_SYSTIMER=y
# end of ESP Timer (High Resolution Timer)

#
# Wi-Fi
#
CONFIG_ESP_WIFI_ENABLED=y
CONFIG_ESP_WIFI_STATIC_RX_BUFFER_NUM=3
CONFIG_ESP_WIFI_DYNAMIC_RX_BUFFER_NUM=6
# CONFIG_ESP_WIFI_STATIC_TX_BUFFER is not set
CONFIG_ESP_WIFI_DYNAMIC_TX_BUFFER=y
CONFIG_ESP_WIFI_TX_BUFFER_TYPE=1
CONFIG_ESP_WIFI_DYNAMIC_TX_BUFFER_NUM=32
# CONFIG_ESP_WIFI_STATIC_RX_MGMT_BUFFER is not set
CONFIG_ESP_WIFI_DYNAMIC_RX_MGMT_BUFFER=y
CONFIG_ESP_WIFI_DYNAMIC_RX_MGMT_BUF=1
CONFIG_ESP_WIFI_RX_MGMT_BUF_NUM_DEF=5
# CONFIG_ESP_WIFI_CSI_ENABLED is not set
CONFIG_ESP_WIFI_AMPDU_TX_ENABLED=y
CONFIG_ESP_WIFI_TX_BA_WIN=6
CONFIG_ESP_WIFI_AMPDU_RX_ENABLED=y
CONFIG_ESP_WIFI_RX_BA_WIN=3
CONFIG_ESP_WIFI_NVS_ENABLED=y
CONFIG_ESP_WIFI_TASK_PINNED_TO_CORE_0=y
# CONFIG_ESP_WIFI_TASK_PINNED_TO_CORE_1 is not set
CONFIG_ESP_WIFI_SOFTAP_BEACON_MAX_LEN=752
CONFIG_ESP_WIFI_MGMT_SBUF_NUM=32
# CONFIG_ESP_WIFI_IRAM_OPT is not set
# CONFIG_ESP_WIFI_EXTRA_IRAM_OPT is not set
# CONFIG_ESP_WIFI_RX_IRAM_OPT is not set
CONFIG_ESP_WIFI_ENABLE_WPA3_SAE=y
CONFIG_ESP_WIFI_ENABLE_SAE_PK=y
CONFIG_ESP_WIFI_SOFTAP_SAE_SUPPORT=y
CONFIG_ESP_WIFI_ENABLE_WPA3_OWE_STA=y
# CONFIG_ESP_WIFI_SLP_IRAM_OPT is not set
CONFIG_ESP_WIFI_SLP_DEFAULT_MIN_ACTIVE_TIME=50
CONFIG_ESP_WIFI_SLP_DEFAULT_MAX_ACTIVE_TIME=10
CONFIG_ESP_WIFI_SLP_DEFAULT_WAIT_BROADCAST_DATA_TIME=15
# CONFIG_ESP_WIFI_FTM_ENABLE is not set
CONFIG_ESP_WIFI_STA_DISCONNECTED_PM_ENABLE=y
# CONFIG_ESP_WIFI_GCMP_SUPPORT is not set
CONFIG_ESP_WIFI_GMAC_SUPPORT=y
CONFIG_ESP_WIFI_SOFTAP_SUPPORT=y
# CONFIG_ESP_WIFI_SLP_BEACON_LOST_OPT is not set
CONFIG_ESP_WIFI_ESPNOW_MAX_ENCRYPT_NUM=7
CONFIG_ESP_WIFI_MBEDTLS_CRYPTO=y
# CONFIG_ESP_WIFI_WAPI_PSK is not set
# CONFIG_ESP_WIFI_11KV_SUPPORT is not set
# CONFIG_ESP_WIFI_MBO_SUPPORT is not set
# CONFIG_ESP_WIFI_DPP_SUPPORT is not set
# CONFIG_ESP_WIFI_11R_SUPPORT is not set
# CONFIG_ESP_WIFI_WPS_SOFTAP_REGISTRAR is not set

#
# WPS Configuration Options
#
# CONFIG_ESP_WIFI_WPS_STRICT is not set
# CONFIG_ESP_WIFI_WPS_PASSPHRASE is not set
# end of WPS Configuration Options

# CONFIG_ESP_WIFI_DEBUG_PRINT is not set
# CONFIG_ESP_WIFI_TESTING_OPTIONS is not set
# CONFIG_ESP_WIFI_ENTERPRISE_SUPPORT is not set
# end of Wi-Fi

#
# Core dump
#
# CONFIG_ESP_COREDUMP_ENABLE_TO_FLASH is not set
# CONFIG_ESP_COREDUMP_ENABLE_TO_UART is not set
CONFIG_ESP_COREDUMP_ENABLE_TO_NONE=y
# end of Core dump

#
# FAT Filesystem support
#
CONFIG_FATFS_VOLUME_COUNT=2
CONFIG_FATFS_LFN_NONE=y
# CONFIG_FATFS_LFN_HEAP is not set
# CONFIG_FATFS_LFN_STACK is not set
# CONFIG_FATFS_SECTOR_512 is not set
CONFIG_FATFS_SECTOR_4096=y
# CONFIG_FATFS_CODEPAGE_DYNAMIC is not set
CONFIG_FATFS_CODEPAGE_437=y
# CONFIG_FATFS_CODEPAGE_720 is not set
# CONFIG_FATFS_CODEPAGE_737 is not set
# CONFIG_FATFS_CODEPAGE_771 is not set
# CONFIG_FATFS_CODEPAGE_775 is not set
# CONFIG_FATFS_CODEPAGE_850 is not set
# CONFIG_FATFS_CODEPAGE_852 is not set
# CONFIG_FATFS_CODEPAGE_855 is not set
# CONFIG_FATFS_CODEPAGE_857 is not set
# CONFIG_FATFS_CODEPAGE_860 is not set
# CONFIG_FATFS_CODEPAGE_861 is not set
# CONFIG_FATFS_CODEPAGE_862 is not set
# CONFIG_FATFS_CODEPAGE_863 is not set
# CONFIG_FATFS_CODEPAGE_864 is not set
# CONFIG_FATFS_CODEPAGE_865 is not set
# CONFIG_FATFS_CODEPAGE_866 is not set
# CONFIG_FATFS_CODEPAGE_869 is not set
# CONFIG_FATFS_CODEPAGE_932 is not set
# CONFIG_FATFS_CODEPAGE_936 is not set
# CONFIG_FATFS_CODEPAGE_949 is not set
# CONFIG_FATFS_CODEPAGE_950 is not set
CONFIG_FATFS_CODEPAGE=437
CONFIG_FATFS_FS_LOCK=0
CONFIG_FATFS_TIMEOUT_MS=10000
CONFIG_FATFS_PER_FILE_CACHE=y
CONFIG_FATFS_ALLOC_PREFER_EXTRAM=y
# CONFIG_FATFS_USE_FASTSEEK is not set
CONFIG_FATFS_USE_STRFUNC_NONE=y
# CONFIG_FATFS_USE_STRFUNC_WITHOUT_CRLF_CONV is not set
# CONFIG_FATFS_USE_STRFUNC_WITH_CRLF_CONV is not set
CONFIG_FATFS_VFS_FSTAT_BLKSIZE=0
# CONFIG_FATFS_IMMEDIATE_FSYNC is not set
# CONFIG_FATFS_USE_LABEL is not set
CONFIG_FATFS_LINK_LOCK=y
# CONFIG_FATFS_USE_DYN_BUFFERS is not set

#
# File system free space calculation behavior
#
CONFIG_FATFS_DONT_TRUST_FREE_CLUSTER_CNT=0
CONFIG_FATFS_DONT_TRUST_LAST_ALLOC=0
# end of File system free space calculation behavior
# end of FAT Filesystem support

#
# FreeRTOS
#

#
# Kernel
#
# CONFIG_FREERTOS_SMP is not set
# CONFIG_FREERTOS_UNICORE is not set
CONFIG_FREERTOS_HZ=100
# CONFIG_FREERTOS_CHECK_STACKOVERFLOW_NONE is not set
# CONFIG_FREERTOS_CHECK_STACKOVERFLOW_PTRVAL is not set
CONFIG_FREERTOS_CHECK_STACKOVERFLOW_CANARY=y
CONFIG_FREERTOS_THREAD_LOCAL_STORAGE_POINTERS=1
CONFIG_FREERTOS_IDLE_TASK_STACKSIZE=1536
# CONFIG_FREERTOS_USE_IDLE_HOOK is not set
# CONFIG_FREERTOS_USE_TICK_HOOK is not set
CONFIG_FREERTOS_MAX_TASK_NAME_LEN=16
# CONFIG_FREERTOS_ENABLE_BACKWARD_COMPATIBILITY is not set
CONFIG_FREERTOS_USE_TIMERS=y
CONFIG_FREERTOS_TIMER_SERVICE_TASK_NAME="Tmr Svc"
# CONFIG_FREERTOS_TIMER_TASK_AFFINITY_CPU0 is not set
# CONFIG_FREERTOS_TIMER_TASK_AFFINITY_CPU1 is not set
CONFIG_FREERTOS_TIMER_TASK_NO_AFFINITY=y
CONFIG_FREERTOS_TIMER_SERVICE_TASK_CORE_AFFINITY=0x7FFFFFFF
CONFIG_FREERTOS_TIMER_TASK_PRIORITY=1
CONFIG_FREERTOS_TIMER_TASK_STACK_DEPTH=2048
CONFIG_FREERTOS_TIMER_QUEUE_LENGTH=10
CONFIG_FREERTOS_QUEUE_REGISTRY_SIZE=0
CONFIG_FREERTOS_TASK_NOTIFICATION_ARRAY_ENTRIES=1
CONFIG_FREERTOS_USE_TRACE_FACILITY=y
CONFIG_FREERTOS_USE_STATS_FORMATTING_FUNCTIONS=y
# CONFIG_FREERTOS_USE_LIST_DATA_INTEGRITY_CHECK_BYTES is not set
# CONFIG_FREERTOS_VTASKLIST_INCLUDE_COREID is not set
CONFIG_FREERTOS_GENERATE_RUN_TIME_STATS=y
CONFIG_FREERTOS_RUN_TIME_COUNTER_TYPE_U32=y
# CONFIG_FREERTOS_RUN_TIME_COUNTER_TYPE_U64 is not set
# CONFIG_FREERTOS_USE_APPLICATION_TASK_TAG is not set
# end of Kernel

#
# Port
#
# CONFIG_FREERTOS_WATCHPOINT_END_OF_STACK is not set
CONFIG_FREERTOS_TLSP_DELETION_CALLBACKS=y
# CONFIG_FREERTOS_TASK_PRE_DELETION_HOOK is not set
# CONFIG_FREERTOS_ENABLE_STATIC_TASK_CLEAN_UP is not set
CONFIG_FREERTOS_CHECK_MUTEX_GIVEN_BY_OWNER=y
CONFIG_FREERTOS_ISR_STACKSIZE=1536
CONFIG_FREERTOS_INTERRUPT_BACKTRACE=y
# CONFIG_FREERTOS_FPU_IN_ISR is not set
CONFIG_FREERTOS_TICK_SUPPORT_SYSTIMER=y
CONFIG_FREERTOS_CORETIMER_SYSTIMER_LVL1=y
# CONFIG_FREERTOS_CORETIMER_SYSTIMER_LVL3 is not set
CONFIG_FREERTOS_SYSTICK_USES_SYSTIMER=y
CONFIG_FREERTOS_RUN_TIME_STATS_USING_ESP_TIMER=y
# CONFIG_FREERTOS_PLACE_FUNCTIONS_INTO_FLASH is not set
# CONFIG_FREERTOS_CHECK_PORT_CRITICAL_COMPLIANCE is not set
# end of Port

#
# Extra
#
CONFIG_FREERTOS_TASK_CREATE_ALLOW_EXT_MEM=y
# end of Extra

CONFIG_FREERTOS_PORT=y
CONFIG_FREERTOS_NO_AFFINITY=0x7FFFFFFF
CONFIG_FREERTOS_SUPPORT_STATIC_ALLOCATION=y
CONFIG_FREERTOS_DEBUG_OCDAWARE=y
CONFIG_FREERTOS_ENABLE_TASK_SNAPSHOT=y
CONFIG_FREERTOS_PLACE_SNAPSHOT_FUNS_INTO_FLASH=y
CONFIG_FREERTOS_NUMBER_OF_CORES=2
# end of FreeRTOS

#
# Hardware Abstraction Layer (HAL) and Low Level (LL)
#
CONFIG_HAL_ASSERTION_EQUALS_SYSTEM=y
# CONFIG_HAL_ASSERTION_DISABLE is not set
# CONFIG_HAL_ASSERTION_SILENT is not set
# CONFIG_HAL_ASSERTION_ENABLE is not set
CONFIG_HAL_DEFAULT_ASSERTION_LEVEL=2
CONFIG_HAL_WDT_USE_ROM_IMPL=y
CONFIG_HAL_SPI_MASTER_FUNC_IN_IRAM=y
CONFIG_HAL_SPI_SLAVE_FUNC_IN_IRAM=y
# end of Hardware Abstraction Layer (HAL) and Low Level (LL)

#
# Heap memory debugging
#
CONFIG_HEAP_POISONING_DISABLED=y
# CONFIG_HEAP_POISONING_LIGHT is not set
# CONFIG_HEAP_POISONING_COMPREHENSIVE is not set
CONFIG_HEAP_TRACING_OFF=y
# CONFIG_HEAP_TRACING_STANDALONE is not set
# CONFIG_HEAP_TRACING_TOHOST is not set
# CONFIG_HEAP_USE_HOOKS is not set
# CONFIG_HEAP_TASK_TRACKING is not set
# CONFIG_HEAP_ABORT_WHEN_ALLOCATION_FAILS is not set
# CONFIG_HEAP_PLACE_FUNCTION_INTO_FLASH is not set
# end of Heap memory debugging

#
# Log
#

#
# Log Level
#
# CONFIG_LOG_DEFAULT_LEVEL_NONE is not set
# CONFIG_LOG_DEFAULT_LEVEL_ERROR is not set
# CONFIG_LOG_DEFAULT_LEVEL_WARN is not set
CONFIG_LOG_DEFAULT_LEVEL_INFO=y
# CONFIG_LOG_DEFAULT_LEVEL_DEBUG is not set
# CONFIG_LOG_DEFAULT_LEVEL_VERBOSE is not set
CONFIG_LOG_DEFAULT_LEVEL=3
CONFIG_LOG_MAXIMUM_EQUALS_DEFAULT=y
# CONFIG_LOG_MAXIMUM_LEVEL_DEBUG is not set
# CONFIG_LOG_MAXIMUM_LEVEL_VERBOSE is not set
CONFIG_LOG_MAXIMUM_LEVEL=3

#
# Level Settings
#
# CONFIG_LOG_MASTER_LEVEL is not set
CONFIG_LOG_DYNAMIC_LEVEL_CONTROL=y
# CONFIG_LOG_TAG_LEVEL_IMPL_NONE is not set
# CONFIG_LOG_TAG_LEVEL_IMPL_LINKED_LIST is not set
CONFIG_LOG_TAG_LEVEL_IMPL_CACHE_AND_LINKED_LIST=y
# CONFIG_LOG_TAG_LEVEL_CACHE_ARRAY is not set
CONFIG_LOG_TAG_LEVEL_CACHE_BINARY_MIN_HEAP=y
CONFIG_LOG_TAG_LEVEL_IMPL_CACHE_SIZE=31
# end of Level Settings
# end of Log Level

#
# Format
#
# CONFIG_LOG_COLORS is not set
CONFIG_LOG_TIMESTAMP_SOURCE_RTOS=y
# CONFIG_LOG_TIMESTAMP_SOURCE_SYSTEM is not set
# end of Format
# end of Log

#
# LWIP
#
CONFIG_LWIP_ENABLE=y
CONFIG_LWIP_LOCAL_HOSTNAME="espressif"
# CONFIG_LWIP_NETIF_API is not set
CONFIG_LWIP_TCPIP_TASK_PRIO=18
# CONFIG_LWIP_TCPIP_CORE_LOCKING is not set
# CONFIG_LWIP_CHECK_THREAD_SAFETY is not set
CONFIG_LWIP_DNS_SUPPORT_MDNS_QUERIES=y
# CONFIG_LWIP_L2_TO_L3_COPY is not set
# CONFIG_LWIP_IRAM_OPTIMIZATION is not set
# CONFIG_LWIP_EXTRA_IRAM_OPTIMIZATION is not set
CONFIG_LWIP_TIMERS_ONDEMAND=y
CONFIG_LWIP_ND6=y
# CONFIG_LWIP_FORCE_ROUTER_FORWARDING is not set
CONFIG_LWIP_MAX_SOCKETS=10
# CONFIG_LWIP_USE_ONLY_LWIP_SELECT is not set
# CONFIG_LWIP_SO_LINGER is not set
CONFIG_LWIP_SO_REUSE=y
CONFIG_LWIP_SO_REUSE_RXTOALL=y
# CONFIG_LWIP_SO_RCVBUF is not set
# CONFIG_LWIP_NETBUF_RECVINFO is not set
CONFIG_LWIP_IP_DEFAULT_TTL=64
CONFIG_LWIP_IP4_FRAG=y
CONFIG_LWIP_IP6_FRAG=y
# CONFIG_LWIP_IP4_REASSEMBLY is not set
# CONFIG_LWIP_IP6_REASSEMBLY is not set
CONFIG_LWIP_IP_REASS_MAX_PBUFS=10
# CONFIG_LWIP_IP_FORWARD is not set
# CONFIG_LWIP_STATS is not set
CONFIG_LWIP_ESP_GRATUITOUS_ARP=y
CONFIG_LWIP_GARP_TMR_INTERVAL=60
CONFIG_LWIP_ESP_MLDV6_REPORT=y
CONFIG_LWIP_MLDV6_TMR_INTERVAL=40
CONFIG_LWIP_TCPIP_RECVMBOX_SIZE=16
CONFIG_LWIP_DHCP_DOES_ARP_CHECK=y
# CONFIG_LWIP_DHCP_DOES_ACD_CHECK is not set
# CONFIG_LWIP_DHCP_DOES_NOT_CHECK_OFFERED_IP is not set
# CONFIG_LWIP_DHCP_DISABLE_CLIENT_ID is not set
CONFIG_LWIP_DHCP_DISABLE_VENDOR_CLASS_ID=y
# CONFIG_LWIP_DHCP_RESTORE_LAST_IP is not set
CONFIG_LWIP_DHCP_OPTIONS_LEN=69
CONFIG_LWIP_NUM_NETIF_CLIENT_DATA=0
CONFIG_LWIP_DHCP_COARSE_TIMER_SECS=1

#
# DHCP server
#
CONFIG_LWIP_DHCPS=y
CONFIG_LWIP_DHCPS_LEASE_UNIT=60
CONFIG_LWIP_DHCPS_MAX_STATION_NUM=8
CONFIG_LWIP_DHCPS_STATIC_ENTRIES=y
CONFIG_LWIP_DHCPS_ADD_DNS=y
# end of DHCP server

# CONFIG_LWIP_AUTOIP is not set
CONFIG_LWIP_IPV4=y
CONFIG_LWIP_IPV6=y
# CONFIG_LWIP_IPV6_AUTOCONFIG is not set
CONFIG_LWIP_IPV6_NUM_ADDRESSES=3
# CONFIG_LWIP_IPV6_FORWARD is not set
# CONFIG_LWIP_NETIF_STATUS_CALLBACK is not set
CONFIG_LWIP_NETIF_LOOPBACK=y
CONFIG_LWIP_LOOPBACK_MAX_PBUFS=8

#
# TCP
#
CONFIG_LWIP_MAX_ACTIVE_TCP=16
CONFIG_LWIP_MAX_LISTENING_TCP=16
CONFIG_LWIP_TCP_HIGH_SPEED_RETRANSMISSION=y
CONFIG_LWIP_TCP_MAXRTX=12
CONFIG_LWIP_TCP_SYNMAXRTX=12
CONFIG_LWIP_TCP_MSS=1440
CONFIG_LWIP_TCP_TMR_INTERVAL=250
CONFIG_LWIP_TCP_MSL=60000
CONFIG_LWIP_TCP_FIN_WAIT_TIMEOUT=20000
CONFIG_LWIP_TCP_SND_BUF_DEFAULT=5760
CONFIG_LWIP_TCP_WND_DEFAULT=5760
CONFIG_LWIP_TCP_RECVMBOX_SIZE=6
CONFIG_LWIP_TCP_ACCEPTMBOX_SIZE=6
CONFIG_LWIP_TCP_QUEUE_OOSEQ=y
CONFIG_LWIP_TCP_OOSEQ_TIMEOUT=6
CONFIG_LWIP_TCP_OOSEQ_MAX_PBUFS=4
# CONFIG_LWIP_TCP_SACK_OUT is not set
CONFIG_LWIP_TCP_OVERSIZE_MSS=y
# CONFIG_LWIP_TCP_OVERSIZE_QUARTER_MSS is not set
# CONFIG_LWIP_TCP_OVERSIZE_DISABLE is not set
CONFIG_LWIP_TCP_RTO_TIME=1500
# end of TCP

#
# UDP
#
CONFIG_LWIP_MAX_UDP_PCBS=16
CONFIG_LWIP_UDP_RECVMBOX_SIZE=6
# end of UDP

#
# Checksums
#
# CONFIG_LWIP_CHECKSUM_CHECK_IP is not set
# CONFIG_LWIP_CHECKSUM_CHECK_UDP is not set
CONFIG_LWIP_CHECKSUM_CHECK_ICMP=y
# end of Checksums

CONFIG_LWIP_TCPIP_TASK_STACK_SIZE=3072
CONFIG_LWIP_TCPIP_TASK_AFFINITY_NO_AFFINITY=y
# CONFIG_LWIP_TCPIP_TASK_AFFINITY_CPU0 is not set
# CONFIG_LWIP_TCPIP_TASK_AFFINITY_CPU1 is not set
CONFIG_LWIP_TCPIP_TASK_AFFINITY=0x7FFFFFFF
CONFIG_LWIP_IPV6_MEMP_NUM_ND6_QUEUE=3
CONFIG_LWIP_IPV6_ND6_NUM_NEIGHBORS=5
CONFIG_LWIP_IPV6_ND6_NUM_PREFIXES=5
CONFIG_LWIP_IPV6_ND6_NUM_ROUTERS=3
CONFIG_LWIP_IPV6_ND6_NUM_DESTINATIONS=10
# CONFIG_LWIP_IPV6_ND6_ROUTE_INFO_OPTION_SUPPORT is not set
# CONFIG_LWIP_PPP_SUPPORT is not set
# CONFIG_LWIP_SLIP_SUPPORT is not set

#
# ICMP
#
CONFIG_LWIP_ICMP=y
# CONFIG_LWIP_MULTICAST_PING is not set
# CONFIG_LWIP_BROADCAST_PING is not set
# end of ICMP

#
# LWIP RAW API
#
CONFIG_LWIP_MAX_RAW_PCBS=16
# end of LWIP RAW API

#
# SNTP
#
CONFIG_LWIP_SNTP_MAX_SERVERS=1
# CONFIG_LWIP_DHCP_GET_NTP_SRV is not set
CONFIG_LWIP_SNTP_UPDATE_DELAY=3600000
CONFIG_LWIP_SNTP_STARTUP_DELAY=y
CONFIG_LWIP_SNTP_MAXIMUM_STARTUP_DELAY=5000
# end of SNTP

#
# DNS
#
CONFIG_LWIP_DNS_MAX_HOST_IP=1
CONFIG_LWIP_DNS_MAX_SERVERS=3
# CONFIG_LWIP_FALLBACK_DNS_SERVER_SUPPORT is not set
# CONFIG_LWIP_DNS_SETSERVER_WITH_NETIF is not set
# end of DNS

CONFIG_LWIP_BRIDGEIF_MAX_PORTS=7
CONFIG_LWIP_ESP_LWIP_ASSERT=y

#
# Hooks
#
# CONFIG_LWIP_HOOK_TCP_ISN_NONE is not set
CONFIG_LWIP_HOOK_TCP_ISN_DEFAULT=y
# CONFIG_LWIP_HOOK_TCP_ISN_CUSTOM is not set
CONFIG_LWIP_HOOK_IP6_ROUTE_NONE=y
# CONFIG_LWIP_HOOK_IP6_ROUTE_DEFAULT is not set
# CONFIG_LWIP_HOOK_IP6_ROUTE_CUSTOM is not set
CONFIG_LWIP_HOOK_ND6_GET_GW_NONE=y
# CONFIG_LWIP_HOOK_ND6_GET_GW_DEFAULT is not set
# CONFIG_LWIP_HOOK_ND6_GET_GW_CUSTOM is not set
CONFIG_LWIP_HOOK_IP6_SELECT_SRC_ADDR_NONE=y
# CONFIG_LWIP_HOOK_IP6_SELECT_SRC_ADDR_DEFAULT is not set
# CONFIG_LWIP_HOOK_IP6_SELECT_SRC_ADDR_CUSTOM is not set
CONFIG_LWIP_HOOK_NETCONN_EXT_RESOLVE_NONE=y
# CONFIG_LWIP_HOOK_NETCONN_EXT_RESOLVE_DEFAULT is not set
# CONFIG_LWIP_HOOK_NETCONN_EXT_RESOLVE_CUSTOM is not set
CONFIG_LWIP_HOOK_DNS_EXT_RESOLVE_NONE=y
# CONFIG_LWIP_HOOK_DNS_EXT_RESOLVE_CUSTOM is not set
# CONFIG_LWIP_HOOK_IP6_INPUT_NONE is not set
CONFIG_LWIP_HOOK_IP6_INPUT_DEFAULT=y
# CONFIG_LWIP_HOOK_IP6_INPUT_CUSTOM is not set
# end of Hooks

# CONFIG_LWIP_DEBUG is not set
# end of LWIP

#
# mbedTLS
#
# CONFIG_MBEDTLS_INTERNAL_MEM_ALLOC is not set
CONFIG_MBEDTLS_EXTERNAL_MEM_ALLOC=y
# CONFIG_MBEDTLS_DEFAULT_MEM_ALLOC is not set
# CONFIG_MBEDTLS_CUSTOM_MEM_ALLOC is not set
CONFIG_MBEDTLS_ASYMMETRIC_CONTENT_LEN=y
CONFIG_MBEDTLS_SSL_IN_CONTENT_LEN=16384
CONFIG_MBEDTLS_SSL_OUT_CONTENT_LEN=4096
CONFIG_MBEDTLS_DYNAMIC_BUFFER=y
CONFIG_MBEDTLS_DYNAMIC_FREE_CONFIG_DATA=y
CONFIG_MBEDTLS_DYNAMIC_FREE_CA_CERT=y
# CONFIG_MBEDTLS_DEBUG is not set

#
# mbedTLS v3.x related
#
# CONFIG_MBEDTLS_SSL_VARIABLE_BUFFER_LENGTH is not set
# CONFIG_MBEDTLS_X509_TRUSTED_CERT_CALLBACK is not set
# CONFIG_MBEDTLS_SSL_CONTEXT_SERIALIZATION is not set
# CONFIG_MBEDTLS_SSL_KEEP_PEER_CERTIFICATE is not set
# CONFIG_MBEDTLS_SSL_KEYING_MATERIAL_EXPORT is not set
CONFIG_MBEDTLS_PKCS7_C=y
# end of mbedTLS v3.x related

#
# Certificate Bundle
#
CONFIG_MBEDTLS_CERTIFICATE_BUNDLE=y
CONFIG_MBEDTLS_CERTIFICATE_BUNDLE_DEFAULT_FULL=y
# CONFIG_MBEDTLS_CERTIFICATE_BUNDLE_DEFAULT_CMN is not set
# CONFIG_MBEDTLS_CERTIFICATE_BUNDLE_DEFAULT_NONE is not set
# CONFIG_MBEDTLS_CUSTOM_CERTIFICATE_BUNDLE is not set
# CONFIG_MBEDTLS_CERTIFICATE_BUNDLE_DEPRECATED_LIST is not set
CONFIG_MBEDTLS_CERTIFICATE_BUNDLE_MAX_CERTS=200
# end of Certificate Bundle

# CONFIG_MBEDTLS_ECP_RESTARTABLE is not set
CONFIG_MBEDTLS_CMAC_C=y
CONFIG_MBEDTLS_HARDWARE_AES=y
CONFIG_MBEDTLS_AES_USE_INTERRUPT=y
CONFIG_MBEDTLS_AES_INTERRUPT_LEVEL=0
CONFIG_MBEDTLS_GCM_SUPPORT_NON_AES_CIPHER=y
CONFIG_MBEDTLS_HARDWARE_MPI=y
# CONFIG_MBEDTLS_LARGE_KEY_SOFTWARE_MPI is not set
CONFIG_MBEDTLS_MPI_USE_INTERRUPT=y
CONFIG_MBEDTLS_MPI_INTERRUPT_LEVEL=0
CONFIG_MBEDTLS_HARDWARE_SHA=y
CONFIG_MBEDTLS_ROM_MD5=y
# CONFIG_MBEDTLS_ATCA_HW_ECDSA_SIGN is not set
# CONFIG_MBEDTLS_ATCA_HW_ECDSA_VERIFY is not set
CONFIG_MBEDTLS_HAVE_TIME=y
# CONFIG_MBEDTLS_PLATFORM_TIME_ALT is not set
# CONFIG_MBEDTLS_HAVE_TIME_DATE is not set
CONFIG_MBEDTLS_ECDSA_DETERMINISTIC=y
CONFIG_MBEDTLS_SHA1_C=y
CONFIG_MBEDTLS_SHA512_C=y
# CONFIG_MBEDTLS_SHA3_C is not set
CONFIG_MBEDTLS_TLS_SERVER_AND_CLIENT=y
# CONFIG_MBEDTLS_TLS_SERVER_ONLY is not set
# CONFIG_MBEDTLS_TLS_CLIENT_ONLY is not set
# CONFIG_MBEDTLS_TLS_DISABLED is not set
CONFIG_MBEDTLS_TLS_SERVER=y
CONFIG_MBEDTLS_TLS_CLIENT=y
CONFIG_MBEDTLS_TLS_ENABLED=y

#
# TLS Key Exchange Methods
#
# CONFIG_MBEDTLS_PSK_MODES is not set
CONFIG_MBEDTLS_KEY_EXCHANGE_RSA=y
CONFIG_MBEDTLS_KEY_EXCHANGE_ELLIPTIC_CURVE=y
CONFIG_MBEDTLS_KEY_EXCHANGE_ECDHE_RSA=y
CONFIG_MBEDTLS_KEY_EXCHANGE_ECDHE_ECDSA=y
CONFIG_MBEDTLS_KEY_EXCHANGE_ECDH_ECDSA=y
CONFIG_MBEDTLS_KEY_EXCHANGE_ECDH_RSA=y
# end of TLS Key Exchange Methods

# CONFIG_MBEDTLS_SSL_RENEGOTIATION is not set
CONFIG_MBEDTLS_SSL_PROTO_TLS1_2=y
# CONFIG_MBEDTLS_SSL_PROTO_GMTSSL1_1 is not set
# CONFIG_MBEDTLS_SSL_PROTO_DTLS is not set
CONFIG_MBEDTLS_SSL_ALPN=y
CONFIG_MBEDTLS_CLIENT_SSL_SESSION_TICKETS=y
CONFIG_MBEDTLS_SERVER_SSL_SESSION_TICKETS=y

#
# Symmetric Ciphers
#
CONFIG_MBEDTLS_AES_C=y
# CONFIG_MBEDTLS_CAMELLIA_C is not set
# CONFIG_MBEDTLS_DES_C is not set
# CONFIG_MBEDTLS_BLOWFISH_C is not set
# CONFIG_MBEDTLS_XTEA_C is not set
CONFIG_MBEDTLS_CCM_C=y
CONFIG_MBEDTLS_GCM_C=y
# CONFIG_MBEDTLS_NIST_KW_C is not set
# end of Symmetric Ciphers

# CONFIG_MBEDTLS_RIPEMD160_C is not set

#
# Certificates
#
CONFIG_MBEDTLS_PEM_PARSE_C=y
CONFIG_MBEDTLS_PEM_WRITE_C=y
CONFIG_MBEDTLS_X509_CRL_PARSE_C=y
CONFIG_MBEDTLS_X509_CSR_PARSE_C=y
# end of Certificates

CONFIG_MBEDTLS_ECP_C=y
CONFIG_MBEDTLS_PK_PARSE_EC_EXTENDED=y
CONFIG_MBEDTLS_PK_PARSE_EC_COMPRESSED=y
# CONFIG_MBEDTLS_DHM_C is not set
CONFIG_MBEDTLS_ECDH_C=y
CONFIG_MBEDTLS_ECDSA_C=y
# CONFIG_MBEDTLS_ECJPAKE_C is not set
CONFIG_MBEDTLS_ECP_DP_SECP192R1_ENABLED=y
CONFIG_MBEDTLS_ECP_DP_SECP224R1_ENABLED=y
CONFIG_MBEDTLS_ECP_DP_SECP256R1_ENABLED=y
CONFIG_MBEDTLS_ECP_DP_SECP384R1_ENABLED=y
CONFIG_MBEDTLS_ECP_DP_SECP521R1_ENABLED=y
CONFIG_MBEDTLS_ECP_DP_SECP192K1_ENABLED=y
CONFIG_MBEDTLS_ECP_DP_SECP224K1_ENABLED=y
CONFIG_MBEDTLS_ECP_DP_SECP256K1_ENABLED=y
CONFIG_MBEDTLS_ECP_DP_BP256R1_ENABLED=y
CONFIG_MBEDTLS_ECP_DP_BP384R1_ENABLED=y
CONFIG_MBEDTLS_ECP_DP_BP512R1_ENABLED=y
CONFIG_MBEDTLS_ECP_DP_CURVE25519_ENABLED=y
CONFIG_MBEDTLS_ECP_NIST_OPTIM=y
# CONFIG_MBEDTLS_ECP_FIXED_POINT_OPTIM is not set
# CONFIG_MBEDTLS_POLY1305_C is not set
# CONFIG_MBEDTLS_CHACHA20_C is not set
# CONFIG_MBEDTLS_HKDF_C is not set
# CONFIG_MBEDTLS_THREADING_C is not set
CONFIG_MBEDTLS_ERROR_STRINGS=y
CONFIG_MBEDTLS_FS_IO=y
# CONFIG_MBEDTLS_ALLOW_WEAK_CERTIFICATE_VERIFICATION is not set
# end of mbedTLS

#
# ESP-MQTT Configurations
#
CONFIG_MQTT_PROTOCOL_311=y
# CONFIG_MQTT_PROTOCOL_5 is not set
CONFIG_MQTT_TRANSPORT_SSL=y
CONFIG_MQTT_TRANSPORT_WEBSOCKET=y
CONFIG_MQTT_TRANSPORT_WEBSOCKET_SECURE=y
# CONFIG_MQTT_MSG_ID_INCREMENTAL is not set
# CONFIG_MQTT_SKIP_PUBLISH_IF_DISCONNECTED is not set
# CONFIG_MQTT_REPORT_DELETED_MESSAGES is not set
# CONFIG_MQTT_USE_CUSTOM_CONFIG is not set
# CONFIG_MQTT_TASK_CORE_SELECTION_ENABLED is not set
# CONFIG_MQTT_CUSTOM_OUTBOX is not set
# end of ESP-MQTT Configurations

#
# Newlib
#
CONFIG_NEWLIB_STDOUT_LINE_ENDING_CRLF=y
# CONFIG_NEWLIB_STDOUT_LINE_ENDING_LF is not set
# CONFIG_NEWLIB_STDOUT_LINE_ENDING_CR is not set
# CONFIG_NEWLIB_STDIN_LINE_ENDING_CRLF is not set
# CONFIG_NEWLIB_STDIN_LINE_ENDING_LF is not set
CONFIG_NEWLIB_STDIN_LINE_ENDING_CR=y
CONFIG_NEWLIB_NANO_FORMAT=y
CONFIG_NEWLIB_TIME_SYSCALL_USE_RTC_HRT=y
# CONFIG_NEWLIB_TIME_SYSCALL_USE_RTC is not set
# CONFIG_NEWLIB_TIME_SYSCALL_USE_HRT is not set
# CONFIG_NEWLIB_TIME_SYSCALL_USE_NONE is not set
# end of Newlib

CONFIG_STDATOMIC_S32C1I_SPIRAM_WORKAROUND=y

#
# NVS
#
# CONFIG_NVS_ENCRYPTION is not set
# CONFIG_NVS_ASSERT_ERROR_CHECK is not set
# CONFIG_NVS_LEGACY_DUP_KEYS_COMPATIBILITY is not set
# CONFIG_NVS_ALLOCATE_CACHE_IN_SPIRAM is not set
# end of NVS

#
# OpenThread
#
# CONFIG_OPENTHREAD_ENABLED is not set

#
# OpenThread Spinel
#
# CONFIG_OPENTHREAD_SPINEL_ONLY is not set
# end of OpenThread Spinel

# CONFIG_OPENTHREAD_DEBUG is not set
# end of OpenThread

#
# Protocomm
#
CONFIG_ESP_PROTOCOMM_SUPPORT_SECURITY_VERSION_0=y
CONFIG_ESP_PROTOCOMM_SUPPORT_SECURITY_VERSION_1=y
CONFIG_ESP_PROTOCOMM_SUPPORT_SECURITY_VERSION_2=y
CONFIG_ESP_PROTOCOMM_SUPPORT_SECURITY_PATCH_VERSION=y
# end of Protocomm

#
# PThreads
#
CONFIG_PTHREAD_TASK_PRIO_DEFAULT=5
CONFIG_PTHREAD_TASK_STACK_SIZE_DEFAULT=3072
CONFIG_PTHREAD_STACK_MIN=768
CONFIG_PTHREAD_DEFAULT_CORE_NO_AFFINITY=y
# CONFIG_PTHREAD_DEFAULT_CORE_0 is not set
# CONFIG_PTHREAD_DEFAULT_CORE_1 is not set
CONFIG_PTHREAD_TASK_CORE_DEFAULT=-1
CONFIG_PTHREAD_TASK_NAME_DEFAULT="pthread"
# end of PThreads

#
# MMU Config
#
CONFIG_MMU_PAGE_SIZE_64KB=y
CONFIG_MMU_PAGE_MODE="64KB"
CONFIG_MMU_PAGE_SIZE=0x10000
# end of MMU Config

#
# Main Flash configuration
#

#
# SPI Flash behavior when brownout
#
CONFIG_SPI_FLASH_BROWNOUT_RESET_XMC=y
CONFIG_SPI_FLASH_BROWNOUT_RESET=y
# end of SPI Flash behavior when brownout

#
# Optional and Experimental Features (READ DOCS FIRST)
#

#
# Features here require specific hardware (READ DOCS FIRST!)
#
# CONFIG_SPI_FLASH_HPM_ENA is not set
CONFIG_SPI_FLASH_HPM_AUTO=y
# CONFIG_SPI_FLASH_HPM_DIS is not set
CONFIG_SPI_FLASH_HPM_ON=y
CONFIG_SPI_FLASH_HPM_DC_AUTO=y
# CONFIG_SPI_FLASH_HPM_DC_DISABLE is not set
# CONFIG_SPI_FLASH_AUTO_SUSPEND is not set
CONFIG_SPI_FLASH_SUSPEND_TSUS_VAL_US=50
# CONFIG_SPI_FLASH_FORCE_ENABLE_XMC_C_SUSPEND is not set
# CONFIG_SPI_FLASH_FORCE_ENABLE_C6_H2_SUSPEND is not set
# end of Optional and Experimental Features (READ DOCS FIRST)
# end of Main Flash configuration

#
# SPI Flash driver
#
# CONFIG_SPI_FLASH_VERIFY_WRITE is not set
# CONFIG_SPI_FLASH_ENABLE_COUNTERS is not set
CONFIG_SPI_FLASH_ROM_DRIVER_PATCH=y
# CONFIG_SPI_FLASH_ROM_IMPL is not set
CONFIG_SPI_FLASH_DANGEROUS_WRITE_ABORTS=y
# CONFIG_SPI_FLASH_DANGEROUS_WRITE_FAILS is not set
# CONFIG_SPI_FLASH_DANGEROUS_WRITE_ALLOWED is not set
# CONFIG_SPI_FLASH_BYPASS_BLOCK_ERASE is not set
CONFIG_SPI_FLASH_YIELD_DURING_ERASE=y
CONFIG_SPI_FLASH_ERASE_YIELD_DURATION_MS=20
CONFIG_SPI_FLASH_ERASE_YIELD_TICKS=1
CONFIG_SPI_FLASH_WRITE_CHUNK_SIZE=8192
# CONFIG_SPI_FLASH_SIZE_OVERRIDE is not set
# CONFIG_SPI_FLASH_CHECK_ERASE_TIMEOUT_DISABLED is not set
# CONFIG_SPI_FLASH_OVERRIDE_CHIP_DRIVER_LIST is not set

#
# Auto-detect flash chips
#
CONFIG_SPI_FLASH_VENDOR_XMC_SUPPORTED=y
CONFIG_SPI_FLASH_VENDOR_GD_SUPPORTED=y
CONFIG_SPI_FLASH_VENDOR_ISSI_SUPPORTED=y
CONFIG_SPI_FLASH_VENDOR_MXIC_SUPPORTED=y
CONFIG_SPI_FLASH_VENDOR_WINBOND_SUPPORTED=y
CONFIG_SPI_FLASH_VENDOR_BOYA_SUPPORTED=y
CONFIG_SPI_FLASH_VENDOR_TH_SUPPORTED=y
CONFIG_SPI_FLASH_SUPPORT_ISSI_CHIP=y
CONFIG_SPI_FLASH_SUPPORT_MXIC_CHIP=y
CONFIG_SPI_FLASH_SUPPORT_GD_CHIP=y
CONFIG_SPI_FLASH_SUPPORT_WINBOND_CHIP=y
CONFIG_SPI_FLASH_SUPPORT_BOYA_CHIP=y
CONFIG_SPI_FLASH_SUPPORT_TH_CHIP=y
CONFIG_SPI_FLASH_SUPPORT_MXIC_OPI_CHIP=y
# end of Auto-detect flash chips

CONFIG_SPI_FLASH_ENABLE_ENCRYPTED_READ_WRITE=y
# end of SPI Flash driver

#
# SPIFFS Configuration
#
CONFIG_SPIFFS_MAX_PARTITIONS=3

#
# SPIFFS Cache Configuration
#
CONFIG_SPIFFS_CACHE=y
CONFIG_SPIFFS_CACHE_WR=y
# CONFIG_SPIFFS_CACHE_STATS is not set
# end of SPIFFS Cache Configuration

CONFIG_SPIFFS_PAGE_CHECK=y
CONFIG_SPIFFS_GC_MAX_RUNS=10
# CONFIG_SPIFFS_GC_STATS is not set
CONFIG_SPIFFS_PAGE_SIZE=256
CONFIG_SPIFFS_OBJ_NAME_LEN=32
# CONFIG_SPIFFS_FOLLOW_SYMLINKS is not set
CONFIG_SPIFFS_USE_MAGIC=y
CONFIG_SPIFFS_USE_MAGIC_LENGTH=y
CONFIG_SPIFFS_META_LENGTH=4
CONFIG_SPIFFS_USE_MTIME=y

#
# Debug Configuration
#
# CONFIG_SPIFFS_DBG is not set
# CONFIG_SPIFFS_API_DBG is not set
# CONFIG_SPIFFS_GC_DBG is not set
# CONFIG_SPIFFS_CACHE_DBG is not set
# CONFIG_SPIFFS_CHECK_DBG is not set
# CONFIG_SPIFFS_TEST_VISUALISATION is not set
# end of Debug Configuration
# end of SPIFFS Configuration

#
# TCP Transport
#

#
# Websocket
#
CONFIG_WS_TRANSPORT=y
CONFIG_WS_BUFFER_SIZE=1024
# CONFIG_WS_DYNAMIC_BUFFER is not set
# end of Websocket
# end of TCP Transport

#
# Ultra Low Power (ULP) Co-processor
#
# CONFIG_ULP_COPROC_ENABLED is not set

#
# ULP Debugging Options
#
# end of ULP Debugging Options
# end of Ultra Low Power (ULP) Co-processor

#
# Unity unit testing library
#
CONFIG_UNITY_ENABLE_FLOAT=y
CONFIG_UNITY_ENABLE_DOUBLE=y
# CONFIG_UNITY_ENABLE_64BIT is not set
# CONFIG_UNITY_ENABLE_COLOR is not set
CONFIG_UNITY_ENABLE_IDF_TEST_RUNNER=y
# CONFIG_UNITY_ENABLE_FIXTURE is not set
# CONFIG_UNITY_ENABLE_BACKTRACE_ON_FAIL is not set
# end of Unity unit testing library

#
# USB-OTG
#
CONFIG_USB_HOST_CONTROL_TRANSFER_MAX_SIZE=1536
CONFIG_USB_HOST_HW_BUFFER_BIAS_BALANCED=y
# CONFIG_USB_HOST_HW_BUFFER_BIAS_IN is not set
# CONFIG_USB_HOST_HW_BUFFER_BIAS_PERIODIC_OUT is not set

#
# Hub Driver Configuration
#

#
# Root Port configuration
#
CONFIG_USB_HOST_DEBOUNCE_DELAY_MS=250
CONFIG_USB_HOST_RESET_HOLD_MS=30
CONFIG_USB_HOST_RESET_RECOVERY_MS=30
CONFIG_USB_HOST_SET_ADDR_RECOVERY_MS=10
# end of Root Port configuration

# CONFIG_USB_HOST_HUBS_SUPPORTED is not set
# end of Hub Driver Configuration

# CONFIG_USB_HOST_ENABLE_ENUM_FILTER_CALLBACK is not set
CONFIG_USB_OTG_SUPPORTED=y
# end of USB-OTG

#
# Virtual file system
#
CONFIG_VFS_SUPPORT_IO=y
CONFIG_VFS_SUPPORT_DIR=y
CONFIG_VFS_SUPPORT_SELECT=y
CONFIG_VFS_SUPPRESS_SELECT_DEBUG_OUTPUT=y
CONFIG_VFS_SELECT_IN_RAM=y
CONFIG_VFS_SUPPORT_TERMIOS=y
CONFIG_VFS_MAX_COUNT=8

#
# Host File System I/O (Semihosting)
#
CONFIG_VFS_SEMIHOSTFS_MAX_MOUNT_POINTS=1
# end of Host File System I/O (Semihosting)

CONFIG_VFS_INITIALIZE_DEV_NULL=y
# end of Virtual file system

#
# Wear Levelling
#
# CONFIG_WL_SECTOR_SIZE_512 is not set
CONFIG_WL_SECTOR_SIZE_4096=y
CONFIG_WL_SECTOR_SIZE=4096
# end of Wear Levelling

#
# Wi-Fi Provisioning Manager
#
CONFIG_WIFI_PROV_SCAN_MAX_ENTRIES=16
CONFIG_WIFI_PROV_AUTOSTOP_TIMEOUT=30
CONFIG_WIFI_PROV_STA_ALL_CHANNEL_SCAN=y
# CONFIG_WIFI_PROV_STA_FAST_SCAN is not set
# end of Wi-Fi Provisioning Manager

#
# ESP Emote Graphics Framework
#
# CONFIG_GFX_FONT_FREETYPE_SUPPORT is not set
CONFIG_GFX_EAF_JPEG_DECODE_SUPPORT=y
# end of ESP Emote Graphics Framework

#
# Animation Player Configuration
#
CONFIG_ANIM_PLAYER_DEFAULT_FPS=30
# end of Animation Player Configuration

#
# ADC Battery Estimation
#
CONFIG_OCV_SOC_MODEL_1=y
# CONFIG_OCV_SOC_MODEL_2 is not set
CONFIG_ADC_FILTER_WINDOW_SIZE=10
CONFIG_BATTERY_CAPACITY_LPF_COEFFICIENT=2
CONFIG_BATTERY_STATE_SOFTWARE_ESTIMATION=y

#
# Software Estimation Configuration
#
CONFIG_SOFTWARE_ESTIMATION_SAMPLE_COUNT=10
CONFIG_SOFTWARE_ESTIMATION_SAMPLE_INTERVAL=20000
# end of Software Estimation Configuration
# end of ADC Battery Estimation

#
# ADC Mic
#
CONFIG_ADC_MIC_APPLY_GAIN=3
CONFIG_ADC_MIC_OFFSET=16380
CONFIG_ADC_MIC_TASK_PRIORITY=1
CONFIG_ADC_MIC_TASK_STACK_SIZE=3072
CONFIG_ADC_MIC_TASK_CORE=-1
# end of ADC Mic

#
# IoT Button
#
CONFIG_BUTTON_PERIOD_TIME_MS=5
CONFIG_BUTTON_DEBOUNCE_TICKS=2
CONFIG_BUTTON_SHORT_PRESS_TIME_MS=180
CONFIG_BUTTON_LONG_PRESS_TIME_MS=1500
CONFIG_BUTTON_LONG_PRESS_HOLD_SERIAL_TIME_MS=20
CONFIG_ADC_BUTTON_MAX_CHANNEL=3
CONFIG_ADC_BUTTON_MAX_BUTTON_PER_CHANNEL=8
CONFIG_ADC_BUTTON_SAMPLE_TIMES=1
# end of IoT Button

#
# CMake Utilities
#
# CONFIG_CU_RELINKER_ENABLE is not set
# CONFIG_CU_DIAGNOSTICS_COLOR_NEVER is not set
CONFIG_CU_DIAGNOSTICS_COLOR_ALWAYS=y
# CONFIG_CU_DIAGNOSTICS_COLOR_AUTO is not set
# CONFIG_CU_GCC_LTO_ENABLE is not set
# CONFIG_CU_GCC_STRING_1BYTE_ALIGN is not set
# end of CMake Utilities

#
# DSP Library
#
CONFIG_DSP_OPTIMIZATIONS_SUPPORTED=y
# CONFIG_DSP_ANSI is not set
CONFIG_DSP_OPTIMIZED=y
CONFIG_DSP_OPTIMIZATION=1
# CONFIG_DSP_MAX_FFT_SIZE_512 is not set
# CONFIG_DSP_MAX_FFT_SIZE_1024 is not set
# CONFIG_DSP_MAX_FFT_SIZE_2048 is not set
CONFIG_DSP_MAX_FFT_SIZE_4096=y
# CONFIG_DSP_MAX_FFT_SIZE_8192 is not set
# CONFIG_DSP_MAX_FFT_SIZE_16384 is not set
# CONFIG_DSP_MAX_FFT_SIZE_32768 is not set
CONFIG_DSP_MAX_FFT_SIZE=4096
# end of DSP Library

#
# Audio Codec Configuration
#

#
# Audio Decoder Configuration
#
CONFIG_AUDIO_DECODER_AAC_SUPPORT=y
CONFIG_AUDIO_DECODER_MP3_SUPPORT=y
CONFIG_AUDIO_DECODER_G711_SUPPORT=y
CONFIG_AUDIO_DECODER_AMRNB_SUPPORT=y
CONFIG_AUDIO_DECODER_AMRWB_SUPPORT=y
CONFIG_AUDIO_DECODER_FLAC_SUPPORT=y
CONFIG_AUDIO_DECODER_OPUS_SUPPORT=y
CONFIG_AUDIO_DECODER_VORBIS_SUPPORT=y
CONFIG_AUDIO_DECODER_ADPCM_SUPPORT=y
CONFIG_AUDIO_DECODER_ALAC_SUPPORT=y
CONFIG_AUDIO_DECODER_PCM_SUPPORT=y
CONFIG_AUDIO_DECODER_SBC_SUPPORT=y
CONFIG_AUDIO_DECODER_LC3_SUPPORT=y
# end of Audio Decoder Configuration

#
# Audio Simple Decoder Configuration
#
CONFIG_AUDIO_SIMPLE_DEC_WAV_SUPPORT=y
CONFIG_AUDIO_SIMPLE_DEC_M4A_SUPPORT=y
CONFIG_AUDIO_SIMPLE_DEC_TS_SUPPORT=y
# end of Audio Simple Decoder Configuration

#
# Audio Encoder Configuration
#
CONFIG_AUDIO_ENCODER_AAC_SUPPORT=y
CONFIG_AUDIO_ENCODER_G711_SUPPORT=y
CONFIG_AUDIO_ENCODER_AMRNB_SUPPORT=y
CONFIG_AUDIO_ENCODER_AMRWB_SUPPORT=y
CONFIG_AUDIO_ENCODER_OPUS_SUPPORT=y
CONFIG_AUDIO_ENCODER_ADPCM_SUPPORT=y
CONFIG_AUDIO_ENCODER_ALAC_SUPPORT=y
CONFIG_AUDIO_ENCODER_PCM_SUPPORT=y
CONFIG_AUDIO_ENCODER_SBC_SUPPORT=y
CONFIG_AUDIO_ENCODER_LC3_SUPPORT=y
# end of Audio Encoder Configuration
# end of Audio Codec Configuration

#
# Espressif Camera Sensors Configurations
#

#
# Camera Sensor Configuration
#

#
# Select and Set Camera Sensor
#
# CONFIG_CAMERA_BF3901 is not set
# CONFIG_CAMERA_BF3925 is not set
# CONFIG_CAMERA_BF3A03 is not set
# CONFIG_CAMERA_GC0308 is not set
# CONFIG_CAMERA_GC2145 is not set
# CONFIG_CAMERA_OV2640 is not set
# CONFIG_CAMERA_OV3660 is not set
# CONFIG_CAMERA_OV5640 is not set
# CONFIG_CAMERA_SC030IOT is not set
# CONFIG_CAMERA_SC101IOT is not set
# CONFIG_CAMERA_SC2336 is not set
# end of Select and Set Camera Sensor
# end of Camera Sensor Configuration

#
# Camera XCLK Generator Configuration
#
CONFIG_CAMERA_XCLK_USE_LEDC=y
# end of Camera XCLK Generator Configuration

#
# Camera Motor Control Configuration
#
# CONFIG_CAM_MOTOR_DW9714 is not set
# end of Camera Motor Control Configuration

#
# Camera SPI Controller Configuration
#
CONFIG_CAM_CTLR_SPI_ISR_CACHE_SAFE=y
# CONFIG_CAM_CTLR_SPI_DISABLE_BACKUP_BUFFER is not set
# CONFIG_CAM_CTLR_SPI_DISABLE_AUTO_DECODE is not set
CONFIG_CAM_CTLR_SPI_DECODE_TASK_STACK_SIZE=4096
CONFIG_CAM_CTLR_SPI_DECODE_TASK_PRIORITY=10
# end of Camera SPI Controller Configuration
# end of Espressif Camera Sensors Configurations

#
# Audio Codec Device Configuration
#
# CONFIG_CODEC_I2C_BACKWARD_COMPATIBLE is not set
CONFIG_CODEC_ES8311_SUPPORT=y
CONFIG_CODEC_ES7210_SUPPORT=y
CONFIG_CODEC_ES7243_SUPPORT=y
CONFIG_CODEC_ES7243E_SUPPORT=y
CONFIG_CODEC_ES8156_SUPPORT=y
CONFIG_CODEC_AW88298_SUPPORT=y
CONFIG_CODEC_ES8389_SUPPORT=y
CONFIG_CODEC_ES8374_SUPPORT=y
CONFIG_CODEC_ES8388_SUPPORT=y
CONFIG_CODEC_TAS5805M_SUPPORT=y
# CONFIG_CODEC_ZL38063_SUPPORT is not set
# CONFIG_CODEC_CJC8910_SUPPORT is not set
# end of Audio Codec Device Configuration

#
# ESP IO Expander
#
# CONFIG_IO_EXPANDER_ENABLE_GPIO_API_WRAPPER is not set
# end of ESP IO Expander

#
# ESP LCD TOUCH
#
CONFIG_ESP_LCD_TOUCH_MAX_POINTS=5
CONFIG_ESP_LCD_TOUCH_MAX_BUTTONS=1
# end of ESP LCD TOUCH

#
# ESP LCD TOUCH - CST816S
#
# CONFIG_ESP_LCD_TOUCH_CST816S_DISABLE_READ_ID is not set
# end of ESP LCD TOUCH - CST816S

#
# ESP LVGL PORT
#
# end of ESP LVGL PORT

#
# mmap file support format
#
CONFIG_MMAP_FILE_NAME_LENGTH=16
# end of mmap file support format

#
# Espressif SCCB Configurations
#
CONFIG_ESP_SCCB_TRANS_TIMEOUT_DEFAULT=500
# end of Espressif SCCB Configurations

#
# Espressif Video Configuration
#
CONFIG_ESP_VIDEO_CHECK_PARAMETERS=y
CONFIG_ESP_VIDEO_ENABLE_DVP_VIDEO_DEVICE=y
# CONFIG_ESP_VIDEO_ENABLE_SPI_VIDEO_DEVICE is not set
# CONFIG_ESP_VIDEO_ENABLE_USB_UVC_VIDEO_DEVICE is not set

#
# Video Data Preprocessing
#
# end of Video Data Preprocessing
# end of Espressif Video Configuration

#
# Bus Options
#

#
# I2C Bus Options
#
CONFIG_I2C_BUS_DYNAMIC_CONFIG=y
CONFIG_I2C_MS_TO_WAIT=200
# CONFIG_I2C_BUS_BACKWARD_CONFIG is not set
# CONFIG_I2C_BUS_SUPPORT_SOFTWARE is not set
# CONFIG_I2C_BUS_REMOVE_NULL_MEM_ADDR is not set
# end of I2C Bus Options
# end of Bus Options

#
# IOT Knob
#
CONFIG_KNOB_PERIOD_TIME_MS=3
CONFIG_KNOB_DEBOUNCE_TICKS=2
CONFIG_KNOB_HIGH_LIMIT=1000
CONFIG_KNOB_LOW_LIMIT=-1000
# end of IOT Knob

#
# USB HOST UVC
#
# CONFIG_UVC_PRINTF_CONFIGURATION_DESCRIPTOR is not set
CONFIG_UVC_INTERVAL_ARRAY_SIZE=3
# end of USB HOST UVC

#
# LVGL configuration
#
CONFIG_LV_CONF_SKIP=y
# CONFIG_LV_CONF_MINIMAL is not set

#
# Color Settings
#
# CONFIG_LV_COLOR_DEPTH_32 is not set
# CONFIG_LV_COLOR_DEPTH_24 is not set
CONFIG_LV_COLOR_DEPTH_16=y
# CONFIG_LV_COLOR_DEPTH_8 is not set
# CONFIG_LV_COLOR_DEPTH_1 is not set
CONFIG_LV_COLOR_DEPTH=16
# end of Color Settings

#
# Memory Settings
#
# CONFIG_LV_USE_BUILTIN_MALLOC is not set
CONFIG_LV_USE_CLIB_MALLOC=y
# CONFIG_LV_USE_MICROPYTHON_MALLOC is not set
# CONFIG_LV_USE_RTTHREAD_MALLOC is not set
# CONFIG_LV_USE_CUSTOM_MALLOC is not set
# CONFIG_LV_USE_BUILTIN_STRING is not set
CONFIG_LV_USE_CLIB_STRING=y
# CONFIG_LV_USE_CUSTOM_STRING is not set
# CONFIG_LV_USE_BUILTIN_SPRINTF is not set
CONFIG_LV_USE_CLIB_SPRINTF=y
# CONFIG_LV_USE_CUSTOM_SPRINTF is not set
# end of Memory Settings

#
# HAL Settings
#
CONFIG_LV_DEF_REFR_PERIOD=33
CONFIG_LV_DPI_DEF=130
# end of HAL Settings

#
# Operating System (OS)
#
CONFIG_LV_OS_NONE=y
# CONFIG_LV_OS_PTHREAD is not set
# CONFIG_LV_OS_FREERTOS is not set
# CONFIG_LV_OS_CMSIS_RTOS2 is not set
# CONFIG_LV_OS_RTTHREAD is not set
# CONFIG_LV_OS_WINDOWS is not set
# CONFIG_LV_OS_MQX is not set
# CONFIG_LV_OS_CUSTOM is not set
# end of Operating System (OS)

#
# Rendering Configuration
#
CONFIG_LV_DRAW_BUF_STRIDE_ALIGN=1
CONFIG_LV_DRAW_BUF_ALIGN=4
CONFIG_LV_DRAW_LAYER_SIMPLE_BUF_SIZE=24576
CONFIG_LV_DRAW_LAYER_MAX_MEMORY=0
CONFIG_LV_DRAW_THREAD_STACK_SIZE=8192
CONFIG_LV_DRAW_THREAD_PRIO=3
CONFIG_LV_USE_DRAW_SW=y
CONFIG_LV_DRAW_SW_SUPPORT_RGB565=y
CONFIG_LV_DRAW_SW_SUPPORT_RGB565A8=y
CONFIG_LV_DRAW_SW_SUPPORT_RGB888=y
CONFIG_LV_DRAW_SW_SUPPORT_XRGB8888=y
CONFIG_LV_DRAW_SW_SUPPORT_ARGB8888=y
CONFIG_LV_DRAW_SW_SUPPORT_L8=y
CONFIG_LV_DRAW_SW_SUPPORT_AL88=y
CONFIG_LV_DRAW_SW_SUPPORT_A8=y
CONFIG_LV_DRAW_SW_SUPPORT_I1=y
CONFIG_LV_DRAW_SW_I1_LUM_THRESHOLD=127
CONFIG_LV_DRAW_SW_DRAW_UNIT_CNT=1
# CONFIG_LV_USE_DRAW_ARM2D_SYNC is not set
# CONFIG_LV_USE_NATIVE_HELIUM_ASM is not set
CONFIG_LV_DRAW_SW_COMPLEX=y
# CONFIG_LV_USE_DRAW_SW_COMPLEX_GRADIENTS is not set
CONFIG_LV_DRAW_SW_SHADOW_CACHE_SIZE=0
CONFIG_LV_DRAW_SW_CIRCLE_CACHE_SIZE=4
CONFIG_LV_DRAW_SW_ASM_NONE=y
# CONFIG_LV_DRAW_SW_ASM_NEON is not set
# CONFIG_LV_DRAW_SW_ASM_HELIUM is not set
# CONFIG_LV_DRAW_SW_ASM_CUSTOM is not set
CONFIG_LV_USE_DRAW_SW_ASM=0
# CONFIG_LV_USE_DRAW_VGLITE is not set
# CONFIG_LV_USE_PXP is not set
# CONFIG_LV_USE_DRAW_G2D is not set
# CONFIG_LV_USE_DRAW_DAVE2D is not set
# CONFIG_LV_USE_DRAW_SDL is not set
# CONFIG_LV_USE_DRAW_VG_LITE is not set
# CONFIG_LV_USE_VECTOR_GRAPHIC is not set
# CONFIG_LV_USE_DRAW_DMA2D is not set
# end of Rendering Configuration

#
# Feature Configuration
#

#
# Logging
#
# CONFIG_LV_USE_LOG is not set
# end of Logging

#
# Asserts
#
CONFIG_LV_USE_ASSERT_NULL=y
CONFIG_LV_USE_ASSERT_MALLOC=y
CONFIG_LV_USE_ASSERT_STYLE=y
# CONFIG_LV_USE_ASSERT_MEM_INTEGRITY is not set
# CONFIG_LV_USE_ASSERT_OBJ is not set
CONFIG_LV_ASSERT_HANDLER_INCLUDE="assert.h"
# end of Asserts

#
# Debug
#
# CONFIG_LV_USE_REFR_DEBUG is not set
# CONFIG_LV_USE_LAYER_DEBUG is not set
# CONFIG_LV_USE_PARALLEL_DRAW_DEBUG is not set
# end of Debug

#
# Others
#
# CONFIG_LV_ENABLE_GLOBAL_CUSTOM is not set
CONFIG_LV_CACHE_DEF_SIZE=0
CONFIG_LV_IMAGE_HEADER_CACHE_DEF_CNT=0
CONFIG_LV_GRADIENT_MAX_STOPS=2
CONFIG_LV_COLOR_MIX_ROUND_OFS=128
# CONFIG_LV_OBJ_STYLE_CACHE is not set
# CONFIG_LV_USE_OBJ_ID is not set
# CONFIG_LV_USE_OBJ_NAME is not set
# CONFIG_LV_USE_OBJ_PROPERTY is not set
# end of Others
# end of Feature Configuration

#
# Compiler Settings
#
# CONFIG_LV_BIG_ENDIAN_SYSTEM is not set
CONFIG_LV_ATTRIBUTE_MEM_ALIGN_SIZE=1
# CONFIG_LV_ATTRIBUTE_FAST_MEM_USE_IRAM is not set
# CONFIG_LV_USE_FLOAT is not set
# CONFIG_LV_USE_MATRIX is not set
# CONFIG_LV_USE_PRIVATE_API is not set
# end of Compiler Settings

#
# Font Usage
#

#
# Enable built-in fonts
#
# CONFIG_LV_FONT_MONTSERRAT_8 is not set
# CONFIG_LV_FONT_MONTSERRAT_10 is not set
# CONFIG_LV_FONT_MONTSERRAT_12 is not set
CONFIG_LV_FONT_MONTSERRAT_14=y
# CONFIG_LV_FONT_MONTSERRAT_16 is not set
# CONFIG_LV_FONT_MONTSERRAT_18 is not set
# CONFIG_LV_FONT_MONTSERRAT_20 is not set
# CONFIG_LV_FONT_MONTSERRAT_22 is not set
# CONFIG_LV_FONT_MONTSERRAT_24 is not set
# CONFIG_LV_FONT_MONTSERRAT_26 is not set
# CONFIG_LV_FONT_MONTSERRAT_28 is not set
# CONFIG_LV_FONT_MONTSERRAT_30 is not set
# CONFIG_LV_FONT_MONTSERRAT_32 is not set
# CONFIG_LV_FONT_MONTSERRAT_34 is not set
# CONFIG_LV_FONT_MONTSERRAT_36 is not set
# CONFIG_LV_FONT_MONTSERRAT_38 is not set
# CONFIG_LV_FONT_MONTSERRAT_40 is not set
# CONFIG_LV_FONT_MONTSERRAT_42 is not set
# CONFIG_LV_FONT_MONTSERRAT_44 is not set
# CONFIG_LV_FONT_MONTSERRAT_46 is not set
# CONFIG_LV_FONT_MONTSERRAT_48 is not set
# CONFIG_LV_FONT_MONTSERRAT_28_COMPRESSED is not set
# CONFIG_LV_FONT_DEJAVU_16_PERSIAN_HEBREW is not set
# CONFIG_LV_FONT_SIMSUN_14_CJK is not set
# CONFIG_LV_FONT_SIMSUN_16_CJK is not set
# CONFIG_LV_FONT_SOURCE_HAN_SANS_SC_14_CJK is not set
# CONFIG_LV_FONT_SOURCE_HAN_SANS_SC_16_CJK is not set
# CONFIG_LV_FONT_UNSCII_8 is not set
# CONFIG_LV_FONT_UNSCII_16 is not set
# end of Enable built-in fonts

# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_8 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_10 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_12 is not set
CONFIG_LV_FONT_DEFAULT_MONTSERRAT_14=y
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_16 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_18 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_20 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_22 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_24 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_26 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_28 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_30 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_32 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_34 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_36 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_38 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_40 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_42 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_44 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_46 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_48 is not set
# CONFIG_LV_FONT_DEFAULT_MONTSERRAT_28_COMPRESSED is not set
# CONFIG_LV_FONT_DEFAULT_DEJAVU_16_PERSIAN_HEBREW is not set
# CONFIG_LV_FONT_DEFAULT_SIMSUN_14_CJK is not set
# CONFIG_LV_FONT_DEFAULT_SIMSUN_16_CJK is not set
# CONFIG_LV_FONT_DEFAULT_SOURCE_HAN_SANS_SC_14_CJK is not set
# CONFIG_LV_FONT_DEFAULT_SOURCE_HAN_SANS_SC_16_CJK is not set
# CONFIG_LV_FONT_DEFAULT_UNSCII_8 is not set
# CONFIG_LV_FONT_DEFAULT_UNSCII_16 is not set
CONFIG_LV_FONT_FMT_TXT_LARGE=y
# CONFIG_LV_USE_FONT_COMPRESSED is not set
# CONFIG_LV_USE_FONT_PLACEHOLDER is not set

#
# Enable static fonts
#
# end of Enable static fonts
# end of Font Usage

#
# Text Settings
#
CONFIG_LV_TXT_ENC_UTF8=y
# CONFIG_LV_TXT_ENC_ASCII is not set
CONFIG_LV_TXT_BREAK_CHARS=" ,.;:-_)}"
CONFIG_LV_TXT_LINE_BREAK_LONG_LEN=0
CONFIG_LV_TXT_COLOR_CMD="#"
# CONFIG_LV_USE_BIDI is not set
# CONFIG_LV_USE_ARABIC_PERSIAN_CHARS is not set
# end of Text Settings

#
# Widget Usage
#
CONFIG_LV_WIDGETS_HAS_DEFAULT_VALUE=y
# CONFIG_LV_USE_ANIMIMG is not set
CONFIG_LV_USE_ARC=y
CONFIG_LV_USE_BAR=y
CONFIG_LV_USE_BUTTON=y
CONFIG_LV_USE_BUTTONMATRIX=y
# CONFIG_LV_USE_CALENDAR is not set
CONFIG_LV_USE_CANVAS=y
# CONFIG_LV_USE_CHART is not set
CONFIG_LV_USE_CHECKBOX=y
CONFIG_LV_USE_DROPDOWN=y
CONFIG_LV_USE_IMAGE=y
CONFIG_LV_USE_IMAGEBUTTON=y
# CONFIG_LV_USE_KEYBOARD is not set
CONFIG_LV_USE_LABEL=y
CONFIG_LV_LABEL_TEXT_SELECTION=y
CONFIG_LV_LABEL_LONG_TXT_HINT=y
CONFIG_LV_LABEL_WAIT_CHAR_COUNT=3
# CONFIG_LV_USE_LED is not set
CONFIG_LV_USE_LINE=y
# CONFIG_LV_USE_LIST is not set
# CONFIG_LV_USE_MENU is not set
# CONFIG_LV_USE_MSGBOX is not set
CONFIG_LV_USE_ROLLER=y
CONFIG_LV_USE_SCALE=y
CONFIG_LV_USE_SLIDER=y
# CONFIG_LV_USE_SPAN is not set
# CONFIG_LV_USE_SPINBOX is not set
# CONFIG_LV_USE_SPINNER is not set
CONFIG_LV_USE_SWITCH=y
CONFIG_LV_USE_TEXTAREA=y
CONFIG_LV_TEXTAREA_DEF_PWD_SHOW_TIME=1500
CONFIG_LV_USE_TABLE=y
# CONFIG_LV_USE_TABVIEW is not set
# CONFIG_LV_USE_TILEVIEW is not set
# CONFIG_LV_USE_WIN is not set
# end of Widget Usage

#
# Themes
#
CONFIG_LV_USE_THEME_DEFAULT=y
# CONFIG_LV_THEME_DEFAULT_DARK is not set
CONFIG_LV_THEME_DEFAULT_GROW=y
CONFIG_LV_THEME_DEFAULT_TRANSITION_TIME=80
CONFIG_LV_USE_THEME_SIMPLE=y
# CONFIG_LV_USE_THEME_MONO is not set
# end of Themes

#
# Layouts
#
CONFIG_LV_USE_FLEX=y
CONFIG_LV_USE_GRID=y
# end of Layouts

#
# 3rd Party Libraries
#
CONFIG_LV_FS_DEFAULT_DRIVER_LETTER=0
# CONFIG_LV_USE_FS_STDIO is not set
# CONFIG_LV_USE_FS_POSIX is not set
# CONFIG_LV_USE_FS_WIN32 is not set
# CONFIG_LV_USE_FS_FATFS is not set
# CONFIG_LV_USE_FS_MEMFS is not set
# CONFIG_LV_USE_FS_LITTLEFS is not set
# CONFIG_LV_USE_FS_ARDUINO_ESP_LITTLEFS is not set
# CONFIG_LV_USE_FS_ARDUINO_SD is not set
# CONFIG_LV_USE_FS_UEFI is not set
CONFIG_LV_USE_LODEPNG=y
# CONFIG_LV_USE_LIBPNG is not set
# CONFIG_LV_USE_BMP is not set
# CONFIG_LV_USE_TJPGD is not set
# CONFIG_LV_USE_LIBJPEG_TURBO is not set
# CONFIG_LV_USE_GIF is not set
# CONFIG_LV_BIN_DECODER_RAM_LOAD is not set
# CONFIG_LV_USE_RLE is not set
# CONFIG_LV_USE_QRCODE is not set
# CONFIG_LV_USE_BARCODE is not set
# CONFIG_LV_USE_FREETYPE is not set
# CONFIG_LV_USE_TINY_TTF is not set
# CONFIG_LV_USE_RLOTTIE is not set
# CONFIG_LV_USE_THORVG is not set
# CONFIG_LV_USE_LZ4 is not set
# CONFIG_LV_USE_FFMPEG is not set
# end of 3rd Party Libraries

#
# Others
#
CONFIG_LV_USE_SNAPSHOT=y
# CONFIG_LV_USE_SYSMON is not set
# CONFIG_LV_USE_PROFILER is not set
# CONFIG_LV_USE_MONKEY is not set
# CONFIG_LV_USE_GRIDNAV is not set
# CONFIG_LV_USE_FRAGMENT is not set
CONFIG_LV_USE_IMGFONT=y
CONFIG_LV_USE_OBSERVER=y
# CONFIG_LV_USE_IME_PINYIN is not set
# CONFIG_LV_USE_FILE_EXPLORER is not set
# CONFIG_LV_USE_FONT_MANAGER is not set
# CONFIG_LV_USE_TEST is not set
# CONFIG_LV_USE_XML is not set
# CONFIG_LV_USE_COLOR_FILTER is not set
CONFIG_LVGL_VERSION_MAJOR=9
CONFIG_LVGL_VERSION_MINOR=3
CONFIG_LVGL_VERSION_PATCH=0
# end of Others

#
# Devices
#
# CONFIG_LV_USE_SDL is not set
# CONFIG_LV_USE_X11 is not set
# CONFIG_LV_USE_WAYLAND is not set
# CONFIG_LV_USE_LINUX_FBDEV is not set
# CONFIG_LV_USE_NUTTX is not set
# CONFIG_LV_USE_LINUX_DRM is not set
# CONFIG_LV_USE_TFT_ESPI is not set
# CONFIG_LV_USE_EVDEV is not set
# CONFIG_LV_USE_LIBINPUT is not set
# CONFIG_LV_USE_ST7735 is not set
# CONFIG_LV_USE_ST7789 is not set
# CONFIG_LV_USE_ST7796 is not set
# CONFIG_LV_USE_ILI9341 is not set
# CONFIG_LV_USE_GENERIC_MIPI is not set
# CONFIG_LV_USE_RENESAS_GLCDC is not set
# CONFIG_LV_USE_ST_LTDC is not set
# CONFIG_LV_USE_FT81X is not set
# CONFIG_LV_USE_UEFI is not set
# CONFIG_LV_USE_OPENGLES is not set
# CONFIG_LV_USE_QNX is not set
# end of Devices

#
# Examples
#
# CONFIG_LV_BUILD_EXAMPLES is not set
# end of Examples

#
# Demos
#
# CONFIG_LV_BUILD_DEMOS is not set
# end of Demos
# end of LVGL configuration

#
# SH1106 ESP-IDF Driver
#
# end of Component config

# CONFIG_IDF_EXPERIMENTAL_FEATURES is not set

# Deprecated options for backward compatibility
# CONFIG_APP_BUILD_TYPE_ELF_RAM is not set
# CONFIG_NO_BLOBS is not set
CONFIG_LOG_BOOTLOADER_LEVEL_NONE=y
# CONFIG_LOG_BOOTLOADER_LEVEL_ERROR is not set
# CONFIG_LOG_BOOTLOADER_LEVEL_WARN is not set
# CONFIG_LOG_BOOTLOADER_LEVEL_INFO is not set
# CONFIG_LOG_BOOTLOADER_LEVEL_DEBUG is not set
# CONFIG_LOG_BOOTLOADER_LEVEL_VERBOSE is not set
CONFIG_LOG_BOOTLOADER_LEVEL=0
CONFIG_APP_ROLLBACK_ENABLE=y
# CONFIG_APP_ANTI_ROLLBACK is not set
# CONFIG_FLASH_ENCRYPTION_ENABLED is not set
CONFIG_FLASHMODE_QIO=y
# CONFIG_FLASHMODE_QOUT is not set
# CONFIG_FLASHMODE_DIO is not set
# CONFIG_FLASHMODE_DOUT is not set
CONFIG_MONITOR_BAUD=115200
# CONFIG_OPTIMIZATION_LEVEL_DEBUG is not set
# CONFIG_COMPILER_OPTIMIZATION_LEVEL_DEBUG is not set
# CONFIG_COMPILER_OPTIMIZATION_DEFAULT is not set
CONFIG_OPTIMIZATION_LEVEL_RELEASE=y
CONFIG_COMPILER_OPTIMIZATION_LEVEL_RELEASE=y
CONFIG_OPTIMIZATION_ASSERTIONS_ENABLED=y
# CONFIG_OPTIMIZATION_ASSERTIONS_SILENT is not set
# CONFIG_OPTIMIZATION_ASSERTIONS_DISABLED is not set
CONFIG_OPTIMIZATION_ASSERTION_LEVEL=2
CONFIG_CXX_EXCEPTIONS=y
CONFIG_CXX_EXCEPTIONS_EMG_POOL_SIZE=1024
CONFIG_STACK_CHECK_NONE=y
# CONFIG_STACK_CHECK_NORM is not set
# CONFIG_STACK_CHECK_STRONG is not set
# CONFIG_STACK_CHECK_ALL is not set
# CONFIG_WARN_WRITE_STRINGS is not set
# CONFIG_ESP32_APPTRACE_DEST_TRAX is not set
CONFIG_ESP32_APPTRACE_DEST_NONE=y
CONFIG_ESP32_APPTRACE_LOCK_ENABLE=y
# CONFIG_EXTERNAL_COEX_ENABLE is not set
# CONFIG_ESP_WIFI_EXTERNAL_COEXIST_ENABLE is not set
# CONFIG_CAM_CTLR_DVP_CAM_ISR_IRAM_SAFE is not set
# CONFIG_MCPWM_ISR_IN_IRAM is not set
# CONFIG_EVENT_LOOP_PROFILING is not set
CONFIG_POST_EVENTS_FROM_ISR=y
CONFIG_POST_EVENTS_FROM_IRAM_ISR=y
CONFIG_GDBSTUB_SUPPORT_TASKS=y
CONFIG_GDBSTUB_MAX_TASKS=32
# CONFIG_OTA_ALLOW_HTTP is not set
CONFIG_ESP32S3_DEEP_SLEEP_WAKEUP_DELAY=2000
CONFIG_ESP_SLEEP_DEEP_SLEEP_WAKEUP_DELAY=2000
CONFIG_ESP32S3_RTC_CLK_SRC_INT_RC=y
# CONFIG_ESP32S3_RTC_CLK_SRC_EXT_CRYS is not set
# CONFIG_ESP32S3_RTC_CLK_SRC_EXT_OSC is not set
# CONFIG_ESP32S3_RTC_CLK_SRC_INT_8MD256 is not set
CONFIG_ESP32S3_RTC_CLK_CAL_CYCLES=1024
CONFIG_ESP32_PHY_CALIBRATION_AND_DATA_STORAGE=y
# CONFIG_ESP32_PHY_INIT_DATA_IN_PARTITION is not set
CONFIG_ESP32_PHY_MAX_WIFI_TX_POWER=20
CONFIG_ESP32_PHY_MAX_TX_POWER=20
# CONFIG_REDUCE_PHY_TX_POWER is not set
# CONFIG_ESP32_REDUCE_PHY_TX_POWER is not set
CONFIG_ESP_SYSTEM_PM_POWER_DOWN_CPU=y
CONFIG_PM_POWER_DOWN_TAGMEM_IN_LIGHT_SLEEP=y
CONFIG_ESP32S3_SPIRAM_SUPPORT=y
CONFIG_DEFAULT_PSRAM_CLK_IO=30
CONFIG_DEFAULT_PSRAM_CS_IO=26
# CONFIG_ESP32S3_DEFAULT_CPU_FREQ_80 is not set
# CONFIG_ESP32S3_DEFAULT_CPU_FREQ_160 is not set
CONFIG_ESP32S3_DEFAULT_CPU_FREQ_240=y
CONFIG_ESP32S3_DEFAULT_CPU_FREQ_MHZ=240
CONFIG_SYSTEM_EVENT_QUEUE_SIZE=32
CONFIG_SYSTEM_EVENT_TASK_STACK_SIZE=4096
CONFIG_MAIN_TASK_STACK_SIZE=8192
CONFIG_CONSOLE_UART_DEFAULT=y
# CONFIG_CONSOLE_UART_CUSTOM is not set
# CONFIG_CONSOLE_UART_NONE is not set
# CONFIG_ESP_CONSOLE_UART_NONE is not set
CONFIG_CONSOLE_UART=y
CONFIG_CONSOLE_UART_NUM=0
CONFIG_CONSOLE_UART_BAUDRATE=115200
CONFIG_INT_WDT=y
CONFIG_INT_WDT_TIMEOUT_MS=300
CONFIG_INT_WDT_CHECK_CPU1=y
CONFIG_TASK_WDT=y
CONFIG_ESP_TASK_WDT=y
# CONFIG_TASK_WDT_PANIC is not set
CONFIG_TASK_WDT_TIMEOUT_S=10
CONFIG_TASK_WDT_CHECK_IDLE_TASK_CPU0=y
CONFIG_TASK_WDT_CHECK_IDLE_TASK_CPU1=y
# CONFIG_ESP32_DEBUG_STUBS_ENABLE is not set
CONFIG_ESP32S3_DEBUG_OCDAWARE=y
CONFIG_BROWNOUT_DET=y
CONFIG_ESP32S3_BROWNOUT_DET=y
CONFIG_BROWNOUT_DET_LVL_SEL_7=y
CONFIG_ESP32S3_BROWNOUT_DET_LVL_SEL_7=y
# CONFIG_BROWNOUT_DET_LVL_SEL_6 is not set
# CONFIG_ESP32S3_BROWNOUT_DET_LVL_SEL_6 is not set
# CONFIG_BROWNOUT_DET_LVL_SEL_5 is not set
# CONFIG_ESP32S3_BROWNOUT_DET_LVL_SEL_5 is not set
# CONFIG_BROWNOUT_DET_LVL_SEL_4 is not set
# CONFIG_ESP32S3_BROWNOUT_DET_LVL_SEL_4 is not set
# CONFIG_BROWNOUT_DET_LVL_SEL_3 is not set
# CONFIG_ESP32S3_BROWNOUT_DET_LVL_SEL_3 is not set
# CONFIG_BROWNOUT_DET_LVL_SEL_2 is not set
# CONFIG_ESP32S3_BROWNOUT_DET_LVL_SEL_2 is not set
# CONFIG_BROWNOUT_DET_LVL_SEL_1 is not set
# CONFIG_ESP32S3_BROWNOUT_DET_LVL_SEL_1 is not set
CONFIG_BROWNOUT_DET_LVL=7
CONFIG_ESP32S3_BROWNOUT_DET_LVL=7
CONFIG_IPC_TASK_STACK_SIZE=1280
CONFIG_TIMER_TASK_STACK_SIZE=3584
CONFIG_ESP32_WIFI_ENABLED=y
CONFIG_ESP32_WIFI_STATIC_RX_BUFFER_NUM=3
CONFIG_ESP32_WIFI_DYNAMIC_RX_BUFFER_NUM=6
# CONFIG_ESP32_WIFI_STATIC_TX_BUFFER is not set
CONFIG_ESP32_WIFI_DYNAMIC_TX_BUFFER=y
CONFIG_ESP32_WIFI_TX_BUFFER_TYPE=1
CONFIG_ESP32_WIFI_DYNAMIC_TX_BUFFER_NUM=32
# CONFIG_ESP32_WIFI_CSI_ENABLED is not set
CONFIG_ESP32_WIFI_AMPDU_TX_ENABLED=y
CONFIG_ESP32_WIFI_TX_BA_WIN=6
CONFIG_ESP32_WIFI_AMPDU_RX_ENABLED=y
CONFIG_ESP32_WIFI_RX_BA_WIN=3
CONFIG_ESP32_WIFI_NVS_ENABLED=y
CONFIG_ESP32_WIFI_TASK_PINNED_TO_CORE_0=y
# CONFIG_ESP32_WIFI_TASK_PINNED_TO_CORE_1 is not set
CONFIG_ESP32_WIFI_SOFTAP_BEACON_MAX_LEN=752
CONFIG_ESP32_WIFI_MGMT_SBUF_NUM=32
# CONFIG_ESP32_WIFI_IRAM_OPT is not set
# CONFIG_ESP32_WIFI_RX_IRAM_OPT is not set
CONFIG_ESP32_WIFI_ENABLE_WPA3_SAE=y
CONFIG_ESP32_WIFI_ENABLE_WPA3_OWE_STA=y
CONFIG_WPA_MBEDTLS_CRYPTO=y
# CONFIG_WPA_WAPI_PSK is not set
# CONFIG_WPA_11KV_SUPPORT is not set
# CONFIG_WPA_MBO_SUPPORT is not set
# CONFIG_WPA_DPP_SUPPORT is not set
# CONFIG_WPA_11R_SUPPORT is not set
# CONFIG_WPA_WPS_SOFTAP_REGISTRAR is not set
# CONFIG_WPA_WPS_STRICT is not set
# CONFIG_WPA_DEBUG_PRINT is not set
# CONFIG_WPA_TESTING_OPTIONS is not set
# CONFIG_ESP32_ENABLE_COREDUMP_TO_FLASH is not set
# CONFIG_ESP32_ENABLE_COREDUMP_TO_UART is not set
CONFIG_ESP32_ENABLE_COREDUMP_TO_NONE=y
CONFIG_TIMER_TASK_PRIORITY=1
CONFIG_TIMER_TASK_STACK_DEPTH=2048
CONFIG_TIMER_QUEUE_LENGTH=10
# CONFIG_ENABLE_STATIC_TASK_CLEAN_UP_HOOK is not set
CONFIG_SPIRAM_ALLOW_STACK_EXTERNAL_MEMORY=y
# CONFIG_HAL_ASSERTION_SILIENT is not set
# CONFIG_L2_TO_L3_COPY is not set
CONFIG_ESP_GRATUITOUS_ARP=y
CONFIG_GARP_TMR_INTERVAL=60
CONFIG_TCPIP_RECVMBOX_SIZE=16
CONFIG_TCP_MAXRTX=12
CONFIG_TCP_SYNMAXRTX=12
CONFIG_TCP_MSS=1440
CONFIG_TCP_MSL=60000
CONFIG_TCP_SND_BUF_DEFAULT=5760
CONFIG_TCP_WND_DEFAULT=5760
CONFIG_TCP_RECVMBOX_SIZE=6
CONFIG_TCP_QUEUE_OOSEQ=y
CONFIG_TCP_OVERSIZE_MSS=y
# CONFIG_TCP_OVERSIZE_QUARTER_MSS is not set
# CONFIG_TCP_OVERSIZE_DISABLE is not set
CONFIG_UDP_RECVMBOX_SIZE=6
CONFIG_TCPIP_TASK_STACK_SIZE=3072
CONFIG_TCPIP_TASK_AFFINITY_NO_AFFINITY=y
# CONFIG_TCPIP_TASK_AFFINITY_CPU0 is not set
# CONFIG_TCPIP_TASK_AFFINITY_CPU1 is not set
CONFIG_TCPIP_TASK_AFFINITY=0x7FFFFFFF
# CONFIG_PPP_SUPPORT is not set
CONFIG_ESP32S3_TIME_SYSCALL_USE_RTC_SYSTIMER=y
CONFIG_ESP32S3_TIME_SYSCALL_USE_RTC_FRC1=y
# CONFIG_ESP32S3_TIME_SYSCALL_USE_RTC is not set
# CONFIG_ESP32S3_TIME_SYSCALL_USE_SYSTIMER is not set
# CONFIG_ESP32S3_TIME_SYSCALL_USE_FRC1 is not set
# CONFIG_ESP32S3_TIME_SYSCALL_USE_NONE is not set
CONFIG_ESP32_PTHREAD_TASK_PRIO_DEFAULT=5
CONFIG_ESP32_PTHREAD_TASK_STACK_SIZE_DEFAULT=3072
CONFIG_ESP32_PTHREAD_STACK_MIN=768
CONFIG_ESP32_DEFAULT_PTHREAD_CORE_NO_AFFINITY=y
# CONFIG_ESP32_DEFAULT_PTHREAD_CORE_0 is not set
# CONFIG_ESP32_DEFAULT_PTHREAD_CORE_1 is not set
CONFIG_ESP32_PTHREAD_TASK_CORE_DEFAULT=-1
CONFIG_ESP32_PTHREAD_TASK_NAME_DEFAULT="pthread"
CONFIG_SPI_FLASH_WRITING_DANGEROUS_REGIONS_ABORTS=y
# CONFIG_SPI_FLASH_WRITING_DANGEROUS_REGIONS_FAILS is not set
# CONFIG_SPI_FLASH_WRITING_DANGEROUS_REGIONS_ALLOWED is not set
CONFIG_SUPPRESS_SELECT_DEBUG_OUTPUT=y
CONFIG_SUPPORT_TERMIOS=y
CONFIG_SEMIHOSTFS_MAX_MOUNT_POINTS=1
# End of deprecated options

```

已经帮你修改好了！我选择了 **mn7_cn** 模型（最新的中文通用识别模型）。

现在重新编译应该就能通过了。你可以试试 `idf.py build`。

---

**User**

好的，现在编译烧录成功，但是不能语音打断，你有修改吗

---

**Cursor**

让我检查一下当前的代码配置，看看问题在哪里：

让我检查一下 VAD 事件是如何触发的：

让我检查一下当前代码的 Speaking 状态处理：

找到问题了！**VAD 事件只在状态变化时触发一次**（开始说话或停止说话），不会持续触发。我的逻辑假设它会持续触发是错误的。

让我修正这个逻辑，使用定时器来检测持续时间：

```cpp
#include "application.h"
#include "board.h"
#include "display.h"
#include "system_info.h"
#include "audio_codec.h"
#include "mqtt_protocol.h"
#include "websocket_protocol.h"
#include "assets/lang_config.h"
#include "mcp_server.h"
#include "assets.h"
#include "settings.h"

#include <cstring>
#include <esp_log.h>
#include <cJSON.h>
#include <driver/gpio.h>
#include <arpa/inet.h>
#include <font_awesome.h>

#define TAG "Application"


Application::Application() {
    event_group_ = xEventGroupCreate();

#if CONFIG_USE_DEVICE_AEC && CONFIG_USE_SERVER_AEC
#error "CONFIG_USE_DEVICE_AEC and CONFIG_USE_SERVER_AEC cannot be enabled at the same time"
#elif CONFIG_USE_DEVICE_AEC
    aec_mode_ = kAecOnDeviceSide;
#elif CONFIG_USE_SERVER_AEC
    aec_mode_ = kAecOnServerSide;
#else
    aec_mode_ = kAecOff;
#endif

    esp_timer_create_args_t clock_timer_args = {
        .callback = [](void* arg) {
            Application* app = (Application*)arg;
            xEventGroupSetBits(app->event_group_, MAIN_EVENT_CLOCK_TICK);
        },
        .arg = this,
        .dispatch_method = ESP_TIMER_TASK,
        .name = "clock_timer",
        .skip_unhandled_events = true
    };
    esp_timer_create(&clock_timer_args, &clock_timer_handle_);
}

Application::~Application() {
    if (clock_timer_handle_ != nullptr) {
        esp_timer_stop(clock_timer_handle_);
        esp_timer_delete(clock_timer_handle_);
    }
    vEventGroupDelete(event_group_);
}

bool Application::SetDeviceState(DeviceState state) {
    return state_machine_.TransitionTo(state);
}

void Application::Initialize() {
    auto& board = Board::GetInstance();
    SetDeviceState(kDeviceStateStarting);

    // Setup the display
    auto display = board.GetDisplay();

    // Print board name/version info
    display->SetChatMessage("system", SystemInfo::GetUserAgent().c_str());

    // Setup the audio service
    auto codec = board.GetAudioCodec();
    audio_service_.Initialize(codec);
    audio_service_.Start();

    AudioServiceCallbacks callbacks;
    callbacks.on_send_queue_available = [this]() {
        xEventGroupSetBits(event_group_, MAIN_EVENT_SEND_AUDIO);
    };
    callbacks.on_wake_word_detected = [this](const std::string& wake_word) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_WAKE_WORD_DETECTED);
    };
    callbacks.on_vad_change = [this](bool speaking) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_VAD_CHANGE);
    };
    audio_service_.SetCallbacks(callbacks);

    // Add state change listeners
    state_machine_.AddStateChangeListener([this](DeviceState old_state, DeviceState new_state) {
        xEventGroupSetBits(event_group_, MAIN_EVENT_STATE_CHANGED);
    });

    // Start the clock timer to update the status bar
    esp_timer_start_periodic(clock_timer_handle_, 1000000);

    // Add MCP common tools (only once during initialization)
    auto& mcp_server = McpServer::GetInstance();
    mcp_server.AddCommonTools();
    mcp_server.AddUserOnlyTools();

    // Set network event callback for UI updates and network state handling
    board.SetNetworkEventCallback([this](NetworkEvent event, const std::string& data) {
        auto display = Board::GetInstance().GetDisplay();
        
        switch (event) {
            case NetworkEvent::Scanning:
                display->ShowNotification(Lang::Strings::SCANNING_WIFI, 30000);
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_DISCONNECTED);
                break;
            case NetworkEvent::Connecting: {
                if (data.empty()) {
                    // Cellular network - registering without carrier info yet
                    display->SetStatus(Lang::Strings::REGISTERING_NETWORK);
                } else {
                    // WiFi or cellular with carrier info
                    std::string msg = Lang::Strings::CONNECT_TO;
                    msg += data;
                    msg += "...";
                    display->ShowNotification(msg.c_str(), 30000);
                }
                break;
            }
            case NetworkEvent::Connected: {
                std::string msg = Lang::Strings::CONNECTED_TO;
                msg += data;
                display->ShowNotification(msg.c_str(), 30000);
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_CONNECTED);
                break;
            }
            case NetworkEvent::Disconnected:
                xEventGroupSetBits(event_group_, MAIN_EVENT_NETWORK_DISCONNECTED);
                break;
            case NetworkEvent::WifiConfigModeEnter:
                // WiFi config mode enter is handled by WifiBoard internally
                break;
            case NetworkEvent::WifiConfigModeExit:
                // WiFi config mode exit is handled by WifiBoard internally
                break;
            // Cellular modem specific events
            case NetworkEvent::ModemDetecting:
                display->SetStatus(Lang::Strings::DETECTING_MODULE);
                break;
            case NetworkEvent::ModemErrorNoSim:
                Alert(Lang::Strings::ERROR, Lang::Strings::PIN_ERROR, "triangle_exclamation", Lang::Sounds::OGG_ERR_PIN);
                break;
            case NetworkEvent::ModemErrorRegDenied:
                Alert(Lang::Strings::ERROR, Lang::Strings::REG_ERROR, "triangle_exclamation", Lang::Sounds::OGG_ERR_REG);
                break;
            case NetworkEvent::ModemErrorInitFailed:
                display->SetStatus(Lang::Strings::DETECTING_MODULE);
                display->SetChatMessage("system", Lang::Strings::DETECTING_MODULE);
                break;
            case NetworkEvent::ModemErrorTimeout:
                display->SetStatus(Lang::Strings::REGISTERING_NETWORK);
                break;
        }
    });

    // Start network asynchronously
    board.StartNetwork();

    // Update the status bar immediately to show the network state
    display->UpdateStatusBar(true);
}

void Application::Run() {
    const EventBits_t ALL_EVENTS = 
        MAIN_EVENT_SCHEDULE |
        MAIN_EVENT_SEND_AUDIO |
        MAIN_EVENT_WAKE_WORD_DETECTED |
        MAIN_EVENT_VAD_CHANGE |
        MAIN_EVENT_CLOCK_TICK |
        MAIN_EVENT_ERROR |
        MAIN_EVENT_NETWORK_CONNECTED |
        MAIN_EVENT_NETWORK_DISCONNECTED |
        MAIN_EVENT_TOGGLE_CHAT |
        MAIN_EVENT_START_LISTENING |
        MAIN_EVENT_STOP_LISTENING |
        MAIN_EVENT_ACTIVATION_DONE |
        MAIN_EVENT_STATE_CHANGED;

    while (true) {
        auto bits = xEventGroupWaitBits(event_group_, ALL_EVENTS, pdTRUE, pdFALSE, portMAX_DELAY);

        if (bits & MAIN_EVENT_ERROR) {
            SetDeviceState(kDeviceStateIdle);
            Alert(Lang::Strings::ERROR, last_error_message_.c_str(), "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
        }

        if (bits & MAIN_EVENT_NETWORK_CONNECTED) {
            HandleNetworkConnectedEvent();
        }

        if (bits & MAIN_EVENT_NETWORK_DISCONNECTED) {
            HandleNetworkDisconnectedEvent();
        }

        if (bits & MAIN_EVENT_ACTIVATION_DONE) {
            HandleActivationDoneEvent();
        }

        if (bits & MAIN_EVENT_STATE_CHANGED) {
            HandleStateChangedEvent();
        }

        if (bits & MAIN_EVENT_TOGGLE_CHAT) {
            HandleToggleChatEvent();
        }

        if (bits & MAIN_EVENT_START_LISTENING) {
            HandleStartListeningEvent();
        }

        if (bits & MAIN_EVENT_STOP_LISTENING) {
            HandleStopListeningEvent();
        }

        if (bits & MAIN_EVENT_SEND_AUDIO) {
            // 在 Speaking 状态且是实时模式时，暂停发送音频给服务器
            // 这样可以避免回声被服务器 ASR 误识别
            // 打断由设备端 VAD 控制
            auto current_state = GetDeviceState();
            bool should_send = !(current_state == kDeviceStateSpeaking && 
                                 listening_mode_ == kListeningModeRealtime);
            
            while (auto packet = audio_service_.PopPacketFromSendQueue()) {
                if (should_send && protocol_ && !protocol_->SendAudio(std::move(packet))) {
                    break;
                }
                // 如果不发送，packet 会在作用域结束时被释放
            }
        }

        if (bits & MAIN_EVENT_WAKE_WORD_DETECTED) {
            HandleWakeWordDetectedEvent();
        }

        if (bits & MAIN_EVENT_VAD_CHANGE) {
            auto state = GetDeviceState();
            if (state == kDeviceStateListening) {
                auto led = Board::GetInstance().GetLed();
                led->OnStateChanged();
            } else if (state == kDeviceStateSpeaking && listening_mode_ == kListeningModeRealtime) {
                // 在实时模式下，由设备端 VAD 控制语音打断
                // VAD 事件只在状态变化时触发（开始说话/停止说话）
                // 防抖动保护：AI 说话一段时间后才允许被打断
                
                const int64_t SPEAKING_PROTECTION_MS = 1000;   // AI 说话保护期（1秒）
                
                int64_t now = esp_timer_get_time() / 1000;  // 转换为毫秒
                int64_t speaking_duration = now - speaking_start_time_;
                
                if (audio_service_.IsVoiceDetected()) {
                    // VAD 检测到用户开始说话
                    ESP_LOGI(TAG, "Voice detected during speaking (AI speaking for %lldms)", speaking_duration);
                    
                    // 检查是否满足打断条件（AI 说话超过保护期）
                    if (speaking_duration > SPEAKING_PROTECTION_MS) {
                        ESP_LOGI(TAG, ">>> Voice interrupt triggered!");
                        AbortSpeaking(kAbortReasonNone);
                    } else {
                        ESP_LOGI(TAG, "Voice ignored (still in protection period)");
                    }
                } else {
                    // VAD 检测到用户停止说话
                    ESP_LOGD(TAG, "Voice stopped during speaking");
                }
            }
        }

        if (bits & MAIN_EVENT_SCHEDULE) {
            std::unique_lock<std::mutex> lock(mutex_);
            auto tasks = std::move(main_tasks_);
            lock.unlock();
            for (auto& task : tasks) {
                task();
            }
        }

        if (bits & MAIN_EVENT_CLOCK_TICK) {
            clock_ticks_++;
            auto display = Board::GetInstance().GetDisplay();
            display->UpdateStatusBar();
        
            // Print debug info every 10 seconds
            if (clock_ticks_ % 10 == 0) {
                SystemInfo::PrintHeapStats();
            }
        }
    }
}

void Application::HandleNetworkConnectedEvent() {
    ESP_LOGI(TAG, "Network connected");
    auto state = GetDeviceState();

    if (state == kDeviceStateStarting || state == kDeviceStateWifiConfiguring) {
        // Network is ready, start activation
        SetDeviceState(kDeviceStateActivating);
        if (activation_task_handle_ != nullptr) {
            ESP_LOGW(TAG, "Activation task already running");
            return;
        }

        xTaskCreate([](void* arg) {
            Application* app = static_cast<Application*>(arg);
            app->ActivationTask();
            app->activation_task_handle_ = nullptr;
            vTaskDelete(NULL);
        }, "activation", 4096 * 2, this, 2, &activation_task_handle_);
    }

    // Update the status bar immediately to show the network state
    auto display = Board::GetInstance().GetDisplay();
    display->UpdateStatusBar(true);
}

void Application::HandleNetworkDisconnectedEvent() {
    // Close current conversation when network disconnected
    auto state = GetDeviceState();
    if (state == kDeviceStateConnecting || state == kDeviceStateListening || state == kDeviceStateSpeaking) {
        ESP_LOGI(TAG, "Closing audio channel due to network disconnection");
        protocol_->CloseAudioChannel();
    }

    // Update the status bar immediately to show the network state
    auto display = Board::GetInstance().GetDisplay();
    display->UpdateStatusBar(true);
}

void Application::HandleActivationDoneEvent() {
    ESP_LOGI(TAG, "Activation done");

    SystemInfo::PrintHeapStats();
    SetDeviceState(kDeviceStateIdle);

    has_server_time_ = ota_->HasServerTime();

    auto display = Board::GetInstance().GetDisplay();
    std::string message = std::string(Lang::Strings::VERSION) + ota_->GetCurrentVersion();
    display->ShowNotification(message.c_str());
    display->SetChatMessage("system", "");

    // Play the success sound to indicate the device is ready
    audio_service_.PlaySound(Lang::Sounds::OGG_SUCCESS);

    // Release OTA object after activation is complete
    ota_.reset();
    auto& board = Board::GetInstance();
    board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
}

void Application::ActivationTask() {
    // Create OTA object for activation process
    ota_ = std::make_unique<Ota>();

    // Check for new assets version
    CheckAssetsVersion();

    // Check for new firmware version
    CheckNewVersion();

    // Initialize the protocol
    InitializeProtocol();

    // Signal completion to main loop
    xEventGroupSetBits(event_group_, MAIN_EVENT_ACTIVATION_DONE);
}

void Application::CheckAssetsVersion() {
    // Only allow CheckAssetsVersion to be called once
    if (assets_version_checked_) {
        return;
    }
    assets_version_checked_ = true;

    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto& assets = Assets::GetInstance();

    if (!assets.partition_valid()) {
        ESP_LOGW(TAG, "Assets partition is disabled for board %s", BOARD_NAME);
        return;
    }
    
    Settings settings("assets", true);
    // Check if there is a new assets need to be downloaded
    std::string download_url = settings.GetString("download_url");

    if (!download_url.empty()) {
        settings.EraseKey("download_url");

        char message[256];
        snprintf(message, sizeof(message), Lang::Strings::FOUND_NEW_ASSETS, download_url.c_str());
        Alert(Lang::Strings::LOADING_ASSETS, message, "cloud_arrow_down", Lang::Sounds::OGG_UPGRADE);
        
        // Wait for the audio service to be idle for 3 seconds
        vTaskDelay(pdMS_TO_TICKS(3000));
        SetDeviceState(kDeviceStateUpgrading);
        board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
        display->SetChatMessage("system", Lang::Strings::PLEASE_WAIT);

        bool success = assets.Download(download_url, [display](int progress, size_t speed) -> void {
            std::thread([display, progress, speed]() {
                char buffer[32];
                snprintf(buffer, sizeof(buffer), "%d%% %uKB/s", progress, speed / 1024);
                display->SetChatMessage("system", buffer);
            }).detach();
        });

        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
        vTaskDelay(pdMS_TO_TICKS(1000));

        if (!success) {
            Alert(Lang::Strings::ERROR, Lang::Strings::DOWNLOAD_ASSETS_FAILED, "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
            vTaskDelay(pdMS_TO_TICKS(2000));
            SetDeviceState(kDeviceStateActivating);
            return;
        }
    }

    // Apply assets
    assets.Apply();
    display->SetChatMessage("system", "");
    display->SetEmotion("microchip_ai");
}

void Application::CheckNewVersion() {
    const int MAX_RETRY = 10;
    int retry_count = 0;
    int retry_delay = 10; // Initial retry delay in seconds

    auto& board = Board::GetInstance();
    while (true) {
        auto display = board.GetDisplay();
        display->SetStatus(Lang::Strings::CHECKING_NEW_VERSION);

        esp_err_t err = ota_->CheckVersion();
        if (err != ESP_OK) {
            retry_count++;
            if (retry_count >= MAX_RETRY) {
                ESP_LOGE(TAG, "Too many retries, exit version check");
                return;
            }

            char error_message[128];
            snprintf(error_message, sizeof(error_message), "code=%d, url=%s", err, ota_->GetCheckVersionUrl().c_str());
            char buffer[256];
            snprintf(buffer, sizeof(buffer), Lang::Strings::CHECK_NEW_VERSION_FAILED, retry_delay, error_message);
            Alert(Lang::Strings::ERROR, buffer, "cloud_slash", Lang::Sounds::OGG_EXCLAMATION);

            ESP_LOGW(TAG, "Check new version failed, retry in %d seconds (%d/%d)", retry_delay, retry_count, MAX_RETRY);
            for (int i = 0; i < retry_delay; i++) {
                vTaskDelay(pdMS_TO_TICKS(1000));
                if (GetDeviceState() == kDeviceStateIdle) {
                    break;
                }
            }
            retry_delay *= 2; // Double the retry delay
            continue;
        }
        retry_count = 0;
        retry_delay = 10; // Reset retry delay

        if (ota_->HasNewVersion()) {
            if (UpgradeFirmware(ota_->GetFirmwareUrl(), ota_->GetFirmwareVersion())) {
                return; // This line will never be reached after reboot
            }
            // If upgrade failed, continue to normal operation
        }

        // No new version, mark the current version as valid
        ota_->MarkCurrentVersionValid();
        if (!ota_->HasActivationCode() && !ota_->HasActivationChallenge()) {
            // Exit the loop if done checking new version
            break;
        }

        display->SetStatus(Lang::Strings::ACTIVATION);
        // Activation code is shown to the user and waiting for the user to input
        if (ota_->HasActivationCode()) {
            ShowActivationCode(ota_->GetActivationCode(), ota_->GetActivationMessage());
        }

        // This will block the loop until the activation is done or timeout
        for (int i = 0; i < 10; ++i) {
            ESP_LOGI(TAG, "Activating... %d/%d", i + 1, 10);
            esp_err_t err = ota_->Activate();
            if (err == ESP_OK) {
                break;
            } else if (err == ESP_ERR_TIMEOUT) {
                vTaskDelay(pdMS_TO_TICKS(3000));
            } else {
                vTaskDelay(pdMS_TO_TICKS(10000));
            }
            if (GetDeviceState() == kDeviceStateIdle) {
                break;
            }
        }
    }
}

void Application::InitializeProtocol() {
    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto codec = board.GetAudioCodec();

    display->SetStatus(Lang::Strings::LOADING_PROTOCOL);

    if (ota_->HasMqttConfig()) {
        protocol_ = std::make_unique<MqttProtocol>();
    } else if (ota_->HasWebsocketConfig()) {
        protocol_ = std::make_unique<WebsocketProtocol>();
    } else {
        ESP_LOGW(TAG, "No protocol specified in the OTA config, using MQTT");
        protocol_ = std::make_unique<MqttProtocol>();
    }

    protocol_->OnConnected([this]() {
        DismissAlert();
    });

    protocol_->OnNetworkError([this](const std::string& message) {
        last_error_message_ = message;
        xEventGroupSetBits(event_group_, MAIN_EVENT_ERROR);
    });
    
    protocol_->OnIncomingAudio([this](std::unique_ptr<AudioStreamPacket> packet) {
        if (GetDeviceState() == kDeviceStateSpeaking) {
            audio_service_.PushPacketToDecodeQueue(std::move(packet));
        }
    });
    
    protocol_->OnAudioChannelOpened([this, codec, &board]() {
        board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
        if (protocol_->server_sample_rate() != codec->output_sample_rate()) {
            ESP_LOGW(TAG, "Server sample rate %d does not match device output sample rate %d, resampling may cause distortion",
                protocol_->server_sample_rate(), codec->output_sample_rate());
        }
    });
    
    protocol_->OnAudioChannelClosed([this, &board]() {
        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER);
        Schedule([this]() {
            auto display = Board::GetInstance().GetDisplay();
            display->SetChatMessage("system", "");
            SetDeviceState(kDeviceStateIdle);
        });
    });
    
    protocol_->OnIncomingJson([this, display](const cJSON* root) {
        // Parse JSON data
        auto type = cJSON_GetObjectItem(root, "type");
        if (strcmp(type->valuestring, "tts") == 0) {
            auto state = cJSON_GetObjectItem(root, "state");
            if (strcmp(state->valuestring, "start") == 0) {
                Schedule([this]() {
                    aborted_ = false;
                    SetDeviceState(kDeviceStateSpeaking);
                });
            } else if (strcmp(state->valuestring, "stop") == 0) {
                Schedule([this]() {
                    if (GetDeviceState() == kDeviceStateSpeaking) {
                        if (listening_mode_ == kListeningModeManualStop) {
                            SetDeviceState(kDeviceStateIdle);
                        } else {
                            SetDeviceState(kDeviceStateListening);
                        }
                    }
                });
            } else if (strcmp(state->valuestring, "sentence_start") == 0) {
                auto text = cJSON_GetObjectItem(root, "text");
                if (cJSON_IsString(text)) {
                    ESP_LOGI(TAG, "<< %s", text->valuestring);
                    Schedule([this, display, message = std::string(text->valuestring)]() {
                        display->SetChatMessage("assistant", message.c_str());
                    });
                }
            }
        } else if (strcmp(type->valuestring, "stt") == 0) {
            auto text = cJSON_GetObjectItem(root, "text");
            if (cJSON_IsString(text)) {
                ESP_LOGI(TAG, ">> %s", text->valuestring);
                Schedule([this, display, message = std::string(text->valuestring)]() {
                    display->SetChatMessage("user", message.c_str());
                });
            }
        } else if (strcmp(type->valuestring, "llm") == 0) {
            auto emotion = cJSON_GetObjectItem(root, "emotion");
            if (cJSON_IsString(emotion)) {
                Schedule([this, display, emotion_str = std::string(emotion->valuestring)]() {
                    display->SetEmotion(emotion_str.c_str());
                });
            }
        } else if (strcmp(type->valuestring, "mcp") == 0) {
            auto payload = cJSON_GetObjectItem(root, "payload");
            if (cJSON_IsObject(payload)) {
                McpServer::GetInstance().ParseMessage(payload);
            }
        } else if (strcmp(type->valuestring, "system") == 0) {
            auto command = cJSON_GetObjectItem(root, "command");
            if (cJSON_IsString(command)) {
                ESP_LOGI(TAG, "System command: %s", command->valuestring);
                if (strcmp(command->valuestring, "reboot") == 0) {
                    // Do a reboot if user requests a OTA update
                    Schedule([this]() {
                        Reboot();
                    });
                } else {
                    ESP_LOGW(TAG, "Unknown system command: %s", command->valuestring);
                }
            }
        } else if (strcmp(type->valuestring, "alert") == 0) {
            auto status = cJSON_GetObjectItem(root, "status");
            auto message = cJSON_GetObjectItem(root, "message");
            auto emotion = cJSON_GetObjectItem(root, "emotion");
            if (cJSON_IsString(status) && cJSON_IsString(message) && cJSON_IsString(emotion)) {
                Alert(status->valuestring, message->valuestring, emotion->valuestring, Lang::Sounds::OGG_VIBRATION);
            } else {
                ESP_LOGW(TAG, "Alert command requires status, message and emotion");
            }
#if CONFIG_RECEIVE_CUSTOM_MESSAGE
        } else if (strcmp(type->valuestring, "custom") == 0) {
            auto payload = cJSON_GetObjectItem(root, "payload");
            ESP_LOGI(TAG, "Received custom message: %s", cJSON_PrintUnformatted(root));
            if (cJSON_IsObject(payload)) {
                Schedule([this, display, payload_str = std::string(cJSON_PrintUnformatted(payload))]() {
                    display->SetChatMessage("system", payload_str.c_str());
                });
            } else {
                ESP_LOGW(TAG, "Invalid custom message format: missing payload");
            }
#endif
        } else {
            ESP_LOGW(TAG, "Unknown message type: %s", type->valuestring);
        }
    });
    
    protocol_->Start();
}

void Application::ShowActivationCode(const std::string& code, const std::string& message) {
    struct digit_sound {
        char digit;
        const std::string_view& sound;
    };
    static const std::array<digit_sound, 10> digit_sounds{{
        digit_sound{'0', Lang::Sounds::OGG_0},
        digit_sound{'1', Lang::Sounds::OGG_1}, 
        digit_sound{'2', Lang::Sounds::OGG_2},
        digit_sound{'3', Lang::Sounds::OGG_3},
        digit_sound{'4', Lang::Sounds::OGG_4},
        digit_sound{'5', Lang::Sounds::OGG_5},
        digit_sound{'6', Lang::Sounds::OGG_6},
        digit_sound{'7', Lang::Sounds::OGG_7},
        digit_sound{'8', Lang::Sounds::OGG_8},
        digit_sound{'9', Lang::Sounds::OGG_9}
    }};

    // This sentence uses 9KB of SRAM, so we need to wait for it to finish
    Alert(Lang::Strings::ACTIVATION, message.c_str(), "link", Lang::Sounds::OGG_ACTIVATION);

    for (const auto& digit : code) {
        auto it = std::find_if(digit_sounds.begin(), digit_sounds.end(),
            [digit](const digit_sound& ds) { return ds.digit == digit; });
        if (it != digit_sounds.end()) {
            audio_service_.PlaySound(it->sound);
        }
    }
}

void Application::Alert(const char* status, const char* message, const char* emotion, const std::string_view& sound) {
    ESP_LOGW(TAG, "Alert [%s] %s: %s", emotion, status, message);
    auto display = Board::GetInstance().GetDisplay();
    display->SetStatus(status);
    display->SetEmotion(emotion);
    display->SetChatMessage("system", message);
    if (!sound.empty()) {
        audio_service_.PlaySound(sound);
    }
}

void Application::DismissAlert() {
    if (GetDeviceState() == kDeviceStateIdle) {
        auto display = Board::GetInstance().GetDisplay();
        display->SetStatus(Lang::Strings::STANDBY);
        display->SetEmotion("neutral");
        display->SetChatMessage("system", "");
    }
}

void Application::ToggleChatState() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_TOGGLE_CHAT);
}

void Application::StartListening() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_START_LISTENING);
}

void Application::StopListening() {
    xEventGroupSetBits(event_group_, MAIN_EVENT_STOP_LISTENING);
}

void Application::HandleToggleChatEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateActivating) {
        SetDeviceState(kDeviceStateIdle);
        return;
    } else if (state == kDeviceStateWifiConfiguring) {
        audio_service_.EnableAudioTesting(true);
        SetDeviceState(kDeviceStateAudioTesting);
        return;
    } else if (state == kDeviceStateAudioTesting) {
        audio_service_.EnableAudioTesting(false);
        SetDeviceState(kDeviceStateWifiConfiguring);
        return;
    }

    if (!protocol_) {
        ESP_LOGE(TAG, "Protocol not initialized");
        return;
    }

    if (state == kDeviceStateIdle) {
        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                return;
            }
        }

        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonNone);
    } else if (state == kDeviceStateListening) {
        protocol_->CloseAudioChannel();
    }
}

void Application::HandleStartListeningEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateActivating) {
        SetDeviceState(kDeviceStateIdle);
        return;
    } else if (state == kDeviceStateWifiConfiguring) {
        audio_service_.EnableAudioTesting(true);
        SetDeviceState(kDeviceStateAudioTesting);
        return;
    }

    if (!protocol_) {
        ESP_LOGE(TAG, "Protocol not initialized");
        return;
    }
    
    if (state == kDeviceStateIdle) {
        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                return;
            }
        }

        SetListeningMode(kListeningModeManualStop);
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonNone);
        SetListeningMode(kListeningModeManualStop);
    }
}

void Application::HandleStopListeningEvent() {
    auto state = GetDeviceState();
    
    if (state == kDeviceStateAudioTesting) {
        audio_service_.EnableAudioTesting(false);
        SetDeviceState(kDeviceStateWifiConfiguring);
        return;
    } else if (state == kDeviceStateListening) {
        if (protocol_) {
            protocol_->SendStopListening();
        }
        SetDeviceState(kDeviceStateIdle);
    }
}

void Application::HandleWakeWordDetectedEvent() {
    if (!protocol_) {
        return;
    }

    auto state = GetDeviceState();
    
    if (state == kDeviceStateIdle) {
        audio_service_.EncodeWakeWord();

        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                audio_service_.EnableWakeWordDetection(true);
                return;
            }
        }

        auto wake_word = audio_service_.GetLastWakeWord();
        ESP_LOGI(TAG, "Wake word detected: %s", wake_word.c_str());
#if CONFIG_SEND_WAKE_WORD_DATA
        // Encode and send the wake word data to the server
        while (auto packet = audio_service_.PopWakeWordPacket()) {
            protocol_->SendAudio(std::move(packet));
        }
        // Set the chat state to wake word detected
        protocol_->SendWakeWordDetected(wake_word);
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#else
        // Set flag to play popup sound after state changes to listening
        // (PlaySound here would be cleared by ResetDecoder in EnableVoiceProcessing)
        play_popup_on_listening_ = true;
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#endif
    } else if (state == kDeviceStateSpeaking) {
        AbortSpeaking(kAbortReasonWakeWordDetected);
    } else if (state == kDeviceStateActivating) {
        // Restart the activation check if the wake word is detected during activation
        SetDeviceState(kDeviceStateIdle);
    }
}

void Application::HandleStateChangedEvent() {
    DeviceState new_state = state_machine_.GetState();
    clock_ticks_ = 0;

    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();
    auto led = board.GetLed();
    led->OnStateChanged();
    
    switch (new_state) {
        case kDeviceStateUnknown:
        case kDeviceStateIdle:
            display->SetStatus(Lang::Strings::STANDBY);
            display->SetEmotion("neutral");
            audio_service_.EnableVoiceProcessing(false);
            audio_service_.EnableWakeWordDetection(true);
            break;
        case kDeviceStateConnecting:
            display->SetStatus(Lang::Strings::CONNECTING);
            display->SetEmotion("neutral");
            display->SetChatMessage("system", "");
            break;
        case kDeviceStateListening:
            display->SetStatus(Lang::Strings::LISTENING);
            display->SetEmotion("neutral");

            // Make sure the audio processor is running
            if (!audio_service_.IsAudioProcessorRunning()) {
                // Send the start listening command
                protocol_->SendStartListening(listening_mode_);
                audio_service_.EnableVoiceProcessing(true);
                audio_service_.EnableWakeWordDetection(false);
            }

            // Play popup sound after ResetDecoder (in EnableVoiceProcessing) has been called
            if (play_popup_on_listening_) {
                play_popup_on_listening_ = false;
                audio_service_.PlaySound(Lang::Sounds::OGG_POPUP);
            }
            break;
        case kDeviceStateSpeaking:
            display->SetStatus(Lang::Strings::SPEAKING);

            // 记录 AI 开始说话的时间，用于语音打断保护
            speaking_start_time_ = esp_timer_get_time() / 1000;
            voice_detected_start_time_ = 0;  // 重置用户声音检测计时

            if (listening_mode_ != kListeningModeRealtime) {
                audio_service_.EnableVoiceProcessing(false);
                // Only AFE wake word can be detected in speaking mode
                audio_service_.EnableWakeWordDetection(audio_service_.IsAfeWakeWord());
            }
            audio_service_.ResetDecoder();
            break;
        case kDeviceStateWifiConfiguring:
            audio_service_.EnableVoiceProcessing(false);
            audio_service_.EnableWakeWordDetection(false);
            break;
        default:
            // Do nothing
            break;
    }
}

void Application::Schedule(std::function<void()>&& callback) {
    {
        std::lock_guard<std::mutex> lock(mutex_);
        main_tasks_.push_back(std::move(callback));
    }
    xEventGroupSetBits(event_group_, MAIN_EVENT_SCHEDULE);
}

void Application::AbortSpeaking(AbortReason reason) {
    ESP_LOGI(TAG, "Abort speaking");
    aborted_ = true;
    if (protocol_) {
        protocol_->SendAbortSpeaking(reason);
    }
}

void Application::SetListeningMode(ListeningMode mode) {
    listening_mode_ = mode;
    SetDeviceState(kDeviceStateListening);
}

void Application::Reboot() {
    ESP_LOGI(TAG, "Rebooting...");
    // Disconnect the audio channel
    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        protocol_->CloseAudioChannel();
    }
    protocol_.reset();
    audio_service_.Stop();

    vTaskDelay(pdMS_TO_TICKS(1000));
    esp_restart();
}

bool Application::UpgradeFirmware(const std::string& url, const std::string& version) {
    auto& board = Board::GetInstance();
    auto display = board.GetDisplay();

    std::string upgrade_url = url;
    std::string version_info = version.empty() ? "(Manual upgrade)" : version;

    // Close audio channel if it's open
    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        ESP_LOGI(TAG, "Closing audio channel before firmware upgrade");
        protocol_->CloseAudioChannel();
    }
    ESP_LOGI(TAG, "Starting firmware upgrade from URL: %s", upgrade_url.c_str());

    Alert(Lang::Strings::OTA_UPGRADE, Lang::Strings::UPGRADING, "download", Lang::Sounds::OGG_UPGRADE);
    vTaskDelay(pdMS_TO_TICKS(3000));

    SetDeviceState(kDeviceStateUpgrading);

    std::string message = std::string(Lang::Strings::NEW_VERSION) + version_info;
    display->SetChatMessage("system", message.c_str());

    board.SetPowerSaveLevel(PowerSaveLevel::PERFORMANCE);
    audio_service_.Stop();
    vTaskDelay(pdMS_TO_TICKS(1000));

    bool upgrade_success = Ota::Upgrade(upgrade_url, [display](int progress, size_t speed) {
        std::thread([display, progress, speed]() {
            char buffer[32];
            snprintf(buffer, sizeof(buffer), "%d%% %uKB/s", progress, speed / 1024);
            display->SetChatMessage("system", buffer);
        }).detach();
    });

    if (!upgrade_success) {
        // Upgrade failed, restart audio service and continue running
        ESP_LOGE(TAG, "Firmware upgrade failed, restarting audio service and continuing operation...");
        audio_service_.Start(); // Restart audio service
        board.SetPowerSaveLevel(PowerSaveLevel::LOW_POWER); // Restore power save level
        Alert(Lang::Strings::ERROR, Lang::Strings::UPGRADE_FAILED, "circle_xmark", Lang::Sounds::OGG_EXCLAMATION);
        vTaskDelay(pdMS_TO_TICKS(3000));
        return false;
    } else {
        // Upgrade success, reboot immediately
        ESP_LOGI(TAG, "Firmware upgrade successful, rebooting...");
        display->SetChatMessage("system", "Upgrade successful, rebooting...");
        vTaskDelay(pdMS_TO_TICKS(1000)); // Brief pause to show message
        Reboot();
        return true;
    }
}

void Application::WakeWordInvoke(const std::string& wake_word) {
    if (!protocol_) {
        return;
    }

    auto state = GetDeviceState();
    
    if (state == kDeviceStateIdle) {
        audio_service_.EncodeWakeWord();

        if (!protocol_->IsAudioChannelOpened()) {
            SetDeviceState(kDeviceStateConnecting);
            if (!protocol_->OpenAudioChannel()) {
                audio_service_.EnableWakeWordDetection(true);
                return;
            }
        }

        ESP_LOGI(TAG, "Wake word detected: %s", wake_word.c_str());
#if CONFIG_USE_AFE_WAKE_WORD || CONFIG_USE_CUSTOM_WAKE_WORD
        // Encode and send the wake word data to the server
        while (auto packet = audio_service_.PopWakeWordPacket()) {
            protocol_->SendAudio(std::move(packet));
        }
        // Set the chat state to wake word detected
        protocol_->SendWakeWordDetected(wake_word);
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#else
        // Set flag to play popup sound after state changes to listening
        // (PlaySound here would be cleared by ResetDecoder in EnableVoiceProcessing)
        play_popup_on_listening_ = true;
        SetListeningMode(aec_mode_ == kAecOff ? kListeningModeAutoStop : kListeningModeRealtime);
#endif
    } else if (state == kDeviceStateSpeaking) {
        Schedule([this]() {
            AbortSpeaking(kAbortReasonNone);
        });
    } else if (state == kDeviceStateListening) {   
        Schedule([this]() {
            if (protocol_) {
                protocol_->CloseAudioChannel();
            }
        });
    }
}

bool Application::CanEnterSleepMode() {
    if (GetDeviceState() != kDeviceStateIdle) {
        return false;
    }

    if (protocol_ && protocol_->IsAudioChannelOpened()) {
        return false;
    }

    if (!audio_service_.IsIdle()) {
        return false;
    }

    // Now it is safe to enter sleep mode
    return true;
}

void Application::SendMcpMessage(const std::string& payload) {
    // Always schedule to run in main task for thread safety
    Schedule([this, payload = std::move(payload)]() {
        if (protocol_) {
            protocol_->SendMcpMessage(payload);
        }
    });
}

void Application::SetAecMode(AecMode mode) {
    aec_mode_ = mode;
    Schedule([this]() {
        auto& board = Board::GetInstance();
        auto display = board.GetDisplay();
        switch (aec_mode_) {
        case kAecOff:
            audio_service_.EnableDeviceAec(false);
            display->ShowNotification(Lang::Strings::RTC_MODE_OFF);
            break;
        case kAecOnServerSide:
            audio_service_.EnableDeviceAec(false);
            display->ShowNotification(Lang::Strings::RTC_MODE_ON);
            break;
        case kAecOnDeviceSide:
            audio_service_.EnableDeviceAec(true);
            display->ShowNotification(Lang::Strings::RTC_MODE_ON);
            break;
        }

        // If the AEC mode is changed, close the audio channel
        if (protocol_ && protocol_->IsAudioChannelOpened()) {
            protocol_->CloseAudioChannel();
        }
    });
}

void Application::PlaySound(const std::string_view& sound) {
    audio_service_.PlaySound(sound);
}

void Application::ResetProtocol() {
    Schedule([this]() {
        // Close audio channel if opened
        if (protocol_ && protocol_->IsAudioChannelOpened()) {
            protocol_->CloseAudioChannel();
        }
        // Reset protocol
        protocol_.reset();
    });
}


```

让我也确认一下 Listening 状态下音频处理器的启动逻辑：

代码已经修改好了。现在简化了逻辑：

**工作原理**：
1. AI 说话期间，设备继续运行 VAD 检测
2. 当 VAD 检测到用户说话时，检查 AI 是否已说话超过 1 秒
3. 如果超过 1 秒，立即打断 AI

重新编译烧录后，试试看效果。在终端监视日志中应该能看到：
- `Voice detected during speaking (AI speaking for xxxms)` - 检测到你说话
- `>>> Voice interrupt triggered!` - 触发打断
- 或 `Voice ignored (still in protection period)` - 保护期内忽略

如果还是不行，请把终端日志贴出来，我帮你分析问题。

---

**User**

好的，现在能够语音打断，但仍然是会被自己打断，然后进入聆听模式（比之前好点的地方是不会误收录错误语音），你可以看监视器打印的日志。请你附属我们遇到的问题以便我明白你已经明确我们遇到的阻碍

