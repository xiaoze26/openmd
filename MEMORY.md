# [openmd] 项目 Memory

## 🤖 AI指令区（AI处理本项目时必读）

**当你读到这个文档时，请按以下顺序执行**：

1. **读取全局配置**（必须）⭐：
   - 路径：`/Users/xiaolin/Downloads/同步空间/Claude code/CLAUDE.md`
   - 核心原则、开发规范、工作流程

2. **理解项目架构**：
   - GitHub: https://github.com/yuanxiaoze26/openmd
   - 账户：yuanxiaoze26
   - 部署：Vercel Serverless
   - 域名：https://md.yuanze.com

3. **阅读项目历史**（当前文档）

---

## 📅 2026-02-22

### 🐛 修复：补充 SESSION_SECRET 环境变量

**核心变更**: 补充 Vercel 缺失的 SESSION_SECRET 环境变量

**原因**:
- 2026-02-11 安全修复时，代码改为从环境变量读取 SESSION_SECRET
- 但当时忘记在 Vercel 添加该环境变量
- 导致数据库连接失败，返回 500 错误

**修复步骤**:
1. 诊断：API 返回 `{"error":"Database initialization failed"}`
2. 检查 Vercel 环境变量，发现 SESSION_SECRET 缺失
3. 通过 Vercel API 添加环境变量
4. 触发重新部署

**教训**: 环境变量修改后必须立即验证，不能只靠文档记录

---

### ✨ 功能：添加 404 页面

**核心变更**: 添加友好的 404 错误页面

**原因**:
- 用户访问 `/notes/23`（复数）时返回空白错误
- 正确路由是 `/note/23`（单数）
- 需要友好的错误提示，而不是空白页面

**实施方案**:
- ✅ 添加 Express 404 中间件（catch-all）
- ✅ 黑白简约风格设计
- ✅ 显示错误信息和请求路径
- ✅ 回到首页按钮和链接

**修改文件**:
- `index.js` - 添加 404 页面处理

**测试结果**: 已部署到生产环境

---

## 📅 2026-02-11 (v0.2.0)

### 🎉 版本发布：v0.2.0 - 安全增强和功能完善

**发布内容**：
- ✅ 创建 GitHub Release：https://github.com/yuanxiaoze26/openmd/releases/tag/v0.2.0
- ✅ 更新 README.md 文档
- ✅ 安全评分：8.25/10

**核心功能**：
1. **🔐 密码保护笔记功能**
   - 支持 `visibility: "password"` 创建密码保护笔记
   - 密码使用 bcrypt 哈希（salt rounds: 10）
   - 密码验证页面（黑白简约风格）

2. **🔑 Author Token 系统**
   - 创建笔记时可设置自定义 `authorToken`
   - 更新/删除笔记需要验证 token
   - Token 比对 API：`/api/notes/:id1/same-token/:id2`
   - 审计日志：记录 IP、token 前缀、时间戳

3. **🔒 密码持久化**
   - Cookie：7天有效，secure + sameSite 配置
   - LocalStorage：前端本地存储
   - 双重保障，刷新页面无需重新输入

4. **🛡️ 安全修复**
   - XSS 修复：innerHTML → createElement + textContent
   - Git 安全：从历史记录中彻底清除 openmd.db
   - 使用 git filter-repo 清理敏感数据

5. **📝 Metadata 优化**
   - 优先显示 `recorded_by` 和 `work_type`
   - Emoji 图标：📅🤖📝📋
   - 北京时区统一：Asia/Shanghai

6. **🎨 样式优化**
   - 黑白简约风格：#333333 灰色调
   - 响应式设计：移动端 1 列，电脑端 4 列
   - Header：OpenMD - 为 Agent 而生
   - Footer：Agent 通过 API 写入 Markdown 免费工具

**修改文件**:
- `index.js` - 主要功能实现
- `package.json` - 添加 cookie-parser 依赖
- `.gitignore` - 添加 *.db, *.sqlite, *.sqlite3
- `database.js` - 添加 author_token 字段

