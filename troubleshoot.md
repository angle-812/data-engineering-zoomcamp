# 加速器出问题怎么修

这份是 **dev-sidecar** 的故障排查手册，专门按你这台机器的实际配置写的。遇到问题时不用从头读，先看第 2 节的速查表，找到对应的一节。

---

## 1. 先记住四件事

搞不懂原理没关系，但这四件事决定了你能自己修好八成的故障。

**① 它是个"中间人"。**
加速器在你电脑上开了一个小门（`127.0.0.1:31181`），浏览器访问 github.com 时被送到这个门，由它代你去取内容，再转给你。所以它一关掉，浏览器就上不了网——不是网络坏了，是**没人守在门口**了。

**② 系统代理是独立开关。**
加速器退出时通常会把系统代理关掉，但偶尔关不干净。这时浏览器会一直找一个不存在的门，所有网站都打不开。**"全部网站都打不开、但微信能发消息"= 这个症状。**

**③ 证书是它能工作的前提。**
因为它要"看"你的 https 内容，就必须解密——用一张自己签发的证书。系统得先信任这张证书，否则浏览器会拦截，报"您的连接不是私密连接"。

**④ 你的东西都在这几个位置。**

| 项目 | 位置 / 值 |
|---|---|
| 程序 | `D:\Program Files\dev-sidecar\dev-sidecar.exe` |
| 配置目录 | `C:\Users\a1897\.dev-sidecar\` |
| 端口 | **31181** |
| 根证书 | `C:\Users\a1897\.dev-sidecar\dev-sidecar.ca.crt`（指纹 `9D5F610B…`，2046 年到期） |
| 日志 | `C:\Users\a1897\.dev-sidecar\logs\server.log`（出问题先看这个，最新的在最下面） |
| 当前生效配置 | `.dev-sidecar\running.json` |

---

## 2. 症状速查表

| 你看到的现象 | 最可能的原因 | 看第几节 |
|---|---|---|
| 所有网站都打不开，微信却正常 | 加速器没启动，但系统代理还开着 | **A** |
| 只有 GitHub 打不开，别的正常 | 线路问题 / 加速器总开关没开 | **B** |
| 提示"您的连接不是私密连接" | 根证书没装、被删、或浏览器不认 | **C** |
| GitHub 网页报 *What? Your browser did something unexpected* | 加速器往页面注入了脚本，打乱了表单校验 | **D** |
| 点 **Create fork** 报错 / 登录后又跳回错误页 | 同上 | **D** |
| GitHub 首页一直转圈、头像不出来 | 正常现象，首页本身很重 | **B3** |
| `git clone` / `git push` 卡住、超时 | git 没走代理，或代理端口变了 | **E** |
| `git push` 报 **502** / *RPC failed* | 撞到了只读镜像 | **E5** |
| `git push` 报 *Authentication failed* | 登录凭据过期 | **E6** |
| 关掉加速器后整个电脑上不了网 | 系统代理没恢复 | **A** |
| 淘宝、B站、学校网站变慢或打不开 | 国内域名被误代理了 | **F** |
| 启动后立刻消失 / 日志里 `EADDRINUSE` | 端口被别的程序占了 | **G** |
| 重启电脑后就不管用了 | 没设开机自启 | **H** |
| 同时开了 Clash / 公司 VPN，全乱 | 两个程序抢系统代理 | **I** |
| 浏览器正常，但 `npm` / `pip` / `docker` 下不动 | 命令行工具不读系统代理 | **J** |
| 提示证书过期 / 时间错误 | 系统时间不准 | **K** |

---

## 3. 三十秒自检

修之前先跑这三条，能直接定位问题在哪一层。**在开始菜单搜 `cmd` 打开命令提示符**，逐条粘贴：

```bat
:: ① 加速器在不在跑（有输出=在跑，没输出=没跑）
netstat -ano | findstr :31181

:: ② 网页通道通不通（应该返回 200）
curl -s -o NUL -w "%{http_code}\n" -x http://127.0.0.1:31181 https://api.github.com/

