# MARS-408 产品需求文档（PRD）

> 基于改进 GOMARL 与 FrugalRAG 的计算机 408 考研个性化学习多智能体系统
> 版本：v2.0 | 2026-07-02
> 团队：张敏杰(负责人)、陈健伟、彭凯扬、张文杰、张勋 — 闽江大学网络空间安全学院
> 更新说明：基于代码审计与数据分析报告更新实际数据量、API一致性风险、Python兼容性等

---

## 〇、项目当前状态摘要（基于 2026-07-02 代码审计）

| 维度 | 状态 | 说明 |
|------|------|------|
| 代码规模 | 13,474 行源码（后端 5,960 / 前端 7,514），56+ 文件 | 架构清晰，模块划分合理 |
| 代码完成度 | 75% | 核心功能已实现，前后端联调未完成 |
| 前后端集成 | 40% | 独立可运行但联调未完成，4/22 API 接口不一致 |
| 测试覆盖率 | 5% | 仅 2 个非系统性测试文件（119 行） |
| 知识库建设 | 228 条种子数据（计网 155 chunks + 73 题目） | 实际向量库存储 128 chunks + 73 questions = 201 文档 |
| 综合健康评分 | 53/100 | 需重点关注联调与测试 |
| 关键路径 | Python 修复(1d) → 联调(8d) → Bug修复(6d) → (5d) → 打包(2d) | 总长约 15-18 自然日 |

**API 一致性问题（4 项需修复，1.6 人天）**：
1. `GET /api/sessions/load` — 前端未传 conv_id（studyStore.ts:350 vs sessions.py:88）
2. `POST /api/sessions/save` — 前端传数组，后端期望单个对象（studyStore.ts:371 vs sessions.py:40）
3. `DELETE /api/sessions/delete` — 前端 query param vs 后端 path param（studyStore.ts:401 vs sessions.py:107）
4. `GET /api/knowledge-graph` — 前端调用但后端无此端点（studyStore.ts:681）

---

## 一、产品目标

MARS-408 是一个面向计算机 408 考研学生的个性化学习多智能体系统。项目同时服务于两个目标场景：软件杯 A3 赛题提交（短期内需要可演示 MVP）和大创创新训练项目（中期/结题需更完整的技术实现）。两个版本不是两套系统，而是一套系统的两个版本——Lite 版（软件杯提交）和完整版（大创结题）。

### 目标一：软件杯可演示 MVP —— 跑通完整学习闭环

**截止时间**：约20天后软件杯提交
**交付形态**：可演示的 Web 应用，覆盖画像构建→资源生成→路径规划→练习→评估的完整闭环

- 对话式学生画像 ≥6 维度，随学随新
- LangGraph 7-Agent StateGraph 多智能体协同，生成 ≥5 种学习资源
- 个性化学习路径 DAG + 资源联动推送
- SSE 流式对话，逐节点展示 Agent 工作状态
- 学习效果评估仪表盘（7章热力图 + 趋势 + LLM 建议）
- 防幻觉审阅（Critic + GOMARL 共识校验）
- 讯飞星火 X2 三通道合规接入
- 计网 1 门完整课程知识库（≥515 chunks）

### 目标二：大创中期检查版本 —— 技术栈升级与算法深化

**截止时间**：2026.11 中期检查
**交付形态**：代码追上申报书承诺，FrugalRAG 真版 + GOMARL 真版上线

- 技术栈升级：Milvus 生产部署、PostgreSQL + Redis 全量接入、LangGraph 完整版
- FrugalRAG 真版：SFT 检索策略训练 + GRPO 停止决策模块
- GOMARL 真版：知识一致性校验层（FrugalRAG 证据验证）+ 冲突消解层 + 动态权重层
- 知识库扩展至 408 四科之一（数据结构 + 计网，≥1000 chunks）
- 智能辅导功能增强（MCP 工具 + 代码沙箱）

### 目标三：大创结题完整版 —— 全量 408 覆盖 + 试点验证

**截止时间**：2027.5 结题
**交付形态**：完整的多智能体学习系统 v2.0，经真实用户验证

- 408 四科全量知识库（数据结构 + 计组 + 操作系统 + 计网，≥1800 chunks）
- 知识图谱（≥1000 节点，≥3000 边）+ 强化学习路径规划
- 教师端管理后台 + 数据看板
- ≥50 人试点应用验证（使用周期 ≥4 周）
- 软著申请 + 结题报告

---

## 二、用户故事

### 场景一：初次使用 —— 对话式画像构建

**角色**：小明，大三学生，计划参加 2027 年计算机考研，408 四科中计网最薄弱。

**故事**：
小明首次打开 MARS-408，系统引导他进入对话式画像构建页面。AI 以自然语言提问："你好！我是你的 AI 学习助手。请问你目前的备考目标是什么？408 四科中你最担心哪一科？"

小明回答："我想考本校的电子信息专硕，计网和数据结构基础不太好，特别是 TCP 协议和树的算法。"

系统在对话中自动提取画像信息，逐步构建 8 维度画像：（1）知识掌握度→计网/数据结构标记为"初级"；（2）学习风格→"视觉型+动手型"；（3）薄弱知识点→"TCP协议/树的遍历"；（4）学习进度→"0%"；（5）每日学习时间→"3-4小时"；（6）备考目标→"电子信息专硕"。

对话结束时，系统展示雷达图可视化画像，并自动推荐计网入门学习路径。

**验收标准**：
- 首次对话 ≥4 轮即可完成画像构建
- 画像 ≥6 维度
- 雷达图可视化展示
- 自动生成初始学习路径推荐

### 场景二：知识点学习 —— 多智能体协同资源生成

**角色**：小明，已在系统中学习 1 周，画像显示计网第 4 章（网络层）薄弱。

**故事**：
小明在聊天框中输入："帮我讲解 IP 分组转发和路由选择"。系统 LangGraph StateGraph 启动：

1. **Coordinator** 分析意图："用户请求讲解 IP 网络层核心概念，匹配计网第 4 章，难度中级"
2. **Diagnostician** 诊断："小明在计网第 4 章薄弱，需要重建路由选择的理解基础"
3. **Planner** 规划："先讲 IP 数据报格式 → 分组转发算法 → 路由选择协议，搭配 3 道练习题，生成导图"
4. **Retriever** 检索：FrugalRAG 从计网知识库中检索"IP 分组转发"相关知识块
5. **Generator Cluster** 生成：
   - Teacher 生成 IP 分组转发讲解文档（Markdown + LaTeX 公式）
   - QuizMaster 生成 3 道练习题（1 选择 + 1 填空 + 1 简答）
   - MediaAgent 生成思维导图大纲（markmap 格式）
   - Extension 推荐拓展阅读（CIDR/路由聚合相关）
