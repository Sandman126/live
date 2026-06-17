# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

免费开源的中文 IPTV/广播图标库与配套 Web 工具。纯静态前端项目，**无构建流程、无包管理器、无测试框架**。通过 GitHub Pages + Cloudflare Pages 双栈部署，依赖均通过 CDN 加载。

上游仓库: [fanmingming/live](https://github.com/fanmingming/live)

## 部署模式

- **主站** `live.fanmingming.com` — GitHub Pages（由 `main` 分支驱动），走 Cloudflare CDN
- **镜像站** `live.126413.xyz` — Cloudflare Pages（监听 `main` 分支自动构建）
- 本仓库是上游的 fork，直接向 `main` 分支 push 即可触发双站部署

## 关键文件与目录

| 路径 | 说明 |
|---|---|
| `tv/*.png` | 929 个电视频道图标，`tv/m3u/` 下为 M3U 播放列表 |
| `radio/*.png` | 461 个广播电台图标，`radio/m3u/` 下为 M3U 播放列表 |
| `e.xml` | EPG 电子节目单数据（7.9MB），每 2 小时自动更新 |
| `m3u8/index.html` | **M3U8 视频下载器** — Vue.js SPA，支持 AES 解密、TS→MP4 合成、StreamSaver 流式下载 |
| `m3u8/serviceworker.js` | StreamSaver 的 Service Worker，实现大文件流式下载 |
| `m3u8/aes-decryptor.js` | HLS AES-128 解密逻辑 |
| `m3u8/mux-mp4.js` | TS 封装段转 MP4（mux.js 封装） |
| `player/hls.html` | HLS 播放器（hls.js），URL 参数 `?vurl=` |
| `player/index.html` | 腾讯云 TCPlayer 播放器，支持 HLS/FLV/DASH |
| `player/cq/index.html` | 重庆本地电视播放器，调用中国广电 MD5 签名 API |
| `txt2m3u/index.html` | TXT→M3U 在线转换器，纯前端无上传 |
| `worker/epg.js` | Cloudflare Worker — DIYP 兼容 EPG 接口 |
| `worker/radio.js` | Cloudflare Worker — 云听 FM 代理 |
| `.github/workflows/epg.yml` | GitHub Actions — 每 2 小时更新 e.xml |

## M3U 播放列表结构

播放列表使用标准 M3U 格式，示例：

```
#EXTM3U x-tvg-url="https://live.126413.xyz/e.xml"
#EXTINF:-1 tvg-name="CCTV1" tvg-logo="https://live.126413.xyz/tv/CCTV1.png" group-title="央视",CCTV-1 综合
直播源URL
```

- `tvg-logo` 引用本项目 `tv/` 或 `radio/` 下的图标
- `x-tvg-url` 指向本项目的 EPG XML
- `group-title` 用于播放器频道分组

## 添加电视/广播图标

1. 将图标 PNG 放入 `tv/` 或 `radio/` 目录
2. 文件名作为公开访问路径（如 `CCTV1.png` → `https://live.126413.xyz/tv/CCTV1.png`）
3. 提交到 `main` 分支即自动部署

## Cloudflare Workers 部署

`worker/epg.js` 和 `worker/radio.js` 需手动部署：
- 登录 Cloudflare Dashboard → Workers 和 Pages → 创建 Worker
- 粘贴 JS 源码 → 部署 → 绑定自定义域名
- 无自动化部署管道

## 工具页面参数说明

- **HLS 播放器** `player/hls.html?vurl=直播源URL`
- **TCPlayer** `player/index.html?vurl=直播源URL`
- **M3U8 下载器** `m3u8/index.html` — 在页面输入 URL 交互操作

## EPG 更新机制

GitHub Actions (`epg.yml`) 每 2 小时：
1. 从 `suzukua/epg` 仓库下载最新 `t.xml`
2. 覆盖本仓库 `e.xml`
3. 直接 commit 并 push 回 `main`