:: ③ git 通道通不通（应该打印一行 refs/heads/main）
git ls-remote --heads https://github.com/angle-812/data-engineering-zoomcamp.git
```

怎么判断：

- **① 没输出** → 加速器根本没启动 → 去 **A** 或 **H**
- **① 有输出但 ② 不是 200** → 门开着但取不到东西，线路问题 → **B**
- **② 是 200，③ 报 SSL / 证书错误** → 证书链问题 → **C**
- **② 是 200，③ 报 502** → 只读镜像 → **E5**
- **三条都正常** → 加速器没问题，是浏览器缓存或扩展 → **D**、**C**
- **② 报"证书吊销"相关错误** → 这不是故障，是 curl 的老毛病，加 `--ssl-no-revoke` 再试一次就好

> 我这里刚用你这台机器跑过一遍：① 在听、② `github.com` 和 `youtube.com` 都返回 200、③ 拿到了 `refs/heads/main`。**说明现在整套是通的**，下面这些是给以后准备的。

---

## A. 所有网站都打不开（最常见，也最好修）

**症状**：Chrome/Edge 打开任何网站都报错或转圈，但微信、QQ 能正常发消息（它们不走系统代理，所以不受影响）。

**原因**：加速器退出了（崩溃、被你关掉、开机还没启动），但 Windows 的"系统代理"开关还指着 `127.0.0.1:31181`。浏览器把请求送到这个门，没人接，就全部超时。

**修，二选一：**

**做法一（推荐）**：重新打开加速器
打开 `D:\Program Files\dev-sidecar\dev-sidecar.exe`，等它界面出现、首页那个大开关是**亮/绿**的，再刷新浏览器。

**做法二（着急上网，先救命）**：直接关掉系统代理
在命令提示符里粘这一行，回车，然后**关掉浏览器再重开**：

```bat
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings" /v ProxyEnable /t REG_DWORD /d 0 /f
```

或者走界面：**设置 → 网络和 Internet → 代理 → 手动设置代理 → 关掉**。

关掉之后 GitHub 就打不开了，但至少能正常上网、能查资料。

---

## B. 只有 GitHub 不行

### B1. 先分清"网页"和"命令行"

这是两套完全独立的通道，经常是一个通、一个不通：

| 通道 | 谁在用 | 走什么配置 |
|---|---|---|
| 网页 | 浏览器 | Windows 系统代理 |
| 命令行 | git、npm、pip | 各自的配置文件 |

所以浏览器打不开 **不代表** git 不能用。你之前就遇到过：网页报错，但 `git push` 其实成功了。

### B2. 检查总开关

打开加速器首页，看那个大开关。**必须同时满足**：大开关是开的、下面"系统代理"这一项也是开的（不是"PAC"或"不使用"）。只开大开关、系统代理没开，浏览器是不会走加速的。

### B3. GitHub 首页很慢是正常的，别误判

我实测过你这台机器的 `github.com` 首页：**20 秒只拉到 278 KB，然后就卡住不动了**，但同一时刻 `api.github.com` 是 200、`git` 命令也是通的。

原因：首页要加载几十个头像、图标和一个很重的 JS 包，走加速通道（中间人解密 + 转发）就会很慢，卡在某张图片上就一直转圈。

**判断"通不通"不要看首页，用第 3 节的自检命令。** 首页转圈时你可以直接刷新，或者根本不管它——`fork`、`push`、下载文件这些操作都不受影响。

### B4. 真的完全不通

按顺序试：

1. 加速器首页关掉再打开（相当于重连）
2. 加速器 **设置 → DNS** 里点一次"重新测速"/"优选 IP"，让它重新挑节点
3. 换个网络试（手机热点最快能验证是不是你这条宽带的问题）
4. 还是不行就绕开网页：下载文件时给链接加镜像前缀 `https://ghfast.top/` + 原链接（实测约 700 KB/s），这和加速器是两条独立的路

---

## C. 证书错误："您的连接不是私密连接"

**症状**：`ERR_CERT_AUTHORITY_INVALID`、`NET::ERR_CERT_INVALID`，点"高级"能看到颁发者是 dev-sidecar。

**原因**：它在解密你的 https，用的是一张自己签的证书。系统必须信任这张证书，浏览器才不报警。你的证书本来是装好的（配置里 `setuped: true`），报错一般是下面四种。