6. **Critic** 审阅 + GOMARL 共识校验：验证 IP 子网计算正确性，检测矛盾

小明在前端看到 SSE 流式进度（6 个 Agent 节点依次点亮），最终收到完整的学习资源包。

**验收标准**：
- 从输入知识点到完整资源返回，全流程 ≤30 秒
- 生成 ≥4 种资源（讲解/题目/导图/拓展）
- SSE 流式推送 ≥4 个 Agent 节点状态
- GOMARL 共识通过率 ≥90%（首次通过或至多重生成 1 轮）

### 场景三：练习与评估 —— 答题闭环 + 画像动态更新

**角色**：小明，完成了计网第 4 章的学习，准备做练习检验。

**故事**：
小明进入练习页面，系统根据画像（薄弱点为路由选择）自动推送 5 道针对性练习题。小明完成答题后提交，系统即时判分：5 题对 3 题（路由选择部分 2 题全错）。

系统自动触发画像更新流程：
- 计网第 4 章掌握度从 "beginner" 上调为 "intermediate"（因 IP 分组转发做对）
- 薄弱点更新："路由选择协议"优先级提升
- 答题正确率趋势更新（加入当前 60%）
- 学习进度更新（第 4 章 +8%）

同时系统自动推荐"重新学习路由选择协议"的学习路径调整建议，并在评估仪表盘中更新 7 章热力图。

**验收标准**：
- 答题提交后画像 3 秒内更新
- 至少 3 个画像维度随答题结果变化
- 评估仪表盘实时更新热力图
- 自动推送薄弱点针对性资源

### 场景四：考前冲刺 —— 学习路径规划 + 薄弱点突破

**角色**：小明，距考研还有 2 个月，需在有限时间内高效补强所有薄弱科目。

**故事**：
小明打开学习路径页面查看整体进度。系统基于 7 章知识点 DAG + 当前画像，自动生成个性化学习路径：
- 已掌握章节（绿色）：计网 1-3 章，只需定期复习
- 进行中（黄色）：计网第 4 章，当前重点学习
- 未开始（灰色）：计网 5-7 章、数据结构全部

系统分析薄弱点分布后，推荐优先学习顺序：计网第 4 章 → 计网第 5 章 → 数据结构第 3 章（树），并为每个节点推荐 2-3 个学习资源。

小明点击计网第 5 章节点，系统联动推送该章节的核心概念讲解 + 练习题。

**验收标准**：
- 7 章 DAG 可视化展示（绿/黄/灰三色区分）
- 每章节点点击可联动推荐 ≥2 个资源
- 路径排序基于画像薄弱点 + 知识依赖关系
- 进度百分比实时显示

### 场景五：效果评估 —— 全面了解学习状况

**角色**：小明，使用系统 2 周后，想了解自己的整体学习效果。

**故事**：
小明打开学习效果评估仪表盘，看到：
- **7 章热力图**：计网各章掌握度从浅黄（低）到深绿（高）变化
- **活跃度趋势**：过去 14 天每日学习时长折线图，周末明显下降
- **答题正确率趋势**：从 40% 逐步提升到 65%
- **薄弱知识点排行**：Top 3 薄弱点——路由选择协议、二叉平衡树、IP 子网划分
- **易错题回顾**：近 7 天错误率最高的 5 道题，附正确答案
- **LLM 学习建议**：AI 生成针对性的学习策略建议——"建议你每天投入 1 小时专门攻克路由选择协议，先理解 RIP 再学 OSPF"

小明将评估页截图分享给导师讨论。

**验收标准**：
- 仪表盘包含 ≥5 种可视化图表（热力图/折线图/排行/易错回顾/建议）
- 数据来源 ≥3 种（答题记录/学习时长/画像变化）
- LLM 建议基于实际数据生成
- 页面加载时间 ≤2 秒

---

## 三、功能需求池

### 3.1 P0 优先级 —— 软件杯提交前必须完成（约 20 天）

