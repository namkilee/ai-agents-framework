<style>code { white-space: pre; overflow-x: auto; }</style>

[![LangGraph Conceptual](https://img.shields.io/badge/LangGraph-Conceptual-blue?logo=langgraph)](https://langchain-ai.github.io/langgraph/concepts/low_level/)


# LangGraph Glossary

<br>

## Graphs

LangGraph는 에이전트 워크플로우를 **그래프** 형태로 모델링합니다. 이를 정의하기 위해 세 가지 주요 구성 요소를 사용합니다:

1. **State(상태)**:  
   - 애플리케이션의 현재 스냅샷을 나타내는 공유 데이터 구조입니다.  
   - Python의 `TypedDict` 또는 Pydantic `BaseModel`로 정의할 수 있습니다.

2. **Nodes(노드)**:  
   - 에이전트의 로직을 캡슐화하는 Python 함수입니다.  
   - 현재 상태를 입력으로 받아 계산이나 부수 효과를 수행한 뒤, 업데이트된 상태를 반환합니다.

3. **Edges(엣지)**:  
   - 현재 상태를 기반으로 다음에 실행할 노드를 결정하는 Python 함수입니다.  
   - 조건부 분기나 고정된 전환을 정의할 수 있습니다.

**요약**:  
- *노드는 작업을 수행하고, 엣지는 다음에 무엇을 할지 결정합니다.*

LangGraph는 Google Pregel 시스템에서 영감을 받아 메시지 전달 방식을 사용하여 그래프 실행을 관리합니다. 각 노드는 작업 완료 후 메시지를 엣지를 통해 다음 노드로 전달하며, 이 과정이 반복됩니다. 실행은 **슈퍼스텝(Super-step)** 단위로 진행됩니다.

**슈퍼스텝(Super-step)**  
- 그래프 노드의 단일 반복(iteration)을 나타냅니다.  
- 병렬로 실행되는 노드는 동일한 슈퍼스텝에 속하며, 순차적으로 실행되는 노드는 별도의 슈퍼스텝에 속합니다.  
- 모든 노드는 초기에는 비활성 상태이며, 메시지를 수신하면 활성화되어 작업을 수행하고 응답을 반환합니다.  
- 모든 노드가 비활성화되고 전송 중인 메시지가 없으면 그래프 실행이 종료됩니다.


### StateGraph

`StateGraph` 클래스는 그래프를 정의하는 주요 클래스입니다. 사용자 정의 `State` 객체를 매개변수로 사용하여 그래프를 구성합니다.


### Compiling your graph

그래프를 빌드하려면 먼저 상태(state)를 정의하고, 그 다음 노드와 엣지를 추가한 후 컴파일합니다. 그렇다면 그래프를 컴파일하는 것이 무엇이며 왜 필요한 것일까요?

컴파일은 매우 간단한 단계입니다. 그래프의 구조에 대한 몇 가지 기본적인 검사를 수행합니다(고아 노드가 없는지 등). 또한 실행 시 인수(checkpointers 및 breakpoints와 같은)를 지정할 수 있는 곳이기도 합니다. 그래프를 컴파일하려면 단순히 `.compile` 메서드를 호출하면 됩니다:

```python
graph = graph_builder.compile(...)
```

그래프를 사용하기 전에 반드시 그래프를 컴파일해야 합니다.

<br>

## State

그래프를 정의할 때 처음 해야 할 일은 그래프의 상태(State)를 정의하는 것입니다. 상태는 그래프의 스키마와 상태에 대한 업데이트를 적용하는 방법을 지정하는 리듀서 함수로 구성됩니다. 상태의 스키마는 그래프의 모든 노드와 엣지의 입력 스키마가 되며, `TypedDict` 또는 Pydantic 모델일 수 있습니다. 모든 노드는 상태에 대한 업데이트를 발생시키며, 이 업데이트는 지정된 리듀서 함수를 사용하여 적용됩니다.


### Schema

그래프의 스키마를 지정하는 주된 문서화된 방법은 `TypedDict`를 사용하는 것입니다. 그러나 우리는 기본값과 추가 데이터 검증을 추가하기 위해 Pydantic `BaseModel`을 그래프 상태로 사용하는 것도 지원합니다.

기본적으로 그래프는 동일한 입력 및 출력 스키마를 가집니다. 이를 변경하려면 명시적인 입력 및 출력 스키마를 직접 지정할 수도 있습니다. 이는 많은 키가 있고 일부는 명시적으로 입력용이고 다른 일부는 출력용인 경우에 유용합니다. 사용 방법은 [이 노트북](#)을 참조하세요.

**Multiple schema**

일반적으로 모든 그래프 노드는 단일 스키마로 통신합니다. 이는 동일한 상태 채널을 읽고 쓴다는 것을 의미합니다. 그러나 더 많은 제어가 필요한 경우가 있습니다:

- 내부 노드는 그래프의 입력/출력에 필요하지 않은 정보를 전달할 수 있습니다.
- 그래프에 대해 다른 입력/출력 스키마를 사용하고 싶을 수 있습니다. 예를 들어, 출력에는 단일 관련 출력 키만 포함될 수 있습니다.

그래프 내부 노드 통신을 위해 노드가 개인 상태 채널에 기록할 수 있습니다. 단순히 `PrivateState`라는 개인 스키마를 정의할 수 있습니다. 자세한 내용은 [이 노트북](#)을 참조하세요.

그래프에 대한 명시적인 입력 및 출력 스키마를 정의하는 것도 가능합니다. 이러한 경우, 그래프 작업과 관련된 모든 키를 포함하는 "내부" 스키마를 정의합니다. 그러나 입력 및 출력 스키마를 "내부" 스키마의 하위 집합으로 정의하여 그래프의 입력 및 출력을 제한합니다. 자세한 내용은 [이 노트북](#)을 참조하세요.

다음 예시를 살펴봅시다:

```python
class InputState(TypedDict):
    user_input: str

class OutputState(TypedDict):
    graph_output: str

class OverallState(TypedDict):
    foo: str
    user_input: str
    graph_output: str

class PrivateState(TypedDict):
    bar: str

def node_1(state: InputState) -> OverallState:
    # Write to OverallState
    return {"foo": state["user_input"] + " name"}

def node_2(state: OverallState) -> PrivateState:
    # Read from OverallState, write to PrivateState
    return {"bar": state["foo"] + " is"}

def node_3(state: PrivateState) -> OutputState:
    # Read from PrivateState, write to OutputState
    return {"graph_output": state["bar"] + " Lance"}

builder = StateGraph(OverallState, input=InputState, output=OutputState)
builder.add_node("node_1", node_1)
builder.add_node("node_2", node_2)
builder.add_node("node_3", node_3)
builder.add_edge(START, "node_1")
builder.add_edge("node_1", "node_2")
builder.add_edge("node_2", "node_3")
builder.add_edge("node_3", END)

graph = builder.compile()
graph.invoke({"user_input": "My"})
# {'graph_output': 'My name is Lance'}
```

여기서 주의해야 할 두 가지 미묘하고 중요한 점이 있습니다:
1. 우리는 `node_1`에 입력 스키마로 `InputState`를 전달합니다. 그러나 `OverallState`의 채널인 `foo`에 기록합니다. 입력 스키마에 포함되지 않은 상태 채널에 어떻게 기록할 수 있을까요? 이는 노드가 그래프 상태의 모든 상태 채널에 기록할 수 있기 때문입니다. **그래프 상태(Graph state)는 초기화 시 정의된 상태 채널의 합집합**으로, 여기에는 `OverallState`, `InputState`, `OutputState`가 포함됩니다.

2. 우리는 `InputState`와 `OutputState`를 사용하여 그래프를 초기화합니다. 그렇다면 `node_2`에서 `PrivateState`에 어떻게 기록할 수 있을까요? 그래프가 초기화 시 전달되지 않은 스키마에 어떻게 접근할 수 있을까요? 이는 노드가 상태 스키마 정의가 존재하는 한 추가 상태 채널을 선언할 수 있기 때문입니다. 이 경우, `PrivateState` 스키마가 정의되어 있으므로 그래프에 새로운 상태 채널 `bar`를 추가하고 기록할 수 있습니다.


### Reducers

리듀서는 노드에서 상태로 업데이트가 적용되는 방식을 이해하는 데 중요합니다. 상태의 각 키는 고유한 리듀서 함수를 가집니다. 명시적으로 리듀서 함수가 지정되지 않은 경우, 해당 키에 대한 모든 업데이트가 덮어쓰는 것으로 간주됩니다. 리듀서에는 몇 가지 유형이 있으며, 기본 리듀서 유형부터 시작합니다:

**Default Reducer**

다음 두 예시는 기본 리듀서를 사용하는 방법을 보여줍니다:

예시 A:
```python
from typing_extensions import TypedDict

class State(TypedDict):
    foo: int
    bar: list[str]
```

이 예시에서는 어떤 키에도 리듀서 함수가 지정되지 않았습니다. 그래프의 입력이 `{"foo": 1, "bar": ["hi"]}`라고 가정해봅시다. 첫 번째 노드가 `{"foo": 2}`를 반환한다고 가정하면, 이는 상태에 대한 업데이트로 처리됩니다. 노드가 전체 상태 스키마를 반환할 필요는 없으며, 업데이트만 반환합니다. 이 업데이트를 적용한 후 상태는 `{"foo": 2, "bar": ["hi"]}`가 됩니다. 두 번째 노드가 `{"bar": ["bye"]}`를 반환하면 상태는 `{"foo": 2, "bar": ["bye"]}`가 됩니다.

예시 B:
```python
from typing import Annotated
from typing_extensions import TypedDict
from operator import add

class State(TypedDict):
    foo: int
    bar: Annotated[list[str], add]
```

이 예시에서는 Annotated 타입을 사용하여 두 번째 키(bar)에 대한 리듀서 함수(operator.add)를 지정했습니다. 첫 번째 키는 변경되지 않습니다. 그래프의 입력이 `{"foo": 1, "bar": ["hi"]}`라고 가정해봅시다. 첫 번째 노드가 `{"foo": 2}`를 반환한다고 가정하면, 이는 상태에 대한 업데이트로 처리됩니다. 노드가 전체 상태 스키마를 반환할 필요는 없으며, 업데이트만 반환합니다. 이 업데이트를 적용한 후 상태는 `{"foo": 2, "bar": ["hi"]}`가 됩니다. 두 번째 노드가 `{"bar": ["bye"]}`를 반환하면 상태는 `{"foo": 2, "bar": ["hi", "bye"]}`가 됩니다. 여기서 bar 키는 두 리스트를 더하여 업데이트됩니다.


### Working with Messages in Graph State

**Why use messages?**

대부분의 최신 LLM 제공업체는 입력으로 메시지 목록을 수락하는 채팅 모델 인터페이스를 제공합니다. 특히 LangChain의 `ChatModel`은 입력으로 메시지 객체의 목록을 수락합니다. 이러한 메시지는 `HumanMessage`(사용자 입력)나 `AIMessage`(LLM 응답)와 같은 다양한 형태로 제공됩니다. 메시지 객체에 대한 자세한 내용은 [이 개념 가이드](https://langchain-ai.github.io/langgraph/concepts/low_level/#default-reducer)를 참조하세요.

**Using Messages in your Graph**

많은 경우, 이전 대화 기록을 그래프 상태에 메시지 목록으로 저장하는 것이 유용합니다. 이를 위해, 메시지 객체 목록을 저장하는 키(채널)를 그래프 상태에 추가하고 리듀서 함수로 주석을 달 수 있습니다(아래 예시의 `messages` 키 참조). 리듀서 함수는 상태 업데이트마다 상태에서 메시지 객체 목록을 업데이트하는 방법을 그래프에 알려주는 데 중요합니다. 리듀서를 지정하지 않으면, 모든 상태 업데이트는 마지막에 제공된 값으로 메시지 목록을 덮어씁니다. 기존 목록에 메시지를 단순히 추가하려면 `operator.add`를 리듀서로 사용할 수 있습니다.

그러나 그래프 상태에서 수동으로 메시지를 업데이트하고 싶을 수도 있습니다(예: 사람이 개입하는 경우). `operator.add`를 사용하면 수동 상태 업데이트가 기존 메시지 목록에 추가되기 때문에 기존 메시지를 업데이트하는 대신 새로운 메시지가 추가됩니다. 이를 피하기 위해, 메시지 ID를 추적하고 기존 메시지를 덮어쓸 수 있는 리듀서가 필요합니다. 이를 달성하기 위해, 미리 정의된 `add_messages` 함수를 사용할 수 있습니다. 새로운 메시지의 경우 기존 목록에 단순히 추가되지만, 기존 메시지에 대한 업데이트도 올바르게 처리합니다.

**Serialization**

메시지 ID를 추적하는 것 외에도, `add_messages` 함수는 메시지 채널에서 상태 업데이트를 받을 때마다 메시지를 LangChain 메시지 객체로 역직렬화하려고 시도합니다. LangChain의 직렬화/역직렬화에 대한 자세한 내용은 [여기](https://langchain-ai.github.io/langgraph/concepts/low_level/#default-reducer)에서 확인할 수 있습니다. 이를 통해 그래프 입력 / 상태 업데이트를 다음과 같은 형식으로 보낼 수 있습니다:

```python
# 지원됨
{"messages": [HumanMessage(content="message")]}
# 또한 이 형식도 지원됨
{"messages": [{"type": "human", "content": "message"}]}
```

`add_messages`를 사용할 때 상태 업데이트는 항상 LangChain 메시지로 역직렬화되므로, 점 표기법을 사용하여 메시지 속성에 접근해야 합니다. 예를 들어, `state["messages"][-1].content`처럼 사용합니다.

아래는 `add_messages`를 리듀서 함수로 사용하는 그래프의 예시입니다:

```python
from langchain_core.messages import AnyMessage
from langgraph.graph.message import add_messages
from typing import Annotated
from typing_extensions import TypedDict

class GraphState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]
```

**MessagesState**

상태에 메시지 목록을 두는 것이 매우 일반적이기 때문에, 메시지를 쉽게 사용할 수 있도록 하는 사전 정의된 상태인 `MessagesState`가 존재합니다. `MessagesState`는 `AnyMessage` 객체 목록인 단일 `messages` 키로 정의되며 `add_messages` 리듀서를 사용합니다. 일반적으로 추적해야 할 상태는 메시지 외에도 더 많기 때문에, 사람들이 이 상태를 서브클래싱하여 더 많은 필드를 추가하는 경우가 있습니다:

```python
from langgraph.graph import MessagesState

class State(MessagesState):
    documents: list[str]
```

<br>

## Nodes

LangGraph에서 노드는 일반적으로 Python 함수(동기 또는 비동기)이며, 첫 번째 매개변수는 [state](#state)이고, 두 번째 매개변수(optional)는 "config"로, `thread_id`와 같은 선택적 [configurable parameter](#configuration)를 포함합니다.

`NetworkX`와 유사하게, `add_node` 메서드를 사용하여 노드를 그래프에 추가할 수 있습니다:

```python
from langchain_core.runnables import RunnableConfig
from langgraph.graph import StateGraph

builder = StateGraph(dict)

def my_node(state: dict, config: RunnableConfig):
    print("In node: ", config["configurable"]["user_id"])
    return {"results": f"Hello, {state['input']}!"}

# 두 번째 인수는 선택 사항입니다
def my_other_node(state: dict):
    return state

builder.add_node("my_node", my_node)
builder.add_node("other_node", my_other_node)
```
API Reference: [RunnableConfig](https://python.langchain.com/api_reference/core/runnables/langchain_core.runnables.config.RunnableConfig.html) | [StateGraph](https://langchain-ai.github.io/langgraph/reference/graphs/#langgraph.graph.state.StateGraph)

백그라운드에서는 함수가 RunnableLambda로 변환되며, 이를 통해 함수에 배치 처리 및 비동기 지원이 추가되고, 네이티브 추적(tracing) 및 디버깅 기능도 제공됩니다.

그래프에 노드를 추가할 때 이름을 지정하지 않으면, 해당 노드는 함수 이름과 동일한 기본 이름이 부여됩니다.
```python
builder.add_node(my_node)
# 이후 `"my_node"`를 사용하여 해당 노드와 연결된 엣지를 생성할 수 있습니다.
```


### `START` Node

`START` 노드는 사용자 입력을 그래프에 보내는 특수 노드입니다. 이 노드를 참조하는 주요 목적은 어떤 노드가 가장 먼저 호출되어야 하는지를 결정하는 데 있습니다.

```python
from langgraph.graph import START

graph.add_edge(START, "node_a")
```
API Reference: [START](https://langchain-ai.github.io/langgraph/reference/constants/#langgraph.constants.START)


### `END` Node

`END` 노드는 종료 노드를 나타내는 특수 노드입니다. 이 노드는 특정 엣지가 완료된 후 더 이상 수행할 작업이 없음을 표시하고자 할 때 참조됩니다.

```python
from langgraph.graph import END

graph.add_edge("node_a", END)
```

<br>

## Edges

엣지는 로직이 어떻게 라우팅되고 그래프가 언제 중지될지를 결정하는 요소를 정의합니다. 이는 에이전트가 작동하는 방식과 다양한 노드간 통신하는 방식을 크게 좌우하는 중요한 요소입니다. 몇 가지 주요 엣지 유형이 있습니다:

- Normal Edges: 하나의 노드에서 다음 노드로 직접 이동합니다.
- Conditional Edges: 다음으로 갈 노드(들)를 결정하기 위해 함수를 호출합니다.
- Entry Point: 사용자 입력이 도착했을 때 가장 먼저 호출할 노드를 지정합니다.
- Conditional Entry Point: 사용자 입력이 도착했을 때 어떤 노드(들)를 먼저 호출할지를 결정하기 위해 함수를 호출합니다.

하나의 노드는 여러 개의 출력 엣지를 가질 수 있습니다. 만약 하나의 노드에 여러 개의 출력 엣지가 있다면, 그 노드들은 모두 다음 슈퍼 스텝의 일환으로 병렬로 실행됩니다.


### Normal Edges

항상 노드 A에서 노드 B로 이동하고 싶다면, add_edge 메서드를 사용할 수 있습니다.

```python
graph.add_edge("node_a", "node_b")
```


### Conditional Edges

1개 이상의 엣지로 선택적으로 라우팅하거나, 종료하고 싶다면 `add_conditional_edges` 메서드를 사용할 수 있습니다. 이 메서드는 노드의 이름과 해당 노드가 실행된 후 호출할 "라우팅 함수(routing function)"를 받습니다.

```python
graph.add_conditional_edges("node_a", routing_function)
```

노드와 마찬가지로, `routing_function`은 그래프의 현재 상태를 입력으로 받아 값을 반환합니다.

기본적으로, `routing_function`의 반환값은 상태를 전달할 다음 노드(또는 노드 리스트)의 이름입니다. 반환된 모든 노드는 다음 슈퍼스텝(superstep)의 일부로 병렬 실행됩니다.

라우팅 함수의 출력값을 다음 노드의 이름에 매핑하는 딕셔너리를 선택적으로 제공할 수도 있습니다.

```python
graph.add_conditional_edges("node_a", routing_function, {True: "node_b", False: "node_c"})
```

> **Tip**  
> 상태 업데이트와 라우팅을 단일 함수로 결합하려면 조건부 엣지 대신 [`Command`](#command)를 사용하세요.


### Entry Point

진입점은 그래프가 시작될 때 처음 실행되는 노드(들)입니다. 가상의 `START` 노드에서 첫 번째로 실행할 노드로의 `add_edge` 메서드를 사용하여 그래프의 진입 지점을 지정할 수 있습니다.

```python
from langgraph.graph import START

graph.add_edge(START, "node_a")
```


### Conditional Entry Point

조건부 진입점은 사용자 정의 로직에 따라 다른 노드에서 시작할 수 있게 합니다. 이를 위해 가상의 `START` 노드에서 `add_conditional_edges`를 사용할 수 있습니다.

```python
from langgraph.graph import START

graph.add_conditional_edges(START, routing_function)
```

라우팅 함수의 출력값을 다음 노드의 이름에 매핑하는 딕셔너리를 선택적으로 제공할 수도 있습니다.


```python
graph.add_conditional_edges(START, routing_function, {True: "node_b", False: "node_c"})
```

<br>

## `Send`

기본적으로, 노드와 엣지는 미리 정의되어 잇고 같은 state를 가지고 작동합니다. 그러나 경우에 따라 정확한 엣지가 사전에 알려지지 않거나, 동시에 여러 버전의 State가 존재해야 할 수 있습니다. 이러한 상황의 일반적인 예로 맵-리듀스(`map-reduce`) 디자인 패턴이 있습니다. 이 패턴에서는 첫 번째 노드가 객체 목록을 생성하고, 생성된 각 객체에 대해 다른 노드를 적용하고자 할 수 있습니다. 이때 객체의 개수(즉, 엣지의 수)는 사전에 알 수 없으며, 하위 노드로 전달되는 입력 상태는 각 객체마다 달라야 합니다.

이 디자인 패턴을 지원하기 위해, LangGraph는 조건부 엣지에서 [`Send`](https://langchain-ai.github.io/langgraph/reference/types/#langgraph.types.Send) 객체를 반환하는 기능을 제공합니다. Send는 두 개의 인수를 받습니다: 첫 번째는 노드의 이름이고, 두 번째는 그 노드에 전달할 state입니다.

```python
...
class OverallState(TypedDict):
    subjects: list[str]
    jokes: Annotated[list[str], operator.add]

...

def continue_to_jokes(state: OverallState):
    return [Send("generate_joke", {"subject": s}) for s in state['subjects']]

builder = StateGraph(OverallState)
builder.add_node("generate_joke", lambda state: {"jokes": [f"Joke about {state['subject']}"]})
builder.add_conditional_edges(START, continue_to_jokes)
builder.add_edge("generate_joke", END)
graph = builder.compile()
# Invoking with two subjects results in a generated joke for each
graph.invoke({"subjects": ["cats", "dogs"]})
# {'subjects': ['cats', 'dogs'], 'jokes': ['Joke about cats', 'Joke about dogs']}
```

<br>

## `Command`

LangGraph에서는 제어 흐름(엣지)과 상태 업데이트(노드)를 결합하는 것이 유용할 수 있습니다. 예를 들어, 동일한 노드에서 상태를 업데이트하면서 동시에 다음으로 이동할 노드를 결정하고 싶을 때 [`Command`](https://langchain-ai.github.io/langgraph/reference/types/#langgraph.types.Command) 객체를 반환하여 이를 구현할 수 있습니다.

```python
def my_node(state: State) -> Command[Literal["my_other_node"]]:
    return Command(
        # 상태 업데이트
        update={"foo": "bar"},
        # 제어 흐름
        goto="my_other_node"
    )
```

`Command`를 사용하면 [conditional edges](#conditional-edges)와 동일하게 동적 제어 흐름을 구현할 수도 있습니다.

```python
def my_node(state: State) -> Command[Literal["my_other_node"]]:
    if state["foo"] == "bar":
        return Command(update={"foo": "baz"}, goto="my_other_node")
```

> **Important**  
> 노드 함수에서 `Command`를 반환할 때는 `Command[Literal["my_other_node"]]`와 같은 리턴 타입 주석을 추가해야 합니다. 이는 그래프 렌더링을 위해 필요하며 `my_node`가 `my_other_node`로 이동할 수 있음을 LangGraph에 알려줍니다. 

Command 사용 방법에 대한 종합적인 예시는 [이 가이드](../how_to/how_to_combine_control_flow_and_state_updates_with_command.md)를 참조하세요.


### When should I use Command instead of conditional edges?

그래프 상태를 업데이트하면서 다른 노드로 라우팅할 필요가 있을 때 Command를 사용하면 좋습니다. [multi-agent handoffs](./multi_agent_systems.md#handoffs)를 구현할 때 다른 에이전트로 라우팅하면서 해당 에이전트에 정보를 전달하는 것이 중요한 경우가 예시입니다.

상태를 업데이트하지 않고 노드 간에 조건부로 라우팅하려면 조건부 엣지를 사용해야 합니다.


### Navigating to node in a parent graph

[서브그래프](#subgraphs)를 사용하는 경우 서브그래프 내의 노드에서 다른 서브그래프(즉, 상위 그래프의 다른 노드)로 이동하고 싶을 수 있습니다. 이를 위해 Command에서 graph=Command.PARENT를 지정할 수 있습니다:

```python
def my_node(state: State) -> Command[Literal["my_other_node"]]:
    return Command(
        update={"foo": "bar"},
        goto="other_subgraph",  # 여기서 `other_subgraph`는 상위 그래프의 노드입니다
        graph=Command.PARENT
    )
```

> **Note**  
> `graph`를 `Command.PARENT`로 설정하면 가장 가까운 상위 그래프로 이동합니다.

> **Start updates with `Command.PARENT`**  
> 서브그래프 노드에서 상위 그래프 노드로 상태 업데이트를 보낼 때, 서브그래프와 상위 그래프 상태 스키마에 공유 키가 있다면, 상위 그래프 상태에서 해당 키에 대한 리듀서를 정의해야 합니다. 자세한 예시는 [이 가이드](../how_to/how_to_combine_control_flow_and_state_updates_with_command.md#navigating-to-a-node-in-a-parent-graph)를 참조하세요.

이 기능은 특히 [multi-agent handoffs](./multi_agent_systems.md#handoffs)를 구현할 때 유용합니다.


### Using inside tools

도구 내부에서 그래프 상태를 업데이트하는 것은 일반적인 사례입니다. 예를 들어, 고객 지원 애플리케이션에서는 대화 시작 시 고객의 계정 번호나 ID를 기반으로 고객 정보를 조회하고자 할 수 있습니다. 이 경우 도구에서 그래프 상태를 업데이트하려면 도구에서 `Command(update={"my_custom_key": "foo", "messages": [...]})`를 반환할 수 있습니다:

```python
@tool
def lookup_user_info(tool_call_id: Annotated[str, InjectedToolCallId], config: RunnableConfig):
    """사용자의 질문을 더 잘 도와주기 위해 사용자 정보를 조회하는 데 사용합니다."""
    user_info = get_user_info(config.get("configurable", {}).get("user_id"))
    return Command(
        update={
            # 상태 키 업데이트
            "user_info": user_info,
            # 메시지 기록 업데이트
            "messages": [ToolMessage("사용자 정보를 성공적으로 조회했습니다", tool_call_id=tool_call_id)]
        }
    )
```

> **Important**  
> 도구에서 Command를 반환할 때는 메시지 기록에 사용하는 상태 키(예: messages)를 반드시 포함해야 하며, 메시지 리스트에는 반드시 ToolMessage가 포함되어야 합니다. 이는 메시지 기록이 유효하도록 하기 위함입니다(LLM 제공자는 도구 호출 후 AI 메시지에 도구 결과 메시지가 이어져야 합니다).

`Command`를 통해 상태를 업데이트하는 도구를 사용하는 경우, LangGraph에서 제공하는 미리 제작된 ToolNode를 사용하는 것이 좋습니다. 이는 도구가 반환한 Command 객체를 자동으로 처리하고 그래프 상태에 반영합니다. 만약 사용자 정의 노드에서 도구를 호출한다면, 도구가 반환한 Command 객체를 수동으로 처리하여 업데이트해야 합니다.


### Human-in-the-loop

`Command`는 human-in-the-loop 워크플로우에서 중요한 역할을 합니다. `interrupt()`를 사용하여 사용자 입력을 받을 때, `Command`는 입력을 제공하고 `Command(resume="User input")`를 통해 실행을 재개하는 데 사용됩니다. 자세한 내용은 [이 가이드](./human_in_the_loop.md)를 확인하세요.

<br>

## Persistence

LangGraph는 [체크포인터](https://langchain-ai.github.io/langgraph/reference/checkpoints/#langgraph.checkpoint.base.BaseCheckpointSaver)를 사용하여 에이전트의 상태에 대한 내장된 영속성을 제공합니다. 체크포인터는 그래프 상태의 스냅샷을 각 슈퍼스텝마다 저장하여 언제든지 재개할 수 있도록 합니다. 이를 통해 human-in-the-loop 상호작용, 메모리 관리, 내결함성과 같은 기능이 가능해집니다. 그래프 실행 후에도 적절한 get 및 update 메서드를 사용하여 그래프 상태를 직접 조작할 수 있습니다. 자세한 내용은 [이 가이드](./persistence.md)를 참조하세요.

<br>

## Threads

LangGraph에서 스레드는 그래프와 사용자 간의 개별 세션 또는 대화를 나타냅니다. 체크포인팅을 사용할 때, 단일 대화의 턴(심지어 단일 그래프 실행 내의 단계)도 고유한 스레드 ID로 정리됩니다.

<br>

## Storage

LangGraph는 BaseStore 인터페이스를 통해 내장된 문서 저장소를 제공합니다. 체크포인터가 스레드 ID에 따라 상태를 저장하는 것과 달리, 저장소는 데이터를 조직하기 위해 사용자 정의 네임스페이스를 사용합니다. 이를 통해 크로스 스레드 영속성이 가능해져 에이전트가 장기적인 기억을 유지하고, 과거 상호작용에서 학습하며, 시간이 지남에 따라 지식을 축적할 수 있습니다. 일반적인 사용 사례로는 사용자 프로필 저장, 지식 기반 구축 및 모든 스레드에서의 전역 설정 관리 등이 포함됩니다.

<br>

## Graph Migrations

이해가 안되서 패스~

<br>

## Configuration

그래프를 생성할 때 특정 부분을 설정 가능하게(configurable) 표시할 수도 있습니다. 이는 모델이나 시스템 프롬프트를 쉽게 변경할 수 있게 합니다. 이를 통해 단일 "cognitive architecutre"(그래프)를 생성하면서도 여러 인스턴스를 가질 수 있습니다. 

그래프를 생성할 때 `config_schema`를 선택적으로 지정할 수 있습니다.

```python
class ConfigSchema(TypedDict):
    llm: str

graph = StateGraph(State, config_schema=ConfigSchema)
```

그런 다음 설정 가능(configurable) 필드를 사용하여 설정을 그래프에 전달할 수 있습니다.

```python
config = {"configurable": {"llm": "anthropic"}}
graph.invoke(inputs, config=config)
```

그런 다음 노드 내부에서 이 설정을 액세스하고 사용할 수 있습니다:

```python
def node_a(state, config):
    llm_type = config.get("configurable", {}).get("llm", "openai")
    llm = get_llm(llm_type)
    ...
```

configuration에 대한 전체 설명은 [이 가이드](../how_to/how_to_add_runtime_configuration_to_your_graph.md)를 참조하세요.


### Recursion Limit

Recursion limit은 그래프가 단일 실행 중에 실행할 수 있는 최대 [슈퍼 스텝(super-step)](#graphs) 수를 설정합니다. 이 한도에 도달하면 LangGraph는 `GraphRecursionError`를 발생시킵니다. 기본적으로 이 값은 25 스텝으로 설정되어 있습니다. Recusion limit은 실행 시 어떤 그래프에서든 설정할 수 있으며, config 딕셔너리를 통해 `.invoke/.stream`에 전달됩니다. 중요한 점은, recursion_limit은 독립적인 설정 키이며, 다른 사용자 정의 구성과 같이 configurable 키 내부에 포함되어서는 안 됩니다. 아래 예제를 참고하세요:

```python
graph.invoke(inputs, config={"recursion_limit": 5, "configurable":{"llm": "anthropic"}})
```

Recursion limit이 어떻게 작동하는지 더 자세히 알아보려면 [이 가이드](../how_to/how_to_create_and_control_loops.md)를 참고하세요.

<br>

## `Interrupt`

특정 지점에서 그래프를 일시 중지하고 사용자 입력을 받으려면 [interrupt](https://langchain-ai.github.io/langgraph/reference/types/#langgraph.types.interrupt) 함수를 사용하세요.  
`interrupt` 함수는 클라이언트에 인터럽트 정보를 노출하여, 사용자 입력을 받거나 개발자가 그래프 상태를 검증하거나 실행을 재개하기 전에 결정을 내릴 수 있도록 합니다.

```python
from langgraph.types import interrupt

def human_approval_node(state: State):
    ...
    answer = interrupt(
        # 이 값은 클라이언트로 전송됩니다.
        # JSON으로 직렬화할 수 있는 값이면 무엇이든 가능합니다.
        {"question": "is it ok to continue?"},
    )
    ...
```
API Reference: interrupt

그래프를 다시 실행하려면, `Command` 객체를 그래프에 전달하면서 `resume` 키를 `interrupt` 함수가 반환한 값으로 설정하면 됩니다.

`interrupt`가 human-in-the-loop workflows 에서 어떻게 사용되는지에 대한 자세한 내용은 [이 가이드](./human_in_the_loop.md)를 참고하세요.

<br>

## Breakpoints

중단점은 특정 지점에서 그래프 실행을 일시 중지하고 단계별로 실행을 진행할 수 있게 합니다. 중단점은 LangGraph의 [영속성 계층](./persistence.md)에 의해 지원되며, 각 그래프 단계 후 상태를 저장합니다. 중단점은 [human-in-the-loop](./human_in_the_loop.md) 워크플로우를 가능하게 할 수도 있지만, 이 목적을 위해서는 [`interrupt` 함수](#persistence)를 사용하는 것이 좋습니다.

중단점에 대한 자세한 내용은 [이 가이드](./breakpoints.md)를 참조하세요.

<br>

## Subgraphs

서브그래프는 다른 그래프에서 [노드](#nodes)로 사용되는 [그래프](#graphs)입니다. 이는 캡슐화의 오래된 개념과 동일합니다. 서브그래프를 사용하는 이유는 다음과 같습니다:

- 다중 에이전트 시스템 구축
- 여러 그래프에서 상태를 공유할 수 있는 노드를 재사용하고자 할 때, 서브그래프에서 한 번 정의한 후 여러 상위 그래프에서 사용할 수 있습니다.
- 그래프의 서로 다른 부분을 독립적으로 작업하고자 하는 팀이 있을 때, 각 부분을 서브그래프로 정의하고 서브그래프 인터페이스(입력 및 출력 스키마)가 준수된다면, 세부 사항을 서로 알 필요 없이 상위 그래프를 구성할 수 있습니다.

서브그래프를 상위 그래프에 추가하는 방법은 두 가지가 있습니다:

* **컴파일된 서브그래프**를 사용하여 노드 추가: 상위 그래프와 서브그래프가 상태 키를 공유하고 들어오거나 나가는 상태를 변환할 필요가 없는 경우에 유용합니다.

```python
builder.add_node("subgraph", subgraph_builder.compile())
```

* **서브그래프를 호출하는 함수**를 사용하여 노드 추가: 상위 그래프와 서브그래프가 서로 다른 상태 스키마를 가지고 있고, 서브그래프 호출 전후에 상태를 변환해야 하는 경우에 유용합니다.

```python
subgraph = subgraph_builder.compile()

def call_subgraph(state: State):
    return subgraph.invoke({"subgraph_key": state["parent_key"]})

builder.add_node("subgraph", call_subgraph)
```

각 방법의 예를 살펴보겠습니다.


###  As a compiled graph
서브그래프 노드를 생성하는 가장 간단한 방법은 [컴파일된 서브그래프](#compiling-your-graph)를 사용하는 것입니다. 이 경우 상위 그래프와 서브그래프 상태 스키마가 통신(communicate) 할 수 있는 최소한 하나의 [키](#state)를 서로 가지고 있는 것이 중요합니다. 그래프와 서브그래프가 어떤 키도 공유하지 않는다면 [서브그래프를 호출하는 함수](#as-a-function)를 사용해야 합니다.

> **Note**  
> 서브그래프 노드에 추가 키(공유 키 외)를 전달하면 서브그래프 노드에서 무시됩니다. 마찬가지로 서브그래프에서 추가 키를 반환하면 상위 그래프에서 무시됩니다.

```python
from langgraph.graph import StateGraph
from typing import TypedDict

class State(TypedDict):
    foo: str

class SubgraphState(TypedDict):
    foo: str  # 상위 그래프 상태와 공유되는 이 키에 주목하세요.
    bar: str

# 서브그래프 정의
def subgraph_node(state: SubgraphState):
    # 이 서브그래프 노드는 공유된 "foo" 키를 통해 상위 그래프와 통신할 수 있습니다.
    return {"foo": state["foo"] + "bar"}

subgraph_builder = StateGraph(SubgraphState)
subgraph_builder.add_node(subgraph_node)
...
subgraph = subgraph_builder.compile()

# 상위 그래프 정의
builder = StateGraph(State)
builder.add_node("subgraph", subgraph)
...
graph = builder.compile()
```

API 참조: [StateGraph](https://langchain-ai.github.io/langgraph/reference/graphs/#langgraph.graph.state.StateGraph)


### As a function

상위 그래프와 완전히 다른 스키마를 사용하는 서브그래프를 정의 할 수 있습니다. 이 경우 서브그래프를 호출하는 노드 함수를 사용해야 합니다. 이 노드(함수)에서는 서브그래프를 호출하기 전에 입력(상위) 상태를 서브그래프 상태로 [변환](../how_to/how_to_transform_inputs_and_outputs_of_a_subgraph.md)하고, 서브그래프 호출 후 결과를 상위 상태로 변환하여 상태 업데이트를 반환해야 합니다.

```python
class State(TypedDict):
    foo: str

class SubgraphState(TypedDict):
    # 이 키들은 상위 그래프 상태와 공유되지 않음을 주목하세요.
    bar: str
    baz: str

# 서브그래프 정의
def subgraph_node(state: SubgraphState):
    return {"bar": state["bar"] + "baz"}

subgraph_builder = StateGraph(SubgraphState)
subgraph_builder.add_node(subgraph_node)
...
subgraph = subgraph_builder.compile()

# 상위 그래프 정의
def node(state: State):
    # 상위 상태를 서브그래프 상태로 변환
    response = subgraph.invoke({"bar": state["foo"]})
    # 응답을 상위 상태로 변환
    return {"foo": response["bar"]}

builder = StateGraph(State)
# 컴파일된 서브그래프 대신 `node` 함수를 사용함에 유의하세요.
builder.add_node(node)
...
graph = builder.compile()
```

<br>

## Visualization

그래프가 더 복잡해질수록 시각화 기능은 유용합니다. LangGraph는 그래프를 시각화할 수 있는 여러가지  방법을 제공합니다. 자세한 내용은 [이 가이드](../how_to/how_to_visualize_your_graph.md)를 참조하십시오.

<br>

## Streaming
LangGraph는 실행 중인 그래프 노드의 업데이트나 LLM 호출 시 토큰 등을 포함한 스트리밍을 최우선으로 지원합니다. 자세한 내용은 [이 가이드](./streaming.md)를 참조하십시오.

