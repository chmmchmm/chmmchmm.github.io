# chmmchmm.github.io — 开发者网站（域名根）

`https://chmmchmm.github.io/` 的源仓库。它的存在只为了一件事：**占住 `chmmchmm.github.io` 这个域名的根路径**，
因为有两类外部校验只认域名根，不认子目录：

* **广告网络**（AdMob 等）：按 App Store listing 里 **marketing URL（开发者网站）的 hostname** 抓
  `https://chmmchmm.github.io/app-ads.txt`。iOS 上认的是 marketing URL 字段，不是 support URL、也不是隐私政策 URL。
* **App Store 审核员**：会从自己所在地区的网络点商店页上的链接，链接落到哪儿就得能打开。

## 目录

```
/                    开发者主页（index.html）
/app-ads.txt         App 广告授权方（当前为占位：只有注释，无有效行）
/ads.txt             网站展示广告授权方（当前为占位）
/robots.txt          不做任何抓取限制（禁止 Disallow 挡 ads 文件）
/404.html            站点级 404（Pages 每站只认根目录这一个）
/.nojekyll           关掉 Jekyll 处理：静态文件原样发布，也免得构建排队延迟
/apps/fs/            Face Shield 支持页          /apps/fs/privacy/  隐私政策
/apps/dcm/           DICOM Viewer 支持页         /apps/dcm/privacy/ 隐私政策
```

## ⚠️ 目录命名的硬规矩

**这里不能建名为 `dcmview` 或 `faceshield` 的目录。** 那两个是同名仓库的项目页
（`/dcmview/`、`/faceshield/`），项目页的优先级**高于** user-site 的同名目录，会被抢走；
而且这两条项目页仍在给商店里的 `supportUrl` / `privacyPolicyUrl` 服务（版本审核期间不动它们），
所以这里是 `/apps/dcm/`、`/apps/fs/` 而不是 `/dcmview/`、`/faceshield/`。

## 同步：页面正文的唯一来源还是各 app 仓库

应用页内容的源是 `../app0/site`（Face Shield）和 `../app1/site`（DICOM Viewer）。改了那边就整份复制过来，
再一起推送（`apps/**` 全部是从 app 仓库复制来的，不要在这个仓库里直接改正文）：

```bash
cd /Users/lianglin/Desktop/2026.10/chmmchmm.github.io

# DICOM：index 与 privacy
cp ../app1/site/index.html            apps/dcm/index.html
cp ../app1/site/privacy/index.html    apps/dcm/privacy/index.html
# Face Shield：index 与 privacy
cp ../app0/site/index.html            apps/fs/index.html
cp ../app0/site/privacy/index.html    apps/fs/privacy/index.html

git add -A && git commit -m "Sync app pages from app0/site, app1/site" && git push
```

推送后 Pages 部署通常十几秒到一两分钟，可用下面命令确认（都应 200）：

```bash
for u in / /app-ads.txt /ads.txt /apps/dcm/ /apps/dcm/privacy/ /apps/fs/ /apps/fs/privacy/; do
  printf "%-26s %s\n" "$u" "$(curl -s -o /dev/null -w '%{http_code}' -x http://127.0.0.1:7890 "https://chmmchmm.github.io$u")"
done
```

## 以后真接广告时要做的事

1. 把 AdMob 后台给出的 personalized snippet（核心一行 `google.com, pub-XXXXXXXX, DIRECT, f08c47fec0942fa0`）
   贴进 `app-ads.txt`，删掉这里的说明性注释也行，但保留 `#` 注释不影响校验。
2. 在 App Store Connect 里把该版本的 **marketing URL** 填成 `https://chmmchmm.github.io/`
   （API：`PATCH /v1/appStoreVersionLocalizations/<id>`，字段 `marketingUrl`；注意这个资源不允许 GET 集合，只能按 id 单查）。
3. 等 Google 自己来爬（几天到一周），AdMob 后台变绿再接 SDK。
4. 同步改：App Privacy 标签（广告 SDK 采集设备标识 ⇒ Data Collected）、ATT 弹窗、隐私政策与商店文案里
   "不收集数据 / 无第三方 SDK" 的表述。