| ID | 功能 | 说明 | 验收标准 |
|----|------|------|----------|
| P0-1 | 对话式画像构建 | 自然语言对话 ≥4 轮构建学生画像，≥6 维度，雷达图可视化 | ① 对话前端可用，LLM 理解意图；② 画像 JSON 含 ≥6 个维度字段；③ 雷达图正确渲染；④ 画像存入 localStorage/后端 |
| P0-2 | 多智能体协同资源生成 | LangGraph 7-Agent StateGraph 协同工作，生成 ≥5 种学习资源 | ① StateGraph 6 节点 + 2 条件边正常运行；② 返回 teacher_doc/quiz/media_plan/extension_doc/critic_report；③ SSE 流式推送 ≥4 个节点状态 |
| P0-3 | FrugalRAG Lite 检索引擎 | E5 向量检索 + 余弦阈值过滤 + BM25 关键词融合排序 | ① E5-base-v2 embedding 正常加载；② ChromaDB/Milvus 检索结果正确；③ 检索响应时间 ≤3 秒；④ 阈值 0.65 噪声过滤生效 |
| P0-4 | GOMARL Lite 共识引擎 | 加权投票 + 知识一致性校验 + 最多 2 轮重生成 | ① 4 类矛盾对检测正常；② 质量分 ≥7 通过、<7 重生成的逻辑正确；③ 历史权重正确累计 |
| P0-5 | SSE 流式对话 | 主聊天界面 SSE 流式消费，逐节点推送 Agent 进度 | ① `/api/agents/langgraph/stream` 端点正常推送；② 前端 ChatView 正确渲染流式内容；③ 完整对话链路无中断 |
| P0-6 | 学习路径规划 + 资源联动 | 7 章知识点 DAG + 画像薄弱点 → LLM 动态排序 + 资源推荐 | ① DAG 可视化正常（绿/黄/灰）；② 每章点击联动推荐资源；③ 路径排序覆盖全部章节 |
| P0-7 | 在线练习 + 答题闭环 | 基于知识点生成练习题，答题后判分 + 画像更新 | ① 题目生成 ≥3 道；② 自动判分 + 正确答案展示；③ 画像 ≥2 维度随答题更新 |
| P0-8 | 学习效果评估仪表盘 | 7 章热力图 + 活跃度趋势 + 正确率 + 易错点 + LLM 建议 | ① 仪表盘 ≥5 种可视化组件；② 数据来自真实答题记录；③ LLM 建议可生成 |
| P0-9 | 内容安全 + 防幻觉 | Critic 事实核查 + GOMARL 共识校验 + 敏感词过滤 + 幻觉检测 | ① 13 类敏感词可过滤；② 4 类常见知识错误可检测；③ 矛盾输出可触发重生成 |
| P0-10 | 三通道 LLM 容灾路由 | DeepSeek/讯飞星火 X2/Qwen2.5 自动切换 | ① 三通道均能正常调用；② 任一通道失败自动回退下一通道；③ 讯飞为第二优先级（合规） |
| P0-11 | 知识库（计网 1 门） | 计网 7 章 ≥228 条种子数据完整可用（实际向量库存储 201 文档） | ① 向量检索可检出相关知识块；② 种子数据（48 知识点 + 35 题）导入完整；③ 7 章全覆盖 |
| P0-12 | 前端 12 视图联调 | 画像/聊天/资源/路径/练习/评估/沙箱/仪表盘等全部联调通过 | ① 12 个视图均可正常访问；② 与后端 API 完整对接；③ 无 JS 报错/白屏 |
| P0-13 | **前后端 API 一致性修复** | 修复 4 个接口不一致问题（sessions load/save/delete + knowledge-graph） | ① sessions.load 正确传 conv_id；② sessions.save payload 格式统一；③ sessions.delete 参数方式统一为 path param；④ knowledge-graph 端点补充或前端调用调整 |
| P0-14 | **Python 版本兼容性修复** | pyproject.toml Python 版本 ≥3.14 改为 ≥3.12，兼容 3.13 运行环境 | ① 项目在 Python 3.12/3.13 环境下可正常安装运行；② uv sync 无报错 |
| P0-15 | 开源声明 | 代码中标注所有开源项目 + AI 工具（≥16 项） | ① 技术方案文档 §0 完整列表；② 前端/后端代码注释标注关键依赖 |

### 3.2 P1 优先级 —— 大创中期检查前（2026.7-2026.11）

| ID | 功能 | 说明 | 验收标准 |
|----|------|------|----------|
| P1-1 | Milvus 生产部署 | 从 ChromaDB 迁移到 Milvus 2.3，支持 HNSW 索引 | ① Milvus 集群正常启动；② 数据完整迁移无丢失；③ 检索性能 ≥ChromaDB |
| P1-2 | PostgreSQL + Redis 全量接入 | 画像/答题/Agent 表现持久化 + LLM 缓存 | ① 4 表 schema 正确建表；② 读写正常；③ Redis 缓存命中率 ≥60% |
| P1-3 | FrugalRAG 真版 · SFT 训练 | 500 条标注样本监督微调，学习最优检索查询策略 | ① SFT pipeline 可运行；② 训练后准确率提升 ≥5%；③ 训练时间 ≤2 小时 |
| P1-4 | FrugalRAG 真版 · GRPO 停止决策 | 强化学习动态决策检索停止时机 | ① GRPO 模块可运行；② 检索成本降低 ≥15%；③ 准确率不下降 |
| P1-5 | GOMARL 真版 · 知识校验层 | 调用 FrugalRAG 检索证据验证 Agent 间冲突 | ① 校验层接入 FrugalRAG；② 可检测 ≥4 类知识冲突；③ 校验结果可追溯 |
| P1-6 | GOMARL 真版 · 冲突消解层 | 基于证据的冲突消解算法，修正矛盾输出 | ① 消解算法可运行；② 消解后准确率提升 ≥10%；③ 消解逻辑可解释 |
| P1-7 | GOMARL 真版 · 动态权重层 | 结合学生画像的 EWMA 动态权重调整 | ① 权重随 Agent 表现实时调整；② 高质量 Agent 权重高于低质量；③ 权重存储在 Redis |
| P1-8 | 知识库扩展（数据结构） | 严蔚敏教材 + 王道 25 版 + 真题做题本全量导入 | ① 数据结构知识库 ≥500 chunks；② 含代码实操案例；③ 计网+DS 合计 ≥1000 chunks |
| P1-9 | 智能辅导增强 | MCP 工具（TCP 握手模拟/子网计算）+ JavaScript 代码沙箱 | ① MCP 工具可被 Agent 调用；② 沙箱安全隔离执行；③ 最终工具 ≥3 种 |
| P1-10 | 教师端管理后台 | 学生数据看板 + 知识库管理 + 题目管理 | ① 学生列表 + 画像浏览；② 整体学习统计分析；③ 知识库内容增删改查 |
| P1-11 | 知识图谱 + RL 路径规划 | 知识图谱先修关系 + 强化学习动态路径排序 | ① 知识图谱 ≥200 节点 + 500 边；② RL 排序优于 LLM 排序（准确率/效率） |

### 3.3 P2 优先级 —— 大创结题前（2026.11-2027.5）

| ID | 功能 | 说明 | 验收标准 |
|----|------|------|----------|
| P2-1 | 408 四科全量知识库 | 计组 + 操作系统知识库完成，四科合计 ≥1800 chunks | ① 计组 ≥300 chunks；② OS ≥300 chunks；③ 四科合计 ≥1800 chunks |
| P2-2 | 知识图谱全量 | 408 四科全覆盖，≥1000 节点 + 3000 边 | ① 四科先修关系完整；② 知识图谱可视化；③ 路径规划基于全量图 |
| P2-3 | 试点应用 | 闽江大学 ≥50 人使用 ≥4 周 | ① 用户注册/登录系统；② 使用数据可追溯；③ 满意度 ≥80% |
| P2-4 | Docker 一键部署 | 生产环境 Docker Compose 一键部署 | ① `docker compose up -d` 可完整启动；② 包含 Milvus+PG+Redis+前后端 |
| P2-5 | 并发支撑 | 支持 50 人同时在线 | ① 50 并发压测通过；② 响应时间 ≤5 秒；③ 无资源泄漏 |
| P2-6 | 调研报告 | 《面向计算机考研的个性化学习多智能体系统技术实现与应用研究》1.5 万字 | ① 格式规范；② 查重率 ≤30%；③ 包含试点数据 |
| P2-7 | 软著申请 | 《MARS-408 V1.0》软件著作权 | ① 软著受理通知书 |