**① 换了浏览器**
Firefox **不用** Windows 的证书库，它有自己的。所以在 Firefox 里必然报错，除非单独导入：
Firefox → 设置 → 隐私与安全 → 证书 → 查看证书 → 证书颁发机构 → 导入 → 选 `C:\Users\a1897\.dev-sidecar\dev-sidecar.ca.crt` → 勾选"信任由此证书颁发机构来标识网站"。

Chrome / Edge 用 Windows 证书库，正常情况下不用管。

**② 证书被安全软件删了**
360、火绒、卡巴斯基这类有时会把自签根证书当威胁清掉。重新装一次：
双击 `C:\Users\a1897\.dev-sidecar\dev-sidecar.ca.crt` → **安装证书** → 选 **当前用户** → **将所有的证书都放入下列存储** → 浏览 → **受信任的根证书颁发机构** → 确定。

也可以直接在加速器界面里找"安装证书"按钮，一键搞定。

**③ 装完没重启浏览器**
证书在浏览器启动时加载。装完必须**把所有浏览器窗口全部关掉**（包括后台进程）再打开。

**④ 只是 curl 报错，浏览器其实正常**
curl 在 Windows 上会去检查证书吊销列表，这一步经常超时。所以命令要加 `--ssl-no-revoke`：

```bat
curl -s --ssl-no-revoke -o NUL -w "%{http_code}\n" -x http://127.0.0.1:31181 https://api.github.com/
```

**验证是否修好**：上面这条返回 `200` 就说明证书链没问题。返回 `000` 或报错，说明证书还是没被信任。

---

## D. GitHub 网页报错 / 点 Fork 失败 / 登录不上

**症状**：

- 页面显示 `What? Your browser did something unexpected`
- 点 **Create fork** 后跳回错误页
- 登录完又回到登录页

**原因（我查过你的配置，确实有这一条）**：

加速器对 `github.com` 的规则里做了两件事：

1. 把 TLS 握手里的域名（SNI）伪装成 `baidu.com` —— 这是为了绕过封锁
2. **往网页里注入一段油猴脚本**（`GithubEnhanced-High-Speed-Download.user.js`）—— 这是为了加"高速下载"按钮

第 2 条会改动页面的 JS 运行环境。GitHub 的表单提交有防篡改校验（CSRF token 加前端完整性检查），被改动后就判定"浏览器行为异常"，于是报那个错。

**修，按成功率从高到低：**

**① 用无痕窗口重试（最快，10 秒）**
`Ctrl + Shift + N` 打开无痕窗口，访问
`https://github.com/DataTalksClub/data-engineering-zoomcamp/fork`
登录后点 **Create fork**。

无痕窗口默认不加载扩展、不用旧 cookie，一次能排除两个原因。**你上次就是靠这个成功的。**

**② 清掉 github.com 的 cookie**
`F12` → Application（应用）→ Cookies → 删掉 `github.com` 和 `www.github.com` 下的全部 → 刷新。

**③ 关掉脚本注入（治本）**
加速器 **设置 → 拦截规则**（有的版本叫"拦截"标签页）→ 找到 `github.com` → 里面有一条带 `script` 或 `tampermonkeyScript` 的规则，把脚本地址清空，或者把这条规则禁用。
**保留 `sni` 那部分就行**，加速本身不受影响，只是少了个"高速下载"按钮。

**④ 换浏览器 / 换设备**
Chrome 不行换 Edge 试一次。能成功就说明是浏览器配置问题，不是账号或网络问题。

**⑤ 走命令行，不碰网页**
`git push` 和 GitHub API 都不受影响（实测通）。fork 这一步可以让别人帮点，或者用 `gh` 命令行工具。

> **关键一点**：这个错**不影响推送**。你上次网页报错，推送照样成功了（远端提交 `a403b01`）。看到这个错误别慌，先用 `git ls-remote` 确认通道是通的。

---

## E. git 走不通

### E1. 你机器上的 git 配置长这样

我当初为了让 git 能穿过加速器，改了 5 项全局配置。记住它们，出问题好排查：

