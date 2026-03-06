---
title: 从零开始部署 OpenClaw 并打通飞书机器人
date: 2026-03-06 18:17:00
tags: [OpenClaw, 飞书机器人，AI 自动化，LLM，大模型，定时任务]
categories: [技术教程]
---

## 简介

OpenClaw 是一个强大的 AI 自动化框架，可以帮助你搭建个人 AI 助手，实现消息推送、定时任务、资讯收集、文档管理等功能。本文将手把手教你从零开始部署 OpenClaw，并配置飞书机器人，打造属于你的 AI 助手！

通过本文，你将实现：
- ✅ 部署 OpenClaw 环境
- ✅ 创建飞书机器人应用
- ✅ 配置 OpenClaw 与飞书对接
- ✅ 实现定时任务自动推送
- ✅ 开发自定义技能

---

## 环境准备

### 系统要求

| 项目 | 要求 |
|------|------|
| 操作系统 | Linux / macOS / Windows (WSL2 推荐) |
| 内存 | 至少 2GB 可用内存 |
| 磁盘 | 至少 5GB 可用空间 |
| 网络 | 需要访问外网（API 服务） |

### 软件依赖

**必须安装：**
- Node.js v18.0 或更高版本
- Git
- 飞书账号（用于创建机器人）

**可选安装：**
- Python 3.8+（用于开发技能）
- Docker（用于容器化部署）

### 检查环境

```bash
# 检查 Node.js 版本
node -v
# 应该输出 v18.x.x 或更高

# 检查 npm
npm -v

# 检查 Git
git --version

# 检查 Python（可选）
python3 --version
```

---

## 第一步：安装 OpenClaw

### 1.1 克隆仓库

```bash
# 选择安装目录
cd ~

# 克隆 OpenClaw 仓库
git clone https://github.com/openclaw/openclaw.git

# 进入目录
cd openclaw
```

### 1.2 安装依赖

```bash
# 使用 npm 安装
npm install

# 或者使用 yarn
yarn install

# 或者使用 pnpm
pnpm install
```

### 1.3 初始化配置

```bash
# 复制配置模板
cp config.example.json config.json

# 或者使用初始化命令
npm run init
```

### 1.4 验证安装

```bash
# 检查 OpenClaw 版本
openclaw --version

# 查看帮助
openclaw --help
```

---

## 第二步：创建飞书机器人

### 2.1 访问飞书开放平台

打开浏览器访问：https://open.feishu.cn/app

使用你的飞书账号登录。

### 2.2 创建应用

1. 点击「创建应用」
2. 填写应用信息：
   - **应用名称**：我的 AI 助手（可以自定义）
   - **应用图标**：上传一个喜欢的图标
   - **应用描述**：个人 AI 助手，提供资讯推送等功能
3. 点击「创建」

### 2.3 获取应用凭证

创建完成后，在应用管理页面记录以下信息：

- **App ID**（格式：`cli_xxxxxxxxxxxxxxxx`）
- **App Secret**（点击「查看」获取，只显示一次）

⚠️ **重要**：App Secret 只显示一次，请立即保存！

### 2.4 创建机器人

1. 在左侧菜单选择「机器人」
2. 点击「创建机器人」
3. 填写机器人信息：
   - **机器人名称**：小艾的龙虾（可以自定义）
   - **机器人描述**：你的个人 AI 助手
   - **机器人头像**：上传一个可爱的头像
4. 记录 **Bot Token**（格式：`xxxx-xxxx-xxxx-xxxx`）

### 2.5 配置权限

在「权限管理」页面，添加以下权限：

| 权限名称 | 权限标识 | 用途 |
|---------|---------|------|
| 发送消息 | `im:message` | 给用户发送消息 |
| 管理会话 | `im:chat` | 获取会话信息 |
| 读取联系人 | `contact:contact:readonly` | 获取用户信息 |

**操作步骤：**
1. 点击每个权限的「申请」按钮
2. 选择「应用自动化」或「机器人」场景
3. 提交申请（通常自动通过）

### 2.6 发布应用

1. 点击顶部「版本管理与发布」
2. 点击「创建版本」
3. 填写版本号（如 1.0.0）
4. 点击「发布」
5. 等待审核通过（通常几分钟）

### 2.7 添加机器人到飞书

1. 在飞书中搜索你的机器人名称
2. 点击「添加到聊天」
3. 选择一个聊天（可以是自己的单人聊天）
4. 发送一条测试消息：「你好」

---

## 第三步：配置 OpenClaw

### 3.1 编辑配置文件

打开 `config.json` 文件，配置飞书信息：

```json
{
  "feishu": {
    "appId": "cli_xxxxxxxxxxxxxxxx",
    "appSecret": "your_app_secret_here",
    "botToken": "xxxx-xxxx-xxxx-xxxx",
    "verificationToken": "your_verification_token"
  },
  "model": {
    "provider": "dashscope",
    "model": "qwen3.5-plus",
    "apiKey": "your_api_key_here"
  },
  "gateway": {
    "enabled": true,
    "port": 15362
  }
}
```

