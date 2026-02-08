# 通过 Cloudflare Worker 代理给 CF Pages 项目加速

## 方案背景
CF Pages 项目原本可通过华为云国际版实现分流加速，但华为云目前不支持绑定二级域名；而 CF Worker 项目可通过 Worker 路由加速、Argo 和 VPS 项目可通过 CF SaaS 加速，因此本文提供一种**仅用二级域名即可为 CF Pages 项目加速**的新方法，核心是利用 Worker 作为中转层转发流量。

## 基础知识
- **CF Worker 加速**：适用于 Worker 项目，通过 Worker 路由实现流量加速。
- **CF Pages 加速（华为云方式）**：原支持分流加速，但**不支持二级域名**，当前方案已失效。
- **CF SaaS 加速**：适用于 Argo 和 VPS 项目，与本次 Pages 加速无关。

##  Pages 加速核心步骤
### 第一步：搭建 CF Pages 项目
先完成 Pages 项目部署，项目默认会分配官方地址，示例为 `https://your-project.pages.dev`（后续需用到此地址）。

### 第二步：创建 Worker 转发 Pages 流量
1. **创建 Worker**：进入 Cloudflare 控制台，找到「Workers 和 Pages」，新建一个 Worker（可自定义名称）。
2. **复制并粘贴代码**：获取对应加速代码（原文提供 GitHub 代码地址，可直接访问复制），粘贴到新建的 Worker 编辑界面。
3. **修改代码关键配置**：找到代码约第七行的 `targetBase` 变量，将默认地址替换为自己的 Pages 项目官方地址，示例修改如下：
   - 原代码：`const targetBase = 'https://link.yutian81.top'`
   - 修改后：`const targetBase = 'https://your-project.pages.dev'`（替换为实际 Pages 地址）。
4. **绑定 Worker 路由**：假设用二级域名 `moontv.ip-ddns.com` 访问 Pages 项目，路由配置如下：
   - 路由区域：选择二级域名的根域（示例：`ip-ddns.com`）。
   - 路由填写：`moontv.ip-ddns.com/*`（`*` 表示匹配所有路径，确保全页面可访问）。
<img width="441" height="371" alt="image" src="https://github.com/user-attachments/assets/50620a04-a79c-405f-98a8-46808255ffd3" />

5. **添加 CNAME 解析记录**：进入二级域名（如 `ip-ddns.com`）的 DNS 记录管理页面，新增一条 CNAME 记录：
   - 类型：CNAME
   - 名称：子域名前缀（示例：`mtv`，最终访问子域为 `mtv.ip-ddns.com`）。
   - 内容：填入优选域名（示例：`cloudflare.182682.xyz`）。
   - 小黄云（代理状态）：关闭（仅保留 DNS 解析功能，避免与 Worker 路由冲突）。

## 结果验证
完成所有配置后，打开浏览器访问绑定的二级域名（示例：`https://moontv.ip-ddns.com`），若能正常加载 CF Pages 项目内容，且访问速度流畅，说明加速配置成功。

## 标签
#cloudflare #建站 #CDN
