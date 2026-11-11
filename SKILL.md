---
name: aws-lightsail-windows
description: 在用户已自行注册好的 AWS 账号上完成两件事：①做完控制台 Explore AWS 的 5 个新手任务领取抵扣金（目标 $200，建完即删）；②部署 Lightsail Windows 云电脑（实例、静态 IP、RDP 防火墙、数据盘、交付连接凭证）。不含账号注册，不生成任何姓名/地址等身份信息。触发词：Lightsail、云电脑、新手任务、抵扣金、静态 IP、远程桌面。
---

# AWS Lightsail Windows 云电脑部署（已有账号版）

## 边界（必读，优先级最高）

- 只操作**用户已用真实身份注册好的 AWS 账号**。不注册账号，不生成姓名 / 地址 / 邮编等任何身份信息。
- **登录由用户本人在浏览器完成**，不要索要、代管或记录 AWS 登录密码；通过浏览器自动化工具操作用户已登录的会话。
- 遇到绑卡 / 付款验证、身份验证（照片证件）、CAPTCHA：**暂停并交由用户本人完成**，完成后继续。不代做、不绕过、不换环境重试。
- 全程不提交任何付款信息。页面要求升级付费计划 / 绑卡时，立即停止并报告用户。

## 第零步：确认模式与前提

先问用户执行模式：

```
开始之前确认一下，做到哪一步？

A. 领抵扣金 + 部署服务器（完整流程）
B. 只领抵扣金（做完 5 个新手任务，不买服务器）
C. 只部署服务器（跳过新手任务）

回复 A、B 或 C 即可。
```

前提确认：

- 用户已有可用 AWS 账号。**本 skill 不含注册环节，也不提供注册指引**：没注册的话用户自行完成后再来。
- 确认浏览器自动化通道可用（chrome-devtools MCP、browse skill 或会话内可用的浏览器控制工具），以本机浏览器为准。

必填信息（按模式收集）：

| 项目 | 说明 |
| --- | --- |
| 配置档位 | $22/月（2 vCPU 2GB）或 $44/月（2 vCPU 4GB），仅模式 A / C |
| 服务器地区 | 悉尼（默认）/ 美国 / 日本东京 |
| 附加数据盘 | 默认 32 GB，可说不需要 |
| 实例名称 | 默认 `Claude-Workspace-01` |

## 第一步：用户登录，AI 接管

1. 请用户在自己浏览器打开登录地址并登录：
   - Builder ID 账号（近期注册的多为这种）：`https://signin.aws.amazon.com/signup?request_type=builderId` —— 必须用这个地址，用 console.aws.amazon.com 会报错
   - 传统 root 账号：`https://console.aws.amazon.com/`
2. 登录成功（邮箱验证码由用户自己收自己填）后，AI 通过浏览器自动化接管当前会话。
3. **登录后的所有页面导航由 AI 负责**：跳转到指定页面（Console Home、Explore AWS 卡片、Billing → Credits、Lightsail 控制台等），用户只负责登录和输验证码，不需要自己找入口。

地区与 Lightsail 区域对照（注册时选的国家决定默认区域）：

| 注册时选的国家 | Lightsail 区域 |
| --- | --- |
| China | ap-southeast-2（悉尼，默认） |
| United States | us-east-1 或 us-west-2 |
| Japan | ap-northeast-1（东京） |

## 第二步（模式 A / B）：新手任务领满 $200

### 规则

- 控制台首页 **Explore AWS** 卡片列有 5 个新手任务，每完成一个得一份额度（通常每个 $20），合计最多再领 $100，加上注册赠送的 $100 合计 $200（以控制台显示为准）。
- 抵扣金只能抵扣服务费，自开户起 12 个月过期；免费计划满 6 个月或额度用完自动结束。
- 4 个任务本身会消耗额度（EC2 / Bedrock / Lambda / RDS），核心原则：**跟着向导建出来 → 任务显示 Completed → 立刻删除资源**。
- 所有任务资源统一 `-task` 后缀命名；一律在账号当前默认区域创建，不切区域。

### 找入口

Console Home 向下滚找 **Explore AWS / Get started with AWS / Earn credits** 卡片，记录各任务状态；已 Completed 的跳过。优先从每个任务的 **Start activity** 按钮进入，AWS 才能正确记账。找不到卡片 → 查 Billing → Credits，仍没有则如实报告（可能账号早于活动规则），不要乱建资源。

### 执行顺序（先免费后付费，RDS 最慢放最后）

| 顺序 | 任务 | 花额度 | 完成后 |
| --- | --- | --- | --- |
| 1 | AWS Budgets 成本预算 | 否 | **保留**（扣费提醒） |
| 2 | EC2 实例 | 是 | 必须删除 |
| 3 | Bedrock 试用模型 | 极少 | 无资源 |
| 4 | Lambda Web 应用 | 极少 | 删除 |
| 5 | RDS 数据库 | 是 | 必须删除 |

每完成一个汇报一次进度，如：`✅ 任务 2/5 EC2 已完成并已删除实例，当前额度 $140`。

