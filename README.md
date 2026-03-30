# Spring AI MCP + RAG 知识库

基于 **Spring AI 1.0.0** 构建的智能对话系统，集成 **MCP (Model Context Protocol)** 工具调用和 **RAG (Retrieval-Augmented Generation)** 知识库检索，支持自然语言驱动的商品管理、文档问答和互联网搜索。

## 🏗️ 项目架构

```
┌─────────────┐     SSE      ┌──────────────────┐     MCP SSE     ┌──────────────┐
│   Frontend   │◄───────────►│   MCP Client      │◄──────────────►│  MCP Server   │
│  (HTML/Vue)  │             │  (Spring Boot)    │                │ (Spring Boot) │
│  Port: 5501  │             │  Port: 9090       │                │ Port: 9060    │
└─────────────┘              └────────┬──────────┘                └──────┬───────┘
                                      │                                  │
                              ┌───────┴────────┐                ┌───────┴───────┐
                              │  OpenAI API     │                │    MySQL      │
                              │  (LLM 推理)     │                │  (商品数据)   │
                              │                 │                │  Port: 5506   │
                              │  Redis Stack    │                └───────────────┘
                              │  (向量存储)     │
                              │  Port: 9379     │
                              └─────────────────┘
```

## ✨ 核心功能

### 🔧 MCP 工具调用 (Tool Use)
AI 通过 MCP 协议调用后端工具，实现自然语言操作：

| 工具 | 功能 | 示例 |
|------|------|------|
| `ProductTool` | 商品 CRUD | "帮我查价格低于 10000 的商品" |
| `DateTool` | 时区时间查询 | "现在上海几点了？" |
| `EmailTool` | 邮件发送 | "给 xxx 发封邮件" |

### 📚 RAG 知识库
- 支持上传 PDF/TXT/DOCX 文档
- 自动分块 → 向量化 → 存入 Redis Stack
- 对话时自动检索相关文档片段，增强回答质量

### 🌐 互联网搜索
- 集成 SearXNG 自托管搜索引擎
- AI 可实时搜索互联网获取最新信息

## 🛠️ 技术栈

| 组件 | 技术 | 版本 |
|------|------|------|
| 后端框架 | Spring Boot | 3.5.0 |
| AI 框架 | Spring AI | 1.0.0 |
| MCP 协议 | Spring AI MCP (SSE + STDIO) | 1.0.0 |
| LLM | OpenAI 兼容 API | - |
| 向量存储 | Redis Stack | - |
| 嵌入模型 | all-MiniLM-L6-v2 (本地) | - |
| 数据库 | MySQL 9 | 9.x |
| ORM | MyBatis-Plus | 3.5.10 |
| 搜索引擎 | SearXNG (自托管) | - |
| 前端 | HTML + Vue.js + Element UI | - |
| 通信 | SSE (Server-Sent Events) | - |

## 📁 项目结构

```
├── mcp-client/                    # MCP 客户端 (主应用)
│   └── src/main/java/com/itzixi/
│       ├── controller/
│       │   ├── ChatController     # 对话接口
│       │   ├── RagController      # RAG 文档管理
│       │   ├── InternetController # 互联网搜索
│       │   └── SSEController      # SSE 连接管理
│       ├── service/
│       │   ├── ChatServiceImpl    # AI 对话 + MCP 工具编排
│       │   ├── DocumentServiceImpl# 文档分块 + 向量化
│       │   └── SesrXngServiceImpl # SearXNG 搜索集成
│       └── config/
│           └── CorsConfig         # 跨域配置
│
├── mcp-server/                    # MCP 服务端 (工具提供方)
│   └── src/main/java/com/itzixi/
│       ├── mcp/tool/
│       │   ├── ProductTool        # 商品管理工具 (CRUD)
│       │   ├── DateTool           # 日期时间工具
│       │   └── EmailTool          # 邮件工具
│       ├── mapper/
│       │   └── ProductMapper      # MyBatis-Plus 数据映射
│       └── pojo/
│           └── Product            # 商品实体
│
└── frontend/                      # 前端页面
    ├── pages/spring-ai.html       # 主对话界面
    ├── js/                        # API 请求封装
    └── libs/                      # 第三方库
```

