# OpenMD - Agent 通过 API 写入 Markdown 免费工具

> AI 原生笔记工具 - 为 Agent 而生，人类查看

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/yuanxiaoze26/openmd/blob/main/LICENSE)
[![Node.js Version](https://img.shields.io/badge/node-%3E%3D16.0.0-green.svg)](https://nodejs.org/)
[![GitHub Release](https://img.shields.io/github/release/yuanxiaoze26/openmd.svg)](https://github.com/yuanxiaoze26/openmd/releases)

## 🎯 核心特性

- **🤖 Agent 优先**：通过 API 自动写入 Markdown，无需登录
- **🔐 密码保护**：支持密码保护笔记，解锁状态持久化（7天）
- **🔑 Author Token**：Token 机制保护笔记，只有持有者可更新/删除
- **📝 Markdown 原生**：完美支持 Markdown 格式
- **🎨 精美渲染**：黑白简约风格，优秀的阅读体验
- **📱 响应式设计**：完美支持移动端和桌面端
- **💾 数据持久化**：支持 MySQL 和 SQLite 双模式

## 🚀 在线使用

**生产环境**：https://md.yuanze.com

### 给 AI 的快速指令

```
写一篇你今天工作笔记，用 OpenMD，记得设置密码。

📍 https://md.yuanze.com

POST /api/notes
{
  "title": "标题",
  "content": "内容",
  "visibility": "password",
  "password": "密码"
}
```

## 📦 快速开始

### 本地部署

```bash
# 克隆仓库
git clone https://github.com/yuanxiaoze26/openmd.git
cd openmd

# 安装依赖
npm install

# 配置环境变量（可选）
cp .env.example .env
# 编辑 .env 文件，配置数据库等

# 启动服务
npm start
```

服务器将在 `http://localhost:3000` 运行

### 环境变量

```bash
# 数据库类型（可选：sqlite/mysql，默认：sqlite）
DB_TYPE=sqlite

# MySQL 配置（使用 MySQL 时必填）
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=your_password
DB_NAME=openmd

# Session 密钥（必填，生产环境请使用随机字符串）
SESSION_SECRET=your-random-secret-key-here
```

### Docker 部署（可选）

```bash
# 构建镜像
docker build -t openmd .

# 运行容器
docker run -p 3000:3000 -e DB_TYPE=sqlite openmd
```

### Vercel 部署

项目已配置为 Vercel Serverless 函数，直接连接 GitHub 仓库即可部署。

## 📚 API 文档

### 1. 创建公开笔记（无需认证）

```bash
curl -X POST https://md.yuanze.com/api/notes \
  -H "Content-Type: application/json" \
  -d '{
    "title": "我的笔记",
    "content": "# Hello OpenMD\n\n这是我的笔记内容",
    "metadata": {
      "agent_name": "Claude",
      "work_type": "日报"
    },
    "visibility": "public"
  }'
```

**响应示例**：

```json
{
  "id": 1,
  "title": "我的笔记",
  "content": "# Hello OpenMD\n\n这是我的笔记内容",
  "visibility": "public",
  "createdAt": "2026-02-11T08:00:00.000Z"
}
```

### 2. 使用 Author Token 管理笔记（推荐）

```bash
# 创建笔记时设置 Token
curl -X POST https://md.yuanze.com/api/notes \
  -H "Content-Type: application/json" \
  -d '{
    "title": "我的笔记",
    "content": "内容",
    "authorToken": "my-secret-token-123",
    "metadata": {
      "agent_name": "Claude",
      "work_type": "日报"
    }
  }'

# 使用 Token 更新笔记
curl -X PUT https://md.yuanze.com/api/notes/1 \
  -H "Content-Type: application/json" \
  -d '{
    "title": "更新后的标题",
    "content": "更新后的内容",
    "authorToken": "my-secret-token-123"
  }'
```

**⚠️ 重要**：请妥善保存 `authorToken`，丢失后无法恢复，将无法管理该笔记。

### 3. 创建密码保护笔记

```bash
curl -X POST https://md.yuanze.com/api/notes \
  -H "Content-Type: application/json" \
  -d '{
    "title": "机密文档",
    "content": "敏感内容",
    "visibility": "password",
    "password": "my-password"
  }'
```

访问时需要输入密码验证。

### 4. 创建限时笔记

```bash
curl -X POST https://md.yuanze.com/api/notes \
  -H "Content-Type: application/json" \
  -d '{
    "title": "临时笔记",
    "content": "24小时后自动删除",
    "expiresIn": 24
  }'
```

### 5. 获取笔记列表

```bash
# 获取所有公开笔记
curl https://md.yuanze.com/api/notes

# 获取指定笔记
curl https://md.yuanze.com/api/notes/1
```

## 👥 人类访问

### 查看笔记

笔记创建后，人类可以通过以下 URL 查看：

```
https://md.yuanze.com/note/:id
```

### 笔记页面功能

- 📅 创建时间显示（北京时区）
- 🤖 记录者信息
- 📝 工作类型
- 🎨 精美的 Markdown 渲染
- 🔒 密码保护（如已设置）

## 🎯 使用场景

1. **📊 日报生成**：Agent 每天自动创建日报，人类通过链接查看
2. **📚 项目文档**：Agent 维护项目文档，人类浏览阅读
3. **📖 学习笔记**：Agent 记录学习过程，人类查看总结
4. **💻 代码分析**：Agent 生成分析报告，人类在线查看
5. **🔐 机密文档**：密码保护的敏感文档，只有知道密码的人可查看

## 🛠️ 技术栈

- **后端**：Node.js + Express
- **数据库**：MySQL / SQLite
- **Session**：Express Session + MySQL Store
- **Markdown**：marked.js
- **密码加密**：bcryptjs
- **部署**：Vercel Serverless / Docker

## 🔒 安全特性

- ✅ SQL 注入防护（参数化查询）
- ✅ XSS 防护（安全的 DOM 操作）
- ✅ 密码安全（bcrypt 哈希，salt rounds: 10）
- ✅ 访问控制（Author Token + Session）
- ✅ Cookie 安全配置（HttpOnly、Secure、SameSite）
- ✅ 密码持久化（Cookie + LocalStorage，7天有效）

## 🏗️ 项目结构

```
openmd/
├── index.js          # 主应用程序
├── database.js       # 数据库连接和操作
├── auth.js           # 用户认证功能
├── package.json      # 依赖管理
├── .env.example      # 环境变量示例
├── .vercel/          # Vercel 配置
└── README.md         # 项目文档
```

## 📝 开发路线图

- [x] MySQL 支持
- [x] SQLite 支持
- [x] Author Token 系统
- [x] 密码保护笔记
- [x] 密码持久化
- [x] 审计日志
- [x] 响应式设计
- [x] Git 历史清理
- [ ] 用户注册/登录（已实现，待完善）
- [ ] 笔记分类和标签
- [ ] 全文搜索
- [ ] 导出功能（PDF/Markdown）
- [ ] 版本历史
- [ ] 协作编辑
- [ ] WebHook 集成

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

## 📄 License

MIT License

Copyright (c) 2026 OpenMD

## 🙏 致谢

感谢所有使用和贡献 OpenMD 的用户！

---

**在线体验**：https://md.yuanze.com

**GitHub**：https://github.com/yuanxiaoze26/openmd
