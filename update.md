# OpenClaw 运维手册

## 一、核心配置文件 (`openclaw.json`)

配置文件路径：`~/.openclaw/openclaw.json`

### 关键配置项说明

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `gateway.bind` | `"loopback"` | 网关绑定地址，`openclaw doctor` 校验通过的标准值 |
| `channels.telegram.allowFrom` | `7939882359` | 你的 Telegram 数字 ID，限制机器人只响应你的指令 |
| `agents.defaults.model.primary` | `"gemini/gemini-2.5-flash"` | 默认使用 Gemini 2.5 Flash 模型 |
| `browser.defaultProfile` | `"chrome"` | 启用浏览器中继模式 |
| `auth.mode` | `"none"` | 网关身份验证模式（可选） |

---

## 二、服务管理 (Systemd)

OpenClaw 已配置为 systemd 用户服务，开机自动启动，后台静默运行。

### 常用命令

```bash
# 重启机器人（修改配置后必做）
systemctl --user restart openclaw-gateway

# 查看运行状态
systemctl --user status openclaw-gateway

# 停止机器人
systemctl --user stop openclaw-gateway

# 启动机器人
systemctl --user start openclaw-gateway
```

### 监控与调试

```bash
# 健康检查
openclaw doctor

# 实时日志
openclaw logs --follow

# 交互式界面
openclaw tui
```

---

## 三、Windows 端桥梁维护

### 端口转发

WSL IP 变动后需更新转发规则。以 **管理员权限** 打开 PowerShell：

```powershell
# 1. 在 WSL 中获取 IP: hostname -I
# 2. 删除旧规则
netsh interface portproxy delete v4tov4 listenport=18792 listenaddress=127.0.0.1
# 3. 添加新规则 (将 <WSL_IP> 替换为实际 IP)
netsh interface portproxy add v4tov4 listenport=18792 listenaddress=127.0.0.1 connectport=18792 connectaddress=<WSL_IP>
```

### 浏览器插件

- 确保 Chrome 插件 **OpenClaw Browser Relay** 图标为 **彩色 (ON)**
- 若图标为灰色，点击点亮中继

---

## 四、机器人对话指令

在 Telegram 或 TUI 中使用：

| 指令 | 功能 |
|-----|------|
| `/restart` | 重启机器人进程（需配置中开启 `commands.restart=true`） |
| `/reboot` | 强制重置当前会话，刷新浏览器工具访问权限 |
| `截图/读取页面` | 自然语言命令，如"读取当前网页标题" |

---

## 五、故障排查

| 现象 | 原因 | 解决方法 |
|-----|------|---------|
| TG 无回应（进程在线） | 网络无法连接 Telegram | 在启动脚本中添加 `export https_proxy` |
| 日志显示 Token 错误 | 网关开启了身份验证 | 修改 `openclaw.json` 将 `auth.mode` 设为 `none` |
| 插件显示已连接但机器人说"看不见" | 会话未刷新 | 发送 `/reboot` 重置会话 |
| doctor 报错 | `gateway.bind` 配置错误 | 设为 `"loopback"` |

---

## 六、维护常用命令

```bash
# 查看后台日志
tail -f ~/openclaw_debug.log

# 杀掉残留进程
killall node

# 测试 Telegram 连通性
curl -v https://api.telegram.org
```

---

## 💡 快速检查清单

当机器人失效时，按此顺序排查：

1. **IP 匹配**：`hostname -I` 查到的 IP 是否与 Windows `netsh` 转发的目标 IP 一致？
2. **插件开关**：Chrome 插件图标是否为 **ON (彩色)**？
3. **服务状态**：`systemctl --user status openclaw-gateway` 是否显示为 **active (running)**？
4. **配置校验**：`openclaw doctor` 是否返回 **Doctor complete**？

---

## 七、技能扩展

```bash
# 安装新技能
openclaw install <skill-name>

# 手动配置技能
openclaw configure --section <name>
```
