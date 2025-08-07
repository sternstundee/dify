以下是一份完整的 **内网部署 Dify 的技术文档 / README**，可直接用于归档或团队使用说明。

---

## 📘 内网部署 Dify 平台技术文档

> 本文档说明如何在无公网访问的内网环境下，通过配置代理与公共 DNS，成功部署 Dify 开源大模型应用平台。

---

### 📦 一、项目简介

[Dify](https://github.com/langgenius/dify) 是一个支持多模型（如 OpenAI、Qwen、Baichuan、Gemini、本地部署模型等）的开源 LLM 应用开发平台，支持知识库、插件调用、Agent 流程、多轮对话等功能。

---

### 🧩 二、部署目标

* 内网部署（无公网 IP）
* 可访问公网模型 API（通过代理）
* 正确解析外部 DNS（如 `api.openai.com`）

---

### 🛠 三、环境准备

#### ✅ 系统依赖

| 项目             | 说明                      |
| -------------- | ----------------------- |
| OS             | Linux / CentOS / Ubuntu |
| Docker         | ≥ 20.10                 |
| Docker Compose | ≥ 2.0                   |
| Git            | 拉取项目源码                  |

---

### 🔐 四、代理配置（CNTLM）

若内网只能通过 HTTP 代理访问公网（如 CNTLM 代理），需设置如下：

#### 1. CNTLM 本地代理安装

参考 CNTLM 官方文档或系统包安装：

```bash
# Debian / Ubuntu
sudo apt install cntlm

# CentOS / RHEL
sudo yum install cntlm
```

#### 2. 验证 CNTLM 工作正常

```bash
curl -x http://localhost:3128 http://www.google.com
```

#### 3. 获取 `cntlm` 代理地址

例如：

```bash
http://172.30.1.10:3128
```

---

### 📁 五、项目拉取与目录结构

```bash
git clone https://github.com/langgenius/dify.git
cd dify
```

关键文件结构如下：

```text
dify/
├── docker-compose.yaml
├── .env                  # 主配置文件（需自定义）
├── .env.middleware       # 中间件配置文件（需自定义）
├── frontend/             # 前端应用
├── backend/              # 后端服务
```

---

### ⚙️ 六、环境变量配置（重点）

#### 1. 修改 `.env` 文件

```env
# === HTTP 代理配置 ===
HTTP_PROXY=http://172.30.1.10:3128
HTTPS_PROXY=http://172.30.1.10:3128
NO_PROXY=localhost,127.0.0.1,dify-db,redis,dify-api,dify-web,dify-worker

# === DNS 配置（部分容器无法解析内网 DNS 时）===
DNS=8.8.8.8,114.114.114.114,10.54.12.44,10.187.50.203

# === 模型 API 配置（示例：OpenAI）===
OPENAI_API_KEY=sk-xxxx
OPENAI_API_BASE=https://api.openai.com/v1
```

> ⚠️ 注意 `NO_PROXY` 一定要包含内部服务的容器名，否则会因为代理反而访问不了服务本体。

---

#### 2. 修改 `.env.middleware` 文件（或 middleware.env）

```env
HTTP_PROXY=http://172.30.1.10:3128
HTTPS_PROXY=http://172.30.1.10:3128
NO_PROXY=localhost,127.0.0.1,mysql,redis,mongo,rabbitmq
```

用于 Redis、Mongo、MySQL 等中间件容器。

---

#### 3. 修改 `docker-compose.yaml` 增加 DNS 配置

```yaml
services:
  dify-api:
    ...
    dns:
      - 8.8.8.8
      - 114.114.114.114
```

你可以将 `dns:` 配置加到所有服务中，或在 `docker-compose.yml` 顶部全局指定：

```yaml
x-common-dns: &common-dns
  dns:
    - 8.8.8.8
    - 114.114.114.114
    - 10.54.12.44

services:
  dify-api:
    <<: *common-dns
    ...
```

---

### 🚀 七、启动服务

```bash
docker-compose up -d
```

---

### ✅ 八、访问服务

* 根据你部署时暴露的端口访问/install

---

### 🧪 九、常见问题排查

| 问题                  | 解决方案                             |
| ------------------- | -------------------------------- |
| `DNS resolve error` | 添加 DNS 配置，优先解析 8.8.8.8           |
| `API 404`           | 模型 URL 设置错误，检查 `OPENAI_API_BASE` |
| 无法访问公网              | 检查 CNTLM 是否配置成功                  |
| 模型验证失败              | 代理可能无法访问外部模型服务                   |

---

### 🧠 十、建议

* 推荐使用 CNTLM + docker-compose 全面代理流量
* 插件调用需确保目标 API 可在内网中访问
* 可部署私有模型（如 Qwen）以避免访问外网
* 建议将 `.env`、`middleware.env` 加入 `.gitignore`，避免泄露代理信息

---




