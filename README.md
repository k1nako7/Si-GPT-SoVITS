# 祀 · GPT-SoVITS 语音克隆模型

《明日方舟：终末地》角色 **祀**（中文配音：幽舞越山，内部代号 `chr_0036_jsspsi`）的 GPT-SoVITS v2ProPlus 音色模型。

从游戏资源中提取全部中文语音，经切分、清洗、ASR 标注后训练而成。开箱即用，支持中文任意文本合成，可通过参考音频切换情绪。

> 音色与韵律仅供个人学习研究，角色语音版权归鹰角网络所有，请勿用于商业用途或冒充他人。

---

## 模型信息

| 项目 | 内容 |
| --- | --- |
| 框架版本 | GPT-SoVITS **v2ProPlus**（32kHz） |
| SoVITS (s2) | v2ProPlus + LoRA (rank=32) · 15 epoch / 705 iter |
| GPT (s1) | 基于 `s1v3` 底模微调 · 18 epoch / 558 iter |
| 训练集准确率 | top-3 **0.847**（s1） |
| 训练数据 | 184 段中文语音，约 **14.7 分钟** |
| 素材来源 | 游戏内角色语音：战斗、战斗指令、日常闲聊、赠礼、问候、台词独白 |
| 训练设备 | RTX 4060 Ti 8GB（SoVITS batch=4 / GPT batch=6） |
| 生成日期 | 2026-11-04 |

---

## 目录结构

```
.
├─ Si（新版）/
│  ├─ Si.ckpt              GPT 模型     「说什么」  148MB  (Git LFS)
│  ├─ Si.pth               SoVITS 模型  「音色」    165MB  (Git LFS)
│  └─ slicer_opt/          训练用切片   185 段 32kHz 单声道 wav
├─ 参考音频实例/            按语气分类的参考音频，推理时控制情绪
│  ├─ 日常闲聊/  问候/  赠礼/
│  ├─ 战斗/      战斗指令/  台词独白/
├─ 训练集文本.txt           训练标注，格式：路径|说话人|语言|文本
├─ 参考音频文本对照.txt      参考音频 ↔ 台词对照
├─ 试听样例_合成.wav        本模型合成示例（约 13 秒）
└─ 使用说明.txt             原版说明文档
```

> 仓库使用 **Git LFS** 存放 `.pth` / `.ckpt`。克隆前请先安装 [Git LFS](https://git-lfs.com/)：
> ```bash
> git lfs install
> git clone https://github.com/k1nako7/Si-GPT-SoVITS.git
> ```

---

## 快速开始

**1. 准备 GPT-SoVITS**

必须是带 **v2ProPlus** 的版本，例如 [GPT-SoVITS-v2pro-20250604](https://github.com/RVC-Boss/GPT-SoVITS)。旧版 v1/v2 环境无法加载本模型。

**2. 放置权重**

```
GPT-SoVITS根目录/
├─ GPT_weights_v2ProPlus/    ← 放 Si.ckpt
└─ SoVITS_weights_v2ProPlus/ ← 放 Si.pth
```

**3. 启动推理**

```bash
go-webui.bat
```

进入 `1-GPT-SoVITS-TTS` → `1C-inference` 页面，模型选择处分别选 `Si.ckpt` 与 `Si.pth`。

---

## 推理参数

| 参数 | 建议值 | 说明 |
| --- | --- | --- |
| 参考音频 | 3~8 秒 | 从 `参考音频实例/` 挑选，越贴合目标语气越好 |
| 参考文本 | **必填** | 必须是参考音频的原文，见 `参考音频文本对照.txt` |
| 参考语言 | 中文 | |
| 合成文本 | 中文 | |
| top_k | 15 | 默认 |
| top_p | 1 | |
| temperature | 1 | |

**参考文本写错会明显影响音色还原度**，这是最常见的踩坑点。

### 情绪参考选择

| 语气 | 适用场景 |
| --- | --- |
| 日常闲聊 | 最通用，建议作为默认参考 |
| 战斗 / 战斗指令 | 喊招、命令语气，适合激烈台词 |
| 问候 | 温和、带笑意 |
| 赠礼 | 轻松俏皮 |
| 台词独白 | 偏平静叙述 |

---

## 已知限制

1. **标注文本是 ASR 自动识别结果**，非官方台本，个别同音字、专有名词可能有误。已做过标点与异常字符清洗。
2. 素材中有较多 1~2 秒短句和战斗喊声，**长句自然度优于极短句**；合成短句时建议在文本中补足上下文。
3. 14.7 分钟属于「够用」级别，**情绪覆盖有限**。若要更贴近角色，可补充剧情对话语音（按章节混排、人工挑选）后重训。
4. 音色与韵律仅供个人学习研究，请勿商用或冒充他人。

---

## 复现信息

| 项目 | 路径 / 说明 |
| --- | --- |
| 训练脚本 | `prep_dataset.py` / `run_asr.py` / `clean_labels.py` / `make_s1_cfg.py` / `make_s2_cfg.py` |
| 训练日志 | `logs/s2_train2.log`、`logs/s1_train.log` |
| 实验目录 | `GPT-SoVITS根目录/logs/Si` |

### 训练参数概要

- **SoVITS**：15 epoch / 705 iter，`batch_size=4`（8GB 显存下的稳定上限，bs=6 会 OOM）
- **GPT**：18 epoch / 558 iter，`batch_size=6`，底模 `s1v3.ckpt`
- 数据预处理：去静音 → RMS 归一 → 超长音频按 10s 切分 → 185 段切片