---

## 四、功能规格详解

### 4.1 对话式画像构建

**功能 ID**：P0-1

**概述**：
通过自然语言对话自动构建学生画像，替代传统表单填写。系统在对话中逐步提取学生的知识基础、学习风格、薄弱点、进度、目标等信息，构建多维动态画像，并以雷达图可视化展示。后续答题/学习行为触发画像随学随新。

**输入**：
- 用户自然语言消息（字符串，≥1 字符）
- Session ID（标识对话会话）
- 可选：已有画像 JSON（增量更新时传入）

**输出**：
- 画像 JSON（含 8 个维度字段）
- 对话回复文本（SSE 流式）
- 画像雷达图（前端渲染）

**画像维度定义**：

| 维度 | 字段 | 类型 | 取值 |
|------|------|------|------|
| 知识掌握度 | `knowledge_base` | string | `none` / `beginner` / `intermediate` / `advanced` |
| 学习风格 | `learning_style` | string | `visual` / `reading` / `hands-on` / `auditory` |
| 备考目标 | `goal` | string | `exam` / `practical` / `theory` / `general` |
| 薄弱知识点 | `weak_points` | string | 逗号分隔知识点名称 |
| 学习进度 | `progress` | integer | 1-7（表示已完成的章节数） |
| 兴趣方向 | `interest_area` | string | `networking` / `security` / `protocol` / `programming` / `general` |
| 每日学习时间 | `study_time` | string | `0-1h` / `1-2h` / `2-4h` / `4h+` |
| 难度偏好 | `preferred_difficulty` | string | `easy` / `medium` / `hard` / `adaptive` |

**交互流程**：

```
[前端] 用户首次打开画像页面
  → 系统发送引导消息："你好！我是你的AI学习助手..."
  
[前端] 用户输入回复
  → POST /api/profile/build { message, session_id }
  
[后端] LLM 处理对话
  → 提取画像信息（意图识别 + NER）
  → 返回: { reply, profile_update, is_complete }
  
[前端] 渲染对话 + 更新画像
  → 当 is_complete=true 时展示雷达图
  → 画像存入 localStorage / 后端 DB
  
[后续] 答题后触发随学随新
  → POST /api/quiz/submit { answers, profile }
  → 画像维度动态更新（至少 knowledge_base/weak_points/progress）
```

**边界条件**：
- 首次对话最少 2 轮，最多 10 轮完成
- 最少收集 4 个维度即可标记 `is_complete=true`
- 用户可随时跳过/重新开始画像构建
- 历史画像作为种子传给 LLM（避免重复提问）
- 对话超时 60 秒无响应则提示

**前端组件**：
- `views/ProfileBuilder.vue`：对话式画像页面
- `components/ProfilePanel.vue`：雷达图可视化面板

**后端接口**：
- `POST /api/profile/build`：对话式画像构建
- `GET /api/profile/{session_id}`：获取已有画像

---

### 4.2 多智能体协同资源生成

**功能 ID**：P0-2

**概述**：
当用户请求学习特定知识点时，LangGraph StateGraph 编排 7 个 Agent 节点协同工作，生成多类型学习资源包。前端通过 SSE 实时展示 Agent 工作进度。

**Agent 角色定义**：

| 节点 | 角色 | 职责 | 依赖 |
|------|------|------|------|
| Coordinator | 全局协调 | 分析用户意图，决定 Agent 调用路径 | 用户输入 + 画像 |
| Diagnostician | 学情诊断 | 薄弱点分析 + 知识缺口定位 | 画像 + Coordinator 输出 |
| Planner | 教学规划 | 拆解教学任务 → 分派给 Generator Cluster | Diagnostician 输出 |
| Retriever | 检索优化 | FrugalRAG 检索相关知识片段 | Planner 输出 + 知识库 |
| Generator Cluster | 资源生成集群 | 4 子 Agent 并行生成资源 | Retriever 检索结果 |
| - Teacher | 讲解文档 | Markdown 知识点讲解文档 | 检索结果 + 画像 |
| - QuizMaster | 题目生成 | ≥3 道练习题（选择/填空/简答/计算） | 检索结果 + 画像 |
| - MediaAgent | 思维导图 | markmap 格式导图大纲 | 检索结果 |
| - Extension | 拓展阅读 | 论文/实践/前沿/开源推荐 | 检索结果 + 画像 |
| Critic | 审阅+共识 | 事实核查 + GOMARL 共识校验 | 所有 Generator 输出 |

**输入**：
```json
{
  "message": "讲解TCP三次握手",
  "topic": "TCP",
  "difficulty": "medium",
  "course": "computer_network",
  "session_id": "xxx",
  "profile": { /* 可选，当前画像 */ }
}
```

**输出**（SSE 流式）：
```
data: {"type":"status","field":"coordinating","content":"Coordinator 正在解析请求..."}
data: {"type":"node_done","field":"coordinator","content":"请求解析完成"}
data: {"type":"status","field":"diagnosing","content":"Diagnostician 正在分析薄弱点..."}
data: {"type":"node_done","field":"diagnostician","content":"薄弱点分析完成"}
data: {"type":"status","field":"planning","content":"Planner 正在制定教学计划..."}
data: {"type":"node_done","field":"planner","content":"教学计划完成"}
data: {"type":"status","field":"retrieving","content":"Retriever 正在检索知识库..."}
data: {"type":"node_done","field":"retriever","content":"检索到5条相关知识"}
data: {"type":"content","field":"teacher","content":"讲解文档内容..."}
data: {"type":"content","field":"quiz","content":"题目内容..."}
data: {"type":"content","field":"media","content":"{\"nodes\":[...]}"}
data: {"type":"content","field":"extension","content":"拓展阅读推荐..."}
data: {"type":"status","field":"critic","content":"Critic 正在审阅..."}
data: {"type":"node_done","field":"critic","content":"审阅通过"}
data: [DONE]
```

**GOMARL 共识流程**：
1. 各 Agent 对自身输出自评分（准确性/完整性/适配性，1-10）
2. 加权投票：当前权重 × 自评分 → 综合质量分
3. 关键词矛盾检测（4 类预设矛盾对：TCP/UDP、握手次数、设备层级、IP分类）
4. 决策：质量分 ≥7 且无矛盾 → 通过；否则 → 重生成（最多 2 轮）
5. Critic 输出审核报告：`critic_report` 含通过/拒绝/建议

