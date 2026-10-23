# HTTPS 与证书部署

## 已有反向代理

下面是 Caddy 配置示意。将域名替换为本人域名，先确认 DNS、网络入口和证书申请条件，再由现有运维流程部署。不要将公网入口指向原始 DeepSeek Host，只代理本工具的回环网关。

```caddyfile
remote.example.test {
    reverse_proxy 127.0.0.1:8766
}
```

Caddy 自动处理常规 WebSocket 代理。若使用其他代理，需开启 HTTP/1.1 Upgrade、延长实时通道读取超时，保留原始 Host、Origin 与 Cookie。HTTPS 入口必须为单独域名的根路径。

私有 VPN 的 HTTPS 服务也可指向同一回环网关。远程连接能力以所用私有通道的账户策略为准，配对网关仍单独验证本人设备。

## 无公网 80/443 时的证书

DNS-01 可在独立设备上完成证书申请；然后只把对应入口的证书链和私钥交给本机，DNS API 凭据继续留在证书管理设备。

使用官方 acme.sh 时，按其 DNS 服务商说明配置环境变量。示例为域名占位符，不包含任何真实账户设置：

```sh
# 先按官方 DNS API 文档，在证书管理设备的私有环境中配置最小权限凭据。
acme.sh --issue --dns dns_ali -d remote.example.test --keylength ec-256
acme.sh --install-cert -d remote.example.test --ecc \
  --key-file /private/cert-output/privkey.pem \
  --fullchain-file /private/cert-output/fullchain.pem
```

不要把凭据直接写进终端历史、公开脚本或本工具配置。申请证书不会自动建立公网端口映射。本工具不自动安装第三方 ACME 客户端。

## 续期与不中断换证

“诊断”会显示证书到期时间。内置 HTTPS 模式下，使用原证书管理器续期后，将新的证书链与私钥安全更新至当前配置 `tls.directory` 下的 `fullchain.pem`、`privkey.pem`。保持私有目录访问权限，再执行：

```sh
node bin/dsh-remote.mjs reload-cert
```

重载失败会保留当前证书，不关闭正在运行的 Host。成功重载不撤销手机登录。

重新执行 `configure --mode tls ...` 会创建新的证书目录供下次启动使用，不隐式重启当前服务。旧证书目录留在私有状态中，可在确认不再使用后清除；完全移除可用 `uninstall --purge`。

## 外网与常见故障

- **页面能打开但看不到任务**：执行 `doctor`。连接器凭据存在和上游已认证是两项不同状态。不要反复重新下载模型。
- **配对码无效**：新生成的配对页会替代旧未兑换码；用最新完整链接，十分钟内在系统浏览器打开。
- **来源校验拒绝**：使用配置中的完整 HTTPS 入口，检查反向代理是否重写了 Host 或 Origin。不要通过关闭来源校验解决。
- **加载后十秒断线**：检查代理的 WebSocket 支持与读取超时。项目回归包含升级后空闲十一秒再交换数据。
- **端口占用**：选择一个空闲本机端口或明确管理旧实例。工具不会停止占用端口的未知程序。
- **Windows 状态目录拒绝访问**：检查当前账户和目录 ACL，不要用授予“所有人完全控制”绕过。
- **Linux 无法开启自启**：确认登录会话有用户 systemd。容器或无 systemd 环境仍可前台 `run` 或手动后台 `start`，不要冒充原生自启已验收。
