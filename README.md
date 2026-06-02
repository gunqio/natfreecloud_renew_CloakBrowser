# 🌐 NAT Freecloud 自动签到 & 续费

> 基于 GitHub Actions + CloakBrowser 实现 [nat.freecloud.ltd](https://nat.freecloud.ltd) 的每日自动签到、积分累积与套餐自动续费，支持 WxPusher 微信推送通知。

---

## ✨ 功能特性

- 🕐 **每日自动签到**：定时触发，自动完成验证码识别与数学题答题
- 🔄 **自动续费**：检测套餐到期时间，剩余 ≤1 天时自动使用积分续期
- 🤖 **反检测浏览器**：使用 CloakBrowser 源码级指纹伪装，配合代理 IP 自动匹配时区/语言
- 🔒 **代理流量**：通过 Xray v25.5.16 (SOCKS5) 代理访问，绕过 Cloudflare 检测
- 📱 **微信推送**：签到/续费结果通过 WxPusher 推送到微信
- 🎬 **可选录屏**：手动触发时可开启录屏，便于调试排查问题
- 🧹 **自动清理**：每次运行后自动删除旧的 workflow runs，只保留最近 2 条

---

## 📁 项目结构

```
.
├── .github/
│   └── workflows/
│       └── checkin.yml       # GitHub Actions 工作流
├── nat_sign_pydoll.py         # 主签到/续费脚本
├── requirements.txt           # Python 依赖
└── README.md
```

---

## ⚙️ 配置说明

### 1. Fork 本仓库

Fork 到自己的 GitHub 账号下。

### 2. 配置 Secrets

在仓库 **Settings → Secrets and variables → Actions → New repository secret** 中添加以下变量：

| Secret 名称 | 说明 | 是否必填 |
|---|---|---|
| `USER_EMAIL` | nat.freecloud.ltd 登录邮箱 | ✅ 必填 |
| `USER_PASSWORD` | nat.freecloud.ltd 登录密码 | ✅ 必填 |
| `V2RAY_CONFIG` | Xray 代理配置 JSON 内容 | ✅ 必填 |
| `APP_TOKEN` | WxPusher AppToken | ⭕ 选填 |
| `WX_PUSHER_UID` | WxPusher 用户 UID | ⭕ 选填 |

> `APP_TOKEN` 和 `WX_PUSHER_UID` 不填则跳过微信推送，不影响签到功能。

### 3. 开启 Workflow 写权限

**Settings → Actions → General → Workflow permissions** → 选择 **Read and write permissions**

> 这是自动清理旧 runs 功能所必须的权限。

### 4. V2RAY_CONFIG 说明

将你的 Xray 客户端配置文件内容（JSON 格式）完整粘贴到 Secret 中。  
脚本会在本地启动 Xray，监听 `127.0.0.1:10808` 作为 SOCKS5 代理。

推荐使用 **v2rayN 导出的配置**，确保入站包含：

```json
{
  "inbounds": [{
    "port": 10808,
    "protocol": "mixed"
  }]
}
```

---

## 🚀 使用方式

### 自动触发（定时）

工作流默认每天 **UTC 01:30**（北京时间 09:30）自动运行，**不录屏**。

### 手动触发

在 **Actions → NAT签到 → Run workflow** 手动触发，可选择是否开启录屏：

| 参数 | 说明 |
|---|---|
| `true` | 开启录屏，录像会作为 artifact 上传，便于调试 |
| `false` | 不录屏（默认） |

---

## 📲 微信推送配置

1. 前往 [wxpusher.zjiecode.com](https://wxpusher.zjiecode.com) 注册账号
2. 创建应用，获取 `AppToken`
3. 关注应用公众号，获取个人 `UID`
4. 将两者填入对应 Secret

推送示例：

```
✅ 签到成功
账户余额剩余 520 积分
到期时间 2026-08-01
不用续期，等到 2026-07-31 再续期
```

---

## 🔧 工作流步骤说明

```
检出代码
  ↓
安装系统依赖（xvfb / ffmpeg / 字体 / 浏览器运行库）
  ↓
安装 Python 依赖（cloakbrowser / ddddocr / playwright）
  ↓
下载并启动 Xray v25.5.16 代理
  ↓
运行签到脚本
  ├── 登录（含验证码 OCR 识别，最多重试 3 次）
  ├── 签到（答数学题，自动提交）
  └── 检查到期时间，必要时自动续费
  ↓
上传截图 / 录屏（可选）
  ↓
清理旧 Workflow Runs（只保留最近 2 条）
```

---

## 📦 依赖说明

| 依赖 | 用途 |
|---|---|
| `cloakbrowser[geoip]` | 反指纹浏览器，自动匹配代理 IP 时区/语言 |
| `ddddocr` | 验证码 OCR 识别 |
| `playwright` | 浏览器自动化底层支持 |
| `Pillow` | 图像处理辅助 |
| Xray-core v25.5.16 | 代理内核，提供 SOCKS5 本地代理 |

---

## ⚠️ 注意事项

- 本项目仅用于个人自动化学习用途，请勿滥用
- Xray 配置请使用**新版格式**（支持 `mixed` 协议，无 `allowInsecure` 字段）
- 若签到页结构变化，可通过手动触发 + 开启录屏排查问题
- GitHub Actions 免费额度每月 2000 分钟，每次运行约 5~10 分钟，日常使用完全够用

---

## 📄 License

MIT