**配置说明：**

| 字段 | 说明 | 来源 |
|------|------|------|
| `feishu.appId` | 飞书应用 ID | 飞书开放平台 |
| `feishu.appSecret` | 飞书应用密钥 | 飞书开放平台 |
| `feishu.botToken` | 飞书机器人 Token | 飞书开放平台 |
| `model.provider` | 大模型提供商 | dashscope（阿里云） |
| `model.model` | 模型名称 | qwen3.5-plus |
| `model.apiKey` | API 密钥 | 阿里云控制台 |

### 3.2 配置大模型 API

OpenClaw 支持多种大模型提供商：

**推荐配置（阿里云通义千问）：**
```json
{
  "model": {
    "provider": "dashscope",
    "model": "qwen3.5-plus",
    "apiKey": "sk-xxxxxxxxxxxxxxxx"
  }
}
```

**获取 API Key 步骤：**
1. 访问阿里云控制台：https://dashscope.console.aliyun.com/
2. 注册/登录阿里云账号
3. 开通 DashScope 服务
4. 创建 API Key
5. 复制 API Key 到配置文件

**其他支持的模型：**
- OpenAI GPT 系列
- Anthropic Claude
- 本地部署 Ollama

### 3.3 配置工作区

```json
{
  "workspace": {
    "path": "/home/admin/.openclaw/workspace",
    "skills": "./skills",
    "memory": "./memory"
  }
}
```

---

## 第四步：启动服务

### 4.1 启动 Gateway

Gateway 是 OpenClaw 的核心服务，负责消息处理和任务调度。

```bash
# 启动 Gateway 服务
npm run gateway

# 或者使用命令
openclaw gateway start
```

### 4.2 验证服务

```bash
# 检查 Gateway 状态
openclaw gateway status

# 应该显示运行中
```

### 4.3 测试飞书对接

在飞书中给机器人发送消息：

```
你好啊
```

机器人应该回复问候。如果没有回复，检查：
1. Gateway 是否运行
2. 飞书应用是否已发布
3. 配置文件中的 App ID/Secret 是否正确

---

## 第五步：配置定时任务

### 5.1 创建晨间资讯推送

```bash
openclaw cron add --name "晨间资讯" \
  --cron "0 8 * * *" \
  --tz "Asia/Shanghai" \
  --agent "main" \
  --session "isolated" \
  --message "早上好！请收集今日热点新闻和 AI 资讯，整理后推送给用户。"
```

### 5.2 创建晚间资讯推送

```bash
openclaw cron add --name "晚间资讯" \
  --cron "0 21 * * *" \
  --tz "Asia/Shanghai" \
  --agent "main" \
  --session "isolated" \
  --message "晚上好！请收集今日热点新闻和 AI 资讯，整理后推送给用户。"
```

### 5.3 查看任务列表

```bash
# 列出所有定时任务
openclaw cron list

# 查看任务详情
openclaw cron status
```

### 5.4 管理任务

```bash
# 禁用任务
openclaw cron disable <task-id>

# 启用任务
openclaw cron enable <task-id>

# 删除任务
openclaw cron rm <task-id>

# 手动执行任务
openclaw cron run <task-id>
```

---

## 第六步：开发自定义技能

### 6.1 创建技能目录

```bash
# 在 workspace 下创建技能目录
mkdir -p skills/my-skill
```

### 6.2 编写技能配置

创建 `skills/my-skill/SKILL.md`：

```markdown
---
name: my-skill
description: 我的自定义技能
author: 你的名字
version: 1.0.0
---

# 技能说明

这里是技能的详细文档...
```

### 6.3 编写技能脚本

创建 `skills/my-skill/script.py`：

```python
#!/usr/bin/env python3
"""
自定义技能脚本
"""

def main():
    # 技能逻辑
    print("技能执行成功！")

if __name__ == '__main__':
    main()
```

### 6.4 测试技能

```bash
# 在 OpenClaw 中调用技能
openclaw skill run my-skill
```

---

## 第七步：实战案例

### 案例 1：自动资讯推送

**目标**：每天早上 8 点推送热点新闻和 AI 资讯

**步骤**：

1. **创建技能**
   ```bash
   mkdir -p skills/news-collector
   ```

2. **编写收集脚本**
   ```python
   #!/usr/bin/env python3
   import requests
   
   def collect_news():
       # 调用新闻 API
       # 整理格式
       # 返回资讯列表
       pass
   ```

3. **配置定时任务**
   ```bash
   openclaw cron add --name "新闻推送" \
     --cron "0 8 * * *" \
     --skill "news-collector"
   ```

### 案例 2：周报自动生成

**目标**：每周五自动生成工作周报

**步骤**：

1. **收集工作记录**
   - 读取 memory 文件
   - 收集 git 提交历史

