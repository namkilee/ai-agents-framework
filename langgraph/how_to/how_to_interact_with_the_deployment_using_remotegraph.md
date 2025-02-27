[![LangGraph How to](https://img.shields.io/badge/LangGraph-How_to-yellow?logo=langgraph)](https://langchain-ai.github.io/langgraph/how-tos/use-remote-graph/)


# How to interact with the deployment using RemoteGraph

`RemoteGraph`는 로컬에서 정의된 LangGraph 그래프(예: `CompiledGraph`)를 LangGraph Platform 과 상호작용할 수 있게 해주는 인터페이스입니다. 이 가이드는 `RemoteGraph`를 초기화하고 상호작용하는 방법을 보여줍니다.

<br>

## Initailizing the graph

`RemoteGraph`를 초기화할 때 항상 설정해야 하는 항목:
- `name`: 상호작용하려는 그래프의 이름입니다. 이는 `langgraph.json` 설정 파일에서 사용하는 그래프 이름과 동일합니다.
- `api_key`: 유효한 LangSmith API 키입니다. 환경 변수(`LANGSMITH_API_KEY`)를 설정하거나 `api_key` 인수를 통해 직접 전달할 수 있습니다. API 키는 `LangGraphClient` 또는 `SyncLangGraphClient`가 `api_key` 인수로 초기화된 경우 `client` / `sync_client` 인수를 통해서도 제공될 수 있습니다.

추가로 다음 중 하나를 제공해야 합니다:
- `url`: 상호작용하려는 URL입니다. `url` 인수가 전달되면 제공된 URL, 헤더(있는 경우) 및 기본 구성 값(예: 시간 초과 등)을 사용하는 동기 및 비동기 클라이언트가 모두 생성됩니다.
- `client`: 비동기적으로 상호작용하기 위한 `LangGraphClient` 인스턴스입니다(예: .astream(), .ainvoke(), .aget_state(), .aupdate_state() 등 사용).
- `sync_client`: 동기적으로 상호작용하기 위한 `SyncLangGraphClient` 인스턴스입니다(예: .stream(), .invoke(), .get_state(), .update_state() 등 사용).

> **Note**  
> `client` 또는 `sync_client`와 `url` 인수를 모두 전달하면 `client` 또는 `sync_client`가 `url` 인수보다 우선합니다. `client` / `sync_client` / `url` 중 하나도 제공되지 않으면 `RemoteGraph`는 런타임에 `ValueError`를 발생시킵니다.


### Using URL

```python
from langgraph.pregel.remote import RemoteGraph

url = <DEPLOYMENT_URL>
graph_name = "agent"
remote_graph = RemoteGraph(graph_name, url=url)
```


### Using clients

```python
from langgraph_sdk import get_client, get_sync_client
from langgraph.pregel.remote import RemoteGraph

url = <DEPLOYMENT_URL>
graph_name = "agent"
client = get_client(url=url)
sync_client = get_sync_client(url=url)
remote_graph = RemoteGraph(graph_name, client=client, sync_client=sync_client)
```

<br>

## Invoking the graph

`RemoteGraph`는 `Runnable`로, `CompiledGraph`와 동일한 메서드를 구현하므로 `.invoke()`, `.stream()`, `.get_state()`, `.update_state()` 등과 같이 `CompiledGraph`와 동일한 방식으로 상호작용할 수 있습니다(비동기 버전도 포함).


### Asynchronously

> **Note**  
> 그래프를 비동기적으로 사용하려면 `RemoteGraph`를 초기화할 때 `url` 또는 `client`를 제공해야 합니다.

```python
# 그래프 호출
result = await remote_graph.ainvoke({
    "messages": [{"role": "user", "content": "what's the weather in sf"}]
})

# 그래프에서 출력 스트리밍
async for chunk in remote_graph.astream({
    "messages": [{"role": "user", "content": "what's the weather in la"}]
}):
    print(chunk)
```


### Synchronously

> **Note**  
> 그래프를 동기적으로 사용하려면 `RemoteGraph`를 초기화할 때 `url` 또는 `sync_client`를 제공해야 합니다.

```python
# 그래프 호출
result = remote_graph.invoke({
    "messages": [{"role": "user", "content": "what's the weather in sf"}]
})

# 그래프에서 출력 스트리밍
for chunk in remote_graph.stream({
    "messages": [{"role": "user", "content": "what's the weather in la"}]
}):
    print(chunk)
```

<br>

## Thread-level persistence

기본적으로 그래프 실행(`.invoke()` 또는 `.stream()` 호출)은 stateless-the-checkpoint로 실행되며, 최종 상태가 저장되지 않습니다. 그래프의 출력을 저장하려면(예: human-in-the-loop 을 활성화) compiled graph와 동일하게 스레드를 생성하고 `config` 인수를 통해 스레드 ID를 제공해야합니다.


```python
from langgraph_sdk import get_sync_client
url = <DEPLOYMENT_URL>
graph_name = "agent"
sync_client = get_sync_client(url=url)
remote_graph = RemoteGraph(graph_name, url=url)

# 스레드 생성(또는 기존 스레드 사용)
thread = sync_client.threads.create()

# 스레드 구성으로 그래프 호출
config = {"configurable": {"thread_id": thread["thread_id"]}}
result = remote_graph.invoke({
    "messages": [{"role": "user", "content": "what's the weather in sf"}]
}, config=config)

# 상태가 스레드에 지속되는지 확인
thread_state = remote_graph.get_state(config)
print(thread_state)
```

<br>

## Using as a subgraph

> **Note**  
> `RemoteGraph` 서브 그래프 노드가 있는 그래프에서 `checkpointer`를 사용하려면 스레드 ID로 UUID를 사용해야 합니다.

`RemoteGraph`는 일반 `CompiledGraph`와 동일한 방식으로 동작하므로 다른 그래프의 서브 그래프로도 사용할 수 있습니다. 예를 들어:

```python
from langgraph_sdk import get_sync_client
from langgraph.graph import StateGraph, MessagesState, START
from typing import TypedDict

url = <DEPLOYMENT_URL>
graph_name = "agent"
remote_graph = RemoteGraph(graph_name, url=url)

# 부모 그래프 정의
builder = StateGraph(MessagesState)
# 서브 그래프로 RemoteGraph를 직접 노드로 추가
builder.add_node("child", remote_graph)
builder.add_edge(START, "child")
graph = builder.compile()

# 부모 그래프 호출
result = graph.invoke({
    "messages": [{"role": "user", "content": "what's the weather in sf"}]
})
print(result)

# 부모 그래프 및 서브 그래프에서 출력 스트리밍
for chunk in graph.stream({
    "messages": [{"role": "user", "content": "what's the weather in sf"}]
}, subgraphs=True):
    print(chunk)
```
