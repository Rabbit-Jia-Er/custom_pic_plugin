# 麦麦绘卷 (Mai's Art Journal)

MaiBot 多模型图片生成插件，支持 9 种 API 格式、文生图/图生图自动识别、自拍模式、自动自拍发说说。

## 功能

### 智能图片生成 (Action 组件)

由 MaiBot LLM 自动触发，不需要手动命令。

- **文生图**: 根据对话内容自动生成图片
- **图生图**: 检测到消息中有图片时自动使用图生图模式
- **自拍模式**: 根据手部动作库生成自拍提示词，可配置参考图
- **提示词优化**: 自动将中文描述优化为专业英文 SD 提示词（自拍模式仅优化场景，不干扰角色外观）
- **LLM 智能选尺寸**: 根据内容自动选择竖图/横图/方图
- **结果缓存**: 相同参数复用之前的结果
- **自动撤回**: 可按模型配置延时撤回

### /dr 命令系统 (Command 组件)

#### 图片生成

| 命令 | 说明 |
|------|------|
| `/dr <风格名>` | 对最近的图片应用预设风格（图生图） |
| `/dr <描述>` | 自然语言生成图片（自动判断文/图生图） |
| `/dr 用model2画一只猫` | 指定模型生成 |

#### 风格管理

| 命令 | 说明 |
|------|------|
| `/dr styles` | 列出所有可用风格 |
| `/dr style <名>` | 查看风格详情 |
| `/dr help` | 帮助信息 |

#### 配置管理（需管理员权限）

| 命令 | 说明 |
|------|------|
| `/dr list` | 列出所有模型 |
| `/dr config` | 显示当前聊天流配置 |
| `/dr set <模型ID>` | 设置 /dr 命令使用的模型 |
| `/dr default <模型ID>` | 设置 Action 组件默认模型 |
| `/dr model on\|off <模型ID>` | 开关指定模型 |
| `/dr recall on\|off <模型ID>` | 开关指定模型的撤回 |
| `/dr on` / `/dr off` | 开关插件（当前聊天流） |
| `/dr reset` | 重置当前聊天流的所有运行时配置 |

> 运行时配置（模型切换、开关等）仅保存在内存中，重启后恢复为 config.toml 的全局设置。

### 自动自拍

定时生成自拍图片并发布到 QQ 空间说说。

**依赖**:
- `autonomous_planning` 插件 — 提供日程数据（当前活动）
- `Maizone` 插件 — 发布到 QQ 空间

**流程**: 获取当前活动 → 生成场景提示词 → 调用生图 API → LLM 生成配文 → 发布说说

**特性**:
- 可配置间隔（默认 2 小时）
- 安静时段控制（默认 00:00-07:00 不发）
- 12 种活动类型的场景/动作/表情/光线随机组合
- 配文基于日程活动 + MaiBot 人设 + 表达风格自然生成，生成失败则跳过不发
- 无日程数据时自动跳过，不会发空内容

---

## 支持的 API 格式

| format | 平台 | 说明 |
|--------|------|------|
| `openai` | OpenAI / 硅基流动 / Grok / NewAPI 等 | 通用 `/images/generations` 接口，自动适配硅基流动参数差异 |
| `openai-chat` | 支持生图的 Chat 模型 | 通过 `/chat/completions` 生图，多策略提取图片 |
| `doubao` | 豆包（火山引擎） | 使用 Ark SDK，支持 seed/guidance_scale/watermark |
| `gemini` | Google Gemini | 原生 `generateContent` 接口，支持 Gemini 2.5/3 系列 |
| `modelscope` | 魔搭社区 | 异步任务模式，自动轮询结果 |
| `shatangyun` | 砂糖云 (NovelAI) | GET 请求，URL 参数传递 |
| `mengyuai` | 梦羽 AI | 支持多模型切换，不支持图生图（须设 `support_img2img = false`） |
| `zai` | Zai (Gemini 转发) | OpenAI 兼容的 chat/completions，支持宽高比/分辨率 |
| `comfyui` | 本地 ComfyUI | 加载工作流 JSON，替换占位符，轮询结果（支持代理配置） |

---

## 快速开始

### 1. 安装插件

```shell
cd MaiBot/plugins
git clone https://github.com/Rabbit-Jia-Er/mais-art-journal.git mais_art_journal
```

重启 MaiBot 后会在 `MaiBot/plugins/mais_art_journal` 中自动生成配置文件 `config.toml`。

