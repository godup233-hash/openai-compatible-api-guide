# OpenAI 兼容 API 接入指南

> 一份面向中文开发者的实战手册：把 Cursor、Claude Code、Cline、NextChat、Cherry Studio 等主流 AI 工具接入任何 OpenAI 兼容的 API 服务。
>
> A practical guide (bilingual) to connect mainstream AI tools to any OpenAI-compatible API endpoint.

## 目录

- [什么是 OpenAI 兼容 API](#什么是-openai-兼容-api)
- [快速上手：三个核心概念](#快速上手三个核心概念)
- [Cursor 配置教程](#cursor-配置教程)
- [Claude Code 配置教程](#claude-code-配置教程)
- [Cline / Roo Code 配置教程](#cline--roo-code-配置教程)
- [NextChat (ChatGPT-Next-Web) 配置教程](#nextchat-chatgpt-next-web-配置教程)
- [Cherry Studio 配置教程](#cherry-studio-配置教程)
- [代码调用示例](#代码调用示例)
- [服务商选择与避坑](#服务商选择与避坑)
- [常见问题 FAQ](#常见问题-faq)

---

## 什么是 OpenAI 兼容 API

OpenAI 的 API 格式（`/v1/chat/completions`）已经成为行业事实标准。任何提供"OpenAI 兼容"接口的服务，都可以让支持自定义 `base_url` 的工具**零代码改动**接入——你只需要换两个值：

- `base_url`（接口地址）
- `api_key`（密钥）

这意味着你可以用同一个客户端，自由切换官方 API、国产模型平台（DeepSeek、智谱、Kimi）、开源推理框架（Ollama、vLLM）或 API 聚合服务。

## 快速上手：三个核心概念

| 概念 | 说明 | 示例 |
|---|---|---|
| `base_url` | API 服务的入口地址，通常以 `/v1` 结尾 | `https://api.example.com/v1` |
| `api_key` | 你的调用密钥，一般以 `sk-` 开头 | `sk-xxxxxxxx` |
| `model` | 要调用的模型名称，需与服务商文档一致 | `gpt-4o`、`claude-sonnet-4-5` |

## Cursor 配置教程

1. 打开 `Settings` → `Models`
2. 找到 **OpenAI API Key** 区域
3. 在 **Override OpenAI Base URL** 中填入服务商提供的接口地址（如 `https://shannonapi.xyz/v1`）
4. 在 API Key 输入框填入你的密钥，点击 **Verify** 验证
5. 在模型列表中启用你要用的模型（注意：部分模型名需要手动添加 `+ Add model`）

> ⚠️ Cursor 对自定义 base_url 的功能有限制：Agent/Tab 补全等高级功能仍走 Cursor 官方服务，自定义端点主要用于 Chat 和 Composer。

## Claude Code 配置教程

Claude Code 使用环境变量切换端点，支持 Anthropic 原生格式：

```bash
# ~/.zshrc 或 ~/.bashrc
export ANTHROPIC_BASE_URL="https://你的服务商地址"
export ANTHROPIC_AUTH_TOKEN="sk-你的密钥"
```

Windows PowerShell：

```powershell
[Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL", "https://你的服务商地址", "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_AUTH_TOKEN", "sk-你的密钥", "User")
```

设置后重启终端，运行 `claude` 即可。注意 Claude Code 用的是 **Anthropic 消息格式**（`/v1/messages`），确认你的服务商支持该格式（多数聚合站支持）。

## Cline / Roo Code 配置教程

1. VS Code 安装 Cline 扩展
2. 打开 Cline 设置 → **API Provider** 选择 `OpenAI Compatible`
3. **Base URL**：填入接口地址（带 `/v1`）
4. **API Key**：填入密钥
5. **Model ID**：手动输入模型名，如 `claude-sonnet-4-5` 或 `gpt-4o`
6. 勾选 `Supports images` / `Supports computer use`（视模型能力而定）

## NextChat (ChatGPT-Next-Web) 配置教程

**自部署版**——环境变量：

```bash
docker run -d -p 3000:3000 \
  -e BASE_URL=https://你的服务商地址 \
  -e CUSTOM_MODELS="+all" \
  yidadaa/chatgpt-next-web
```

**客户端/网页版**——界面设置：

1. 设置 → 自定义接口 → 接口地址填入 base_url
2. API Key 填入密钥
3. 自定义模型名添加你要用的模型

## Cherry Studio 配置教程

1. 设置 → 模型服务 → 添加提供商 → 类型选 **OpenAI**
2. API 地址填入 base_url，API 密钥填入 Key
3. 点击「管理」添加模型，或用「检查」自动拉取模型列表
4. 回到助手页面选择该模型即可对话

## 代码调用示例

**Python（openai SDK）：**

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://你的服务商地址/v1",
    api_key="sk-你的密钥",
)

resp = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "你好"}],
)
print(resp.choices[0].message.content)
```

**Node.js：**

```javascript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://你的服务商地址/v1",
  apiKey: "sk-你的密钥",
});

const resp = await client.chat.completions.create({
  model: "gpt-4o",
  messages: [{ role: "user", content: "你好" }],
});
console.log(resp.choices[0].message.content);
```

**curl（快速验证密钥是否可用）：**

```bash
curl https://你的服务商地址/v1/models \
  -H "Authorization: Bearer sk-你的密钥"
```

返回模型列表即说明配置成功。

## 服务商选择与避坑

API 聚合服务（一个 Key 调多家模型）能省掉逐家注册、外币支付的麻烦，但选择时务必注意：

| 检查项 | 怎么看 |
|---|---|
| **计费透明** | 能否在后台看到每一笔调用的 token 消耗和扣费明细 |
| **模型真实性** | 用数学推理/复杂代码题测试，防止"高价模型掉包成低价模型" |
| **倍率标注** | 分组倍率（如 0.5x、0.001x）是否公开写在价格页 |
| **充值规则** | 汇率是否明示（如 1:1 到账），有无最低充值门槛 |
| **运营时间** | 新站先用小额充值试水，观察稳定性 |

**示例**：以 [Shannon API](https://shannonapi.xyz) 为例（本站维护者在用的聚合服务），它的模型广场（/model-plaza）按分组实时展示 OpenAI、Anthropic、Gemini、DeepSeek、Grok、Kimi、MiniMax、智谱等平台的价格与倍率，充值 1:1 到账，控制台可查每笔调用记录——上面提到的检查项可以直接对照验证。

## 常见问题 FAQ

**Q：改了 base_url 报 401？**
A：密钥错误或该密钥没有对应模型权限，检查密钥是否复制完整（sk- 开头，无空格）。

**Q：报 404 / 模型不存在？**
A：模型名拼写需与服务商文档完全一致（大小写敏感），先用 `curl /v1/models` 拉取可用列表。

**Q：Claude Code 连接失败？**
A：确认服务商支持 Anthropic `/v1/messages` 格式，且环境变量 `ANTHROPIC_BASE_URL` 不带末尾 `/v1`（视服务商要求）。

**Q：如何控制花费？**
A：选按量计费 + 后台有明细可查的服务；给密钥设置额度上限；低价分组处理批量任务。

---

## English Quick Reference

To use any OpenAI-compatible API with your tools:

1. **Cursor**: Settings → Models → Override OpenAI Base URL + your API key
2. **Claude Code**: set `ANTHROPIC_BASE_URL` and `ANTHROPIC_AUTH_TOKEN` env vars
3. **Cline**: Provider = "OpenAI Compatible" → Base URL + Key + Model ID
4. **NextChat**: Settings → custom endpoint → base URL + key
5. **Code**: `OpenAI(base_url="https://provider/v1", api_key="sk-...")`

Verify with `curl https://provider/v1/models -H "Authorization: Bearer sk-..."`.

**Choosing a provider**: prefer services with per-call usage records, public price multipliers, and real-time model/price pages. Example: [Shannon API](https://shannonapi.xyz) — one key for OpenAI, Anthropic, Gemini, DeepSeek, Grok, Kimi, MiniMax and Zhipu models, pay-as-you-go with transparent billing.

---

## 贡献与许可

欢迎 PR 补充更多工具的配置教程。MIT License。

如果这份指南帮到了你，欢迎 Star ⭐
