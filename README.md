# gubo-wafermap-skill-installer

`@guwave/wafermap-v2` 晶圆图渲染组件 AI Skill 一键安装器。

支持将 Wafer Map Skill 安装到 Cursor / Claude Code / Gemini CLI / Codex 等 AI 编程助手中，让 AI 掌握 `@guwave/wafermap-v2` 的使用方法。

---

## 快速安装

```bash
npx gubo-wafermap-skill-installer
```

CLI 会引导你选择安装目标路径，支持：

- 当前目录
- Cursor（项目级 / 全局）
- Claude Code
- Gemini CLI
- Codex
- 全部安装

---

## Skill 能力

安装后，AI 编程助手将掌握以下知识：

- `.npmrc` 配置（`@guwave` 私有仓库）
- `@guwave/wafermap-v2` 的安装与导入
- 核心概念：Wafer、Die、Reticle、坐标系
- 两种渲染模式：`dashboard` / `waferMap-config`
- `WaferMapOptions` 完整配置项
- React / Vue 3 框架集成
- BinMap、Reticle、高亮与框选、SublinePlugin 辅助线
- 颜色工具函数
- 完整 TypeScript 类型参考

---

## 本地开发

> Node >= 18

```bash
pnpm install
node bin/cli.mjs
```

---

## 项目结构

```text
gubo-wafermap-skill-installer/
├── bin/
│   └── cli.mjs              # CLI 入口
├── src/
│   ├── config.mjs            # Skill 配置 (名称、下载地址等)
│   ├── install.mjs           # 安装逻辑 (下载、解压、写入)
│   └── targets.mjs           # 安装目标路径定义
└── skills/
    └── wafer-map/
        ├── SKILL.md           # Skill 主文件
        └── reference.md       # TypeScript 类型参考
```

---

## License

MIT