### 2. 基本配置

编辑 `config.toml`，至少配置一个模型：

```toml
[plugin]
enabled = true

[models.model1]
name = "我的生图模型"
base_url = "https://api.openai.com/v1"
api_key = "Bearer your_api_key_here"
format = "openai"
model = "dall-e-3"
support_img2img = true
```

### 3. 使用示例

```
用户：麦麦，画一张美少女
麦麦：[生成图片]

用户：麦麦，来张自拍！
麦麦：[生成Bot角色的自拍照片]

用户：[发送图片] 麦麦，把背景换成海滩
麦麦：[生成修改后的图片]

用户：/dr cartoon
麦麦：[应用卡通风格]

用户：/dr 用model1画一只猫
麦麦：[使用指定模型生成]
```

---

## 配置说明

配置文件: `config.toml`，首次启动自动生成。版本更新时自动备份到 `old/` 目录。

### 基础设置

```toml
[plugin]
enabled = true                    # 启用插件

[generation]
default_model = "model1"          # Action 组件默认使用的模型 ID

[components]
enable_unified_generation = true  # 启用智能生图 Action
enable_pic_command = true         # 启用 /dr 图片生成命令
enable_pic_config = true          # 启用 /dr 配置管理命令
enable_pic_style = true           # 启用 /dr 风格管理命令
pic_command_model = "model1"      # /dr 命令默认模型（可通过 /dr set 动态切换）
admin_users = ["12345"]           # 管理员 QQ 号列表（字符串格式）
max_retries = 2                   # API 失败重试次数
enable_debug_info = false         # 显示调试信息
enable_verbose_debug = false      # 打印完整请求/响应报文
```

### 模型配置

```toml
[models.model1]
name = "我的模型"                          # 显示名称
base_url = "https://api.siliconflow.cn/v1" # API 地址
api_key = "Bearer sk-xxx"                  # API 密钥（统一 Bearer 格式）
format = "openai"                          # API 格式
model = "Kwai-Kolors/Kolors"               # 模型标识
fixed_size_enabled = false                 # 固定尺寸（关闭=LLM 自动选）
default_size = "1024x1024"                 # 默认尺寸
seed = 42                                  # 随机种子（-1=随机）
guidance_scale = 2.5                       # 引导强度
num_inference_steps = 20                   # 推理步数
custom_prompt_add = ", best quality"       # 追加正面提示词
negative_prompt_add = "lowres, bad anatomy" # 追加负面提示词
support_img2img = true                     # 是否支持图生图（mengyuai 格式必须设为 false）
auto_recall_delay = 0                      # 自动撤回延时（秒），0=不撤回
```

添加更多模型：复制 `[models.model1]` 整节，改名为 `model2`、`model3` 等。

#### 各平台配置要点

**硅基流动**:
```toml
format = "openai"
base_url = "https://api.siliconflow.cn/v1"
# 插件自动适配 image_size/batch_size 等参数差异
```

**豆包（火山引擎）**:
```toml
format = "doubao"
base_url = "https://ark.cn-beijing.volces.com/api/v3"
api_key = "Bearer xxx"       # 会自动去除 Bearer 前缀传给 SDK
fixed_size_enabled = true    # 必须开启，豆包不接受像素格式
default_size = "2K"          # seedream: "2K"，seededit: "adaptive"
guidance_scale = 5.5         # seededit 推荐
watermark = false
```

**Gemini**:
```toml
format = "gemini"
base_url = "https://generativelanguage.googleapis.com"
api_key = "Bearer AIzaSy..."  # 会自动去除 Bearer 前缀
model = "gemini-2.0-flash-exp"
default_size = "16:9"         # 宽高比格式
# 或 "16:9-2K" (宽高比+分辨率，仅 Gemini 3)
# 或 "-2K" (仅指定分辨率，LLM 选宽高比)
```

**魔搭社区**:
```toml
format = "modelscope"
base_url = "https://api-inference.modelscope.cn/v1"
# 自动使用异步任务模式，轮询获取结果
```

**砂糖云 (NovelAI)**:
```toml
format = "shatangyun"
base_url = "https://std.loliyc.com"
api_key = "Bearer token_here"
model = "nai-diffusion-4-5-full"
artist = "artist_tag"           # 艺术家标签（砂糖云专用）
cfg = 0                         # CFG Rescale
sampler = "k_euler_ancestral"   # 采样器
nocache = 0                     # 禁用缓存
noise_schedule = "karras"       # 噪声调度
```

