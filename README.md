# Birthday Biosystem v22.0

一个为周毅芬 22 岁生日设计的 Vue 惊喜页，结合北京协和医学院、生物科研和朋友共同成长的主题。包含全屏检测开场、彩纸爆发、照片风暴、照片放大、显微镜彩蛋、同行评审式祝福、祝愿培养皿，以及长按吹灭蜡烛的结尾仪式。

## 本地运行

```powershell
pnpm install
pnpm dev
```

生产构建：

```powershell
pnpm build
```

## 内容配置

所有姓名、年龄、日期、祝福和照片说明都集中在：

```text
src/data/birthday.js
```

最终照片位于：

```text
public/photos/
```

当前版本使用 10 张毕业、校园、朋友和共同荣誉照片组成回忆相册，`11-personal.jpg` 专用于最终许愿蛋糕旁。人物裁切焦点、相册文案和弹窗说明均维护在 `birthday.js`。

## GitHub Pages

仓库已经包含 `.github/workflows/deploy.yml`。推送到 GitHub 的 `main` 分支后，在仓库设置中将 Pages 的 Source 设为 **GitHub Actions**，之后每次推送都会自动构建并发布。

Vite 已使用相对资源路径，因此仓库名无需写死在配置中。

## 技术栈

- Vue 3 + Vite
- GSAP
- canvas-confetti
- Lucide icons

正式发布前，请确认照片中所有出镜者同意将照片放在公开网页中；奖状照片中可能包含可识别姓名，也应一并确认。
