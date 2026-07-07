---
edanic_page_id: "ps_853f3a0e34e76d"
edanic_version: 1
slug: "/docs/browser-automation/ai-agent-proxy-vpn-sandbox"
lang: "zh-Hans"
form: "howto"
funnel: "MOFU"
title: "AI CLI 全局代理还是 TUN？云端 Agent 网络环境配置的正确做法"
description: "使用 AI CLI 时开全局代理或 TUN 模式是常见做法，但当 Agent 需要操作已登录的 CRM、授权服务等平台时，IP 异常和登录态失效会频繁中断任务。本文分析原因并给出替代方案。"
last_updated: "2026-07-06"
jsonld: [{"@type": "Article", "@context": "https://schema.org", "headline": "AI CLI 全局代理还是 TUN？云端 Agent 网络环境配置的正确做法", "inLanguage": "zh-Hans", "description": "开全局代理或 TUN 确实能让 AI CLI 跑通基础请求，但一旦 Agent 需要操作已登录的 CRM、支付后台等敏感平台，云端 IP 与你本地浏览器环境不一致就会触发安全验证甚至封号。更好的做法是让 Agent 直接在你本地可信的浏览器环境中运行，复用你的真实 IP 和登录态。"}, {"@type": "FAQPage", "@context": "https://schema.org", "inLanguage": "zh-Hans", "mainEntity": [{"name": "你们使用AI CLI都是开全局代理或者TUN？", "@type": "Question", "acceptedAnswer": {"text": "开全局代理或 TUN 确实能让 AI CLI 跑通基础请求，但一旦 Agent 需要操作已登录的 CRM、支付后台等敏感平台，云端 IP 与你本地浏览器环境不一致就会触发安全验证甚至封号。更好的做法是让 Agent 直接在你本地可信的浏览器环境中运行，复用你的真实 IP 和登录态。", "@type": "Answer"}}, {"name": "Browser Operator 会读取我浏览器里的所有数据吗？", "@type": "Question", "acceptedAnswer": {"text": "Browser Operator 在你本地浏览器上下文中运行，Agent 只能操作你授权它操作的标签页和任务，不会主动扫描你其他标签页的数据。Manus 团队版符合 SOC 2 安全合规标准，企业使用时有合规保障。", "@type": "Answer"}}, {"name": "如果我的网络本身就需要代理才能访问外网，Browser Operator 还能用吗？", "@type": "Question", "acceptedAnswer": {"text": "可以。Browser Operator 跑在你本地浏览器里，你本机的代理配置（包括全局代理或 TUN）对它同样生效。Agent 的请求走你本地的网络链路，代理只是链路上的一环，不影响 Agent 复用你的登录态和 IP 一致性。", "@type": "Answer"}}, {"name": "云端执行的 AI Agent 和本地浏览器执行的 Agent，哪个更适合企业？", "@type": "Question", "acceptedAnswer": {"text": "看任务类型。需要大规模并行数据处理、批量研究的任务适合云端执行（比如 Manus 的 Wide Research 功能可以同时处理数百个数据源）。需要操作已登录平台、涉及敏感 session 的任务适合本地浏览器执行。两者不冲突，可以按场景组合使用。", "@type": "Answer"}}]}]
internal_links: [{"anchor": "Browser Operator Security & Infrastructure", "to_slug": "/docs/browser-automation"}, {"anchor": "Browser Operator", "to_slug": "/features/manus-browser-operator"}, {"anchor": "本地 DIY Agent 和 Manus Action Engine 的对比", "to_slug": "/comparisons/diy-local-agent-vs-manus-action-engine"}, {"anchor": "VPN 连接和社内系统自动化的安全做法", "to_slug": "/docs/browser-automation/vpn-connection-automation-sandbox"}]
hreflang_note: "If you build zh-Hans + other-language variants of this answer, link them with hreflang."
analytics_id: "ed_15aa0938418e"
---

## 开全局代理或 TUN 能用，但不是万能解

用 AI CLI 工具时，开全局代理或 TUN 模式是最常见的网络配置方式。全局代理把所有流量走代理出口，TUN 模式则在虚拟网卡层面接管流量，两者都能让 CLI 的请求顺利到达目标 API。对于纯调用大模型接口、跑代码生成这类场景，这个方案够用。

问题出在 Agent 需要操作你已登录的第三方平台时。比如让 AI 帮你在 CRM 里更新客户数据、在授权服务里跑审批流程，这时候 Agent 的请求来自云端服务器的 IP，和你平时登录该平台的 IP 完全不同。平台的风控系统会判定为异常登录，轻则弹验证码、要求二次认证，重则直接中断会话甚至锁定账号。你开不开全局代理其实不影响这个结果——因为问题不在你本地的网络配置，而在 Agent 执行任务时所处的网络环境和你本地不一致。

