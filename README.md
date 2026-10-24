# Ecommerce scraping services：托管交付还是 API 自抓，看懂计费逻辑再选不踩坑

搜 ecommerce scraping services 的人，多半卡在同一个岔路口：一边是“你给清单、我交数据表”的托管服务商（Actowiz、ScrapeCrew 这类，按项目报价，ScrapeCrew 官方列的管线方案从 $900 起、视范围 $900–$3,000），另一边是抓取 API 工具——自己写代码调接口，按用量付费。两条路没有绝对优劣，但适合的人完全不同：如果你有开发能力、需要长期持续监控商品价格和库存，API 方案几乎总是更便宜也更可控；如果只要一次性拿某个类目的数据表，托管服务反而省事。

ScraperAPI 属于后者。它把代理轮换、JS 渲染、验证码处理封装成一个 API：你丢给它一个 URL 或 ASIN，它返回页面内容或解析好的 JSON。电商场景下它真正值钱的有两块：对 Amazon、Walmart、eBay 等站点有现成的结构化数据端点，不用自己写解析器；以及一套完全透明的 credit 计费体系——透明到能把真实成本算出来，这一点大部分对比文章都没做。

> **选购速览**：月抓取量百万级以内、目标以 Amazon/Walmart/eBay 为主的电商数据需求，ScraperAPI 的 Startup（$149/月，100 万 credits）或 Business（$299/月，300 万 credits）基本够用；年付统一 9 折，注册即有 7 天免费试用（5,000 credits，无需信用卡）。想直接对着价格表选，可以👉 [查看 ScraperAPI 全部套餐的当前价格](https://www.scraperapi.com/pricing/?fp_ref=coupons)。

## 托管服务和抓取 API，电商场景该选哪条路

结论先说：这个决定不由预算决定，由数据更新频率和团队构成决定。

托管服务商的典型做法在 ScrapeCrew 那类对比文章里写得很清楚：先 mapping 目标站点、字段、格式和刷新频率，再走 QA、部署、监控、重试、告警、交付的完整流程。Actowiz 宣称覆盖 5,000+ 零售商和 40+ 国家，刷新间隔最低 15 分钟，按月固定报价。适合没有开发团队、或者只需要一次数据交付的情况。

抓取 API 是自己写请求，服务方解决“怎么抓到”的问题。ScraperAPI 把复杂度压进了参数里：加 `render=true` 走无头浏览器，用结构化端点直接返回 JSON，用 DataPipeline 可以把重复任务排成定时抓取、输出 JSON 或 CSV。适合会写点代码、数据要每天甚至每小时刷新、要直接进自己系统的场景。

判断标准很简单：抓取是你产品流程的一部分、会长期运行——选 API；数据一锤子买卖、拿一次就用完——选托管。两边报价方式也没法直接比：托管按项目，API 按量。

## ScraperAPI 全套餐对比

官网目前展示 7 个付费套餐加免费计划，年付全部 9 折。所有付费套餐的功能列表一致（JS 渲染、住宅与移动代理、验证码处理、JSON 自动解析、DataPipeline、Full Crawler Access 等），差异集中在额度、并发和地域定位。

| 套餐 | 月付价 | 年付价（9 折） | Credits/月 | 并发数 | 订购 |
| --- | --- | --- | --- | --- | --- |
| Hobby | $49 | $44.10 | 100,000 | 20 | [ 开通 Hobby](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [ 开通 Startup](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [ 开通 Business](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [ 开通 Scaling](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [ 开通 Professional](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [ 开通 Advanced](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | 定制 | 定制 | 22,000,000+ | 500+ | [ 联系销售获取报价](https://www.scraperapi.com/contact-sales/?fp_ref=coupons) |

免费侧还有一层：注册送 1,000 credits（最高 5 并发），加上 7 天 5,000 credits 的试用，不花钱能把流程跑通再决定。官方口径是只为成功的结果付费，失败请求不计费——至少标准目标上，试错的成本很低。

月付和年付的差距不算戏剧性，就是 10%，但换算成绝对值：Startup 档一年省 $178.80，Business 省 $358.80。真正影响选档的从来不是标价，是下一节这笔账。

## Credit 系统的真实账本：多数对比文章没算的部分

一个 credit 等于一次标准页面请求——这是官方定义，但“标准”两个字后面藏着一套乘数。同样是抓一页，成本差得很远：

- 标准网页：1 credit
- Amazon 页面：5 credits
- Google 和 Bing（含子域）：25 credits
- LinkedIn：30 credits
- Cloudflare、Datadome、PerimeterX 防护站点：每请求额外加 10 credits

实际影响一下就出来了。Hobby 的 100,000 credits 听起来能抓 10 万页，目标要是 Amazon 商品页，实际只能抓 2 万页。有第三方评测把这个问题概括成"credits are not requests"——headline 数字和实际拿到的页面数之间，隔着一个量级。这个批评主要针对困难目标，常规站点的成本和官方口径基本对得上，但选套餐前自己算一遍总是没错的。

好在 ScraperAPI 把这套乘数完全公开，还给了两个控制工具。Dashboard 里的 Domain Cost Estimator 输入具体 URL 就能查出单次请求的真实消耗；请求里可以带 `max_cost` 参数，直接封顶单次消耗。也就是说，在选套餐之前你就能把账算清：拿免费额度测你真正要抓的站点，用 Cost Estimator 查出单页成本，倒推月需求量和对应套餐。这一步做完，选档基本不会错，👉 [用免费试用先算清单页成本再选套餐](https://www.scraperapi.com/?fp_ref=coupons)。

## Amazon / Walmart / eBay 场景：结构化端点省掉的是解析维护

电商抓取最贵的从来是解析，抓取本身反而便宜。页面结构一改，解析器就崩，维护成本随站点数量线性增长。ScraperAPI 的结构化数据端点（Structured Data Endpoints）就是冲这个来的：官方文档确认支持 Amazon、Google、Walmart、eBay、Redfin 五个站点，以 API key 加 ASIN 这类标识符调用固定端点，直接拿回解析好的 JSON——商品名、价格、评分、库存、卖家信息这些字段全在里面，页面结构变化由服务端维护，官方对支持域名宣称 99.99% 成功率。

端点颗粒度比很多文章写的细。Amazon 有 Product、Search、Offers 三个端点（Offers 那个专门拿第三方卖家的报价、状况和运费，做竞品价格监控用得上）；Walmart 有 Product、Search、Category、Reviews 四个；eBay 有 Product 和 Search。想不写代码，还能把端点接进 DataPipeline 或 Async Scraper，提交输入列表后数据走 webhook 回来。

成本上有个很多人不知道的好消息：端点的 credit 消耗跟该域名的核心 API 一样，按域名算，不按端点加价。也就是说 Walmart 端点每次请求只花 1 credit——标准价；Amazon 端点 5 credits，Google 端点 25 credits。全部套餐（包括 Hobby 和 Startup）都能用这些端点。

所以 SDE 划不划算，完全取决于你的目标站点：监控 Walmart 的话，解析免维护、credits 又是标准价，基本没有理由不用；Amazon 每次请求是标准页的 5 倍，目标站点结构稳定、只抓一种页面类型时自己写解析可能更省，跨站点多页面类型的监控场景则是长期更赚。这里还有个多数对比文章没讲清的边界：SDE 只覆盖上面列的站点，Shopify 这类自建站平台没有现成端点，自定义 URL（比如带参数的 Amazon 商品链接）官方推荐改走 Auto Parsing 功能。选购前最好对照官网的端点目录确认你的目标站点在不在列，👉 [确认结构化端点是否覆盖你的目标站点](https://www.scraperapi.com/?fp_ref=coupons)。

## 和 Bright Data、Apify、Oxylabs 比，ScraperAPI 处在什么位置

把几家有公开定价的工具放在一起看（价格来自 Extralt 2026 年 5 月核验的横向评测，是评测方的核验口径，未逐一在对方官网复核，下单前请自行确认）：

| 工具 | 价格信号 | 适合谁 |
| --- | --- | --- |
| ScraperAPI | Hobby $49/月（100K credits）、Startup $149/月（1M credits） | 要结构化电商数据、不想维护解析器的开发者 |
| Bright Data | Web Scraper API 按 $1.50/千条成功记录计；Scale 档 $499/月含 384K 记录 | 大规模公共网页采集、需要代理基础设施 |
| Apify | Starter $29/月、Scale $199/月、Business $999/月，外加按量付费 | 想跑现成 Actor 或自定义爬虫的用户 |
| Oxylabs | 电商 scraper $49/月起，按目标与 JS 渲染计价 | 企业级采集，高并发 API 工作负载 |
| Firecrawl | Standard $83/月（年付，100K credits），JSON 模式每页加 4 credits | 给 AI 应用喂干净的任意页面内容 |

再往上是 Zyte（API 优先的提取层，按目标复杂度计价）和纯托管路线（ScrapeCrew 从 $900 起、DataOx 按报价），那条路的买家不碰代码，维护全在服务方。

ScraperAPI 的位置比较好概括：SDE 覆盖到的站点你不用维护解析器，覆盖不到的自己写。相比按成功记录计费的 Bright Data，credit 模型在标准站点上更便宜、在防护重的站点上更贵；相比 Apify，它省去挑 Actor 和调试输出格式的环节，灵活性低一些。另外 ScrapeCrew 的对比表里 Bright Data 每月送 5,000 条免费记录、Apify Starter 只要 $29——预算极其有限、月量不到 10 万页的话，这两家的起步档确实更低，ScraperAPI 的优势区间从几十万 credits 起才开始拉开。

## 限制和不适合谁

这部分官网条款里写得清楚，值得单独拎出来：

- **地域定位分档**：Hobby 和 Startup 只能定位 US 和 EU 两个地区，要按国家精准定位（抓日本站、巴西站之类）得上 Business 及以上。
- **额度用尽的默认行为是暂停**。Hobby 到 Professional 可以手动开启 auto-upgrade（用完自动升下一档，opt-in），pay-as-you-go 按固定费率续抓只有 Scaling 及以上才有。低于 Scaling 的套餐额度烧完就停，做监控类任务记得开 auto-upgrade，别让数据断档。
- **超并发直接返回 429**，不排队也不收费，重试逻辑需要自己的调度处理。
- **硬防护站点是弱项**。多家 2026 年的第三方评测口径一致：常规站点表现扎实，Cloudflare/DataDome 这类目标成功率明显下滑（其中一篇引独立基准测试给出六成出头的成功率，且针对的正是困难目标），防护绕过的 10 credits 附加费照样扣。如果目标是大面积硬防护站点，先跑免费试用验证成功率再掏钱。

## 常见问题

**免费试用到底给多少？** 7 天、5,000 API credits，不要信用卡；注册本身另送 1,000 credits（5 并发上限）。这个量足够用 Cost Estimator 测出目标站点的单页成本。

**年付和月付差多少？** 统一 9 折。Startup 从 $149 降到 $134.10/月，一年省 $178.80；Business 一年省 $358.80。

**credits 用完了怎么办？** 默认暂停到下个周期。Hobby–Professional 可开 auto-upgrade 自动升档；Scaling 及以上可设月度上限、按固定费率 pay-as-you-go 继续抓；年付还能提前续费拿新额度。

**支持哪些电商站点的现成解析？** Amazon、Walmart、eBay 有结构化端点（另有 Google Shopping 和 Redfin，以及 Etsy 等专用端点）。关键差别在成本：Walmart 端点 1 credit/次，Amazon 5 credits/次。Shopify 这类自建站没有端点，走通用 API 自行解析，或用 Auto Parsing 处理自定义 URL。

**月付能随时取消吗？** 官网展示为按月计费订阅，升降级在 billing 页操作；具体取消条款以结账页为准，付款前建议确认当期政策。

## 按你的情况选

- 月抓取 10 万页以内、标准网页为主 → Hobby（$49）够用，别为用不上的并发付钱。
- Amazon/Walmart/eBay 监控主力场景、月需求百万级 → Startup 或 Business，年付锁 9 折；Walmart 监控因为端点只要 1 credit，成本压力比 Amazon 小得多。
- 需要国家级地域定位或 pay-as-you-go 弹性 → Business 起步，Scaling 以上才有 PAYG。
- 目标全是硬防护站点，或要抓 Shopify 这类无端点平台 → 先跑免费试用验证，不行就往 Bright Data 或 Apify 那类方案找。

下单前最后确认两件事：用 Domain Cost Estimator 算过目标站点的单页 credit 成本；确认你要的地域定位档位够用。都没问题，👉 [开通 ScraperAPI 并领取 7 天免费试用](https://www.scraperapi.com/?fp_ref=coupons) 就可以开始了。
