# -Agent
我搭建了一个企业内部知识库 Agent，主要解决新人培训、制度查询和工单重复答复成本高的问题。系统会先通过 RAG 检索公司文档、历史工单和 FAQ，再由推理 Agent 判断用户问题属于制度咨询、技术问题还是流程办理。对于简单问题，Answer Agent 会直接给出带出处的回答；对于复杂问题，Workflow Agent 会自动拆解处理步骤，例如查询权限、生成申请模板、提醒负责人审批；如果置信度不足，则转交人工并生成问题摘要。目前试点覆盖了常见 HR、IT 和业务流程问题，能自动处理大约 60%-70% 的重复咨询，人工客服只需要处理例外情况，响应时间从小时级缩短到分钟级。
[README.md](https://github.com/user-attachments/files/27224431/README.md)
# 企业知识库问答 + 工单处理 Agent 系统

这是一个可直接运行的企业内部支持系统 Demo，包含知识库检索、问答生成、工单创建、工单状态流转和多 Agent 执行链路展示。项目只使用 Python 标准库和原生前端，不需要安装第三方依赖。

## 功能

- 知识库问答：从 `data/knowledge_base.json` 检索企业制度、IT、财务、HR、合规等知识。
- 工单处理：根据用户描述自动识别分类、优先级、负责人和 SLA，并写入 SQLite。
- 多 Agent 编排：Planner、Retriever、Answer、Workflow/Ticket Agent 分步执行。
- 前端控制台：支持对话、查看来源、查看 Agent 链路、查看和更新工单、新增知识。
- API 服务：提供健康检查、知识库、工单、对话处理等接口。

## 运行

```powershell
python app.py
```

打开：

```text
http://127.0.0.1:8000
```

指定端口：

```powershell
python app.py --port 8080
```

## 测试

```powershell
python -m unittest discover -s tests
```

## 主要接口

### 对话处理

```http
POST /api/chat
Content-Type: application/json
```

```json
{
  "message": "VPN 连不上，提示账号无权限，请帮我提交工单",
  "user": "张三",
  "department": "产品运营部"
}
```

返回内容包含：

- `answer`：知识库问答结果
- `sources`：命中的知识来源
- `trace`：Agent 执行链路
- `ticket`：自动创建的工单，若无需工单则为 `null`

### 工单列表

```http
GET /api/tickets
GET /api/tickets?status=open
```

### 更新工单

```http
PATCH /api/tickets/{ticket_id}
Content-Type: application/json
```

```json
{
  "status": "in_progress"
}
```

### 新增知识

```http
POST /api/documents
Content-Type: application/json
```

```json
{
  "title": "会议室设备故障处理",
  "category": "行政设备",
  "keywords": ["会议室", "投屏", "设备"],
  "content": "会议室投屏失败时，先检查网络、HDMI 线缆和投屏器电源。影响多人会议时按 P1 处理。"
}
```

## 项目结构

```text
.
├── app.py                    # 后端、检索、Agent 编排、API 服务
├── data/
│   └── knowledge_base.json   # 种子知识库
├── static/
│   ├── index.html            # 前端页面
│   ├── styles.css            # 控制台样式
│   └── app.js                # 前端交互
└── tests/
    └── test_agents.py        # 核心流程测试
```

## 可扩展方向

- 接入真实大模型：把 `EnterpriseAgentSystem.build_answer` 替换为 LLM 调用，并保留当前检索结果作为上下文。
- 接入企业系统：将 `TicketStore` 换成 Jira、飞书工单、ServiceNow 或企业微信审批。
- 引入向量检索：用 embedding + 向量数据库替换当前轻量词法检索。
- 加权限控制：为知识条目和工单增加用户角色、部门隔离和审计日志。
