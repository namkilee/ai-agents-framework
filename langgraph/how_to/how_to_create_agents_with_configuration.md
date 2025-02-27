<style>code { white-space: pre; overflow-x: auto; }</style>

[![LangGraph How to](https://img.shields.io/badge/LangGraph-How_to-yellow?logo=langgraph)](https://langchain-ai.github.io/langgraph/cloud/how-tos/configuration_cloud/)


# How to create agents with configuration

LangGraph API의 이점 중 하나는 다양한 설정으로 에이전트를 생성할 수 있다는 점입니다. 이는 다음과 같은 상황에서 유용합니다:
- LangGraph로 cognitive architecture를 한 번 정의한 후,
- 그 LangGraph가 일부 속성(예: 시스템 메시지 또는 사용할 LLM)을 설정할 수 있도록 하여,
- 사용자가 임의의 설정을 가진 에이전트를 생성하고, 이를 저장하여 미래에 사용할 수 있도록 합니다.

이 가이드에서는 기본적으로 내장하고 있는 에이전트를 수행하는 방법을 보일 것 입니다..

정의된 에이전트를 살펴보면, `call_model` 노드 내부에서 일부 설정을 기반으로 모델을 생성한 것을 알 수 있습니다. 그 노드는 다음과 같습니다:

```python
def call_model(state, config):
    messages = state["messages"]
    model_name = config.get('configurable', {}).get("model_name", "anthropic")
    model = _get_model(model_name)
    response = model.invoke(messages)
    # 이 결과는 기존 목록에 추가되기 때문에 목록을 반환합니다.
    return {"messages": [response]}
```

설정 내부에서 `model_name` 매개변수를 찾고 있으며(default 값으로 anthropic를 사용), 기본적으로 Anthropic 모델을 사용하고 있습니다. 이 예제에서는 OpenAI 모델로 설정하여 에이전트를 생성하도록 하겠습니다.

우선 클라이언트와 스레드를 설정해 봅시다:

```python
from langgraph_sdk import get_client

client = get_client(url=<DEPLOYMENT_URL>)
# 설정이 지정되지 않은 어시스턴트 선택
assistants = await client.assistants.search()
assistant = [a for a in assistants if not a["config"]][0]
```

이제 `.get_schemas`를 호출하여 이 그래프와 관련된 스키마를 가져올 수 있습니다:

```python
schemas = await client.assistants.get_schemas(
    assistant_id=assistant["assistant_id"]
)
# 여러 유형의 스키마가 있습니다.
# 설정 가능한 매개변수를 보기 위해 `config_schema`를 가져올 수 있습니다.
print(schemas["config_schema"])
```

출력:

```json
{
    'model_name': {
        'title': 'Model Name',
        'enum': ['anthropic', 'openai'],
        'type': 'string'
    }
}
```

이제 어시스턴트를 설정값으로 초기화할 수 있습니다:

```python
openai_assistant = await client.assistants.create(
    # "agent"는 배포한 그래프의 이름입니다.
    "agent", config={"configurable": {"model_name": "openai"}}
)

print(openai_assistant)
```

출력:
```json
{
    "assistant_id": "62e209ca-9154-432a-b9e9-2d75c7a9219b",
    "graph_id": "agent",
    "created_at": "2024-08-31T03:09:10.230718+00:00",
    "updated_at": "2024-08-31T03:09:10.230718+00:00",
    "config": {
        "configurable": {
            "model_name": "open_ai"
        }
    },
    "metadata": {}
}
```

설정이 실제로 적용되었는지 확인할 수 있습니다:

```python
thread = await client.threads.create()
input = {"messages": [{"role": "user", "content": "who made you?"}]}
async for event in client.runs.stream(
    thread["thread_id"],
    openai_assistant["assistant_id"],
    input=input,
    stream_mode="updates",
):
    print(f"Receiving event of type: {event.event}")
    print(event.data)
    print("\n\n")
```

출력:

```
Receiving event of type: metadata {'run_id': '1ef6746e-5893-67b1-978a-0f1cd4060e16'}

Receiving event of type: updates {'agent': {'messages': [{'content': 'I was created by OpenAI, a research organization focused on developing and advancing artificial intelligence technology.', 'additional_kwargs': {}, 'response_metadata': {'finish_reason': 'stop', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_157b3831f5'}, 'type': 'ai', 'name': None, 'id': 'run-e1a6b25c-8416-41f2-9981-f9cfe043f414', 'example': False, 'tool_calls': [], 'invalid_tool_calls': [], 'usage_metadata': None}]}}
```
