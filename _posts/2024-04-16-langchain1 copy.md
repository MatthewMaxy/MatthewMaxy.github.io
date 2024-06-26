---
layout: post
title:  "LangChain 学习笔记（一） "
date:   2024-04-16 23:00:00 +0800
tags:   LLM Project LangChain
color: rgb(226,129,98)
cover: '../assets/Blogs/LangChain/title.png'
subtitle: 'LangChain for LLM Application Development'
---
<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']],
            splayMath: [ ['$$','$$'], ["\\[","\\]"] ]
            }
        });
    </script>
</head>

## 简介


LangChain 是一个用于开发由语言模型驱动的应用程序的框架。它使得应用程序能够：

+ 具有上下文感知能力：将语言模型连接到上下文来源（提示指令，少量的示例，需要回应的内容等）
+ 具有推理能力：依赖语言模型进行推理（根据提供的上下文如何回答，采取什么行动等）

本文素材，代码参考：[Deeplearning.ai](https://learn.deeplearning.ai/courses/langchain/lesson/1/introduction)

## LangChain: Models, Prompts and Output Parsers

### 环境配置相关


```bash
    pip install python-dotenv
    pip install openai
    pip install langchain
```

```python
import os
import openai

from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv()) # read local .env file
openai.api_key = os.environ['OPENAI_API_KEY']

# API-KEY 载入也可以
# os.environ['OPENAI_API_KEY'] = '...'
```

### Model

```python
from langchain.chat_models import ChatOpenAI
chat = ChatOpenAI(temperature=0.0, model=llm_model)
'''
chat:
    ChatOpenAI(verbose=False, callbacks=None, callback_manager=None, 
    client=<class 'openai.api_resources.chat_completion.ChatCompletion'>,
    model_name='gpt-3.5-turbo-0301', temperature=0.0, model_kwargs={} 
    openai_api_key=None, openai_api_base=None, openai_organization=None, 
    request_timeout=None, max_retries=6, streaming=False, n=1, max_tokens=None)
'''
```

### Prompt template

**Template 模板**

```python
template_string = """Translate the text \
that is delimited by triple backticks \
into a style that is {style}. \
text: ```{text}```
"""

from langchain.prompts import ChatPromptTemplate

# langchain prompt模板构建
prompt_template = ChatPromptTemplate.from_template(template_string)

```

**基于模板的信息组织**

```python
customer_text = """
因为颅骨的损伤，也耽搁了大量时间，\
雷锋同志因工牺牲了。他生的伟大，死的光荣。\
在雷锋的追悼会上有上万人为雷锋送行，\
他是人们心中永远的英雄。
"""

customer_style = """
Chinese \
used in ancient China
"""

customer_messages = prompt_template.format_messages(
                    style=customer_style,
                    text=customer_email)
```

**使用模板后的Prompt**

```python
# Call the LLM to translate to the style of the customer message
customer_response = chat(customer_messages)
'''
print(customer_response.content)
    因為顱骨的損傷，也耽擱了大量時間，雷鋒同志因工牺牲了。
    他生的偉大，死的光榮。在雷鋒的追悼會上有上萬人為雷鋒送行，
    他是人們心中永遠的英雄。
'''
```

### Output Parsers

有时我们希望输出答案是结构化的文本而不是简单的 (str)，例如我们对评论进行提取，希望每条评论返回结果是一个 JSON 文件，格式如下：

```python
{
  "gift": False,
  "delivery_days": 5,
  "price_value": "pretty affordable!"
}
```

此时如果直接采用 2 中的 prompt 方法，返回的结果在形式上可能与需求很相似，但我们会发现他的数据类型还是 string

```python
''' output
response.content:
    {
        "gift": true,
        "delivery_days": 2,
        "price_value": ["It's slightly more expensive than
        the other leaf blowers out there, but I think 
        it's worth it for the extra features."]
    }
'''

''' output
type(response.content):
    str
'''
```

可以借用 langchain 的 output_parsers 让输出转化为 dict 数据类型

```python
from langchain.output_parsers import ResponseSchema
from langchain.output_parsers import StructuredOutputParser

gift_schema = ResponseSchema(name="gift",
                             description="Was the item purchased\
                             as a gift for someone else? \
                             Answer True if yes,\
                             False if not or unknown.")
delivery_days_schema = ResponseSchema(name="delivery_days",
                                      description="How many days\
                                      did it take for the product\
                                      to arrive? If this \
                                      information is not found,\
                                      output -1.")
price_value_schema = ResponseSchema(name="price_value",
                                    description="Extract any\
                                    sentences about the value or \
                                    price, and output them as a \
                                    comma separated Python list.")

response_schemas = [gift_schema, 
                    delivery_days_schema,
                    price_value_schema]

output_parser = StructuredOutputParser.from_response_schema(response_schemas) 
format_instructions = output_parser.get_format_instructions()
```

Langchain会创建一个非常精准的对格式的描述：

![format instruction](/assets/Blogs/LangChain/1.png){:height="70%" width="70%"}

再对 prompt 模板进行拼接，并进行 query

```python
review_template_2 = """\
For the following text, extract the following information:

gift: Was the item purchased as a gift for someone else? \
Answer True if yes, False if not or unknown.

delivery_days: How many days did it take for the product\
to arrive? If this information is not found, output -1.

price_value: Extract any sentences about the value or price,\
and output them as a comma separated Python list.

text: {text}

{format_instructions}
"""

prompt = ChatPromptTemplate.from_template(template=review_template_2)

messages = prompt.format_messages(text=customer_review, 
                                format_instructions=format_instructions)

response = chat(messages)

''' output
    ```json
    {
        "gift": true,
        "delivery_days": "2",
        "price_value": ["It's slightly more expensive than 
        the other leaf blowers out there, but I think 
        it's worth it for the extra features."]
    }
    ```
'''
```

最后对其进行解释即可得到 dict 类型数据：

```python
output_dict = output_parser.parse(response.content)

'''output
{'gift': True,
 'delivery_days': '2',
 'price_value': ["It's slightly more expensive than 
 the other leaf blowers out there, but I think 
  it's worth it for the extra features."]}
'''
```

## LangChain: Memory

LLMs本身不具备记忆能力，之所以能展现出记忆上下文的效果，本质是将问答历史记录也作为 Text 提交给 LLMs，但是由于文本逐渐变长，问答会逐渐变得昂贵，因此可以利用 LangChain 的 各种 Memory 模块对 context 进行加工与限制。

![Memory](/assets/Blogs/LangChain/3.png){:height="70%" width="70%"}

### ConversationBufferMemory

```python
from langchain.chat_models import ChatOpenAI
from langchain.chains import ConversationChain
from langchain.memory import ConversationBufferMemory
llm = ChatOpenAI(temperature=0.0, model=llm_model)
memory = ConversationBufferMemory()

# verbose：是否显示历史日志
conversation = ConversationChain(
    llm=llm,  
    memory = memory,
    verbose=True
)
# 前两个输出省略
conversation.predict(input="Hi, my name is Andrew")
conversation.predict(input="What is 1+1?")
conversation.predict(input="What is my name?")
```

最后输出日志如下：

![verbose](/assets/Blogs/LangChain/2.png){:height="70%" width="70%"}

同时可以直接打印 memory buffer 内容 以及对 Memory buffer 内容进行修改

```python

print(memory.buffer)
print(memory.load_memory_variables({}))

# 增加记忆内容
memory.save_context({"input": "Not much, just hanging"}, 
                    {"output": "Cool"})

```

### ConversationBufferWindowMemory

相比 Buffer Memory，只记忆窗口大小的 Memory

```python
from langchain.memory import ConversationBufferWindowMemory

# k : 记忆问答轮数
memory = ConversationBufferWindowMemory(k=1)     

memory.save_context({"input": "Hi"},
                    {"output": "What's up"})
memory.save_context({"input": "Not much, just hanging"},
                    {"output": "Cool"})

memory.load_memory_variables({})

'''output:
    {'history': 'Human: Not much, just hanging\nAI: Cool'}
'''

# 真实调用场景

llm = ChatOpenAI(temperature=0.0, model=llm_model)
memory = ConversationBufferWindowMemory(k=1)
conversation = ConversationChain(
    llm=llm, 
    memory = memory,
    verbose=False
)
conversation.predict(input="Hi, my name is Andrew")
conversation.predict(input="What is 1+1?")
conversation.predict(input="What is my name?")
''' Output
    "I'm sorry, I don't have access to that information.
    Could you please tell me your name?"
'''
```

### ConversationTokenBufferMemory

利用调用 LLM 的 token 计算模式去记忆有限 token 内容

```python
from langchain.memory import ConversationTokenBufferMemory
from langchain.llms import OpenAI

llm = ChatOpenAI(temperature=0.0, model=llm_model)
memory = ConversationTokenBufferMemory(llm=llm, max_token_limit=50)

memory.save_context({"input": "AI is what?!"},
                    {"output": "Amazing!"})
memory.save_context({"input": "Backpropagation is what?"},
                    {"output": "Beautiful!"})
memory.save_context({"input": "Chatbots are what?"}, 
                    {"output": "Charming!"})

memory.load_memory_variables({})
''' Output
    {'history': 'AI: Amazing!\nHuman: Backpropagation is what? \n
    AI: Beautiful!\nHuman: Chatbots are what?\nAI: Charming!'}
'''

```

### ConversationSummaryMemory

将历史对话从最近开始存储并与 limit token 进行比较，如果超出限制则会以总结的形式存储

```python
from langchain.memory import ConversationSummaryBufferMemory
schedule = "There is a meeting at 8am with your product team. \
You will need your powerpoint presentation prepared. \
9am-12pm have time to work on your LangChain \
project which will go quickly because Langchain is such a powerful tool. \
At Noon, lunch at the italian resturant with a customer who is driving \
from over an hour away to meet you to understand the latest in AI. \
Be sure to bring your laptop to show the latest LLM demo."

memory = ConversationSummaryBufferMemory(llm=llm, max_token_limit=1000)
memory.save_context({"input": "Hello"}, {"output": "What's up"})
memory.save_context({"input": "Not much, just hanging"},
                    {"output": "Cool"})
memory.save_context({"input": "What is on the schedule today?"}, 
                    {"output": f"{schedule}"})

print(memory.load_memory_variables({}))
```

（左图）上下文长度小于 token limit；（右图）上下文长度大于 token limit

![result](/assets/Blogs/LangChain/4.png)

## Chains in LangChain

### LLMChain

最基本的 Chain，也是很多种类 Chain 的基础 (Only LLM + prompt)

```python
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.chains import LLMChain

llm = ChatOpenAI(temperature=0.9, model=llm_model)
prompt = ChatPromptTemplate.from_template(
    "What is the best name to describe \
    a company that makes {product}?"
)

chain = LLMChain(llm=llm, prompt=prompt)

product = "Queen Size Sheet Set"
chain.run(product)
```

### SequentialChain

所谓 SequentialChain 即将多个 Chain 组合起来，其中每个 Chain 的输出可能成为另一个 Chain 的输入。具体而言可以分为 SimpleSequentialChain（单输入/输出） 和 SequentialChain（多输入/输出）

#### SimpleSequentialChain

**基本结构:**

![struct](/assets/Blogs/LangChain/6.png){:height="70%" width="70%"}

```python
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.chains import SimpleSequentialChain
llm = ChatOpenAI(temperature=0.9, model=llm_model)

# prompt template 1
first_prompt = ChatPromptTemplate.from_template(
    "What is the best name to describe \
    a company that makes {product}?"
)

# Chain 1
chain_one = LLMChain(llm=llm, prompt=first_prompt)

# prompt template 2
second_prompt = ChatPromptTemplate.from_template(
    "Write a 20 words description for the following \
    company:{company_name}"
)
# chain 2
chain_two = LLMChain(llm=llm, prompt=second_prompt)

overall_simple_chain = SimpleSequentialChain(chains=[chain_one, chain_two],
                                             verbose=True
                                            )

overall_simple_chain.run(product)
```

输出日志：

![result](/assets/Blogs/LangChain/5.png){:height="70%" width="70%"}

#### SequentialChain

**基本结构：**

![struct](/assets/Blogs/LangChain/7.png){:height="70%" width="70%"}

```python
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.chains import SequentialChain
from langchain.chains import LLMChain

llm = ChatOpenAI(temperature=0.9, model=llm_model)

# prompt template 1: translate to english
first_prompt = ChatPromptTemplate.from_template(
    "Translate the following review to english:"
    "\n\n{Review}"
)
# chain 1: input= Review and output= English_Review
chain_one = LLMChain(llm=llm, prompt=first_prompt, 
                     output_key="English_Review"
                    )

second_prompt = ChatPromptTemplate.from_template(
    "Can you summarize the following review in 1 sentence:"
    "\n\n{English_Review}"
)
# chain 2: input= English_Review and output= summary
chain_two = LLMChain(llm=llm, prompt=second_prompt, 
                     output_key="summary"
                    )

# prompt template 3: translate to english
third_prompt = ChatPromptTemplate.from_template(
    "What language is the following review:\n\n{Review}"
)
# chain 3: input= Review and output= language
chain_three = LLMChain(llm=llm, prompt=third_prompt,
                       output_key="language"
                      )

# prompt template 4: follow up message
fourth_prompt = ChatPromptTemplate.from_template(
    "Write a follow up response to the following "
    "summary in the specified language:"
    "\n\nSummary: {summary}\n\nLanguage: {language}"
)
# chain 4: input= summary, language and output= followup_message
chain_four = LLMChain(llm=llm, prompt=fourth_prompt,
                      output_key="followup_message"
                     )

# overall_chain: input= Review 
# and output= English_Review,summary, followup_message
overall_chain = SequentialChain(
    chains=[chain_one, chain_two, chain_three, chain_four],
    input_variables=["Review"],
    output_variables=["English_Review", "summary","followup_message"],
    verbose=True
)

# review string from database
review = '...' 
overall_chain(review)
```

![result](/assets/Blogs/LangChain/8.png){:height="70%" width="70%"}

### Router Chain

![struct](/assets/Blogs/LangChain/9.png){:height="70%" width="70%"}

```python
# prompt 设计
# 注意 ... 为节省篇幅
physics_template = """You are a very smart physics professor. You are ... \
Here is a question:{input}"""

math_template = """You are a very good mathematician. You are ... \
Here is a question:{input}"""

history_template = """You are a very good historian. You have ... \
Here is a question:{input}"""

computerscience_template = """ You are a grear computer scientist.You have ... \
Here is a question:{input}"""

prompt_infos = [{
        "name": "physics", 
        "description": "Good for answering questions about physics", 
        "prompt_template": physics_template
    },{
        "name": "math", 
        "description": "Good for answering math questions", 
        "prompt_template": math_template
    },{
        "name": "History", 
        "description": "Good for answering history questions", 
        "prompt_template": history_template
    },{
        "name": "computer science", 
        "description": "Good for answering computer science questions", 
        "prompt_template": computerscience_template
    }
]

# 正式建立 RouterChain
from langchain.chains.router import MultiPromptChain
from langchain.chains.router.llm_router import LLMRouterChain,RouterOutputParser
from langchain.prompts import PromptTemplate
from langchain.chat_models import ChatOpenAI
from langchain.chains import LLMChain

llm = ChatOpenAI(temperature=0, model=llm_model)

destination_chains = {}
for p_info in prompt_infos:
    name = p_info["name"]
    prompt_template = p_info["prompt_template"]
    prompt = ChatPromptTemplate.from_template(template=prompt_template)
    chain = LLMChain(llm=llm, prompt=prompt)
    destination_chains[name] = chain  
    
destinations = [f"{p['name']}: {p['description']}" for p in prompt_infos]
destinations_str = "\n".join(destinations)

# 当所有 subchain 未被调用时
default_prompt = ChatPromptTemplate.from_template("{input}")
default_chain = LLMChain(llm=llm, prompt=default_prompt)

MULTI_PROMPT_ROUTER_TEMPLATE = """Given a raw text input to a \
language model select the model prompt best suited for the input. \
You will be given the names of the available prompts and a \
description of what the prompt is best suited for. \
You may also revise the original input if you think that revising\
it will ultimately lead to a better response from the language model.

<< FORMATTING >>
Return a markdown code snippet with a JSON object formatted to look like:
    ```json
    {{{{
        "destination": string \ name of the prompt to use or "DEFAULT"
        "next_inputs": string \ a potentially modified version of the original input
    }}}}
    ```

REMEMBER: "destination" MUST be one of the candidate prompt \
names specified below OR it can be "DEFAULT" if the input is not\
well suited for any of the candidate prompts.
REMEMBER: "next_inputs" can just be the original input \
if you don't think any modifications are needed.

<< CANDIDATE PROMPTS >>
{destinations}

<< INPUT >>
{{input}}

<< OUTPUT (remember to include the ```json)>>"""

router_template = MULTI_PROMPT_ROUTER_TEMPLATE.format(
    destinations=destinations_str
)
router_prompt = PromptTemplate(
    template=router_template,
    input_variables=["input"],
    output_parser=RouterOutputParser(),
)

router_chain = LLMRouterChain.from_llm(llm, router_prompt)

chain = MultiPromptChain(router_chain=router_chain, 
                         destination_chains=destination_chains, 
                         default_chain=default_chain, verbose=True
                        )

chain.run("What is black body radiation?")
```

![result](/assets/Blogs/LangChain/10.png){:height="70%" width="70%"}