# astrbot_plugin_index_tts（fork 增强版）

> 本 fork（[xiaoyuyu6420/astrbot_plugin_index_tts](https://github.com/xiaoyuyu6420/astrbot_plugin_index_tts)）基于 [xiewoc/astrbot_plugin_index_tts](https://github.com/xiewoc/astrbot_plugin_index_tts) v1.0.4，配合自研后端 [xiaoyuyu6420/indextts2-astrbot-api](https://github.com/xiaoyuyu6420/indextts2-astrbot-api)（IndexTTS 2.0 整合包 API 服务）使用。

## 本 fork 的改动（v1.0.6）

1. **合成超时可配置**（`serve_config.request_timeout`，默认 600 秒）——原版硬编码 120 秒，长文本在消费级显卡（如 RTX 3070 低显存模式）上必超时；
2. **服务端口可配置**（`serve_config.server_port`，默认 5210）——原版硬编码 5210；
3. **新增生成参数面板配置**：`generation` 组新增 语速 `speaking_speed`（0.5~1.5）与采样参数 `do_sample / top_p / top_k / temperature / repetition_penalty / length_penalty / num_beams / max_mel_tokens`，启动时随 /config 推送给后端；默认值与 IndexTTS webui 一致，原版 service.py 会自动忽略这些字段（互不冲突）；
4. **全 0 情感向量不再推送**——原版总是推送 `[0]*8`，会让后端进入向量模式、丢失音色参考音频自带的情感；
5. 修复 register 元数据里的仓库地址笔误（原版指向 spark_tts 仓库）；
6. **复读指令 `/tts_say_it <内容>`**（v1.0.6 新增）：机器人一字不差地念出你输入的内容（不经过 LLM），同时发送语音消息和 wav 文件。权限由 `custom` 组控制：`say_admin_only`（默认开，仅群管理/AstrBot 管理员）、`say_whitelist`（额外放行的 QQ 号）、`say_send_wav_file`（是否附带 wav 文件，默认开）。

## 推荐部署方式（免下载模型）

如果显卡机器上有 **IndexTTS 2.0 整合包**（yzylauncher 系列），推荐分离部署：

1. 显卡机器：部署 [indextts2-astrbot-api](https://github.com/xiaoyuyu6420/indextts2-astrbot-api)（复用整合包自带模型，5210 端口）；
2. AstrBot：安装本插件，`serve_config.if_seperate_serve = true`，`server_ip` 填显卡机器 IP；
3. 音色放插件 `sounds/` 或显卡机器整合包 `voices/`（后端会自动按文件名回退解析）。

---

# 前言

本插件是基于index-tts对AstrBot的语音转文字(TTS)补充

建议使用 `Conda` 构建本插件的虚拟环境

需要FFmpeg在系统路径下

## 性能要求

建议显卡显存大于4Gib(IndexTTS 1, 1.5),或12Gib(IndexTTS 2)

>[!TIP]
>建议`AMD`用户使用`WSL`或`Linux`以使用`torch-ROCm`

## 环境配置

根据[Index TTS 官方文档](https://github.com/index-tts/index-tts)配置环境

...配置虚拟环境...

建议使用官方提供的方式安装

```bash
uv sync --extra webui --default-index "https://mirrors.aliyun.com/pypi/simple"
```

>[!IMPORTANT]
>目前还不知道怎么将两个uv的环境合并
>
>故建议如果出现问题先转用分隔式服务

如果发生版本冲突，以报错为准去下载合适版本的库

>[!TIP]
>若使用 `uv` 配置环境，则需使用
>```bash
>cd /path/to/AstrBot
>uv pip install xxx
>```
>现已不推荐使用 `Conda` 作为虚拟环境

>[!WARNING]
>本插件目前仅在python3.10上测试，如果为Python >=`3.11` && <=`3.12`，可能不稳定

## `AstrBot` 内配置

添加`OpenAI`的tts适配器

apikey默认`1145141919810`

超时建议`30`~`60`s

其他默认便可

## 插件自定义

插件目前可自定义：apikey、音频输入文件

音频输入文件在`path/to/the/plugin/sounds`下，自定义时仅用填文件全名即可

如需修改端口号,可在 `main.py` 和 `service.py` 中 找到 
`port = xxx`
字样，更改即可，但注意不要更改 `port` 的 `数据类型`

# 使用

## 正常使用

开启/关闭TTS功能：

```cli
/tts
```

更换源语音文件：

```cli
/tts_cfg_it set voice xxx.wav
```

>[!NOTE]
>此处 `/` 为你的自定义唤醒词，会根据你的自定义而改变
>
>此处使用的`xxx.wav`不需要带文件路径

## 作为 `LLM Tool` 调用 

建议在提示词之中加入对于使用`LLM Tool`(A.K.A. `Function Calling`) 的提示

其余便不用管了

>[!NOTE]
>Index TTS目前仅支持中英两种语言，不支持俄语和日语及其他
>
>如有跨语言需求，请使用其他的TTS模型

# 后记

时间原因，文档写的较潦草，如有不懂或报错，请issue

本次(1.0.4)更新属实匆忙，如果偶遇任何bug，请issue

如果有想法，也欢迎issue

如果你认为这个插件对你有帮助，请star

# 版本历史

`1.0.0` -> `1.0.1`:

    (1)添加了LLM工具
    (2)使用了线程池进行语音生成
    (3)优化逻辑

`1.0.1` -> `1.0.2`:

    (1)添加了随机TTS功能
    (2)添加命令以实时更改语音源文件

`1.0.2` -> `1.0.3`:

    (1)添加了预加载模型的功能
    (2)优化异步与传参
    (3)优化仓库下载逻辑
    (4)添加去除emoji和分段的功能
    (5)更改临时文件储存地址

`1.0.3` -> `1.0.4`:

    (1)添加了对IndexTTS的支持

>[!NOTE]
>对于每次更新，建议将原先版本的配置文件删除后再更新，以防出现错误

# TODO

添加GitHub代理

添加对vllm版本的快捷使用


