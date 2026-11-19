# Cloudflare Worker 中转部署教程（真 AI + Key 不泄露）

把 API Key 藏在 Cloudflare Worker 里，网页只连 Worker，不接触真 Key。别人拿到你的网址也刷不了你的额度（还能加口令双保险）。全程免费，10 分钟搞定。

## 原理一句话

```
访客网页 → Cloudflare Worker（藏 Key + 验口令）→ 硅基流动/DeepSeek
```

Key 只写在 Worker 里，访客看不到；Worker 地址是公开的，但你可以设一道口令，没口令的人用不了。

---

## 第一步：注册 Cloudflare

1. 打开 https://dash.cloudflare.com/sign-up ，用邮箱注册（免费）。
2. 进入控制台后，左侧点 **Workers 和 Pages** → **Worker** → **创建 Worker**。
3. 名字随便填，比如 `sun-bot`，点部署。

---

## 第二步：粘贴 Worker 代码

创建后点 **编辑代码**，把里面所有内容删掉，粘贴下面这段：

```js
// ===== 在这里填你的配置 =====
const API_KEY = 'sk-你的真实Key';                       // 硅基流动的 Key
const UPSTREAM = 'https://api.siliconflow.cn/v1/chat/completions'; // 上游地址
const ACCESS_PASS = 'sunxiaochuan666';                  // 访问口令，改成你自己的
const MODEL = 'Qwen/Qwen2.5-7B-Instruct';               // 免费模型
// ============================

export default {
  async fetch(request) {
    // 只允许跨域从你的网站访问
    const cors = {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'POST, OPTIONS',
      'Access-Control-Allow-Headers': 'Content-Type, X-Access-Pass'
    };
    if (request.method === 'OPTIONS') return new Response(null, { headers: cors });
    if (request.method !== 'POST') return new Response('Only POST', { status: 405, headers: cors });

    // 校验口令：前端配置里填的口令要和后面对上
    if (ACCESS_PASS && request.headers.get('X-Access-Pass') !== ACCESS_PASS) {
      return new Response(JSON.stringify({ error: { message: '口令错误，勿cue' } }), {
        status: 403, headers: { ...cors, 'Content-Type': 'application/json' }
      });
    }

    let body;
    try { body = await request.json(); } catch (e) {
      return new Response(JSON.stringify({ error: { message: '请求格式错误' } }), {
        status: 400, headers: { ...cors, 'Content-Type': 'application/json' }
      });
    }
    // 强制使用 Worker 里指定的模型，防止别人拿你的 Key 跑贵模型
    body.model = MODEL;

    const upstream = await fetch(UPSTREAM, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer ' + API_KEY
      },
      body: JSON.stringify(body)
    });
    const text = await upstream.text();
    return new Response(text, { status: upstream.status, headers: { ...cors, 'Content-Type': 'application/json' } });
  }
};
```

改完这 4 个变量：
- `API_KEY`：你的真实 Key（硅基流动申请：https://cloud.siliconflow.cn → 账户管理 → API 密钥，有免费额度）。
- `ACCESS_PASS`：访问口令，自己随便设一串，网页里要填一样的。
- `UPSTREAM` / `MODEL`：用 DeepSeek 就改成 `https://api.deepseek.com/chat/completions` 和 `deepseek-chat`（DeepSeek 是收费的，注意余额）。

点右上角 **部署 (Deploy)**。

---

## 第三步：拿到 Worker 地址

部署后，顶部会显示你的 Worker 网址，形如：

```
https://sun-bot.你的子域名.workers.dev
```

复制这个地址。

---

## 第四步：网页里配置

1. 打开你的网站 → 点右下角孙笑川悬浮球 → 顶部「演示模式 ⚙」进入配置。
2. 接入方式选 **Cloudflare Worker（中转）**。
3. API 地址：粘贴刚才的 Worker 地址。
4. 访问口令：填 Worker 代码里 `ACCESS_PASS` 的值。
5. API Key：**留空**（Key 在 Worker 里，网页不需要）。
6. 模型：留空（Worker 里已强制指定）。
7. 保存。看到「已切换真 AI」就成了，直接对线测试。

---

## 关于安全（回答你担心的盗刷）

- **Key 写死在网页里**：任何人按 F12 都能看到，必被盗刷。❌
- **Key 放 Worker + 口令**：访客只看到 Worker 地址和口令，看不到 Key。就算有人拿到网址，没口令也调不动；有口令的人也只能用你指定的免费模型，跑不了贵模型。✅
- 万一口令泄露：去 Worker 里改 `ACCESS_PASS` 重新部署即可，Key 完全不用动。

> 口令是要发给演示对象的（比如老师），属于「自己人」范围。真正面向公网的产品还要加限流、域名白名单，但交作业这个级别，口令已经足够。

---

## 免费额度参考

- 硅基流动：注册送额度，Qwen2.5-7B-Instruct 等小模型长期免费，交作业绰绰有余。
- Cloudflare Worker：免费版每天 10 万次请求，作业场景用不完。

两个都免费，不用绑卡（Cloudflare 注册可能会让你填卡，但免费版不扣钱，也可以跳过）。