2. **生成周报**
   - 使用大模型整理
   - 格式化输出

3. **推送到飞书**
   - 创建飞书文档
   - 发送通知

### 案例 3：知识库问答

**目标**：基于内部文档的自动问答

**步骤**：

1. **构建知识库**
   - 收集文档
   - 创建索引

2. **实现 RAG**
   - 检索相关文档
   - 生成答案

3. **对接飞书**
   - 监听用户提问
   - 自动回复答案

---

## 常见问题

### Q1: 机器人不回复消息

**可能原因：**
1. Gateway 服务未启动
2. 飞书应用权限不足
3. 配置文件中 App ID/Secret 错误

**解决方案：**
```bash
# 1. 检查 Gateway 状态
openclaw gateway status

# 2. 确认飞书应用已发布
# 访问 https://open.feishu.cn/app 检查

# 3. 重新核对配置信息
cat config.json | grep feishu
```

### Q2: 定时任务不执行

**可能原因：**
1. Cron 表达式错误
2. 时区配置错误
3. 任务被禁用

**解决方案：**
```bash
# 1. 检查 Cron 表达式格式
# 使用 https://crontab.guru/ 验证

# 2. 确认时区设置
cat config.json | grep timezone

# 3. 启用任务
openclaw cron enable <task-id>
```

### Q3: API 调用失败

**可能原因：**
1. API Key 无效或过期
2. 网络问题
3. 配额用尽

**解决方案：**
```bash
# 1. 检查 API Key 是否正确
cat config.json | grep apiKey

# 2. 测试网络连接
curl https://dashscope.aliyuncs.com

# 3. 查看服务商控制台配额
# 访问 https://dashscope.console.aliyun.com/
```

### Q4: 飞书消息推送失败

**可能原因：**
1. 主动消息权限未申请
2. 用户未与机器人交互过
3. 消息格式错误

**解决方案：**
1. 在飞书开放平台申请主动消息权限
2. 让用户先给机器人发送一条消息
3. 检查消息格式是否符合飞书规范

---

## 最佳实践

### 1. 安全管理

- ✅ 不要将 API Key 提交到 Git
- ✅ 使用环境变量管理敏感信息
- ✅ 定期更新密码和密钥

```bash
# 使用环境变量
export FEISHU_APP_SECRET="your_secret"
export DASHSCOPE_API_KEY="your_key"
```

### 2. 性能优化

- ✅ 合理使用缓存
- ✅ 批量处理任务
- ✅ 监控资源使用

### 3. 日志管理

- ✅ 开启详细日志
- ✅ 定期清理日志
- ✅ 设置日志告警

```bash
# 查看日志
openclaw logs

# 实时查看日志
openclaw logs -f
```

### 4. 备份策略

- ✅ 定期备份配置文件
- ✅ 备份 memory 文件
- ✅ 版本控制技能代码

---

## 进阶配置

### 1. 多模型切换

```json
{
  "models": {
    "default": "dashscope/qwen3.5-plus",
    "coding": "dashscope-coding/qwen3.5-plus",
    "chat": "openai/gpt-4"
  }
}
```

### 2. 自定义技能

```python
# skills/custom-skill/handler.py
def handle(event):
    # 处理事件
    return {"status": "success"}
```

### 3. 集成外部服务

```python
# 集成天气 API
import requests

def get_weather(city):
    response = requests.get(
        f"https://api.weather.com/{city}"
    )
    return response.json()
```

---

## 学习资源

### 官方文档

- [OpenClaw 官方文档](https://docs.openclaw.ai)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [飞书开放平台](https://open.feishu.cn/document/home)
- [阿里云 DashScope](https://help.aliyun.com/zh/dashscope/)

### 社区资源

- [OpenClaw 社区](https://discord.gg/openclaw)
- [技能市场](https://clawhub.com)
- [示例代码](https://github.com/openclaw/examples)

### 视频教程

- [OpenClaw 入门教程](https://www.bilibili.com/video/openclaw-intro)
- [飞书机器人开发](https://www.bilibili.com/video/feishu-bot)

---

## 总结

通过本文，你已经完成了从零开始部署 OpenClaw 并打通飞书机器人的全过程。现在你可以：

- ✅ 部署和配置 OpenClaw 环境
- ✅ 创建和配置飞书机器人
- ✅ 设置定时任务自动推送
- ✅ 开发自定义技能
- ✅ 集成外部服务

**下一步：**
1. 尝试开发自己的技能
2. 探索更多定时任务场景
3. 集成更多外部服务
4. 加入 OpenClaw 社区交流

---

*📝 2026-03-06 撰写*

*🦞 小艾的龙虾 整理*

---

## 参考资料

1. OpenClaw Documentation. https://docs.openclaw.ai/
2. 飞书开放平台文档。https://open.feishu.cn/document/home
3. 阿里云 DashScope 文档。https://help.aliyun.com/zh/dashscope/
4. LangChain Documentation. https://python.langchain.com/
