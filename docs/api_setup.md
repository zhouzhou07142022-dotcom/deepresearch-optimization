# API Setup Guide

> Deep Research Agent 项目 · 外部服务接入说明  
> 维护人：Yulan Pei ｜ 更新日期：2026-09-28  
> 用途：任意一台机器按本文档即可完成外部服务接入，无需翻源码找 key。

---

## 0. 一句话结论

**最小可跑通推理，只需要 3 样东西：**

1. `SERPER_KEY_ID` —— Serper.dev 的搜索 key
2. `JINA_API_KEYS` —— Jina.ai 的网页读取 key
3. `API_KEY` + `API_BASE` —— OpenAI 兼容接口（用 OpenRouter 即可，无需 GPU）

其余（Dashscope、SandboxFusion、MODEL_PATH）按需选配。

---

## 1. API 总览表

| 环境变量                      | 服务                  | 用途                      | 必选 | 注册地址                           | 免费额度      |
| ------------------------- | ------------------- | ----------------------- | -- | ------------------------------ | --------- |
| `SERPER_KEY_ID`           | Serper.dev          | Web 搜索 / Google Scholar | ✅  | <https://serper.dev>           | 有（新用户送额度） |
| `JINA_API_KEYS`           | Jina.ai             | 网页正文读取（Reader）          | ✅  | <https://jina.ai>              | 有（免费档）    |
| `API_KEY`                 | OpenAI / OpenRouter | 模型调用、页面摘要               | ✅  | <https://openrouter.ai>        | 有（送少量余额）  |
| `API_BASE`                | 同上                  | API 网关地址                | ✅  | 同上                             | —         |
| `DASHSCOPE_API_KEY`       | 阿里云百炼               | 文件解析（docmind）           | ⚠️ | <https://dashscope.aliyun.com> | 有         |
| `SANDBOX_FUSION_ENDPOINT` | SandboxFusion       | 代码执行沙箱                  | ⚠️ | 见 SandboxFusion 文档             | —         |
| `MODEL_PATH`              | 本地权重                | 本地推理                    | ❌  | HuggingFace / ModelScope       | —         |
| `DATASET`                 | 评测集                 | 评测数据                    | ⚠️ | —                              | —         |
| `OUTPUT_PATH`             | 结果目录                | 结果保存                    | ⚠️ | —                              | —         |

---

## 2. `.env` 完整示例

复制仓库的 `.env.example` 为 `.env` 后填写：

```bash
# 1) 搜索（必选）
SERPER_KEY_ID=你的_serper_key

# 2) 网页读取（必选）
JINA_API_KEYS=你的_jina_key

# 3) 模型调用（必选，走 OpenRouter）
API_KEY=sk-or-v1-xxxxxxxx
API_BASE=https://openrouter.ai/api/v1

# 4) 文件解析（可选）
DASHSCOPE_API_KEY=你的_dashscope_key

# 5) 代码沙箱（可选）
SANDBOX_FUSION_ENDPOINT=http://your-sandbox:8080

# 6) 本地推理（可选，用 OpenRouter 则留空）
MODEL_PATH=

# 7) 评测（评测时填）
DATASET=
OUTPUT_PATH=
```

> 注意：`.env` 已被 gitignore，密钥不会提交进仓库。

---

## 3. 各 API 注册与拿 key 步骤

### 3.1 Serper.dev（搜索）

1. 打开 <https://serper.dev> 注册账号
2. 登录后进入 Dashboard → API Key 页面
3. 复制 key，填入 `SERPER_KEY_ID`

### 3.2 Jina.ai（网页读取）

1. 打开 <https://jina.ai> 注册（可用邮箱或 Google 登录）
2. 进入 API Keys 页面创建 key
3. 复制 key，填入 `JINA_API_KEYS`

### 3.3 OpenRouter（模型调用，替代 OpenAI）

1. 打开 <https://openrouter.ai> 注册并充值少量余额（或用新用户赠送额度）
2. 进入 Keys 页面创建 API Key
3. 填入 `API_KEY`，`API_BASE` 设为 `https://openrouter.ai/api/v1`
4. 模型名使用 `alibaba/tongyi-deepresearch-30b-a3b`
5. 需同步修改 `inference/react_agent.py` 的 `call_server`：
   - API key / URL → 换成 OpenRouter 的
   - 模型名 → `alibaba/tongyi-deepresearch-30b-a3b`
   - 内容拼接 → 按 88–90 行注释调整

### 3.4 Dashscope（文件解析，可选）

1. 打开 <https://dashscope.aliyun.com> 用阿里云账号登录
2. 开通百炼服务，进入 API-KEY 管理创建 key
3. 填入 `DASHSCOPE_API_KEY`

---

## 4. 验证命令

```bash
# 验证 key 已加载
python -c "from dotenv import load_dotenv; import os; load_dotenv(); print('SERPER:', bool(os.getenv('SERPER_KEY_ID'))); print('JINA:', bool(os.getenv('JINA_API_KEYS'))); print('API_KEY:', bool(os.getenv('API_KEY')))"

# 验证 OpenRouter 连通性（示例）
curl -s https://openrouter.ai/api/v1/models -H "Authorization: Bearer $API_KEY" | head -c 200
```

三个 `True` 且 curl 返回模型列表，即代表外部服务接入成功。

---

## 5. 常见坑

- **`JINA_API_KEYS` 是复数**：变量名末尾带 `S`，别写成单数。
- **OpenRouter 的 `API_BASE` 要以 `/api/v1` 结尾**。
- **`API_KEY` 前缀**：OpenRouter 的 key 一般以 `sk-or-v1-` 开头。
- **`.env` 不要提交**：确认在 `.gitignore` 内。
