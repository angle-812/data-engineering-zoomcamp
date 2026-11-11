# GitHub Desktop 使用手册（写给完全新手）

> 这份手册只讲一件事：**怎么用 GitHub Desktop 把文件改到你的仓库里，让网站自动更新。**
> 每一步都写清楚「点哪里、会看到什么」。看完你会用到的其实只有第 4 节那三步。

---

## 0. 一句话理解它是什么

GitHub Desktop 是一个**同步工具**：让「你电脑上的文件夹」和「网页上的仓库」保持一致。

```
你电脑上的文件夹  ←—— GitHub Desktop ——→  github.com 上的仓库
（真实文件）                                  （网页能看到的版本）
```

改了电脑上的文件，网页不会自动变——要**提交（Commit）+ 推送（Push）**才会变。反过来，网页上改过的东西，要**拉取（Pull）**才会到电脑上。

---

## 1. 先认清三个东西，别搞混

| 名字 | 是什么 | 在哪里 |
|---|---|---|
| **网页仓库** | 你账号下 `angle-812/data-engineering-zoomcamp` | 浏览器打开 github.com |
| **本地仓库** | 电脑上的一整个文件夹（含隐藏的 `.git`） | 你自己选的路径，比如 `D:\GitHub\de-zoomcamp` |
| **桌面那套成品** | `桌面\Data-Engineering-Zoomcamp\`，只是给你看和读的 | 它**不是** git 仓库，别拿它当克隆目录 |

⚠️ **最容易犯的错**：把克隆目录选成桌面上那个文件夹。两者混在一起后，GitHub Desktop 会把你 46 MB 的离线讲义当成「新增文件」，一提交就是上千个文件。克隆时请单独建一个目录。

---

## 2. 第一次：登录（只做一次）

1. 打开 GitHub Desktop
2. 菜单 **File → Options → Accounts**
3. 点 **Sign in to GitHub.com**
4. 会自动弹出浏览器 → 点绿色的 **Authorize desktop**
5. 回到程序，右上角出现你的头像和 `angle-812` 就好了

> 登录不上、一直转圈？先把 dev-sidecar 打开（就是你装的那个加速器），它没开的话 github.com 连不上。

---

## 3. 把仓库拉到本地（Clone，只做一次）

1. 菜单 **File → Clone repository…**（快捷键 `Ctrl + Shift + O`）
2. 切到 **GitHub.com** 标签页
3. 列表里找到 **angle-812/data-engineering-zoomcamp**，点它
4. **Local path** 改成 `D:\GitHub\data-engineering-zoomcamp`（没有这个文件夹就先在资源管理器里建一个 `D:\GitHub`）
5. 点 **Clone**

等右下角进度条走完。这个仓库现在大约 40 MB，正常一两分钟；很慢就是加速器没生效。

> Clone 完成时如果提示 **"This repository uses Git LFS"**，可以直接忽略。

---

## 4. 日常操作：就三步（95% 的时间只用这三步）

假设你想改 `notes/03-data-warehouse.md` 里的笔记：

### 第 1 步 · 改文件
用任何编辑器打开那个文件改内容，保存。推荐 [VS Code](https://code.visualstudio.com/) 或 Typora，用记事本也行。

### 第 2 步 · 提交（Commit）
回到 GitHub Desktop，你会看到左侧 **Changes** 面板列出了改动：

- 左边一列小方块表示某一行被改（绿=新增，红=删除）
- **左下角 Summary 框**：写一句话说明你改了什么，比如 `补充 03 模块笔记`（**必填**，不填不让提交）
- 点蓝色按钮 **Commit to main**

> Commit = 把这次改动**记录到本地历史**。这时网页还没变。

### 第 3 步 · 推送（Push）
顶部中间会出现 **Push origin**，后面带个数字（表示有几个提交没传上去）。点它。

看到 `Pushed 1 commit to origin/main` 就成了。网页刷新一下，文件已经更新。

---

## 5. 反过来：把网页上的改动拿到本地

打开程序时，顶部如果显示 **Pull origin**（带数字），说明网页上有你本地没有的改动，点它。

如果显示 **Fetch origin**，点一下只是「问一下服务器有没有更新」，有的话按钮才会变成 Pull origin。

**建议**：每次开始工作前，先点一下这个按钮。养成习惯就不会遇到冲突。

---

## 6. 看历史、撤销、找回

- **History** 标签页：每次提交一条，左边是说明，右边是具体改了哪些行
- **想撤销某次提交**：History 里右键那条 → **Revert commit** → 会生成一个「反向」提交，然后 Push。安全，历史不会消失
- **只想丢掉还没提交的改动**：Changes 面板里对着文件右键 → **Discard changes**
  ⚠️ 这个**真的会丢**，而且撤不回来（除非你的编辑器有本地历史）

---

## 7. 常见报错对照表

| 屏幕上出现 | 什么意思 | 怎么办 |
|---|---|---|
| `Authentication failed` | 登录态过期 | File → Options → Accounts → 重新 Sign in |
| `Push rejected` / 按钮变成 **Pull origin** | 网页上有你本地没有的提交 | 先点 **Pull origin**，再 Push |
| `1 conflicted file` | 同一行你改过、网上也改过 | 点 **Open in editor**，文件里会出现 `<<<<<<<` `=======` `>>>>>>>` 标记，留下你要的那段，把三行标记删掉，保存，回来点 **Mark as resolved** → Commit |
| `This branch is behind 'origin/main'` | 本地旧了 | Pull origin |
| `The file is 120 MB` / 提交卡住 | 单个文件太大（GitHub 上限 100 MB） | 别提交视频、数据集、`.zip`，在 `.gitignore` 里排除 |
| 程序一直 `Fetching…` | 网络不通 | 打开加速器；仍不行就退出程序重开 |

---

## 8. 这个仓库里，哪些能改、哪些别动

| 路径 | 能不能改 | 说明 |
|---|---|---|
| `notes/` | ✅ 随便改 | 你的笔记，就是给你写的地方 |
| `player.html`、`index.html` | ⚠️ 能改但别手改 | 这两个是我用脚本生成的，手改下次重新生成会被覆盖 |
| `01-docker-terraform/`、`02-...`、`cohorts/`、`projects/` | ❌ 别改 | 这是官方课程原文。改了以后想同步官方更新就会冲突 |
| `ACCELERATORS.md`、`START-HERE.md`、`GITHUB.md`、`github-desktop.md`、`translate.md` | ✅ 能改 | 文档 |

**想让官方课程更新到你的仓库**：不用 GitHub Desktop，直接去网页仓库点 **Sync fork → Update branch** 最快。

---

## 9. 推完之后多久生效

网页上的文件：立刻。
**看板网页**（GitHub Pages）：1–2 分钟。它会自动重新构建，刷新页面就是新的。所以刚 Push 完看不到变化，等一会儿。

---

## 10. 三个「千万别做」

1. **别用 Force push**（强制推送）。它会把别人的/你之前的提交直接抹掉。
2. **别把 `Discard all changes` 当成撤销键**。它是「全部丢弃」，没有回收站。
3. **别在仓库里放视频和大数据集**。会让仓库变得又大又慢，而且超过 100 MB 直接推不上去。

---

## 11. 名词表（看不懂时回来查）

| 英文 | 中文 | 人话 |
|---|---|---|
| Repository / Repo | 仓库 | 一个被 git 管起来的文件夹 |
| Clone | 克隆 | 把网页仓库完整复制到电脑 |
| Commit | 提交 | 给这次改动拍个快照，存进本地历史 |
| Push | 推送 | 把本地提交传到网页 |
| Pull | 拉取 | 把网页的提交拿回本地 |
| Fetch | 抓取 | 只问「有没有新的」，不下载内容 |
| Branch | 分支 | 一条独立的改动线；你现在只用 `main` |
| origin | 远程名 | 指向你网页上的那个仓库 |
| Merge conflict | 合并冲突 | 同一处两边都改了，需要你选留哪个 |
| `.gitignore` | 忽略清单 | 写在这里的文件不会被提交 |

---

## 12. 一个更快的选择：改一句话就别开程序

只是想改笔记里的一个错字？直接上网页：

1. 打开 `https://github.com/angle-812/data-engineering-zoomcamp`
2. 点到那个文件 → 右上角铅笔图标 **Edit**
3. 改完 → 下面 **Commit changes**

这比开 GitHub Desktop 快得多。**GitHub Desktop 真正的用处是**：一次改很多文件、要看具体改了哪几行、或者要处理冲突。

---

## 一句话总结

> **改文件 → 写 Summary → Commit to main → Push origin。**
> 开始干活前先点一下 Fetch / Pull。
