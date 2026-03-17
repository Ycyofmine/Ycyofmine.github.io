---
title: WSL mirrored + VS Code 全局代理
date: 2026-3-14 22:18:00 +0800
categories: [随笔]
tags: [开发环境] # TAG names should always be lowercase
media_subpath: /assets/img/装机
---
windows下开发有两个问题：

1. 命令行环境很难受，很多脚本和教程默认都是 unix。
2. 开 `tun` + 全局代理的时候，国内网站和 `QQ` 也会一起走代理，流量掉得很快。

我现在用的是 `WSL(mirrored模式) + VS Code 全局代理`。

`WSL` 里开发，`VS Code` 里挂代理，本机网络保持正常。

## WSL mirrored
先把 `WSL` 装好，安装过程参考文末的知乎文章。

然后在 `%UserProfile%\\.wslconfig` 里加入：

```ini
[wsl2]
networkingMode=mirrored
autoProxy=true
```

改完以后执行：

```powershell
wsl --shutdown
```

微软文档里提到，`mirrored` 模式下，`WSL` 可以直接通过 `127.0.0.1` 访问 windows 上的服务，对 `VPN` 兼容性也更好。

## VS Code 代理
我这里是 `Remote - WSL` 开发。

在 `settings.json` 里加：

```json
{
  "http.proxy": "http://127.0.0.1:{yourPort}",
  "terminal.integrated.env.linux": {
    "HTTP_PROXY": "http://127.0.0.1:{yourPort}",
    "HTTPS_PROXY": "http://127.0.0.1:{yourPort}"
  }
}
```

这样 `VS Code` 扩展和从 `VS Code` 打开的 `WSL` 终端都会走代理，从而防止 openai 的流量不走代理。

现在 `ai` 可以直接在 `unix` 环境里跑命令，本机访问国内网站也正常。

# 参考
[微软官方：如何使用 WSL 在 Windows 上安装 Linux](https://learn.microsoft.com/zh-cn/windows/wsl/install)

[微软官方：WSL 网络访问与 mirrored mode](https://learn.microsoft.com/en-us/windows/wsl/networking)

[微软官方：`.wslconfig` 高级配置](https://learn.microsoft.com/en-us/windows/wsl/wsl-config)

[VS Code 官方：Network Connections in Visual Studio Code](https://code.visualstudio.com/docs/setup/network)

[知乎：WSL安装教程](https://zhuanlan.zhihu.com/p/701837831)