**交互流程**：
```
[前端] 用户在聊天框输入知识点
  → POST /api/agents/langgraph/stream { message, topic, difficulty, course }
  
[后端] LangGraph StateGraph 启动
  → Coordinator → Diagnostician → Planner → Retriever
  → Generator Cluster（4 子 Agent 并行）
  → Critic + GOMARL 共识校验
  → 条件边判断：通过 → 返回；不通过 → 重生成
  
[前端] SSE 逐节点消费
  → ChatView.vue 按 segment 渲染
  → 进度条显示 Agent 节点状态（绿色=完成/蓝色=进行中/灰色=等待）
  → 内容卡片：讲解/题目/导图/拓展 分 tab 展示
```

**边界条件**：
- 用户输入为空 → Coordinator 返回提示
- 知识点不在知识库中 → Retriever 返回空结果 + 提示"该知识点暂无资料"
- GOMARL 连续 2 轮不通过 → 降级返回最佳 Agent 结果 + 标注"未通过共识校验"
- 任一 LLM 调用失败 → 三通道自动回退
- 对话超时 120 秒 → 返回已生成内容 + 提示超时
- 单次资源生成 Token 消耗限制 ≤ 8000 tokens（防止成本爆表）

**资源类型覆盖（赛题 ≥5 种）**：
1. 讲解文档（Markdown + KaTeX 公式）
2. 练习题（选择/填空/简答/计算）
3. 思维导图（markmap 格式）
4. 拓展阅读（论文/实践/前沿/开源）
5. 审核报告（Critic + GOMARL）
6. 代码沙箱实操（P1 补全 MCP 工具）

---

### 4.3 个性化学习路径 + 资源推送

**功能 ID**：P0-6

**概述**：
基于 7 章知识点 DAG + 学生画像薄弱点，动态生成个性化学习路径排序，每章关联推荐学习资源。

**输入**：
```json
{
  "course": "computer_network",
  "profile": { /* 当前画像 */ },
  "current_chapter": 4  // 可选，当前学习章节
}
```

**输出**：
```json
{
  "chapters": [
    {
      "id": 4,
      "name": "网络层",
      "status": "in_progress",    // completed / in_progress / not_started
      "mastery": 0.45,            // 0-1 掌握度
      "recommended_resources": [
        { "type": "teacher", "title": "IP分组转发详解" },
        { "type": "quiz", "title": "网络层练习题(5题)" }
      ],
      "priority": 1               // LLM 排序优先级
    },
    // ... 7 章全部
  ],
  "overall_progress": 0.32,
  "recommended_order": [4, 5, 6, 3, 7, 1, 2]  // 推荐学习顺序
}
```

**DAG 结构**：
- 计网 7 章：概述/物理层/数据链路层/网络层/运输层/应用层/网络安全
- 依赖关系：e.g. 物理层 → 数据链路层 → 网络层 → 运输层 → 应用层
- 每章含种子知识点：计网 48 个 + 数据结构 32 个

**交互流程**：
```
[前端] 学习路径页面加载
  → GET /api/learning-path-with-resources?course=computer_network&session_id=xxx
  
[后端] 计算路径排序
  → 读取画像薄弱点 + DAG 先修关系
  → LLM 动态排序（薄弱章节优先，但尊重先修约束）
  → 每章匹配 FrugalRAG 检索相关资源
  
[前端] 渲染 DAG 可视化
  → 绿/黄/灰三色区分完成/进行中/未开始
  → 点击节点联动右侧推荐资源面板
  → 进度条显示 overall_progress
```

**边界条件**：
- 无画像时按 DAG 默认顺序推荐
- 画像更新后路径自动重排
- 已完成章节标记为绿色，仅推荐复习资源
- 依赖关系约束：前序章节未完成 → 后续章节推荐但不强制

---

### 4.4 在线练习 + 答题闭环

**功能 ID**：P0-7

**概述**：
基于知识点自动生成练习题，学生答题后即时判分、展示解析，并触发画像动态更新（随学随新）。

**输入**：
```json
{
  "topic": "TCP三次握手",
  "difficulty": "medium",
  "count": 5,
  "types": ["single_choice", "fill_blank", "short_answer"],
  "session_id": "xxx",
  "profile": { /* 当前画像 */ }
}
```

**输出**：
```json
{
  "questions": [
    {
      "id": "q1",
      "type": "single_choice",
      "stem": "TCP三次握手中，客户端发送的第一个报文段的标志位是？",
      "options": ["A. SYN=1, ACK=0", "B. SYN=1, ACK=1", "C. SYN=0, ACK=1", "D. FIN=1"],
      "answer": "A",
      "explanation": "第一次握手：客户端发送SYN=1,ACK=0的报文段..."
    }
    // ... 更多题目
  ]
}
```

**答题提交输入**：
```json
{
  "answers": [{"question_id": "q1", "answer": "A"}, ...],
  "session_id": "xxx",
  "profile": { /* 当前画像 */ }
}
```

**答题提交输出**：
```json
{
  "score": 3,
  "total": 5,
  "correct_rate": 0.6,
  "results": [
    {"question_id": "q1", "correct": true, "explanation": "..."},
    {"question_id": "q2", "correct": false, "your_answer": "B", "correct_answer": "C", "explanation": "..."}
  ],
  "profile_update": {
    "knowledge_base": "intermediate",  // 可能变化
    "weak_points": "路由选择协议,TCP流量控制",  // 更新
    "progress": 5  // 可能变化
  }
}
```

**交互流程**：
```
[前端] 练习页面
  → 用户选择知识点/从学习路径进入
  → POST /api/quiz/generate { topic, difficulty, count, types }
  → 渲染题目列表（支持选择/填空/简答不同交互）
  
[前端] 用户答题完提交
  → POST /api/quiz/submit { answers, session_id, profile }
  → 展示判分结果 + 每题解析
  → 画像雷达图实时更新
  → 自动推荐薄弱点针对性资源
```

**边界条件**：
- 题目生成最少 1 题，最多 10 题（防止 Token 过多）
- 简答题判定用 LLM 语义匹配（允许意思对即可）
- 答题时间不限，但 30 分钟无操作提示
- 错题自动加入错题本（P1 功能）
- 连续 3 次同类题正确率 >80% → 降低该知识点权重

---

### 4.5 学习效果评估仪表盘

**功能 ID**：P0-8