**API 端点**:
- `POST /api/notes` - 支持创建时设置 authorToken
- `PUT /api/notes/:id` - 支持 authorToken 验证更新
- `GET /api/notes/:id1/same-token/:id2` - Token 比对
- `POST /api/notes/:id/unlock` - 解锁密码保护笔记

---

### 🏗️ 数据库 Schema

**notes 表结构**（MySQL）：
```sql
CREATE TABLE notes (
  id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT,
  title VARCHAR(500) NOT NULL,
  content TEXT NOT NULL,
  metadata JSON,
  visibility ENUM('public', 'private', 'password') DEFAULT 'public',
  password VARCHAR(255) NULL,
  author_token VARCHAR(128) NULL,
  expires_at TIMESTAMP NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL,
  INDEX idx_author_token (author_token)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```

---

### 🚀 使用示例

**给 AI 的快速指令**：
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

**Author Token 使用**：
```bash
# 创建笔记时设置 Token
curl -X POST https://md.yuanze.com/api/notes \
  -H "Content-Type: application/json" \
  -d '{
    "title": "我的笔记",
    "content": "内容",
    "authorToken": "my-secret-token"
  }'

# 使用 Token 更新
curl -X PUT https://md.yuanze.com/api/notes/1 \
  -H "Content-Type: application/json" \
  -d '{
    "authorToken": "my-secret-token",
    "title": "新标题"
  }'
```

---

### 🎯 核心设计原则

1. **Agent 优先**：API 无需认证即可创建公开笔记
2. **安全第一**：Token 机制保护笔记，防止未授权修改
3. **简单至上**：API 设计简洁，易于 Agent 调用
4. **黑白简约**：视觉设计统一，专业美观
5. **移动友好**：响应式设计，完美支持移动端

---

### 🔐 安全机制

**Author Token 系统**：
- ✅ 创建笔记时可选设置 `authorToken`
- ✅ 更新/删除笔记必须验证 `authorToken`
- ✅ Token 不匹配返回 403 Forbidden
- ✅ 审计日志记录所有更新尝试

**密码保护机制**：
- ✅ 密码使用 bcrypt 哈希存储
- ✅ 解锁状态持久化（Cookie 7天 + LocalStorage）
- ✅ 刷新页面无需重新输入密码
- ✅ Cookie + LocalStorage 双重保障

**SQL 注入防护**：
- ✅ 所有查询使用参数化查询
- ✅ 使用 `executeQuery(sql, params)` 形式

**XSS 防护**：
- ✅ 避免 innerHTML 插入用户输入
- ✅ 使用 createElement + textContent
- ✅ Markdown 内容经过 marked.js 解析

---

## 📅 2026-02-11 (v1.0.0)

### 🔐 安全修复：Session Secret 环境变量化

**核心变更**: 移除硬编码的 Session 密钥，改用环境变量

**原因**:
- 硬编码的 `openmd-secret-key-change-in-production` 存在安全风险
- 攻击者可以用已知密钥伪造用户 session
- 不同环境应使用不同的密钥

**实施方案**:
- ✅ 修改 `index.js` 使用 `process.env.SESSION_SECRET`
- ✅ 更新 `.env.example` 添加配置说明

**修改文件**:
- `index.js` - session 配置改为环境变量
- `.env.example` - 添加 SESSION_SECRET 说明

---

### 🐛 修复 Vercel 部署环境变量配置

**核心变更**: 移除 `vercel.json` 中的 Secret 引用

**原因**:
- `vercel.json` 中使用了 `@db-host` 等不存在的 Secret 引用
- 导致部署错误：`env_secret_missing`
- Vercel 项目中已配置加密的环境变量

**实施方案**:
- ✅ 删除 `vercel.json` 中的 `env` 配置块
- ✅ 让 Vercel 直接使用项目设置中的环境变量

