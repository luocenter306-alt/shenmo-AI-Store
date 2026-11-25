# 神墨 AI STORE

基于所提供 UI 展示板实现的全栈 V1 工程。白色背景、黑灰导航、少量金色、局部立体视觉。首页使用 CSS 3D 实时构建商品模型，配合本地 SVG 标识，支持拖动、触摸、键盘选择与点击商品。已删除旧分类栏、重复宣传区和长版优势区，桌面首页约 1204px。

## 技术与运行

在 `shenmo/store` 目录执行以下命令。需要 Node.js 22.13+ 和 npm。商城使用 React 19、Vinext、Cloudflare Workers、D1（SQLite）和 Drizzle。

```sh
npm ci
npm run build
node --import ./scripts/sites-env.mjs ./node_modules/wrangler/bin/wrangler.js d1 execute DB --local --config dist/server/wrangler.json --persist-to .wrangler/state --file drizzle/0000_whole_newton_destine.sql
node --import ./scripts/sites-env.mjs ./node_modules/wrangler/bin/wrangler.js d1 execute DB --local --config dist/server/wrangler.json --persist-to .wrangler/state --file drizzle/0001_business_constraints.sql
node --import ./scripts/sites-env.mjs ./node_modules/wrangler/bin/wrangler.js d1 execute DB --local --config dist/server/wrangler.json --persist-to .wrangler/state --file drizzle/0002_romantic_fat_cobra.sql
node --import ./scripts/sites-env.mjs ./node_modules/wrangler/bin/wrangler.js d1 execute DB --local --config dist/server/wrangler.json --persist-to .wrangler/state --file drizzle/0003_amused_energizer.sql
npm start
```

打开 `http://127.0.0.1:8787/` 查看构建后的商城。开发时可用 `npm run dev`，默认端口为 `5173`。四个迁移命令按顺序对新建的本地数据库执行一次。已有数据库只需执行尚未应用的迁移。本地数据库保存在 `.wrangler/state`。

站内支持邮箱或手机号加密码注册和登录，密码至少 10 位。手机号默认支持中国大陆 11 位号码，国际号码需带 `+` 和国家区号。当前是账号密码模式，不发送短信或邮箱验证码，也不将注册视为联系方式所有权验证。

Sites 外层访问控制独立于站内账号。若托管平台返回 403，请在托管环境检查站点访问权限。

## 环境配置

在 Sites 的运行环境中配置：

- `ADMIN_EMAIL` 用于指定保留给所有者的站内管理员邮箱。
- `ADMIN_OWNER_EMAIL` 用于指定私有站点所有者已验证的平台邮箱。所有者通过外层身份验证后，使用 `ADMIN_EMAIL` 完成站内注册并自行设置密码。已有同邮箱账号也需要通过所有者证明才能授予管理员角色。
- ENABLE_PLATFORM_LOGIN：默认关闭，仅业务测试明确启用旧平台身份适配。正式站内账号依赖 HttpOnly 会话，不依赖输入的角色。
- LICENSE_KEY：生产加密密钥，已在托管环境设置为 secret。不可随意轮换；已有卡密依赖原密钥解密。
- PAYMENT_WEBHOOK_SECRET：接入支付网关时配置。当前未配置，回调拒绝执行。
- ALLOW_TEST_PAYMENTS：仅本机测试可设为 true，且必须为 localhost / 127.0.0.1 域名。托管环境没有开启。

本地环境使用未跟踪的 .dev.vars；仅填写你自己的开发值，不复制生产密钥。

## 全栈部署

GitHub Pages 发布仓库的 `docs/`，用于公开项目介绍和离线快照。完整商城需要支持 Cloudflare Workers 和 D1 的托管环境。