**概述**：
综合展示学生学习效果的多维度数据看板，包含 7 章热力图、活跃度趋势、正确率趋势、薄弱点排行、易错回顾和 AI 学习建议。

**输入**：
```json
{
  "course": "computer_network",
  "session_id": "xxx",
  "profile": { /* 当前画像 */ }
}
```

**输出**：
```json
{
  "heatmap": {
    "chapters": [
      {"id": 1, "name": "概述", "mastery": 0.85},
      {"id": 4, "name": "网络层", "mastery": 0.45}
    ]
  },
  "activity_trend": [
    {"date": "2026-06-25", "study_minutes": 120},
    {"date": "2026-06-26", "study_minutes": 90}
  ],
  "accuracy_trend": [
    {"date": "2026-06-25", "correct_rate": 0.5},
    {"date": "2026-06-26", "correct_rate": 0.65}
  ],
  "top_weak_points": [
    {"name": "路由选择协议", "error_count": 8, "priority": "high"},
    {"name": "IP子网划分", "error_count": 5, "priority": "medium"}
  ],
  "error_review": [
    {"question": "RIP协议的最大跳数是多少？", "correct_answer": "15", "your_answer": "16", "date": "2026-06-27"}
  ],
  "llm_suggestion": "建议你每天投入1小时专门攻克路由选择协议..."
}
```

**交互流程**：
```
[前端] 评估页面加载
  → GET /api/assessment?course=computer_network&session_id=xxx
  
[后端] 聚合计算
  → 从 DB 读取：答题记录 + 学习时长 + 画像历史
  → LLM 生成学习建议
  
[前端] 渲染仪表盘
  → AssessmentView.vue 多个图表组件
  → 热力图 / 折线图 / 排行榜 / 易错列表 / 建议卡片
```

**可视化组件**：
1. 7 章热力图（绿-黄-红渐变，掌握度映射颜色）
2. 活跃度趋势折线图（近 14/30 天）
3. 答题正确率趋势折线图（近 14/30 天）
4. 薄弱知识点排行榜（Top 5，按错误次数排序）
5. 易错题回顾列表（近 7 天，最多 10 条）
6. LLM 学习建议卡片（AI 生成文字建议）

**边界条件**：
- 无历史数据 → 展示空状态提示 + "开始学习后这里会显示评估数据"
- 数据量过大 → 默认展示近 30 天，可切换时间范围
- LLM 建议生成失败 → 展示默认建议模板
- 页面加载超时 5 秒 → 展示骨架屏

---

### 4.6 FrugalRAG 检索引擎

**功能 ID**：P0-3（Lite 版）、P1-3/P1-4（真版）

**Lite 版（软件杯）**：

```
输入: 用户查询字符串 + 课程标识
  ↓
E5-base-v2 embedding (768维)
  ↓
├─ Milvus/ChromaDB 向量检索 (top_k=5, cosine_threshold=0.65)
│   └─ 过滤低于阈值的噪声结果
├─ BM25 关键词检索 (补充精确匹配)
│
└─ 融合排序: 0.7 × vector_score + 0.3 × bm25_score
  ↓
输出: top_k 个知识片段 (chunk_id, text, score, metadata)
```

**真版增量（大创中期）**：
- SFT 策略训练：用 500 条标注样本微调查询生成策略
- GRPO 停止决策：RL 学习何时停止检索，减少冗余调用
- 目标：检索成本持平，Recall@5 相对提升 15.8%（benchmark 实测，28 题 / 2083 chunks）

**接口**：
- `POST /api/knowledge/search { query, course, top_k? }` → 检索知识片段
- `GET /api/knowledge/stats` → 知识库统计（chunk 数/科目分布等）

---

### 4.7 GOMARL 共识引擎

**功能 ID**：P0-4（Lite 版）、P1-5/P1-6/P1-7（真版）

**Lite 版（软件杯）**：

```
输入: 各 Agent 输出 + 自评分
  ↓
加权投票: base_weight × 历史表现权重 × 自评分 → 综合质量分
  ↓
一致性校验: 4 类关键词矛盾检测
  ├─ TCP 相关 vs UDP 相关
  ├─ 握手次数矛盾（2次/3次/4次）
  ├─ 设备层级矛盾（交换机/路由器/网关）
  └─ IP 分类矛盾（A/B/C/D/E 类）
  ↓
决策: 质量分 ≥ 7 且无矛盾 → 通过
       质量分 < 7 或有矛盾 → 重生成（最多 2 轮）
  ↓
输出: 通过 → consensus_result + critic_report
       最终不通过 → 降级返回最佳 Agent 结果 + 警告标记
```

**真版增量（大创中期）**：
- 知识校验层：调用 FrugalRAG 检索证据验证冲突
- 冲突消解层：基于证据的冲突消解算法
- 动态权重层：结合学生画像的 EWMA 动态权重调整

---

### 4.8 三通道 LLM 容灾路由

**功能 ID**：P0-10

| 优先级 | 通道 | 模型 | 能力 | 用途 |
|--------|------|------|------|------|
| 1 | DeepSeek | `deepseek-chat` | 工具调用 + 流式 + 长上下文 | 首选通道（质量最高） |
| 2 | **讯飞星火 X2** | `Spark-X2-Flash` | **赛题合规要求** + 深度推理 | 赛题合规通道 |
| 3 | Qwen2.5 | `qwen2.5-7b-instruct` | 工具调用 + 流式 | 最后回退 |

**路由逻辑**：
1. `llm_provider = "auto"` → 按优先级 1→2→3 尝试
2. 任一通道失败（超时/限流/错误）→ 自动回退下一通道
3. `llm_provider` 指定通道 → 直连，失败报错
4. 所有通道均不可用 → 返回 503 错误 + "所有 LLM 通道暂不可用"

**6 处 LLM 调用点**：Coordinator / Diagnostician / Planner / Generator Cluster / Critic / 画像对话

**配置**（config.json）：
```json
{
  "llm": {
    "provider": "auto",
    "deepseek": { "api_key": "sk-...", "model": "deepseek-chat" },
    "xunfei": { "app_id": "...", "api_key": "...", "model": "Spark-X2-Flash" },
    "qwen": { "api_key": "sk-...", "model": "qwen2.5-7b-instruct" }
  }
}
```

---

### 4.9 内容安全 + 防幻觉

**功能 ID**：P0-9

**多层防护机制**：