**梦羽 AI**:
```toml
format = "mengyuai"
base_url = "https://sd.exacg.cc"
model = "0"                     # 模型索引数字
support_img2img = false         # 不支持图生图（需要外部图片上传服务）
```

**Zai (Gemini 转发)**:
```toml
format = "zai"
base_url = "https://zai.is/api"
# 尺寸处理同 Gemini：自动转换为宽高比
```

**ComfyUI**:
```toml
format = "comfyui"
base_url = "http://127.0.0.1:8188"  # ComfyUI 服务地址（支持代理配置）
model = "my_workflow.json"           # 工作流文件名（放在 workflow/ 目录下）
api_key = ""                         # 不需要
# 工作流中使用占位符：${prompt}、${seed}、${image}
```

### 网络配置

```toml
[proxy]
enabled = false
url = "http://127.0.0.1:7890"  # 支持 HTTP/HTTPS/SOCKS5
timeout = 60

[cache]
enabled = true
max_size = 10                   # 最大缓存数量
```

### 功能配置

```toml
[selfie]
enabled = true
reference_image_path = ""         # 参考图路径（留空=纯文生图）
prompt_prefix = "blue hair, red eyes, 1girl"  # Bot 外观描述
negative_prompt = ""              # 额外负面提示词（自动附加 anti-dual-hands）

[auto_recall]
enabled = false                   # 总开关，需在模型配置中设置 auto_recall_delay > 0

[prompt_optimizer]
enabled = true                    # 使用 MaiBot LLM 优化提示词
```

### 自动自拍配置

```toml
[auto_selfie]
enabled = false
interval_minutes = 120            # 自拍间隔（分钟）
selfie_model = "model1"           # 使用的模型 ID
selfie_style = "standard"         # standard=前置自拍 / mirror=对镜自拍
quiet_hours_start = "00:00"       # 安静时段（此时段内不发自拍）
quiet_hours_end = "07:00"
caption_enabled = true            # 是否生成配文
```

### 风格配置

```toml
[styles]
cartoon = "cartoon style, anime style, colorful, vibrant colors, clean lines"
watercolor = "watercolor painting style, soft colors, artistic"

[style_aliases]
cartoon = "卡通,动漫"
watercolor = "水彩"
```

使用: `/dr cartoon` 或 `/dr 卡通`

---

## 插件结构

```
mais_art_journal/
├── plugin.py                     # 入口：注册组件、配置管理（MaisArtJournalPlugin）
├── config.toml                   # 配置文件（自动生成）
├── workflow/                     # ComfyUI 工作流目录
├── core/
│   ├── pic_action.py             # Action 组件（MaisArtAction，LLM 触发）
│   ├── pic_command.py            # Command 组件（/dr 命令）
│   ├── config_manager.py         # 增强配置管理（版本检测、备份、合并）
│   ├── api_clients/              # API 客户端
│   │   ├── __init__.py           # 统一入口 + standalone 接口
│   │   ├── base_client.py        # 基类（重试、代理、工具方法）
│   │   ├── openai_client.py      # OpenAI 格式
│   │   ├── openai_chat_client.py # OpenAI Chat 格式
│   │   ├── doubao_client.py      # 豆包
│   │   ├── gemini_client.py      # Gemini
│   │   ├── modelscope_client.py  # 魔搭
│   │   ├── shatangyun_client.py  # 砂糖云
│   │   ├── mengyuai_client.py    # 梦羽 AI
│   │   ├── zai_client.py         # Zai
│   │   └── comfyui_client.py     # ComfyUI
│   ├── selfie/                   # 自动自拍子系统
│   │   ├── auto_selfie_task.py   # 后台定时任务
│   │   ├── schedule_provider.py  # 日程适配（读 autonomous_planning DB）
│   │   ├── scene_action_generator.py # 场景提示词生成
│   │   └── caption_generator.py  # 配文生成（基于日程+人设+表达风格）
│   └── utils/                    # 工具模块
│       ├── model_utils.py        # 模型配置管理
│       ├── image_utils.py        # 图片处理
│       ├── image_send_utils.py   # base64/URL 统一解析
│       ├── size_utils.py         # 尺寸处理 + LLM 选尺寸
│       ├── cache_manager.py      # 结果缓存
│       ├── recall_utils.py       # 自动撤回
│       ├── prompt_optimizer.py   # 提示词优化（普通模式+自拍场景模式）
│       ├── runtime_state.py      # 运行时状态（按聊天流管理）
│       ├── time_utils.py         # 时间工具
│       └── shared_constants.py   # 共享常量
```