| 配置项 | 值 | 作用 |
|---|---|---|
| `http.https://github.com/.proxy` | `http://127.0.0.1:31181` | 让 git 只对 github 走代理 |
| `http.sslBackend` | `openssl` | 换掉 Windows 自带的后端，它太固执 |
| `http.sslCAInfo` | `C:/Users/a1897/.dev-sidecar/git-ca-bundle.crt` | 指向"系统 CA + 加速器 CA"的合并证书包 |
| `http.schannelCheckRevoke` | `false` | 关掉吊销检查，否则每次连接都卡住 |
| `credential.helper` | `manager` | 登录弹窗用图形界面 |

查看当前值：

```bat
git config --global --get-regexp "^http\.|^credential\."
```

### E2. 报"连不上 127.0.0.1:31181"

加速器没开。开了再试；或者临时让 git 直连（直连只对下载有效，推不上去）：

```bat
git config --global --unset http.https://github.com/.proxy
```

要恢复，把这条再加回去：

```bat
git config --global http.https://github.com/.proxy http://127.0.0.1:31181
```

### E3. 改了端口怎么办

如果因为端口冲突把 31181 改成了别的（比如 31182），**git 不会自动跟着改**，必须手动同步：

```bat
git config --global http.https://github.com/.proxy http://127.0.0.1:31182
```

### E4. 报 SSL / 证书错误

```
SSL certificate problem: unable to get local issuer certificate
```

说明那个合并证书包丢了或者路径不对。检查文件在不在：

```bat
dir "C:\Users\a1897\.dev-sidecar\git-ca-bundle.crt"
```

不在就重建（用 git 自带的证书包 + 加速器证书拼起来）：

```bat
copy /b "%ProgramFiles%\Git\mingw64\etc\ssl\certs\ca-bundle.crt"+"%USERPROFILE%\.dev-sidecar\dev-sidecar.ca.crt" "%USERPROFILE%\.dev-sidecar\git-ca-bundle.crt"
```

> 你机器上的 git 是便携版，路径不是 `C:\Program Files\Git`。找不到就用 `where git` 看实际位置，再换上面的路径。

### E5. 报 502 / RPC failed / 403

```
error: RPC failed; HTTP 502
```

这是撞到了**只读镜像**——加速器把 github.com 的某些请求转发给了只能下载、不能上传的镜像站。

**处理：**

1. 先确认到底成没成。502 有时是假警报，实际提交已经上去了：

```bat
curl -s --ssl-no-revoke -x http://127.0.0.1:31181 -H "Accept: application/vnd.github+json" https://api.github.com/repos/angle-812/data-engineering-zoomcamp/commits/main
```

看返回里的 `sha` 是不是你刚提交的那个。

2. 没上去就重试一次（很多时候第二次就过）
3. 还不行：把加速器首页关掉再打开，重新握手
4. 大文件推送失败：`git config --global http.postBuffer 524288000`

### E6. 报 Authentication failed

凭据过期了。三步之一：

- 打开 **GitHub Desktop**，它会提示重新登录，登一次就把凭据刷新了
- 或者命令行触发：`git config --global credential.helper manager`，然后随便跑一个 `git ls-remote`，会弹浏览器登录
- 或者删掉旧凭据：控制面板 → 凭据管理器 → Windows 凭据 → 找 `git:https://github.com` → 删除 → 重新推送

### E7. 不报错，但慢到不可用

`git ls-remote` 秒回、但 `clone` 一个仓库要半小时 —— 这是通道带宽问题，不是故障。

**解法：别完整克隆。** 我在你这台机器上就是这么干的：

```bat
git clone --depth 1 --filter=blob:none --no-checkout https://github.com/angle-812/data-engineering-zoomcamp.git 你的目录
```

只拿提交树，几百 KB，之后再按需取文件。完整克隆那个 41 MB 的仓库，在这条通道上要十几分钟。

---

## F. 国内网站变慢或打不开

**原因**：加速器维护了一份"国内域名不代理"的名单（你的配置里叫绕过名单，已经有 `*.cn`、`*.bilibili.com`、`*.taobao.com`、`*.qq.com` 这一百多条）。名单外的国内站会被错误地送进代理，绕一圈，反而慢甚至打不开。