**任务 1 Budgets**：Create budget → Use a template → **Zero spend budget** → 名称 `budget-task` → 提醒邮箱填用户的注册邮箱 → 创建。**不删除。**

**任务 2 EC2**：Launch instance → Name `ec2-task` → AMI 选带 **Free tier eligible** 的 Amazon Linux / Debian / Ubuntu → 规格选带 Free tier eligible 的最小档（t3.micro / t2.micro，不要高配）→ Key pair 选 Proceed without（强制就建 `ec2-task-key`，最后删）→ 网络和存储（8GB）保持默认 → Launch。等 Running 且 2/2 checks passed（5~15 分钟，可先做任务 3/4 再回来）。回首页看状态：Completed 或仍 In progress **都执行删除**（实测创建并终止后才记完成）：Instances → 勾选 `ec2-task` → Instance state → Terminate → 确认 Terminated；Key Pairs 里的 `ec2-task-key` 一并删除。

**任务 3 Bedrock**：进 Playground（Chat / Text）→ 模型选 **Amazon Nova**（Nova Lite / Micro；不要选 Anthropic 系列，新账号用不了）→ 发一条 `Hello, introduce yourself in one sentence.`。返回报错（配额 / 权限）通常也记完成，不要反复刷请求。需 Request model access 就给 Nova 提交一次再发消息。

**任务 4 Lambda**：Create function → Use a blueprint（选向导推荐的 Hello world / web app 蓝图；没有就 Author from scratch，Runtime 选最新 Node.js 或 Python）→ 名称 `lambda-task` → 默认创建 → 按向导完成一次 Function URL / Test → 回首页确认 Completed 后删除（Actions → Delete → 输确认文字）。自动创建的 IAM 角色保留（不收费）。

**任务 5 RDS**：Create database → **Easy create** → 引擎 PostgreSQL 或 MySQL（**不要 Aurora**，费用高）→ 模板 **Free tier**（db.t4g.micro / db.t3.micro）→ 标识符 `rds-task` → master 密码随机生成（仅本次用，不交付）→ 创建。等 Creating → **Available**（5~15 分钟，**创建中不要删**）。变 Available 后无论任务状态如何都删除：Databases → 勾选 `rds-task` → Actions → Delete → **取消** Create final snapshot 和 Retain automated backups → 勾确认项 → 输 `delete me` → Delete。确认 RDS → Snapshots 无 `rds-task` 残留，有则删。

### 核对额度与收尾

- 任务状态和额度到账有延迟（几分钟到 30 分钟+）。5 个做完后每 3~5 分钟刷新首页，最多等 30 分钟。
- 在 **Billing → Credits**（或首页额度卡片）核对总额度和到期时间。
- 某任务 30 分钟未完成：确认是从 Start activity 进入、资源真的建成功；最多补做 1 次，仍不行如实记录，不无限重试。

离开前逐项核对：

- [ ] EC2 → Instances：`ec2-task` 已 Terminated，无其他 Running 实例
- [ ] EC2 → Volumes：无 available 状态遗留磁盘
- [ ] EC2 → Elastic IPs：无已分配弹性 IP（有则 Release）
- [ ] Lambda：`lambda-task` 已删除
- [ ] RDS → Databases：`rds-task` 已删除或 Deleting；Snapshots 无残留
- [ ] Budgets：`budget-task` **保留**

### 禁止事项

- ❌ 不点 Upgrade plan / 升级付费计划
- ❌ 不加入或创建 AWS Organizations（会把免费计划变付费）
- ❌ 不开任务以外的服务、不选非 Free tier 规格、不切区域、不绑卡

模式 B 到此交付结束（见第六步交付模板的账号 / 额度部分，不含服务器信息）。

## 第三步（模式 A / C）：创建 Lightsail 实例

1. 控制台顶部搜索 `Lightsail` 进入。
2. **Create instance** → Platform 选 **Microsoft Windows** → Blueprint 选 **OS Only**，版本优先 **Windows Server 2022**。
3. 套餐按用户选择：$22/月 = 2 vCPU 2GB 60GB SSD；$44/月 = 2 vCPU 4GB 80GB SSD；其他档位选最接近的并向用户确认。
4. 确认区域可用区（应自动匹配，不改）；Instance name 填实例名 → **Create instance**。

创建失败降级（报"账户无法使用此计划大小"时）：

- 第一轮换版本：Windows Server 2019 → 仍失败换 2025。每次只换版本不换套餐。
- 第二轮降套餐：$44 → $22 再按第一轮重试；$22 全失败 → 问用户是否接受 1 vCPU（约 $12/月，性能明显不足）或换账号。
- 每次尝试间清理残留请求，不连续提交。

等状态 Pending → **Running**（1~3 分钟，5 分钟未变就刷新）。

## 第四步：静态 IP、RDP 防火墙、关 IPv6

**4.1 静态 IP**：实例详情 → Networking → Public IPv4 区 **Attach static IP** → 命名（`Static-IP-01`）→ 确认附加对象是本实例 → **Create and attach**。记录 IP，这是用户连接地址。附加在实例上时不收费。

**4.2 RDP 防火墙**：

