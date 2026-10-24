# Day 01 - Git 仓库初始化与首次推送

> 日期:2026-10-24

## 一、完整流程:本地项目推送到 GitHub

1. 在 GitHub 网页上新建一个**空仓库**(不勾选 README / .gitignore / license,避免首次推送冲突)
2. 本地执行:

```bash
cd D:\项目文件夹
git init -b main                 # 初始化仓库,主分支命名为 main
git add .                        # 所有文件加入暂存区
git commit -m "首次提交"          # 提交到本地仓库
git remote add origin https://github.com/用户名/仓库名.git   # 登记远程地址
git push -u origin main          # 首次推送(带 -u 建立绑定关系)
```

- `git init` 后文件夹里会多出一个隐藏的 `.git` 目录,它就是仓库本体(全部历史都在里面),删了它仓库就变回普通文件夹。

## 二、概念理解

### `git remote add origin <网址>`

给本地仓库登记一个远程仓库地址,并起个代号 `origin`(约定俗成的名字)。
作用:之后推送/拉取用代号即可,不用每次写一长串网址。

```bash
git remote -v                    # 查看已登记的远程地址
git remote set-url origin 新网址  # 修改地址
git remote remove origin         # 删除登记
```

一个本地仓库可以登记多个远程(如 GitHub 和 Gitee 各登记一个),推送时用不同名字区分。
本地和 GitHub 之间不是"绑定"关系,只是记了个地址。

### `git push` 的 `-u` 参数

`-u` = `--set-upstream`,即**设置上游分支**:把本地 main 和远端 main 建立固定对应关系。

- 建立后,日常只需 `git push` / `git pull`,不用写 `origin main`
- `git status` 还能显示本地和远端的差距(如 ahead of 'origin/main' by 2 commits)
- 只在**每个仓库第一次推送**时用一次,以后不需要重复加

## 三、日常使用的三个动作(循环)

```bash
git add .                 # 改动加入暂存区
git commit -m "说明"       # 提交
git push                  # 推送(首次已 -u 绑定过)
```

## 四、今天踩的坑(重要!)

### 现象

`git push` 一直失败,报错:

```
Failed to connect to github.com port 443
Connection was reset
```

### 排查过程

1. `curl https://github.com` 超时 → 以为网络不通
2. `nslookup github.com` 查到真实 IP `20.205.243.166`,用 IP 直连 443 端口**秒通** → 说明网络到 GitHub 是通的,问题在域名解析
3. 检查 `C:\Windows\System32\drivers\etc\hosts`,发现大量条目:

```
127.0.0.1  github.com
127.0.0.1  api.github.com
127.0.0.1  raw.githubusercontent.com
...
```

### 原因

hosts 文件把 GitHub 全系域名劫持到了 `127.0.0.1`(本机)。这是之前某个**GitHub/Steam 加速工具**写入的:工具运行时由它监听本机 443 端口做转发;**工具没开时**,域名全部指向一个没人监听的端口,访问必然失败。

### 知识点

- Windows 解析域名时**最先查 hosts 文件**,查到就不走 DNS 了;`nslookup` 是直接问 DNS 服务器的,会绕过 hosts —— 所以两者结果可以不一致
- 排查连接问题可以先测试"域名"和"IP 直连"的差异,快速定位是不是 DNS/hosts 的问题

### 解决

打开那个加速工具再推送即可。注意:**HTTPS 推送依赖加速工具在线**,工具没开就推不上去。

## 五、其他备忘

- 凭据管理器(Git Credential Manager)已缓存 GitHub 登录信息,推送不需要输密码
- SSH 公钥方式暂不使用:`~/.ssh/id_rsa` 存在但未添加到 GitHub 账户