**修**：加速器 **设置 → 代理 → 忽略域名 / 白名单**，加上这个站点，格式是 `*.域名.com`。多个用分号隔开。

**典型受害者**：学校教务系统、公司内网、新上线的小站点、某些地区性网站。

**内网为什么不影响**：`10.*`、`172.16-31.*`、`192.168.*` 这些网段默认就在忽略列表里。如果你的内网用的是别的网段（比如 `100.64.*`），需要手动加。

---

## G. 端口被占用

**症状**：双击程序后界面一闪就没了，或者界面出来了但首页一直显示"未启动"。看日志 `logs\server.log` 最后几行有 `EADDRINUSE`。

**修**：

1. 先查是谁占的：

```bat
netstat -ano | findstr :31181
```

最后一列是进程号（PID），去任务管理器 → 详细信息 → 按 PID 排序找到它，判断是不是可以关。

2. 不想折腾就直接换端口：加速器 **设置 → 端口** → 改成 `31182` → 保存重启。

3. **改完必须同步 git**，否则 git 还是往旧端口发：

```bat
git config --global http.https://github.com/.proxy http://127.0.0.1:31182
```

系统代理会自动跟着改，浏览器不用管。

---

## H. 重启电脑后就不管用了

**原因**：加速器没有开机自启。重启后系统代理可能还留着上次的设置，于是所有网站都打不开（就是第 A 节那个症状）。

**修**：加速器 **设置** 里找"开机启动 / 自动启动 / 随系统启动"，打开。

如果界面里没有这个选项，手动放一个：

1. `Win + R` 输入 `shell:startup` 回车，打开启动文件夹
2. 把 `D:\Program Files\dev-sidecar\dev-sidecar.exe` 的**快捷方式**拖进去

**注意**：它启动后要 5～10 秒才接管代理。**刚开机就打开浏览器，可能报"无法连接代理服务器"**，等几秒刷新一下就好，不是故障。

---

## I. 和其他代理 / VPN 打架

**症状**：同时开了 Clash、公司 VPN、或者别的加速工具，网络时通时不通，有的网站上得去有的上不去。

**原因**：Windows 系统代理是**全局唯一**的一个。两个程序都想设它，后设的把先设的覆盖了，然后两边互相改来改去。

**修**：

- **只留一个。** 需要切的时候，先退出一个，再启动另一个
- 如果必须共存：让加速器改用"PAC 模式"（只代理指定域名），另一个用 TUN 模式，避免两者都抢系统代理
- 公司 VPN 尤其容易冲突，因为它会改路由表。用 VPN 时先退出加速器

---

## J. 浏览器正常，但命令行下不动

**原因**：系统代理只影响浏览器和"遵守系统代理"的程序。`npm`、`pip`、`docker`、`WSL` 各自读自己的配置，不会自动跟随。

**逐个处理：**

**npm**

```bat
npm config set proxy http://127.0.0.1:31181
npm config set https-proxy http://127.0.0.1:31181
```

**pip**

```bat
pip config set global.proxy http://127.0.0.1:31181
```

**Docker Desktop**
Settings → Resources → Proxies → 填 `http://127.0.0.1:31181`

**WSL**
WSL 里的 `127.0.0.1` 是 WSL 自己，不是 Windows。要用 Windows 主机的 IP：

```bash
export http_proxy=http://$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):31181
```

**但是**：加速器自带 `git`、`node`、`pip` 三个插件（在你的配置里，目前都是关闭状态）。打开对应插件，它会自动帮你配好，比手动改省事。

---

## K. 系统时间不准

TLS 证书有有效期，时间偏差超过几分钟就会报"证书无效/已过期"，但看起来像加速器坏了。

**检查**：

```bat
w32tm /resync
```

或者：设置 → 时间和语言 → 日期和时间 → 打开"自动设置时间" → 点"立即同步"。

---

## L. 彻底卸载、恢复原样

如果决定不用了，**必须按这个顺序**，否则会留下断网和 git 报错两个坑。

**1. 先退出程序**：托盘图标右键 → 退出。这一步会把系统代理恢复原状。

**2. 卸载**：设置 → 应用 → 已安装的应用 → 找 dev-sidecar → 卸载。

