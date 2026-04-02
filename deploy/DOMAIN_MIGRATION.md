# 域名迁移与上线清单（Astro 静态站）

本项目是 Astro 静态站，产物目录为 `dist/`。以下步骤用于把网站内容部署到新域名。

## 1) 确认站点类型与部署方式

- 站点类型：Astro 静态站
- 构建命令：`npm run build`
- 部署产物：`/dist`
- 推荐服务器：Nginx（模板见 `/deploy/nginx`）

## 2) DNS 解析

在域名服务商面板添加：

- `A` 记录：`@` -> 你的服务器 IPv4
- `A` 记录：`www` -> 你的服务器 IPv4（可选）
- `AAAA` 记录：`@`/`www` -> 你的服务器 IPv6（若有）

## 3) 新域名站点配置（Nginx）

1. 复制模板 `/deploy/nginx/site.conf.template` 到服务器并替换：
   - `example.com` -> 你的新域名
   - `/var/www/example.com/dist` -> 你的真实站点目录
2. 软链接到 `sites-enabled` 并重载 Nginx。

## 4) 迁移网站文件与必要数据

```bash
npm install
SITE_URL="https://你的新域名" npm run build
rsync -avz --delete ./dist/ user@your-server:/var/www/your-domain/dist/
```

静态站通常无数据库；若后续加入动态服务，请同步：

- 环境变量（`.env`）
- 上传资源目录
- 数据库备份与恢复

## 5) 更新域名相关配置

- 构建时设置 `SITE_URL=https://你的新域名`
- 若接入第三方服务，更新：
  - OAuth 回调地址
  - CORS 白名单
  - Webhook 回调地址

## 6) 启用 HTTPS 与 HTTP->HTTPS

1. 先使用 80 端口配置申请证书（可基于 `site.conf.template`）。
2. 申请证书示例（Certbot）：
   ```bash
   sudo certbot --nginx -d your-domain.com -d www.your-domain.com
   ```
3. 证书签发后切换为 `/deploy/nginx/site-https.conf.template` 并重载 Nginx。

## 7) 访问验证

最少检查：

- 首页可访问且无资源 404
- 页面跳转与静态资源加载正常
- HTTPS 证书有效
- 响应头与缓存策略符合预期

## 8) 切流与 301 重定向

若旧域名仍在使用，部署 `/deploy/nginx/legacy-redirect.conf.template`：

- `old-example.com` -> 旧域名
- `example.com` -> 新域名

确认所有旧域名请求返回 `301` 到新域名。

## 9) 监控与稳定性观察

上线后持续观察：

- Nginx access/error log（4xx/5xx）
- 证书到期时间与自动续签
- 页面可用性与响应时间

## 快速验证命令

```bash
# 构建验证
npm run build

# 线上状态检查
curl -I http://your-domain.com
curl -I https://your-domain.com
curl -I https://old-domain.com
```