1. 使用 `shenmo/store` 作为应用目录并运行 `npm ci`、`npm run build`。构建产物在 `dist/server/`（Worker 与生成的 Wrangler 配置）和 `dist/client/`（静态资源）。
2. 在 Sites 托管环境中为应用绑定 D1，绑定名为 `DB`，并应用 `drizzle/` 下四个 SQL 迁移。如果直接部署到 Cloudflare Workers，在部署配置中填写实际的 D1 数据库名称和 ID，保留 `DB` 绑定及客户端资源路径。Vite 生成的占位数据库 ID 仅供本地使用。
3. 在托管环境配置上节所需的运行时变量及密钥。保持测试支付关闭，然后验证首页、`/api/bootstrap`、数据库读取和站内登录。

已有 D1 数据库只需应用尚未执行的迁移。数据库应使用独立的生产绑定。[Cloudflare D1 迁移文档](https://developers.cloudflare.com/d1/reference/migrations/)

## 当前可用

首页、分类/搜索/价格排序、商品详情、确认订单、支付状态页、个人订单、卡密安全查看、服务预约和进度、售后工单对话、钱包流水与提现申请、代理申请和审批、代理价格、管理后台。

后台包含商品编辑/来源审核/上下架、TXT 卡密导入和批次、订单和退款、人工派单、技术员、服务状态、售后、提现审核、代理审批、客服角色配置、提现方式设置、操作日志。后台入口位于页脚。没有预设管理员密码。

八条初始商品记录是真实数据库中的待审核目录；全部为 DRAFT、零库存、零销量。管理员填写来源、供应商、凭证及销售依据并审核后才能购买。目录价格来自需求示例，需要运营核实，不代表官方价格或已获授权。

28 张关系表；金额统一使用整数分；主外键、唯一约束、查询索引和状态触发器。卡密 AES-GCM 加密、哈希去重；库存通过原子 batch 保留；售出卡密不可回到可售状态。订单/支付/发卡分别有幂等保护。钱包与操作日志禁止更新、删除，余额从流水汇总。

未支付订单在后续下单清理时释放超过 30 分钟的预留库存。预约订单补货后按时间顺序尝试自动交付。

## 支付适配器边界

当前没有真实收银台，生产支付按钮明确返回「尚未开通」。不生成虚构支付成功或真实收费。

预留 POST /api/payment-webhook：JSON 字段 orderId、eventId、amount（整数分）、status=SUCCEEDED。头部 x-payment-timestamp 为秒级时间戳，x-payment-signature 为 HMAC-SHA256 十六进制摘要；签名内容为 timestamp + '.' + 原始 JSON。五分钟时钟窗口，唯一事件及订单防重放。

这是受控网关适配器协议，并不是微信/支付宝原生回调实现。正式上线前需要商户资质、收银台创建、渠道专用验签/查单与回调可达性联调。当前站点保持仅所有者可访问的私有状态，不能作为对外营业店铺。

## 明确预留的扩展

推广链接已生成，但归因、下级绑定与自动佣金结算尚未实现；分站属于后续范围。product_variants 已建表，当前购买只使用标准规格，多规格编辑与按规格库存尚未实现。已实现独立邮箱/手机号密码账号、随机盐 PBKDF2-SHA256（600,000 轮）、服务端哈希会话、七天有效期、退出撤销、SameSite/HttpOnly/Secure Cookie 与账号/IP 尝试限制。短信/邮件验证码、密码找回与联系方式验证尚未接第三方服务。退款进入钱包；实际提现由管理员人工汇款再确认。技术员后台只返回自己被分配的工单。

商品内部审核以超级管理员人工判断为准；系统只检查必填资料，不能自动认定授权真实性。进一步的审计、支付商户验证和生产运维应在对外营业前完成。

## 验证

```sh
node node_modules/typescript/bin/tsc --noEmit
node tests/business.mjs
npm run build
```

业务测试在真实 SQLite 中运行迁移和 SQL，以事务模拟 D1 批处理。断言覆盖鉴权与归属、商品审核、库存、支付幂等、退款、安装流程、售后、供货目录导入和网站设置。测试输出会列出实际通过的 API 断言数。

## 本次交互与视觉更新

首页收敛为交互 3D 主视觉、四件精选工具（可切换安装服务）、简短服务入口和精简页脚。原分类横栏已删除。商品标识为本地 SVG，来自 LobeHub Icons，商标权归各自权利人；图标许可见 THIRD_PARTY_NOTICES.md。3D 模型采用 DOM/CSS 立体面，旋转范围有限，保留重置、商品切换及 reduced-motion 适配，不依赖模糊位图或 WebGL。

登录弹窗已替换原 ChatGPT 按钮。站内登录与 Sites 外层访问策略独立；当前没有更改站点分享范围。

## Prome 商品目录接入

后台新增「供货商品」：读取 https://prome666.com/user/api/index/data 与公开商品列表接口。首批公开数据来自 2026-10-16：6 个分类，共 31 条分类展示记录、23 个唯一编号；去重并跳过源站代理升级权限后导入 22 件。原始目录文件随交付提供。

新增两张表 supplier_products、supplier_sync_runs，追加迁移 drizzle/0003_amused_energizer.sql。已有本地数据库应用该迁移一次。线上 SEED_SUPPLIER_CATALOG=true 时首次读取商城会将已验证的固定目录快照导入，保持 DRAFT，不需要生产环境临时管理员账号；后续不会重复初始化。

所有源站商品当前 catalog_only=1。后台可以编辑文案和售价、记录审核材料，但客户端按钮和服务端订单接口均阻止采购未接通的目录商品下单。导入不生成卡密，不创建付费订单，不改变任何钱包余额。外部的库存文字只记录为上游库存，不增加本地可售数量；未公开的库存数量保持 null。售价、原价和代理价初始等于抓取时来源公开价，不能视为真实供货成本或代理折扣。

重复同步通过源站编号关联，只更新来源名称、来源价、图片、库存展示和同步时间；保留本站价格、文案及审核状态。网络完整同步后才标记未再出现的来源商品；文件导入不将未包含的商品视为下架。失败时保留原数据并记录日志；同步互斥锁有效期 120 秒。

同步按钮通过服务器请求固定来源域名。外联失败时会显示错误，也可在后台导入仓库中的 [`db/prome-catalog-snapshot.json`](db/prome-catalog-snapshot.json)。托管环境的外联及实时同步需要独立验证。快照记录采集时的供货信息。官方供货 API、自动下单、扣款、卡密获取及自动售后尚未接入。

本机实时读取来源受网络限制时，可在 PowerShell 从项目目录运行 `./scripts/fetch-prome-catalog.ps1` 获取更新目录，随后在后台导入生成的 work/prome-catalog.json。此脚本只读取公开分类与商品，不登录上游、不下单、不付款。

## 商品、分类与公告管理

超级管理员可在首页商品卡片点击“修改价格”，保存售价、原价、代理价；已有订单金额不变。后台“商品管理”可确认删除商品：以持久删除标记和下架状态隐藏，历史订单、卡密、售后及供货映射保留，再次同步目录不会恢复展示。

后台“分类管理”可新增、改名、删除分类；改名同步所属商品，有商品的分类不能删除。分类、公告与删除标记保存在 D1 settings，修改均记录审计日志，无需新增数据库迁移。

后台“公告管理”可修改标题、正文与启用状态。保存产生新版本，访客再次进入会重新提醒。弹窗支持本次关闭、对本版本不再提醒、悬浮入口再次查看、键盘关闭与焦点限制。正文为纯文本，空行分段，不执行 HTML。默认内容明确为用户提供的 Prome 上游公告，并区分上游的联系方式、代理价格、退款承诺与神墨自身服务。

首页默认突出远程安装，3D 展示和默认推荐均为安装服务；购买提醒比较官方账号、API 与本地模型，并说明安装不等于会员充值。