## 🚀 快速开始

### 环境要求
- Java 21+
- Maven 3.9+
- Docker & Docker Compose
- Node.js (可选，用于前端开发服务器)

### 1. 启动基础服务

```bash
# MySQL 9
docker run -d --name mysql9 \
  -e MYSQL_ROOT_PASSWORD=root \
  -p 5506:3306 \
  mysql:9

# Redis Stack (向量存储)
docker run -d --name redis-stack \
  -p 9379:6379 \
  redis/redis-stack-server

# SearXNG (搜索引擎)
docker run -d --name searxng \
  -p 8888:8080 \
  searxng/searxng
```

### 2. 初始化数据库

```sql
CREATE DATABASE `springai-items-mcp`;
USE `springai-items-mcp`;

CREATE TABLE product (
  product_id VARCHAR(50) PRIMARY KEY,
  product_name VARCHAR(200),
  brand VARCHAR(100),
  description TEXT,
  price INT,
  stock INT,
  status INT DEFAULT 1,
  create_time DATETIME DEFAULT CURRENT_TIMESTAMP,
  update_time DATETIME DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO product VALUES
('P001', 'MacBook Pro 16', 'Apple', 'M4 Pro芯片,16GB内存,512GB', 19999, 50, 1, NOW(), NOW()),
('P002', 'ThinkPad X1 Carbon', 'Lenovo', '14寸超轻薄商务笔记本', 12999, 30, 1, NOW(), NOW()),
('P003', 'iPhone 16 Pro', 'Apple', 'A18 Pro芯片,256GB', 8999, 100, 1, NOW(), NOW());
```

### 3. 配置环境变量

```bash
export OPENAI_API_KEY=your-api-key
export OPENAI_BASE_URL=https://api.openai.com    # 或 Azure OpenAI 兼容端点
export OPENAI_MODEL=gpt-4o                        # 或其他模型
```

### 4. 构建并启动

```bash
# 构建
mvn clean package -DskipTests

# 启动 MCP Server (端口 9060)
java -jar mcp-server/target/mcp-server-1.0-SNAPSHOT.jar --spring.profiles.active=dev

# 启动 MCP Client (端口 9090)
java -jar mcp-client/target/mcp-client-1.0-SNAPSHOT.jar --spring.profiles.active=dev
```

### 5. 访问前端

用任意静态文件服务器托管 `frontend/` 目录，例如：

```bash
cd frontend && python3 -m http.server 5501
```

访问 `http://localhost:5501/pages/spring-ai.html`

## 💬 使用示例

```
用户: 帮我查一下所有商品
AI:   [调用 queryProductListByCondition] 当前共有3件商品：
      1. MacBook Pro 16 (Apple) - ¥19,999
      2. ThinkPad X1 Carbon (Lenovo) - ¥12,999
      3. iPhone 16 Pro (Apple) - ¥8,999

用户: 把 iPhone 16 Pro 改成 9999 元
AI:   [调用 modifyProduct] 已将 iPhone 16 Pro 的价格更新为 ¥9,999

用户: 现在上海几点了？
AI:   [调用 getCurrentTimeByZoneId] 上海现在是 2026-03-30 22:30:00 (UTC+8)
```

## 📝 部署笔记

### Azure OpenAI 兼容性
如果使用 Azure OpenAI，需要注意：
- Spring AI OpenAI starter 会在 base-url 后拼接 `/v1/chat/completions`
- Azure OpenAI 需要 `?api-version=` 查询参数
- gpt-5.2-chat 等新模型不支持 `temperature` 参数（仅支持默认值 1）
- 需要将 `max_tokens` 替换为 `max_completion_tokens`

建议使用反向代理处理这些兼容性差异。

## 📄 License

MIT