| Windows 版本 | 默认 RDP 规则 | 本地 mstsc 直连 |
| --- | --- | --- |
| 2025 | 仅 Lightsail 浏览器 RDP | ❌ 需改规则 |
| 2022 / 2019 | 0.0.0.0/0 全开 | ✅ 可直连 |

若创建的是 2025：Networking → 防火墙规则 → RDP（3389）来源改为 **All IPv4 addresses** → Update rules。2022/2019 默认已开，仍核对一下来源是 0.0.0.0/0。

**4.3 关 IPv6（必做）**：同一 Networking 页 → IPv6 networking → 开关切 **Off** → 确认 Yes, disable → 刷新确认 IPv6 地址消失。只关 IPv6，不动 IPv4 和静态 IP。若提示实例重启 / 更新中，等回 Running 再继续。

## 第五步：创建并挂载数据盘（默认 32 GB，用户说不要就跳过）

1. Lightsail → **Storage** → **Create disk** → 区域可用区与实例**完全一致** → 大小 32 GB → 命名 `Data-Disk-01` → 创建。
2. 创建完成后 **Attach** 到实例。

挂载后首次连接还需在 Windows 磁盘管理里初始化分配盘符——写进交付提醒即可。

## 第六步：凭证与交付

实例详情 → **Connect** 标签 → **Connect using your own client**：用户名通常 `Administrator`，点 **Show default password** 取密码。IP 用第四步的静态 IP。

全部完成后打包发送：

```
✅ 你的 AWS 云电脑部署完成！

📋 AWS 账号
━━━━━━━━━━━━━━━━━━━━
邮箱：[用户注册邮箱]（密码用户自持，AI 不经手）
登录：Builder ID 用 https://signin.aws.amazon.com/signup?request_type=builderId
      root 用 https://console.aws.amazon.com/

🖥️ 服务器连接信息
━━━━━━━━━━━━━━━━━━━━
公网 IP ：[静态 IP]
用户名  ：Administrator
密    码：[Windows 服务器密码]
地    区：[区域，如悉尼]

💰 费用信息
━━━━━━━━━━━━━━━━━━━━
当前额度：$[金额]（有效期至 [日期]）
新手任务：[X/5]
实例费用：$[XX]/月　硬盘：$[XX]/月
预估可用：约 [X] 个月

📌 Windows 连接：Win+R → mstsc → 计算机填 IP → 显示选项 → 其他账户 →
   Administrator / 密码 → 连接，提示点是
📌 Mac 连接：App Store 装 Microsoft Remote Desktop → 新建连接填 IP / 用户名 / 密码

⚠️ 首次注意
━━━━━━━━━━━━━━━━━━━━
- 有数据盘先初始化：Win+R → diskmgmt.msc → 新磁盘右键 → 联机 → 初始化 → 新建简单卷 → 出现 D 盘
- 软件 / 项目放 D 盘，别堆 C 盘
- 离开时直接关远程桌面窗口，不要在服务器里点关机
- AWS 密码和服务器密码是两个密码，妥善保管
- 已设零花费预算提醒，超额花费会发邮件
- 免费计划满 6 个月或额度用完自动关闭，到期前备份重要文件
```

模式 B 交付：只发账号 + 额度部分，结尾加"后续需要买服务器随时说"。

## 异常处理

| 情况 | 处理 |
| --- | --- |
| 要求绑卡 / 身份验证 / CAPTCHA | 暂停，用户本人完成后再继续；不代做不绕过 |
| 邮箱验证码超时 | 用户侧点 Resend code 自行查收 |
| 页面改版找不到按钮 | 按页面文字语义找，不乱点；不确定就问用户 |
| 首页无 Explore AWS 卡片 | 查 Billing → Credits；仍无则报告，可能不享受活动 |
| 任务一直 In progress | 等最多 30 分钟；确认 Start activity 入口；补做最多 1 次 |
| Bedrock 全报错 | 发过请求即算，不刷请求 |
| EC2 / RDS 规格不可用 | 换另一个 Free tier eligible 规格 |
| RDS 删除按钮不可用 | 等 Available 再删 |
| 页面要求升级计划 / 绑卡 | 立即停止报告用户 |
| Lightsail 无 Windows 选项 | 确认在 Lightsail（非 EC2）、区域支持 Windows |
| 找不到指定套餐 | 选最接近的并向用户确认 |
| 静态 IP 创建失败 | 查配额上限，报告用户 |
| 磁盘区域不匹配 | 磁盘与实例区域可用区完全一致 |

通用原则：每完成一个关键步骤汇报进度；不确定的选项暂停询问；**出错连续重试不超过 3 次**；服务器密码等凭证完整准确交付；流程失败时报告已完成步骤和失败原因，方便人工接续。

## 检查清单（交付前逐项过）

- [ ] 账号已由用户本人登录，AI 未经手密码 / 付款信息
- [ ] 新手任务已核对，额度已记录（目标 $200）；任务资源全删，零花费预算保留
- [ ] 实例 Running
- [ ] 静态 IP 已附加并记录
- [ ] IPv6 已关闭
- [ ] 数据盘已附加（如用户要求）
- [ ] Windows 用户名密码已获取并交付
- [ ] 费用预估已告知
