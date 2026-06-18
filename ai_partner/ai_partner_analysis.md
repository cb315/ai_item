# `ai_partner.py` 代码逐段分析文档

> **文件路径：** `C:\programer\pythoncode\py_project01\stady_ai01\ai_partner.py`
>
> **项目简介：** 这是一个基于 **Streamlit + DeepSeek 大模型** 构建的 **AI 伴侣** 聊天应用。用户可以为 AI 伴侣设置昵称和性格，通过流式对话与 AI 互动，且支持多会话的创建、加载、保存和删除。

---

## 1. 文件头部注释（第 1 行）

\`\`\`python
#生成一个ai伴侣 1.大模型部署方案  2.http协议  3.大模型交互方案  
# 4.大模型会话记忆方案（滚雪球） 5.streamllit构建页面 6.文件基本操作 7.json操作 8.os/datatime模块
\`\`\`

**作用：** 项目的整体说明注释，列出本 demo 所涵盖的技术要点：

1. **大模型部署方案** — 调用云端 DeepSeek API
2. **HTTP 协议** — 通过 OpenAI SDK 使用 HTTP 请求与 API 通信
3. **大模型交互方案** — 采用流式（Stream）输出，逐 chunk 拼接显示
4. **大模型会话记忆方案（滚雪球）** — 将历史消息累积追加到 messages 列表并每次全量发送
5. **Streamlit 构建页面** — 使用 Streamlit 搭建 Web UI
6. **文件基本操作** — 使用 os 模块读写会话目录和文件
7. **JSON 操作** — 使用 json 模块序列化/反序列化会话数据
8. **os / datetime 模块** — 用 os 操作文件系统，用 datetime 生成时间戳

---

## 2. 模块导入（第 2–8 行）

\`\`\`python
import os
from openai import OpenAI
import streamlit as st
from streamlit import empty
import datetime
import json
\`\`\`

**作用：** 导入项目所有依赖的标准库和第三方库。

| 模块 | 用途 |
|------|------|
| os | 检查/创建目录、列出文件、删除文件等文件系统操作 |
| OpenAI (from openai) | 调用大模型 API（兼容 OpenAI 协议的 DeepSeek） |
| streamlit as st | Web 应用框架，用于构建交互式 UI |
| empty (from streamlit) | 创建一个可被后续内容替换的空占位符（流式输出用） |
| datetime | 生成当前时间字符串，用作会话文件名 |
| json | 将会话数据序列化为 JSON 文件，或从 JSON 文件反序列化 |

---

## 3. 页面配置（第 9–20 行）

\`\`\`python
st.set_page_config(
    page_title="AI伴侣",
    page_icon=":💩:",
    layout="wide",
    initial_sidebar_state="auto",
    menu_items={
        'Get Help': 'https://github.com/xiaoxiao-ai/ai-partner',
        'Report a bug': "https://github.com/xiaoxiao-ai/ai-partner/issues",
        'About': "# This is a demo of AI-Partner"
    }
)
\`\`\`

**作用：** 配置 Streamlit 页面的全局属性。

- **page_title** — 浏览器标签页标题："AI伴侣"
- **page_icon** — 浏览器标签页图标（emoji）
- **layout="wide"** — 使用宽屏布局（而非默认的窄屏居中布局）
- **initial_sidebar_state="auto"** — 侧边栏初始状态自动决定
- **menu_items** — 自定义右上角菜单的三个条目：
  - Get Help — 跳转到 GitHub 仓库
  - Report a Bug — 跳转到 Issues 页面
  - About — 显示简介文本

---

## 4. save_session() — 保存会话到 JSON（第 22–36 行）

\`\`\`python
def save_session():
    if st.session_state.current_datetime:
        session_data = {
            "nick_name": st.session_state.nick_name,
            "natrue": st.session_state.natrue,
            "current_datetime": st.session_state.current_datetime,
            "message": st.session_state.message
        }
    if not os.path.exists("session"):
        os.mkdir("session")
    with open(f"session/{st.session_state.current_datetime}.json", "w", encoding="utf-8") as f:
        json.dump(session_data, f, ensure_ascii=False, indent=4)
\`\`\`

**作用：** 将当前会话的所有数据保存为一个 JSON 文件。

- **数据结构：** 构造一个字典，包含昵称（nick_name）、性格（natrue）、当前时间戳（current_datetime）和完整消息记录（message）。
- **目录检查：** 如果 session/ 目录不存在，自动创建。
- **写入方式：** 以 UTF-8 编码写入 session/{时间戳}.json，使用 ensure_ascii=False 保留中文，indent=4 美化格式。
- **注意点：** 缩进不严谨 — if 块内的 session_data 定义虽不在块外可见的缩进层级，但 Python 允许执行。

---

## 5. create_session_file() — 生成会话标识名（第 37–39 行）

\`\`\`python
def create_session_file():
    return datetime.datetime.now().strftime("%Y-%m-%d-%H-%M-%S")
\`\`\`

**作用：** 生成一个以当前时间精确到秒的时间戳字符串，用作新会话的文件名（不含 .json 后缀）。

**示例输出：** "2026-06-16-17-00-22"

---

## 6. save_creat_session() — 加载所有会话列表（第 40–52 行）

\`\`\`python
def save_creat_session():
    session_list = []
    if os.path.exists("session"):
        file_list = os.listdir("session")
        for filename in file_list:
            if filename.endswith(".json"):
                session_list.append(filename[:-5])
    session_list.sort(reverse=True)
    return session_list
\`\`\`

**作用：** 读取 session/ 目录下所有 .json 文件，返回按时间倒序排列的会话名称列表。

- **过滤条件：** 只取扩展名为 .json 的文件
- **去除后缀：** 用 filename[:-5] 去掉 .json 后缀（共 5 个字符）
- **排序：** 使用 sort(reverse=True) 按文件名字符串倒序排列，因为文件名本身是时间戳（YYYY-MM-DD-HH-MM-SS），字符串倒序等价于最新会话在前

---

## 7. specify_session() — 加载指定会话（第 53–63 行）

\`\`\`python
def specify_session(session_name):
    try:
        if os.path.exists(f"session/{session_name}.json"):
            with open(f"session/{session_name}.json", "r", encoding="utf-8") as f:
                session_data = json.load(f)
                st.session_state.message = session_data["message"]
                st.session_state.nick_name = session_data["nick_name"]
                st.session_state.natrue = session_data["natrue"]
                st.session_state.current_datetime = session_name
    except Exception as e:
        st.error("加载绘画失败")
\`\`\`

**作用：** 从 JSON 文件中读取指定会话的数据，恢复到 st.session_state 中。

- **恢复的内容：** 消息列表（message）、昵称（nick_name）、性格（natrue）、当前会话标识（current_datetime）
- **异常处理：** 如果文件不存在或 JSON 解析失败，在页面上显示错误信息"加载绘画失败"
- **触发时机：** 用户点击侧边栏中的历史会话按钮时调用

---

## 8. delete_session() — 删除会话（第 64–72 行）

\`\`\`python
def delete_session(session_name):
    try:
        if os.path.exists(f"session/{session_name}.json"):
            os.remove(f"session/{session_name}.json")
            if session_name == st.session_state.current_datetime:
                st.session_state.message = []
                st.session_state.current_datetime = create_session_file()
    except Exception as e:
        st.error("加载绘画失败")
\`\`\`

**作用：** 删除指定的会话 JSON 文件。如果删除的是当前正在使用的会话，则自动重置消息列表并生成一个新的会话。

- **删除逻辑：** 先检查文件是否存在，再调用 os.remove() 删除
- **当前会话处理：** 如果删除的是当前会话，清空 st.session_state.message 并生成新的时间戳作为新会话标识
- **异常处理：** 同 specify_session，捕获异常后显示"加载绘画失败"

---

## 9. 主标题与 Logo（第 73–75 行）

\`\`\`python
st.title("AI伴侣")
st.logo("resource/img.png")
\`\`\`

**作用：** 在页面顶部居中显示大标题 "AI伴侣"，并在左上角显示 Logo 图片。

- **Logo 路径：** resource/img.png（相对于工作目录）

---

## 10. 系统提示词模板（第 76–93 行）

\`\`\`python
system_prompt = """
        你叫 %s，现在是用户的真实伴侣，请完全代入伴侣角色。
        规则：
            1. 每次只回1条消息
            2. 禁止任何场景或状态描述性文字
            3. 匹配用户的语言
            4. 回复简短，像微信聊天一样
            5. 有需要的话可以用❤️🌸等emoji表情
            6. 用符合伴侣性格的方式对话
            7. 回复的内容, 要充分体现伴侣的性格特征
        伴侣性格：
            - %s
        你必须严格遵守上述规则来回复用户。
    """
\`\`\`

**作用：** 定义发给大模型的 System Prompt 模板，使用 %s 占位符。

- **%s 占位符 1：** 伴侣的昵称（如"小宝"）
- **%s 占位符 2：** 伴侣的性格描述（如"活泼开朗的台湾妹妹"）
- **7 条规则：** 规范 AI 回复的风格、长度、语言、表情使用等，使 AI 回复更接近真实伴侣对话
- **实际使用时：** 通过 system_prompt % (nick_name, natrue) 填充占位符

---

## 11. 初始化大模型客户端（第 95–96 行）

\`\`\`python
client = OpenAI(api_key=os.environ.get('DEEPSEEK_API_KEY'), base_url="https://api.deepseek.com")
\`\`\`

**作用：** 创建一个 OpenAI SDK 客户端实例，用于调用 DeepSeek 大模型 API。

- **api_key** — 从环境变量 DEEPSEEK_API_KEY 读取 API 密钥
- **base_url** — 指定为 DeepSeek 的 API 端点 https://api.deepseek.com
- **兼容性：** DeepSeek API 兼容 OpenAI 的请求格式，因此可以直接使用 OpenAI SDK

---

## 12. 初始化聊天状态（第 99–112 行）

\`\`\`python
if "message" not in st.session_state:
    st.session_state.message = []

if "nick_name" not in st.session_state:
    st.session_state.nick_name = "小宝"

if "natrue" not in st.session_state:
    st.session_state.natrue = "活泼开朗的台湾妹妹"

if "current_datetime" not in st.session_state:
    st.session_state.current_datetime = create_session_file()
\`\`\`

**作用：** 确保 st.session_state 中的四个关键变量在首次运行时有默认值（Streamlit 的 session state 会在每次用户交互时保持，所以用 if not in 防止重复覆盖）。

| 变量 | 默认值 | 说明 |
|------|--------|------|
| message | []（空列表） | 存储对话历史，元素格式为 {"role": "user"/"assistant", "content": "..."} |
| nick_name | "小宝" | AI 伴侣的昵称 |
| natrue | "活泼开朗的台湾妹妹" | AI 伴侣的性格描述 |
| current_datetime | 当前时间戳字符串 | 当前会话的唯一标识，也用作 JSON 文件名 |

---

## 13. 显示当前会话信息（第 114 行）

\`\`\`python
st.text(f"当前会话：{st.session_state.current_datetime}")
\`\`\`

**作用：** 在页面主体顶部以纯文本形式显示当前会话的时间戳，方便用户确认当前操作的会话。

---

## 14. 渲染聊天历史（第 116–118 行）

\`\`\`python
for message in st.session_state.message:
    st.chat_message(message["role"]).write(message["content"])
\`\`\`

**作用：** 遍历 st.session_state.message 列表，逐条渲染用户和 AI 的历史消息。

- **st.chat_message(role)** — Streamlit 原生聊天组件，"user" 显示在右侧（蓝色气泡），"assistant" 显示在左侧（灰色气泡）
- **write(content)** — 支持字符串、Markdown 等多种格式的内容

---

## 15. 侧边栏 — AI 控制面板（第 120–170 行）

\`\`\`python
with st.sidebar:
    st.title("AI控制面板")
\`\`\`

这是整个应用的控制中心，包含三个功能区。

### 15.1 新建会话按钮（第 123–131 行）

\`\`\`python
if st.button("新建会话", width="stretch", icon="🤡"):
    save_session()
    if st.session_state.message:
        st.session_state.message = []
        st.session_state.current_datetime = create_session_file()
        save_session()
        st.rerun()
\`\`\`

**作用：** 点击后执行以下流程：
1. 先调用 save_session() 保存当前会话
2. 如果当前消息列表非空，清空消息、生成新时间戳、再保存一次空白会话
3. 调用 st.rerun() 让页面立即刷新

### 15.2 历史会话列表（第 132–149 行）

\`\`\`python
st.text("历史会话")
session_list = save_creat_session()
for session in session_list:
    col1, col2 = st.columns([4, 1])
    with col1:
        if st.button(session, icon="❔️️", key=f"load_{session}", width="stretch",
                     type="primary" if session == st.session_state.current_datetime else "secondary"):
            specify_session(session)
            st.rerun()
    with col2:
        if st.button("", icon="❌️", key=f"delete_{session}", width="stretch"):
            delete_session(session)
            st.rerun()
\`\`\`

**作用：** 列出所有已保存的会话。

- **布局：** 每行使用两列（col1:col2 = 4:1），左边显示加载按钮（会话名），右边显示删除按钮（X 图标）
- **加载按钮：** 当前会话对应的按钮样式为 primary（高亮），其他为 secondary。点击后调用 specify_session() 加载该会话
- **删除按钮：** 点击后调用 delete_session() 删除该会话文件。如果删除的是当前会话，会自动重置为新会话

### 15.3 伴侣信息编辑（第 152–170 行）

\`\`\`python
st.divider()
st.subheader("伴侣信息")
nick_name = st.text_input("昵称", placeholder="请输入昵称", value=st.session_state.nick_name)
natrue = st.text_area("性格", placeholder="请输入性格", value=st.session_state.natrue)
if nick_name != st.session_state.nick_name:
    st.session_state.nick_name = nick_name
if natrue != st.session_state.natrue:
    st.session_state.natrue = natrue
\`\`\`

**作用：** 允许用户实时修改 AI 伴侣的昵称和性格描述。

- **st.text_input** — 单行输入框用于昵称
- **st.text_area** — 多行文本框用于性格描述
- **实时同步：** 每次页面重渲染时，如果输入值发生变化，立即更新到 st.session_state，后续对话将使用新的昵称和性格

---

## 16. 聊天输入与 AI 响应（第 173–209 行）

这是整个应用的核心交互逻辑。

### 16.1 消息输入框（第 173 行）

\`\`\`python
prompt = st.chat_input("请输入你的问题：")
\`\`\`

**作用：** 在页面底部显示一个聊天输入框，用户输入文本后按下回车即可发送。

### 16.2 处理用户输入（第 174–176 行）

\`\`\`python
if prompt:
    st.chat_message("user").write(prompt)
    st.session_state.message.append({"role": "user", "content": prompt})
\`\`\`

**作用：** 当用户输入非空时：
1. 立即在聊天区域显示用户的消息气泡
2. 将用户消息追加到 st.session_state.message 列表中

### 16.3 调用大模型 API（第 177–190 行）

\`\`\`python
response = client.chat.completions.create(
    model="deepseek-v4-pro",
    messages=[
        {"role": "system", "content": system_prompt % (st.session_state.nick_name, st.session_state.natrue)},
        *st.session_state.message,
    ],
    stream=True,
    reasoning_effort="high",
    extra_body={"thinking": {"type": "enabled"}}
)
\`\`\`

**作用：** 向 DeepSeek API 发送聊天完成请求。

- **model** — 指定模型为 deepseek-v4-pro（DeepSeek 的最新旗舰模型）
- **messages** — 构造完整的消息列表：第一条是 System Prompt，后面使用 *st.session_state.message 解包所有历史消息，实现"滚雪球"式会话记忆
- **stream=True** — 启用流式输出，模型会逐 token 返回结果
- **reasoning_effort="high"** — 要求模型进行深度推理
- **extra_body={"thinking": {"type": "enabled"}}** — 启用模型的思考过程特性

### 16.4 流式输出处理（第 191–201 行）

\`\`\`python
response_message = st.empty()
full_response = ""
for chunk in response:
    if chunk.choices[0].delta.content:
        content = chunk.choices[0].delta.content
        full_response += content
        response_message.chat_message("assistant").write(full_response)
\`\`\`

**作用：** 以流式方式逐段接收模型返回的内容，并实时更新到聊天界面。

- **st.empty() 占位符** — 创建一个空的可替换 UI 容器
- **逐 chunk 拼接** — 每个 chunk.choices[0].delta.content 是一小段文本，将其拼接到 full_response 中
- **实时更新** — 每次拼接后立即将完整累积内容写入占位符中，实现打字机效果（逐字出现）

### 16.5 保存 AI 回复到会话（第 204–208 行）

\`\`\`python
st.session_state.message.append({"role": "assistant", "content": full_response})
save_session()
\`\`\`

**作用：**
1. 将完整的 AI 回复追加到 st.session_state.message 列表
2. 调用 save_session() 将会话数据持久化到 JSON 文件

---

## 整体架构流程图

\`\`\`mermaid
flowchart TD
    A[用户打开页面] --> B[st.set_page_config 配置页面]
    B --> C[初始化 st.session_state 变量]
    C --> D[渲染历史消息]
    D --> E[显示侧边栏控制面板]
    E --> F{用户操作}
    
    F -->|点击新建会话| G[保存当前会话 清空消息 生成新会话]
    F -->|点击历史会话| H[加载指定会话 JSON 恢复状态]
    F -->|修改昵称/性格| I[实时同步到 session_state]
    F -->|输入消息| J[显示用户消息 调用 DeepSeek API]
    
    J --> K[流式接收 AI 回复并实时显示]
    K --> L[保存 AI 回复到 message 列表]
    L --> M[调用 save_session 持久化到 JSON]
    M --> F
    
    G --> F
    H --> F
    I --> F
\`\`\`

---

## 技术亮点总结

| 方面 | 实现方式 |
|------|----------|
| 大模型调用 | OpenAI SDK + DeepSeek API，兼容 OpenAI 协议 |
| 流式输出 | stream=True + st.empty() 占位符逐 token 刷新 |
| 会话记忆 | "滚雪球"机制：每次请求发送全部历史消息 |
| 数据持久化 | JSON 文件存储，os 模块管理文件目录 |
| 多会话管理 | 时间戳文件名作为会话 ID，支持新建/加载/删除 |
| Web UI | Streamlit 框架，chat_message 组件模拟聊天界面 |
| 深度推理 | 启用 reasoning_effort="high" 和 thinking 模式 |

---

## 潜在改进点

1. **save_session() 缩进问题** — session_data 的赋值在 if 块内，但后续 with open 使用 session_data 时不在同一缩进层级，虽能运行但可读性差，建议统一缩进
2. **函数命名不一致** — save_creat_session 应为 load_session_list，natrue 应为 nature（拼写错误）
3. **环境变量缺失提示** — 如果 DEEPSEEK_API_KEY 未设置，页面应给出用户友好的错误提示而非直接崩溃
4. **会话文件增长** — 长期使用的用户会积累大量 JSON 文件，可考虑增加会话上限或清理策略