| 层级 | 机制 | 实现 | 触发行为 |
|------|------|------|----------|
| 输入过滤 | 敏感词检测 | 13 类违规词正则匹配 | 拒绝生成，返回提示 |
| 事实核查 | Critic Agent | TCP/UDP/握手/设备层级/IP分类关键事实核查 | 标记错误，要求重生成 |
| 共识校验 | GOMARL 一致性检测 | 4 类矛盾对关键词检测 | 触发重生成 |
| 幻觉检测 | 知识错误关键词检测 | 4 类常见 LLM 幻觉模式检测 | 标记 + 告警日志 |

**敏感词类别（13 类）**：政治/色情/暴力/赌博/毒品/诈骗/侵权/人身攻击/歧视/隐私泄露/恶意代码/虚假信息/违法违规。

**幻觉检测（4 类）**：
1. 胡说八道的协议编号（如"RFC 99999"）
2. 不存在的算法名称
3. 错误的年份/时间戳
4. 自相矛盾的技术定义

---

## 五、技术约束与依赖

### 5.1 LLM API 依赖

| 依赖项 | 必要性 | 风险 | 缓解措施 |
|--------|--------|------|----------|
| DeepSeek API | 推荐（首选） | API 限流/服务中断 | 三通道自动回退 |
| 讯飞星火 X2 API | **强制（赛题合规）** | API 限流/服务中断 | 三通道自动回退 |
| Qwen2.5 API | 推荐（第三通道） | API 限流/服务中断 | 作为最后回退 |
| E5-base-v2 Embedding | **强制（本地）** | 模型下载失败 | 本地缓存模型文件 |

### 5.2 向量数据库选择

| 环境 | 方案 | 适用阶段 |
|------|------|----------|
| 开发/软件杯 | ChromaDB（文件存储） | P0 阶段 |
| 生产/大创中期 | Milvus 2.3（HNSW 索引） | P1 阶段 |
| 回退方案 | ChromaDB 自动回退 | 当 Milvus 不可用时 |

### 5.3 部署环境要求

**开发环境**（P0）：
- OS：Windows/macOS/Linux
- Python ≥ 3.12
- Node.js ≥ 20 / bun
- 磁盘：≥ 5GB（含模型文件）
- 网络：需访问 DeepSeek/讯飞 StarFire/Qwen API

**生产环境**（P1-P2）：
- OS：Linux（Ubuntu 20.04+）
- CPU：≥ 4 核
- 内存：≥ 16GB（含 Milvus + PG + Redis）
- GPU：可选，T4 以上用于模型加速
- 磁盘：≥ 50GB（含知识库 + 数据库 + 日志）
- Docker ≥ 24.0 + Docker Compose ≥ 2.0

### 5.4 外部服务依赖

| 服务 | 用途 | 版本 |
|------|------|------|
| Milvus | 向量数据库 | 2.3 |
| PostgreSQL | 关系数据库 | 15+ |
| Redis | 缓存 + Agent 权重 | 7+ |
| Docker | 容器化部署 | 24+ |

### 5.5 前端技术依赖

| 依赖 | 版本 | 协议 |
|------|------|------|
| Vue 3 | 3.5+ | MIT |
| Vite | 8.x | MIT |
| Pinia | 3.x | MIT |
| TypeScript | 5.x | Apache-2.0 |
| marked | latest | MIT |
| KaTeX | latest | MIT |
| markmap-lib | latest | MIT |
| highlight.js | latest | BSD-3-Clause |

---

## 六、Non-goals（明确当前不做什么）

以下功能在当前版本周期内**明确不列入开发计划**：

| Non-goal | 原因 | 可能纳入的时间 |
|----------|------|---------------|
| 移动端 App（iOS/Android） | Web 端 PWA 适配即可满足两个阶段需求 | 大创结题后 |
| 语音交互（ASR/TTS） | 增加显著额外工程复杂度，非核心需求 | 大创结题后 |
| 社交功能（学习社区/讨论区/排行榜） | 偏离"个性化学习"核心定位 | 不确定 |
| 多语言支持（除中文外） | 目标用户为中国考研学生 | 不确定 |
| 在线支付/付费订阅 | 当前为学术研究项目，不涉及商业化 | 不确定 |
| 视频讲解生成 | 工程复杂度高，非赛题/大创要求 | 大创结题后（可探索） |
| 实时协作学习（多人同步） | 需要 WebSocket + 同步协议，复杂度过高 | 不确定 |
| 自适应难度调节（全量 A/B 测试） | 大创版已有难度偏好，全量 A/B 不适配研究场景 | 不确定 |
| 第三方 SSO 登录（微信/QQ） | 学术项目使用本地账号即可 | 大创结题后 |
| 离线学习模式（PWA 离线缓存） | 当前必须依赖 LLM API | 不确定 |

---

## 七、验收标准清单

### 7.1 软件杯提交验收（P0 全部功能）

| # | 验收项 | 验收方法 | 通过标准 |
|---|--------|----------|----------|
| 1 | 对话式画像 ≥6 维度 | 新建会话，完整走一次画像构建对话 | 输出 JSON 含 ≥6 个维度字段，雷达图正确渲染 |
| 2 | 画像随学随新 | 答题后检查画像变化 | ≥2 个维度字段变化（如 knowledge_base/weak_points） |
| 3 | LangGraph 7-Agent 流式协同 | 输入"讲解 TCP 三次握手"，观察 SSE 流 | 收到 ≥4 个 `node_done` 事件，最终返回 ≥4 种资源 |
| 4 | SSE 流式实时推送 | 前端 ChatView 中观察 Agent 进度 | 逐节点推送，进度条 6 节点均点亮 |
| 5 | FrugalRAG 检索正确 | 输入特定知识点，检查检索结果相关性 | Top-3 结果与查询知识点相关度 ≥70%（人工判定） |
| 6 | GOMARL 共识校验 | 故意输入矛盾知识场景（如 TCP 和 UDP 混合） | 系统检测到矛盾并触发重生成或标记警告 |
| 7 | 学习路径 DAG 可视化 | 打开学习路径页面 | 7 章 DAG 正确显示，绿/黄/灰颜色正确区分 |
| 8 | 路径→资源联动 | 点击 DAG 节点 | 右侧面板展示 ≥2 个推荐资源 |
| 9 | 练习→答题→判分→画像更新 | 完成一组 5 题练习 | 即时判分正确，画像更新后雷达图变化 |
| 10 | 评估仪表盘 | 有历史答题数据后打开评估页 | 热力图/折线图/排行/建议至少展示 4 种图表 |
| 11 | 三通道容灾 | 模拟 DeepSeek 不可用（临时修改 API Key） | 自动切换到讯飞星火 X2，功能正常 |
| 12 | 讯飞星火合规 | 检查 LLM 通道优先级配置 | 讯飞为第二优先级（非第一/非第三） |
| 13 | 内容安全防护 | 输入敏感词/知识错误场景 | 敏感词被过滤，幻觉输出被标记 |
| 14 | 前端 12 视图可用 | 逐一访问每个前端路由 | 无白屏/JS 报错/404 |
| 15 | 知识库完整（计网） | 检索计网各章知识点 | 7 章均可检索到相关知识片段 |
| 16 | **前后端 API 一致性** | 检查 4 个不一致接口修复 | sessions load/save/delete 正常工作，knowledge-graph 端点可用 |
| 17 | **Python 版本兼容** | 在 Python 3.12/3.13 环境下 uv sync + 启动服务 | 无依赖冲突报错，服务正常启动 |

