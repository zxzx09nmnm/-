# -
🐟
# -*-编码：utf-8-*-
"""
记忆插件
功能：自动提炼用户长期记忆 → 存本地 sqlite → 每次对话注入上下文
用法：PIP安装OpenAI→python memory_plugin.py
"""
导入sqlite3
import json
从OpenAI导入OpenAI

# ====== 1. 配置 ======
API_KEY = "你的DeepSeek_API_Key"   # 换成你的 key
base_URL="https://api.deepseek.com/v1"
model="deepseek-v4-flash"

客户端=OpenAI(api_key=API_KEY，base_url=BASE_URL)
DB_FILE="memory.db"

# ====== 2. 记忆存储层（sqlite） ======
定义init_db()：
Conn=sqlite3.connect(DB_FILE)
conn.execute("""
如果不存在内存，则创建表(
ID整数主键自动增量，
            content TEXT NOT NULL,          -- 记忆内容
            user_id TEXT DEFAULT 'default', -- 支持多用户隔离
在TIMESTAMP时创建(_A)默认当前_时间戳
        )
    """)
conn.commit()
conn.close()

Def save_memorys(项目，user_id="default")：
    """批量写入新记忆（先查重，避免重复）"""
Conn=sqlite3.connect(DB_FILE)
现在我们可以看到一个新的标签.
    existing = {r[0]用于R in cur.execute(
"从内存中选择内容其中user_id=？"，(user_id，)}
在项目中：
如果它和它不存在：
cur.execute(
"插入存储器(内容，user_id)值(？，？)"，
                (it, user_id))
existing.add(it)
conn.commit()
conn.close()

Def load_memorys(用户ID="default")：
Conn=sqlite3.connect(DB_FILE)
rows=conn.execute(
"从存储器中选择内容，其中user_id=？order BY id DESC"，
(user_id，).fetchall()
conn.close()
返回[r[0]对于R，以行为单位]

Def delete_memory(删除项目，user_id="default")：
Conn=sqlite3.connect(DB_FILE)
时间:
在delete_items中：
cur.execute(
"从存储器中删除，其中内容=？和用户ID=？"，
            (it, user_id))
conn.commit()
conn.close()

#=====3.记忆提炼层(让V4-Flash自己判断该记什么）======
Def提取记忆(历史记录)(_M)：
    """把对话历史交给模型，返回 (该记住的, 该删除的)"""
sys_prompt=(
        "你是记忆管理器。从用户对话中提取值得长期记住的稳定事实："
"用户偏好、身份、习惯、长期目标等。忽略一次性的临时指令。\n"
        "只输出JSON，格式：{\"add\": [\"...\"]，“删除”：[\"...\"]}，无其他文字。"
    )
RESP=client.chat.completions.create(
模型=模型
其他[{"角色"："系统"，"内容"：sys_prompt}，
*历史记录],
response_format={"type"："json_object"}，
    )
没有任何的选择.[0]内容)
return data.get("add"，[])，data.get("remove"，[])

# ====== 4. 主对话函数（自动注入记忆） ======
Def聊天(user_input，user_id="default")：
#读取记忆，拼进系统提示
    历史记录=
    sys_prompt = "你是有帮助的助手。"
如果mems：
没有任何其他方法.[关于用户的长期记忆]\n"+"\n".join(f"-{m}"表示mems中的m)

    #调用V4闪存
    RESP=client.chat.completions.create(
    模型=模型
    messages=[{"角色"："系统"，"内容"：sys_prompt}，
    {"role"："user"，"content"：user_input}]，
    )
    答案=resp.choices[0].message.content
    返回应答，sys_prompt

五,用起来
if __name__ == "__主要的__":
    init_db()

    history = []   # 保留本轮对话，用于提炼记忆
    当为True时：
    user_input=input("你：").带()
    如果user_input.lower()在("exit"，"quit")中：
    打破

    应答，_=聊天(用户输入)
    打印("AI："，回答)

        # 把本轮对话交给模型提炼记忆，存入 sqlite
    history.append({"角色"："用户"，"内容"：user_input})
    history.append({"角色"："助理"，"内容"：答案})
        if len(history) >= 4:   # 每2轮对话提炼一次
    添加、删除=extract_memorys(历史记录)
    如果删除：
    删除内存(删除)(_M)
    如果添加：
    储存记忆(新增)(_M)
    历史记录=历史记录=历史记录=历史记录=历史记录=历史记录=历史记录=历史记录=历史记录=历史记录=历史记录=历史记录=历史记录=历史记录=历史记录=历史记录=历史记录=历史记录=[]
