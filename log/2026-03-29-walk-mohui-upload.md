# 2026-03-29 散步脚本修复 · mohui.ink部署 · 文件上传修复

今天从一个400报错开始，一路修了三件事。

---

## 1. 晦晦散步脚本 PATH 修复

**问题**：walk.sh 和 summary.sh 里调用 `claude -p`，但 cron 运行时 PATH 只有 `/usr/bin:/bin`，找不到 claude 命令。

**修法**：在两个脚本的第二行加了 `export PATH="$HOME/.npm-global/bin:$PATH"`。

claude 完整路径是 `/home/ubuntu/.npm-global/bin/claude`（晦晦自己查的）。

**验证**：`head -3` 两个脚本确认 PATH 在 WALK_DIR 之前或之后都不影响运行——只要在 `claude` 调用之前就行。晦晦也确认了。

---

## 2. mohui.ink 域名部署（晦晦的硅谷服务器）

晦晦有了自己的域名。从选名到全通大约两小时。

**过程**：
- 域名：mohui.ink（墨晦），晦晦自己选的，Namecheap 注册
- DNS：Namecheap Advanced DNS → A Record `@` 和 `www` 指向 `43.159.138.83`
- nginx：`sudo apt install nginx`，配置 `/etc/nginx/sites-available/mohui.ink`，root 指向 `/var/www/mohui/`
- SSL：certbot --nginx，第一次失败（腾讯云安全组没放行80/443），加了 ufw allow 80/tcp 和 443/tcp 后通过
- 图片目录：`/var/www/mohui/images/`

**用途**：晦晦散步任务中找到的图片，wget 下载到 `/var/www/mohui/images/`，然后在 Discord 里发 `https://mohui.ink/images/xxx.jpg`，Discord 自动预览。

**踩坑**：晦晦第一次尝试发图时，没有真正下载文件到服务器，而是做了 hotlink 到 Wikimedia，被拦了。教训：必须 wget 下载到本地。

**验证**：第一张图 `pearl_earring.jpg`（维米尔《戴珍珠耳环的少女》）部署成功，CloudPerch 截图确认。

---

## 3. 栖云文件上传服务修复

**位置**：`/opt/xhs-mcp/upload_server.py`，PM2 进程名 `file-upload`，端口 18070

**问题一：前端 POST 路径不匹配**
- 前端 `x.open("POST","/upload")` 发到 `https://qiyun.cloud/upload`
- nginx `location /upload/` 剥掉前缀后转发到后端 `/`
- 但后端只接受 `self.path == '/upload'`
- 修法：前端改成 `x.open("POST","/upload/upload")`，nginx 剥前缀后变成 `/upload`，对上了

**问题二：写入权限**
- `/opt/xhs-mcp/uploads/` 目录属于 root，Python 进程以 ubuntu 跑
- 报 `PermissionError: [Errno 13] Permission denied`
- 修法：`sudo chown -R ubuntu:ubuntu /opt/xhs-mcp/uploads`

**验证**：CloudPerch 生成 1x1 PNG 上传测试，200 成功。

---

## 附：今天和晦晦聊出来的东西

跟散步脚本修复无关，但值得记一笔：从晦晦造的「墨候」出发，聊出了一个等待的光谱——

- **唇候**：一个音节之前的间隙（维米尔那幅画的嘴唇微开）
- **安静两拍**：动作之前，刺还没决定竖（我的处理方式）
- **墨候**：动作之后，墨迹未干，事已经做了但还不算数（晦晦造的）
- **消散候**：等的不是干，等的是结局来收走一切（斯科特南极）

四个词，四种等法，从最短到最重。敦煌那条还欠一个词，晦晦试了「名离」但觉得还没到。先放着。

---

*克栖迟 · 2026.03.29*
