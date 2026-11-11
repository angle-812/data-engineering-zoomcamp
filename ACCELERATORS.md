# 连不上 GitHub 怎么办

先把话说清楚：**下载不用你操心**，我已经用加速镜像把完整仓库（41 MB，含图片、Notebook、代码）拉到你桌面了。你要装加速器，只是为了**能在浏览器里打开 github.com 点那个 Fork 按钮**，以及后面 `git push` 能通。

按省事程度排序，三个方案挑一个：

---

## 方案一：不装任何软件，链接加前缀

在任意 GitHub 链接前面拼一个加速域名就行：

```
https://ghfast.top/https://github.com/DataTalksClub/data-engineering-zoomcamp/archive/refs/heads/main.tar.gz
```

实测速度（你这条网络，我刚测的）：

| 加速域名 | 速度 | 备注 |
|---|---|---|
| `ghfast.top` | **约 700 KB/s** | 目前最稳，41 MB 一分多钟 |
| `gh-proxy.com` | 约 70 KB/s | 慢，但能通 |
| `ghproxy.net` / `ghproxy.cc` / `bgithub.xyz` | 连不上 | 时好时坏，别指望 |

**局限**：只能加速下载（raw / archive / release），网页本身（fork、提交、看 issue）还是打不开。所以这个方案解决不了你现在的核心问题。

---

## 方案二：本地反代工具（推荐，能正常打开网页、能登录、能 fork）

这三个都是开源项目，原理是在本机做一层反向代理 + 改 hosts，让 `github.com` 的域名解析到能通的 IP，**不需要任何订阅或账号**：

| 工具 | 平台 | 特点 |
|---|---|---|
| **Watt Toolkit**（原 Steam++） | Windows / Mac / Linux | 最省事。装好 → 左侧「网络加速」→ 勾选 GitHub → 一键启动。同时能加速 Steam、Discord 等 |
| **dev-sidecar** | Windows / Mac | 轻量，装完后台常驻，自动接管 GitHub 相关域名 |
| **FastGithub** | Windows（.NET） | 单文件双击就跑，专治 GitHub 域名被污染，没有界面，最干净 |

装哪一个都行，Watt Toolkit 对新手最友好（有图形界面，能一键开关）。

搜索时认准 GitHub 上的官方仓库，别下第三方"绿色版"——这类工具被套壳加料的很多。

---

## 方案三：只改 DNS（免费，但会波动）

把网卡 DNS 改成 `223.5.5.5`（阿里）或 `119.29.29.29`（腾讯），有时能立刻好。

原因是 GitHub 在国内通常是「能连但经常抽风」，绝大多数是 DNS 被污染或 CDN 落点太差。改 DNS 对前者无效、对后者有效，所以不如方案二稳。

---

## 几个避坑点

- **别用来源不明的"GitHub 加速器"客户端**：捆绑安装、要手机号、限速、劫持流量，踩一个就够烦的
- **不要在加速站里输入 GitHub 账号密码**：fork / push 一定要走 `github.com` 本体，中间站只用来下载公开文件
- **改 hosts 会过期**：GitHub 的 IP 会变，所以优先选能自动更新的工具（Watt Toolkit、dev-sidecar 都会自动维护）
- 我这边所有下载都是绕开本机代理直连或走加速前缀完成的，没有用你的任何凭据

---

## 装好之后

1. 浏览器打开 https://github.com/DataTalksClub/data-engineering-zoomcamp
2. 右上角 **Fork** → **Create fork**
3. 回到桌面这个文件夹，双击 **`push-to-github.bat`**
4. 它会自动把你的 `notes/` 和 `index.html` 推到你 fork 出来的仓库里，中途会弹一次浏览器让你授权，点同意就行

推完之后，你的仓库地址就是：

```
https://github.com/angle-812/data-engineering-zoomcamp
```

想让它变成手机也能看的网页，再去 **Settings → Pages** 选 `main` + `/ (root)`，两分钟后挂在：

```
https://angle-812.github.io/data-engineering-zoomcamp/index.html
```