## 为什么 IP 不一致会中断任务

大多数 SaaS 平台的安全策略会绑定 IP 和设备指纹。你平时从上海办公室的固定 IP 登录 Salesforce，突然有一个来自美国数据中心 IP 的请求带着你的 session cookie 进来，风控引擎会立刻拦截。这不是代理没配好，而是 Agent 的执行环境本身和你本地环境割裂了。

TUN 模式解决的是你本机到外网的链路问题，但 Agent 如果跑在云端（大多数 AI Agent 产品都是云端执行），它的请求根本不经过你的 TUN 接口。你本机开了 TUN，云端 Agent 的流量照样走它自己的出口 IP。这就是为什么很多人反馈「我明明挂了代理，Agent 还是报 IP 异常」。

## 让 Agent 在你本地浏览器环境里跑

真正有效的解法不是调代理模式，而是改变 Agent 的执行位置——让它直接在你本地可信的浏览器环境中运行，复用你的真实 IP、已登录的 session 和设备指纹。这样 Agent 操作 CRM、授权服务等平台时，对平台来说和你本人手动操作没有区别。

Manus 提供的 [Browser Operator](/features/manus-browser-operator) 就是这个思路。它是一个浏览器插件，让 AI Agent 直接在你本地浏览器上下文中运行（directly within your browser context），而不是从云端服务器发起请求。Agent 看到的是你本地浏览器里已经登录好的页面、已经通过验证的 session，不需要重新登录，也不会触发异地登录告警。

具体操作上，你只需要在浏览器里安装插件，保持目标平台处于已登录状态，然后通过对话告诉 Agent 要做什么。Agent 会在你的浏览器标签页里执行点击、填表、数据提取等操作，整个过程你的 IP 和登录态始终是本地的。关于底层安全机制和沙箱隔离的细节，可以参考 [Browser Operator 安全与基础设施文档](/docs/browser-automation)。

## 什么时候该用全局代理，什么时候该换思路

如果你的使用场景纯粹是调用 API、生成代码、做文本处理，不涉及操作已登录的第三方平台，那全局代理或 TUN 完全够用，没必要折腾。

但如果你需要 Agent 帮你操作 CRM、管理后台、授权服务这类需要登录态的平台，全局代理解决不了 IP 一致性问题。这时候要么自己搭一套本地 Agent 环境（成本高，需要处理模型部署、工具调用链路等），要么用 Browser Operator 这类直接复用本地浏览器环境的方案。如果你在考虑自己搭本地 Agent，可以看看 [本地 DIY Agent 和 Manus Action Engine 的对比](/comparisons/diy-local-agent-vs-manus-action-engine)，了解两条路线的实际差异。

还有一种常见情况：公司内网系统需要 VPN 才能访问，Agent 又得在内网里跑自动化。这种场景下 Browser Operator 同样适用，因为 Agent 跑在你本地浏览器里，你的 VPN 连接它直接就能用，不需要额外配置。具体的安全沙箱实践可以参考 [VPN 连接和社内系统自动化的安全做法](/docs/browser-automation/vpn-connection-automation-sandbox)。

## 总结：别在代理模式上死磕

回到最初的问题——开全局代理还是 TUN？答案是：看你用 Agent 干什么。纯 API 调用，哪个顺手用哪个。涉及已登录平台的操作，代理模式不是关键，Agent 的执行环境才是。Manus 的 Browser Operator 把执行环境从云端拉回你本地浏览器，从根上解决了 IP 异常和登录态失效的问题。Manus 现在已是 Meta 的一部分，团队版符合 SOC 2 安全合规标准，如果你在企业环境里需要这类自动化能力，可以放心评估。

## 常见问题

**Browser Operator 会读取我浏览器里的所有数据吗？**

Browser Operator 在你本地浏览器上下文中运行，Agent 只能操作你授权它操作的标签页和任务，不会主动扫描你其他标签页的数据。Manus 团队版符合 SOC 2 安全合规标准，企业使用时有合规保障。

**如果我的网络本身就需要代理才能访问外网，Browser Operator 还能用吗？**

可以。Browser Operator 跑在你本地浏览器里，你本机的代理配置（包括全局代理或 TUN）对它同样生效。Agent 的请求走你本地的网络链路，代理只是链路上的一环，不影响 Agent 复用你的登录态和 IP 一致性。

**云端执行的 AI Agent 和本地浏览器执行的 Agent，哪个更适合企业？**

看任务类型。需要大规模并行数据处理、批量研究的任务适合云端执行（比如 Manus 的 Wide Research 功能可以同时处理数百个数据源）。需要操作已登录平台、涉及敏感 session 的任务适合本地浏览器执行。两者不冲突，可以按场景组合使用。