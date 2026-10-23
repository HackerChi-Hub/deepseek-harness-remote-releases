# DeepSeek Harness 手机遥控工具

电脑运行原任务，手机浏览器查看进度、进入原有会话并发送要求。本工具是黑粉科技整理的辅助连接器，不是 DeepSeek 官方手机应用。

**0.1.0 预览版。需要 Node.js 22 或以上，以及已经能正常工作的官方 DeepSeek Harness。** 不包含模型、模型额度、Node.js、Harness 主程序或 HTTPS 证书。

## 下载

- [0.1.0 发行页与版本说明](https://github.com/HackerChi-Hub/deepseek-harness-remote-releases/releases/tag/v0.1.0)
- [下载三平台便携包（ZIP）](https://github.com/HackerChi-Hub/deepseek-harness-remote-releases/releases/download/v0.1.0/deepseek-harness-remote-0.1.0.zip)
- [下载 SHA-256 校验文件](https://github.com/HackerChi-Hub/deepseek-harness-remote-releases/releases/download/v0.1.0/deepseek-harness-remote-0.1.0.zip.sha256)
- [官方 Harness：安装与用法](https://github.com/deepseek-ai/deepseek-harness)
- [Node.js 官方下载](https://nodejs.org/zh-cn/download)

公开的是运行便携包和使用说明，开发仓库与提交历史保持私有。包中的 JavaScript 运行脚本可以被下载者查看；本工具未以“完整开发仓库开源”名义发布。

## 先在电脑上准备

1. 先确认官方 Harness 在电脑端能正常打开原会话并收到模型回复。已有安装和模型继续使用，不用再下载一遍。
2. 安装 Node.js 后重新打开终端，运行 `node --version`，应为 22 或更高版本。
3. 解压便携包到一个长期保留的目录。连接器和可选自启会引用其路径，配置完不要随意移动。

| 系统 | 启动方式 | 备用方式 |
| --- | --- | --- |
| macOS | 双击 `manage.command` | 在解压目录的终端运行 `sh manage.command` |
| Windows | 双击 `manage.cmd` | 在解压目录的 PowerShell 运行 `.\manage.cmd` |
| Linux | 在解压目录打开终端 | 运行 `sh manage.sh` |

## 第一次连接：2 → 3 → 4 → 6 → 5

**菜单 2：挂载连接器。** 选择正在使用的 Harness 配置补丁文件。程序会查找用户目录下 `.dsh/profiles/desktop/cordis.patch.yml`、`.dsh/profiles/web/cordis.patch.yml` 和 `.dsh/cordis.patch.yml`，也可填写完整路径。它只添加自己的区块；不会重启 Host 或重新提交任务。Host 未自动加载时，等任务空闲后自行重启原 Host。

**菜单 3：配置入口。** 三种模式分别为仅本机、已有 HTTPS 反向代理或私有通道、内置 HTTPS。

- 仅本机用于电脑端检查，不提供手机外网访问。
- 已有 HTTPS 代理或私有通道：填写完整 HTTPS 入口，代理目标为 `http://127.0.0.1:8766`。本版要求独立域名根路径，不支持子路径挂载。代理应支持 WebSocket，保留真实 Host、Origin 和 Cookie。
- 内置 HTTPS：明确填写完整入口、本机网卡 IP、监听端口、证书链与私钥文件。DNS、路由器 TCP 映射、防火墙和证书由你明确配置，工具不会自动建立公网可达性。

详细例子见包内 [HTTPS 与证书部署说明](docs/DEPLOYMENT.md)。文档示例域名、IP 和证书路径必须替换为自己的实际值。

**菜单 4：启动，然后菜单 6：诊断。** 同时核对服务运行和上游认证。命令行对应：

```sh
node bin/dsh-remote.mjs status
node bin/dsh-remote.mjs doctor
```

`running` 为真只说明网关运行；`doctor` 中 `upstream.authenticatedHTTP` 还应为 `true`，才能证明接到了原 Host 网页。

**菜单 5：手机配对。** 打开工具提示的本机“手机连接.html”，将最新完整链接交给本人手机，在系统浏览器打开并点击“确认连接”。配对码十分钟有效、只可兑换一次；不要公开链接或配对页。每个浏览器单独配对，网关重启后需重新配对。

第一次进入后，选一个不重要的会话，只发送“只回复：远程连接正常，不调用工具”。先确认消息收发，再接入正式任务。电脑必须保持开机，原模型服务和 Host 执行权限继续有效；工具不会自动批准操作。

## 平时管理

```sh
node bin/dsh-remote.mjs status
node bin/dsh-remote.mjs doctor
node bin/dsh-remote.mjs pair
node bin/dsh-remote.mjs revoke
node bin/dsh-remote.mjs autostart --action install
node bin/dsh-remote.mjs autostart --action remove
node bin/dsh-remote.mjs uninstall
```

菜单 0 退出界面并保留后台；菜单 7 停止网关，有手机登录时需确认；菜单 8 撤销全部手机登录与未兑换码，不停止原 Host 任务；菜单 9 设置当前用户登录后自启。自启不提升管理员权限，是否允许由当前系统策略决定。

升级时先等任务空闲，停止工具，再用新包替换原目录并重新配对；保留独立状态目录。若移动工具目录，应先从旧位置解除连接器和自启。移除默认保留私有资料，只删除自己管理的配置区块；区块被手工改动时要求核对。

## 私有资料在哪里

| 系统 | 默认状态目录 |
| --- | --- |
| macOS | `~/Library/Application Support/DeepSeekHarnessRemote` |
| Windows | `%LOCALAPPDATA%\DeepSeekHarnessRemote` |
| Linux | `$XDG_STATE_HOME/deepseek-harness-remote`，未设置时用 `~/.local/state/deepseek-harness-remote` |

状态目录含真实入口、证书、上游凭据、配对页与日志，不要上传到公开仓库、分享链接或反馈附件。便携包不包含这些资料。配对后的手机能操作原会话，只给本人设备使用。

## 已验证与尚待验证

- 已有真实手机外网入口：本人确认进入并看见原有会话。独立网页另完成实际请求与模型回复。
- macOS 程序回归 20/20，Linux 隔离容器回归 20/20；不将容器结果当作 Linux 原生系统验收。
- macOS 临时网关接真实 Host 配对并读取网页；当前用户 LaunchAgent 完成安装、系统拉起、重复复用、回读与移除。
- Windows 原生 ACL、计划任务与 Host，Linux 原生登录自启，真实手机工具批准、切网和长任务恢复仍待专门验收。

0.1.0 公开便携包的运行代码与上述已测版本逐字节一致，调整的是发包范围与使用文档；这不增加新的运行能力或平台通过项。原始测试材料、真实会话与开发流水线不随包公开。

## 排错

| 现象 | 检查 |
| --- | --- |
| 菜单闪退或没有 node | 新终端运行 `node --version`；从解压目录打开对应入口看报错 |
| 网关已运行却没有原会话 | `doctor` 的上游认证、所选 profile、Host 是否加载连接器 |
| 外网页面打不开 | DNS、TCP 映射目标、本机网卡与监听端口、防火墙、证书分别核对 |
| 配对无效 | 使用新生成的完整链接，在十分钟内用本人系统浏览器兑换 |
| 来源检查拒绝 | 使用登记的完整 HTTPS 地址，检查代理 Host 和 Origin，不关闭校验 |
| 首页能开但进度不更新 | 检查 WebSocket 转发与读取超时 |
| 证书更新 | 由原证书管理器续期并更新正在使用的文件，再执行 `node bin/dsh-remote.mjs reload-cert` |

反馈只附脱敏诊断，不公开配对链接、Cookie、私钥或真实会话。本工具不修改模型、路由器、VPN 或防火墙。