### 7.2 大创中期检查验收（P0+P1 全部功能）

| # | 验收项 | 验收方法 | 通过标准 |
|---|--------|----------|----------|
| 1 | Milvus 生产部署 | 检查向量库状态 | Milvus 可连接，检索性能 ≥ChromaDB |
| 2 | PostgreSQL 持久化 | 创建/读取画像/答题数据 | CRUD 操作正常，数据不丢失 |
| 3 | Redis 缓存有效 | 检查缓存命中日志 | 缓存命中率 ≥60% |
| 4 | FrugalRAG SFT 训练 | 运行训练 pipeline | 训练可完成，检索准确率提升 |
| 5 | GOMARL 证据校验 | 制造知识冲突场景 | FrugalRAG 证据可验证冲突，消解结果可解释 |
| 6 | 动态权重生效 | 多次使用后检查权重变化 | 高质量 Agent 权重大于低质量 Agent |
| 7 | 数据结构知识库 | 检索 DS 知识点 | ≥500 chunks 可用 |
| 8 | MCP 工具可用 | 调用 TCP 握手/子网计算/沙箱 | 至少 3 种工具可被 Agent 调用 |
| 9 | 教师端后台可用 | 登录教师端 | 可查看学生列表/画像/统计数据 |

### 7.3 大创结题验收（全部功能）

| # | 验收项 | 验收方法 | 通过标准 |
|---|--------|----------|----------|
| 1 | 四科知识库 ≥1800 chunks | 统计 all chunks | 四科合计 ≥1800 chunks |
| 2 | Docker 一键部署 | `docker compose up -d` | 全部服务正常启动 |
| 3 | 50 并发无降级 | 压测工具 50 并发 | 响应时间 ≤5 秒，无错误 |
| 4 | 试点满意度 ≥80% | 用户问卷 | ≥80% 用户评分 4/5 以上 |
| 5 | 软著受理 | 提交软著申请 | 受理通知书 |

---

## 附录

### A. 版本差异对照表

| 维度 | 软件杯版（Lite） | 大创中期版 | 大创结题版 |
|------|-----------------|-----------|-----------|
| 画像维度 | 6-8 维度 | 8 维度 | 8 维度 |
| Agent 数量 | 7（6节点+条件边） | 9 节点完整 | 9 节点完整 |
| FrugalRAG | Lite 版（E5+BM25） | 真版（SFT+GRPO） | 真版（全量） |
| GOMARL | Lite 版（加权投票） | 真版（证据校验+消解+动态权重） | 真版（全量） |
| 向量库 | ChromaDB（Milvus 可选） | Milvus 2.3 | Milvus 2.3 |
| 数据库 | 无（内存存储） | PostgreSQL + Redis | PostgreSQL + Redis |
| 知识库 | 计网 1 门（228 条种子数据，实际向量 201 文档） | 计网+DS（≥1000 chunks） | 四科（≥1800 chunks） |
| LLM | DeepSeek/讯飞/Qwen 三通道 | 同左 | 同左 |
| 部署 | 手动启动 | Docker Compose | Docker 一键部署 |
| 后端框架 | FastAPI 13 路由 | 同左 | 同左 |
| Agent 框架 | LangGraph StateGraph | 同左 | 同左 |

### B. 术语表

| 术语 | 全称/说明 |
|------|-----------|
| 408 | 计算机学科专业基础综合（全国统考科目代码） |
| FrugalRAG | 节俭检索增强生成（Frugal Retrieval-Augmented Generation） |
| GOMARL | 基于图优化的多智能体强化学习（Graph Optimization-based Multi-Agent RL） |
| LangGraph | LangChain 提供的 StateGraph 多 Agent 编排框架 |
| StateGraph | LangGraph 中的有状态有向图，节点=Agent，边=数据流/条件路由 |
| SSE | Server-Sent Events，服务端推送事件 |
| E5-base-v2 | 微软开源的文本嵌入模型，768 维向量 |
| BM25 | Best Match 25，经典关键词检索排序算法 |
| HNSW | Hierarchical Navigable Small World，近似最近邻搜索索引 |
| GRPO | Group Relative Policy Optimization，强化学习算法 |
| SFT | Supervised Fine-Tuning，监督微调 |
| EWMA | Exponentially Weighted Moving Average，指数加权移动平均 |
| DAG | Directed Acyclic Graph，有向无环图 |
| MCP | Model Context Protocol，模型上下文协议 |

### C. 参考文档

| 文档 | 路径 |
|------|------|
| 开发说明书 | `documents/开发说明书.md` |
| 大创申报书 | `documents/大创申报书.md` |
| 技术方案文档 | `documents/技术方案文档.md` |
| 软件杯赛题合规清单 | `documents/软件杯赛题合规清单.md` |
| 数据分析报告 | `documents/数据分析报告.md` |
| 项目方向与架构讨论纪要 | `documents/项目方向与架构讨论纪要.md` |
| 申报书原始文本 | `申报书内容.txt` |

---

> 本 PRD 由析客（Specky）基于已有项目文档撰写，覆盖软件杯和大创两个版本的完整需求规格。
> v2.0 更新：基于代码审计数据补充 API 一致性风险、Python 版本兼容、知识库实际数据量、项目健康状态摘要。
> 文档版本：v2.0 | 撰写日期：2026-07-02