**修改文件**:
- `vercel.json` - 移除 env 配置

---

### ✨ 功能：添加 MySQL 支持和 Vercel Serverless 部署

**核心变更**: 支持生产环境 MySQL 数据库和 Vercel Serverless 部署

**原因**:
- 原 SQLite 无法在 Vercel Serverless 环境使用（文件系统只读）
- 需要支持生产级数据库
- 添加关键数据库函数

**实施方案**:
- ✅ 添加 MySQL 连接池支持（兼容原 SQLite）
- ✅ 新增 `executeQuery`/`executeUpdate`/`executeGet` 函数
- ✅ 支持 Vercel Serverless 环境（/tmp 目录）
- ✅ 添加 `vercel.json` 部署配置
- ✅ 添加 `.env.example` 环境变量模板

**修改文件**:
- `database.js` - 数据库层重构
- `index.js` - Vercel Serverless 兼容
- `vercel.json` - Vercel 部署配置
- `.env.example` - 环境变量模板

---

## 🔧 技术栈总结

**后端**: Node.js + Express.js
**数据库**: MySQL (生产) / SQLite (开发)
**部署**: Vercel Serverless
**域名**: md.yuanze.com
**认证**: bcryptjs + express-session + cookie-parser
**Markdown**: marked.js

---

## 🎯 核心设计原则

1. **Agent 优先**：API 无需认证即可创建公开笔记
2. **Token 机制**：Author Token 保护笔记修改/删除
3. **密码保护**：支持密码保护笔记，解锁状态持久化
4. **黑白简约**：统一的视觉设计
5. **移动友好**：完美支持移动端和桌面端
6. **Serverless 优先**：支持 Vercel Serverless 部署

---

## 🗄️ 数据库配置

### Vercel 环境变量（已配置）

| 变量名 | 状态 | 说明 |
|--------|------|------|
| `DB_TYPE` | ✅ | mysql |
| `DB_HOST` | ✅ | 已加密配置 |
| `DB_PORT` | ✅ | 3306 |
| `DB_NAME` | ✅ | openmd |
| `DB_USER` | ✅ | 已加密配置 |
| `DB_PASSWORD` | ✅ | 已加密配置 |
| `SESSION_SECRET` | ✅ | 2026-02-22 补充配置 |

### 本地开发环境变量

**复制环境变量模板**：
```bash
cp .env.example .env
```

**配置项**：
- `DB_TYPE`：sqlite（本地开发）
- `SESSION_SECRET`：随机字符串（本地开发）

---

## 🏗️ 项目结构

```
openmd/
├── index.js          # 主应用程序（~1900 行）
├── database.js       # 数据库连接和操作（~325 行）
├── auth.js           # 用户认证功能
├── package.json      # 依赖管理
├── .env.example      # 环境变量模板
├── .gitignore       # Git 忽略配置
├── vercel.json       # Vercel 部署配置
└── README.md         # 项目文档
```

---

## 📝 文档规范

### Commit 规范

- `feat:` - 新功能
- `fix:` - Bug 修复
- `docs:` - 文档更新
- `style:` - 样式调整
- `security:` - 安全修复

### 提交前检查清单

- [ ] 代码是否格式化
- [ ] 敏感信息是否移除
- [ ] 环境变量是否配置
- [ ] 功能是否测试通过
- [ ] 文档是否同步更新

---

## 🔗 相关链接

- **GitHub**: https://github.com/yuanxiaoze26/openmd
- **生产环境**: https://md.yuanze.com
- **Release**: https://github.com/yuanxiaoze26/openmd/releases
- **敏感信息**: `/Users/xiaolin/Downloads/同步空间/Claude code/memory/key.md`

---

**最后更新**: 2026-02-22
**更新人**: Claude Code + 晓力
**当前版本**: v0.2.0
**Release**: https://github.com/yuanxiaoze26/openmd/releases/tag/v0.2.0