---

## 调用链

### Action 组件

```
用户发消息 → MaiBot LLM 判定触发 → execute()
  → 运行时状态检查（插件开关、模型开关）
  → 提取描述 / 检测自拍模式
  → 提示词优化（LLM，自拍模式使用 scene_only）
  → 检测输入图片（自动判断文/图生图）
  → 获取模型配置 → 尺寸处理 → 合并负面提示词
  → ApiClient.generate_image() → 具体客户端._make_request()
  → process_api_response() 解析响应
  → resolve_image_data() URL→base64
  → send_image() → 缓存 → 自动撤回
```

### Command 组件

```
用户发送 /dr xxx → 正则匹配 → execute()
  → 风格匹配: _execute_style_mode()（图生图）
  → 自然语言: _execute_natural_mode()（文/图生图）
  → 同上的 API 调用链
```

### 自动自拍

```
定时触发 → 安静时段检查
  → ScheduleProvider 查询 autonomous_planning 数据库
  → 获取当前活动（无活动则跳过）
  → 场景提示词生成（动作/环境/表情/光线组合）
  → generate_image_standalone() 独立生图
  → generate_caption() 基于日程+人设+表达风格生成配文（失败则跳过不发）
  → Maizone QZone API 发布说说
```

---

## 常见问题

- **API 调用报错 400**: 参数不正确，请参考报错信息修正请求参数
- **401**: API Key 未正确设置
- **403**: 权限不够，常见原因是模型需要实名认证
- **429**: 触发了频率限制
- **503 / 504**: 服务负载较高，稍后重试
- **图片描述为空**: 需提供明确的图片描述
- **图片尺寸无效**: 支持 `1024x1024` 等格式，宽高范围 100~10000

### 魔搭社区配置步骤

1. 注册[魔搭](https://modelscope.cn)账号
2. 完成[阿里云绑定认证](https://modelscope.cn/docs/accounts/aliyun-binding-and-authorization)
3. 申请 [API Key](https://modelscope.cn/docs/model-service/API-Inference/intro)
4. 在[模型库](https://modelscope.cn/models)选择支持 API 推理的生图模型
5. 在 `config.toml` 中填入 key、请求地址和模型名称

---

## 依赖

- Python 3.12+
- MaiBot 插件系统（兼容 0.10.0+）
- 豆包格式需安装: `pip install 'volcengine-python-sdk[ark]'`

---

## 更新日志

### v3.4.0
- 插件重命名为 **麦麦绘卷 (Mai's Art Journal)**
- 新增 OpenAI-Chat、ComfyUI API 格式（共 9 种）
- 新增自动自拍子系统（日程适配 + 场景提示词 + 配文生成 + QQ空间发布）
- 新增提示词优化 scene_only 模式（自拍仅优化场景，不干扰角色外观）
- 新增配文基于 MaiBot 人设 + 表达风格 + 日程活动生成
- 新增独立生图接口 `generate_image_standalone`
- 优化群聊触发逻辑，必须 @ 或点名才触发，不响应其他机器人命令
- 代码重构：utils 模块拆分，日志统一为 `mais_art.*` 命名空间

### v3.3.3
- 移除基础中文转英文功能，完全依赖提示词优化器
- 优化 OpenAI 客户端响应处理
- 日志增强

### v3.3.1
- 新增砂糖云(NovelAI)、梦羽AI、Zai(Gemini转发) API 格式
- 提示词优化器、自动撤回、聊天流独立配置
- 管理员命令扩展

### v3.2.0
- 自拍模式（40+ 手部动作库）
- 自然语言命令、模型指定、Gemini 尺寸配置
- 智能降级（不支持图生图时自动转文生图）

### v3.1.2
- 智能文生图/图生图自动识别
- 命令式配置管理、风格别名系统、动态模型切换

---

## 贡献和反馈

欢迎提交 Issue 和 Pull Request！

## 版权信息

- 作者：Ptrel, Rabbit-Jia-Er, saberlights Kiuon
- 许可证：GPL-v3.0-or-later
