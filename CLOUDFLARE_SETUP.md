# 孙笑川 AI · 部署与双模式说明

## 当前线上状态（已部署完成 ✅）

| 项目 | 值 |
|---|---|
| 网站 | https://1394641925.github.io/sunxiaochuan/ |
| 默认接入 | **浏览器直连硅基流动** `https://api.siliconflow.cn/v1/chat/completions` |
| 模型 | `THUDM/GLM-4-9B-0414`（硅基流动，输入/输出均 ¥0，免费） |
| 备用 Worker | `https://sunxiaochuan-ai.sunxiaochuan666.workers.dev`（藏 Key 方案，国内可能被污染） |
| 额度 | 走孙哥的免费 Key，访客直接用，不花访客的钱 |

**默认效果**：任何人打开网站 → 点悬浮球对线，直接就是真 AI，不需要任何配置。

> **为什么默认直连而不是 Worker 中转？** `workers.dev` 域名在中国大陆常被 DNS 污染（解析到 Facebook 的 IP），导致网页连不上 Worker。而硅基流动 API 在国内可正常直连、且支持浏览器跨域（`Access-Control-Allow-Origin: *`），更稳。代价是 Key 写在前端——但这是**免费模型**，刷也刷不出钱，交作业完全够用。想藏 Key 可切到 Cloudflare 中转模式。

---

## 双模式：访客可选自带 Key

网站聊天室顶部有「孙哥的AI ⚙」按钮，点开可切换：

1. **孙哥的免费 AI（默认）**：什么都不用填，走内置 Worker + 免费模型。
2. **自带 Key 直连**：接入方式选「硅基流动」或「DeepSeek」，填自己的 Key，走自己的额度（免口令）。
3. **自带 Key 走中转**：接入方式保持默认，只在「自带 API Key」里填自己的 Key，会透传给 Worker 覆盖默认值，同样走自己额度、免口令。
4. **演示模式**：接入方式选「演示模式」，离线词库兜底，不联网也能聊。

配置随时可点「恢复默认」一键回到孙哥的免费 AI。

---

## 原理

```
默认：访客网页 → Worker（藏 Key + 验口令）→ 硅基流动免费模型
自带Key：访客网页 → Worker（透传自带Key，免口令）→ 硅基流动
直连：  访客网页 → 硅基流动/DeepSeek（浏览器直连自己的Key）
```

Key 不写在网页前端，默认 Key 只存在 Worker 里；访客看不到真 Key。自带 Key 由访客自己在配置框填写，存在其浏览器 localStorage。

---

## 自己重新部署 Worker（可选）

如果哪天 Key 失效或想换模型，按下面来：

1. 打开 https://dash.cloudflare.com → Workers 和 Pages → 创建 Worker。
2. 编辑代码，全选删掉，粘贴仓库里的 `worker.js` 完整内容。
3. 改开头三个变量：`API_KEY`（硅基流动 Key）、`ACCESS_PASS`（口令）、`MODEL`（模型 ID）。
4. 点部署，复制形如 `https://xxx.xxx.workers.dev` 的地址。
5. 网站配置里把「API 地址」换成新地址、口令填一样的即可。

> 模型 ID 去硅基流动「模型广场」搜，选标价为 `¥0.000000` 的免费模型，复制它的模型 ID（形如 `THUDM/GLM-4-9B-0414`）。注意：价格详情里 `free-text-model.online.output-tokens` 那串是**计费项代码**，不是模型 ID，别复制错。

---

## 安全说明

- 默认 Key 藏在 Worker，前端看不到；就算有人拿到 Worker 地址，口令错误也调不动。
- Worker 强制指定模型 + `max_tokens=512` 限制，别人拿不到贵模型。
- 口令属于发给演示对象的「自己人」范围；真要面向公网还需加限流/域名白名单，交作业这个级别够用。
- 自带 Key 的访客走自己额度，不消耗孙哥的免费额度。

---

## 免费额度参考

- 硅基流动：`THUDM/GLM-4-9B-0414` 等模型当前输入/输出均免费，交作业绰绰有余。
- Cloudflare Worker：免费版每天 10 万次请求，作业场景用不完。
