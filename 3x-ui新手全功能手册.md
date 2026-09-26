# 3x-ui 全功能新手实战指南（含 IPv6 禁用与 Host 伪装）

## 1. 为什么部署后默认走 IPv6 会断网？如何修复？
- **根因**：大部分云服务器默认开了 IPv6 解析，公共 DNS 会优先返回境外网站的 IPv6 地址；但服务器网卡没有分配实际可用的 IPv6 外网路由，导致流量进死胡同（报 `Network is unreachable`）。
- **永久修复命令（部署时必跑一次）**：
```bash
# 彻底禁用不可达的 IPv6，强制所有流量走 IPv4 直连
echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.default.disable_ipv6 = 1" >> /etc/sysctl.conf
sysctl -p
```

---

## 2. 传输配置中的“主机 (Host)”到底怎么填？
| 使用场景 | 主机 (Host) 填法 | 伪装原理与要求 |
|---|---|---|
| **场景 A：无域名（纯公网 IP 直连）** | **严格留空** | 客户端直接连服务器 IP，填任何假域名都会导致 HTTP 握手 Host 不匹配而直接断开。 |
| **场景 B：自购域名 + 套 Cloudflare CDN** | 填你的**真实域名**（如 `node.yourdomain.com`） | 手机连接 Cloudflare 免费节点 IP，通过 `Host: node.yourdomain.com` 告诉 CDN 转发回你的真实服务器，实现彻底隐藏源站 IP。 |
| **场景 C：免流/白名单免流混淆** | 填免流目标域名（如 `update.microsoft.com`） | 欺骗运营商审计系统，使流量被误判为系统更新或免流业务。 |

---

## 3. 从零添加入站（节点）完整表单
1. **协议 (Protocol)**：`vmess`（全平台客户端 100% 兼容）。
2. **端口 (Port)**：填任意未占用端口（如 `20891`）。
3. **添加客户端**：点 `+ 添加客户端` 自动生成 UUID，`alterId` 填 `0`。
4. **传输配置 (Network)**：
   - 网络：`ws`
   - 路径：`/ray`
   - 主机 (Host)：无域名留空；套 CDN 填自购域名。
5. **保存**。

---

## 4. 出站规则（必须保留一条 direct）
- **路径**：「面板设置」->「Xray 配置」->「出站规则」。
- **参数**：
  - 标签 (Tag): `direct`
  - 协议 (Protocol): `freedom`
- 点击页面上方「保存并重启」。