**3. 删配置目录**：删掉 `C:\Users\a1897\.dev-sidecar\`（里面有证书和 git 证书包）。

**4. 删证书**：`Win + R` 输入 `certmgr.msc` → **受信任的根证书颁发机构** → 证书 → 找到颁发者为 `dev-sidecar` 的那张 → 右键删除。

**5. 清 git 配置（最关键，别漏）**

```bat
git config --global --unset http.https://github.com/.proxy
git config --global --unset http.sslCAInfo
git config --global --unset http.sslBackend
git config --global --unset http.schannelCheckRevoke
```

不做这一步，git 会一直往 `127.0.0.1:31181` 发请求、找那个已经删掉的证书文件，报错会让人一头雾水。

**6. 确认系统代理已关**

```bat
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings" /v ProxyEnable
```

看到 `0x0` 就是关的。

---

## 4. 三条万能修复命令

记不住上面那么多，就记这三条。九成故障能靠它们解决。

**① 上不了网（关系统代理）**

```bat
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings" /v ProxyEnable /t REG_DWORD /d 0 /f
```

**② git 报错（重置代理指向）**

```bat
git config --global http.https://github.com/.proxy http://127.0.0.1:31181
```

**③ git 报证书错（重建证书包）**

```bat
copy /b "%ProgramFiles%\Git\mingw64\etc\ssl\certs\ca-bundle.crt"+"%USERPROFILE%\.dev-sidecar\dev-sidecar.ca.crt" "%USERPROFILE%\.dev-sidecar\git-ca-bundle.crt"
```

---

## 5. 几句实话

**这类工具天生不稳定。** 它的原理是"劫持你的流量 + 转发到镜像站"，链条上任何一环（镜像挂了、节点被墙、证书被清、端口被占）都会让某部分网站不可用。这不是你操作错了，是它的工作方式决定的。

**它也有安全代价。** 为了工作，它必须解密你的 https，并且会从第三方地址（`ds-official-config.bestar.de5.net`）下载脚本注入到 GitHub 页面里。所以：

- **开着它的时候，别登录网银、别输支付密码、别登公司后台**
- 只用它访问 GitHub、YouTube 这类目标站点，用完可以退
- 千万别在这种工具里输入你的 GitHub 账号密码（网页登录走的是 github.com 本体，是安全的；但任何"加速器要求你登录"的弹窗都要警惕）

**什么时候该换方案。** 如果出现下面任一情况，说明它对你的网络环境不合适，别再修了：

- 一周要修三次以上
- GitHub 网页长期打不开，只有命令行能用
- 影响正常上网（国内网站经常打不开）

替代方案按省心程度排序：**Watt Toolkit（开源带界面）→ FastGithub（单文件免安装）→ 纯镜像前缀（`https://ghfast.top/` + 原链接，只解决下载，不解决网页）**。

**最后一条经验**：遇到问题先别急着改配置，**先跑第 3 节那三条自检**。知道自己坏在哪一层，比乱试十次都有用。配置改坏了反而更难查。

---

## 附录：你这台机器的完整参数

排查时对照用。

| 项目 | 值 |
|---|---|
| 程序版本 | dev-sidecar 2.3.5 |
| 程序路径 | `D:\Program Files\dev-sidecar\dev-sidecar.exe` |
| 代理端口 | `31181`（仅监听 127.0.0.1，外部访问不到） |
| 系统代理 | 已开启，`127.0.0.1:31181` |
| 根证书 | `dev-sidecar.ca.crt`，指纹 `9D5F610B…`，到期 2046-11 |
| 证书状态 | 已安装到「当前用户 → 受信任的根证书颁发机构」 |
| 日志文件 | `.dev-sidecar\logs\server.log`（约 6 MB，最新的在末尾） |
| 拦截域名 | 71 个，含 GitHub 全套、YouTube 全套（`*.googlevideo.com` 等） |
| 已开插件 | 无（git / node / pip / free_eye / xget 均未启用） |
| git 代理 | 仅 `https://github.com/` 走 31181 |
| git SSL | openssl 后端 + 合并证书包 |
| 免代理名单 | 一百多条国内域名 + 内网网段 |
