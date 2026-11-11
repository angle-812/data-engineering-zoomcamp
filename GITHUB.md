# 怎么把它弄到你自己的 GitHub 上

先说结论：**用 Fork，不要用「新建仓库 + 上传压缩包」**。这门课仓库约 315 MB（里面全是讲义和图片），直接打包上传又慢又容易丢历史，而且 Fork 会保留原作者归属、还能一键同步官方更新。

你的 GitHub 用户名（从本机 git 配置读到的）是 `angle-812`，下面命令都按这个写，如果实际不同，替换掉即可。

---

## 第一步：Fork（30 秒，网页点两下）

1. 打开 https://github.com/DataTalksClub/data-engineering-zoomcamp
2. 右上角 **Fork** → **Create fork**
3. 完成后你的地址是：`https://github.com/angle-812/data-engineering-zoomcamp`

之后官方更新了课程，进你的仓库点 **Sync fork → Update branch** 就行，不用重新 fork。

---

## 第二步：把看板和笔记放进你的仓库

这里有两个东西要放进去：

- `index.html` —— 我做的学习看板（88 个单元、视频直链、进度勾选）
- `notes/` —— 七个模块的笔记模板（单元清单已经填好，边看边勾）

### 方式 A：网页上传（不想碰命令行就用这个）

1. 打开你的 fork 仓库 → **Add file → Upload files**
2. 把 `notes` 文件夹和 `index.html` 拖进去
3. 下面填一句 `add study dashboard and notes` → **Commit changes**

点开 `index.html` 里的 **Raw** 看不了渲染效果，想看渲染就做第四步（GitHub Pages）。

### 方式 B：命令行（推荐，配好一次以后都好用）

本地已经有一份稀疏克隆在 `de-zoomcamp/repo`（只拉了 Markdown，2.4 MB，不是完整 315 MB）。直接给它加一个指向你 fork 的 remote 再推：

```bash
cd C:/Users/a1897/WorkBuddy/2026-11-06-21-58-12/de-zoomcamp/repo

# 加你自己的远程仓库
git remote add mine https://github.com/angle-812/data-engineering-zoomcamp.git

# 把看板和笔记拷进来
cp -r ../notes ../index.html .

git add notes index.html
git commit -m "add study dashboard and notes"
git push mine main
```

如果 `git push` 报 `CONNECT tunnel failed, response 502`，是本机的代理在拦，绕过它：

```bash
env -u HTTP_PROXY -u HTTPS_PROXY -u http_proxy -u https_proxy git push mine main
```

第一次 push 会弹浏览器让你登录 GitHub 授权，授权一次以后就不用了。

> 用稀疏克隆的好处：以后 `git pull mine main` 只拉 Markdown，几秒钟完事，不像完整克隆每次都几十兆。

---

## 第三步：以后同步官方更新

```bash
cd C:/Users/a1897/WorkBuddy/2026-11-06-21-58-12/de-zoomcamp/repo
git fetch origin main
git merge origin/main        # 只会在 Markdown 上有冲突，你的 notes/ 不受影响
git push mine main
```

或者懒得用命令行，就在 GitHub 网页上点 **Sync fork**。

---

## 第四步（可选但很香）：开 GitHub Pages，手机也能看

仓库 → **Settings → Pages** → Source 选 **Deploy from a branch** → Branch 选 `main`，目录选 `/ (root)` → Save。

等一两分钟，你的看板就挂在这里了：

```
https://angle-812.github.io/data-engineering-zoomcamp/index.html
```

这个链接的好处：加到手机主屏，通勤路上点开就能看今天该学哪个单元、直接跳 YouTube。进度是存在各自浏览器的 localStorage 里的，电脑和手机互不影响。

---

## 如果你想要更"干净"的仓库

Fork 会把整个课程仓库都复制过来，有些人不喜欢自己主页上挂着一堆不是自己写的东西。那就换个做法：

1. 新建一个空仓库，比如 `de-zoomcamp-notes`
2. 只把 `index.html` 和 `notes/` 推上去（方式 A 或 B 都行，去掉 `git remote add mine` 里的 fork 地址，换成新仓库地址）
3. 课程内容靠 `index.html` 里的链接跳官方仓库看

两种都行，看你更在意"完整副本"还是"干净的作品集"。想好告诉我，我把对应的命令给你写死成脚本。

---

## 生成 / 更新看板

```bash
cd C:/Users/a1897/WorkBuddy/2026-11-06-21-58-12/de-zoomcamp
python build_site.py
```

它会重新读取 `repo/` 里的 `module.yaml` 和各单元 Markdown，抓出视频链接，重建 `index.html`；`notes/` 里已存在的笔记**不会被覆盖**。
