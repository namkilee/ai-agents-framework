[![LangGraph Conceptual](https://img.shields.io/badge/LangGraph-Conceptual-blue?logo=langgraph)](https://langchain-ai.github.io/langgraph/concepts/low_level/)


# LangGraph Glossary


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

### **슈퍼스텝(Super-step)**  
- 그래프 노드의 단일 반복(iteration)을 나타냅니다.  
- 병렬로 실행되는 노드는 동일한 슈퍼스텝에 속하며, 순차적으로 실행되는 노드는 별도의 슈퍼스텝에 속합니다.  
- 모든 노드는 초기에는 비활성 상태이며, 메시지를 수신하면 활성화되어 작업을 수행하고 응답을 반환합니다.  
- 모든 노드가 비활성화되고 전송 중인 메시지가 없으면 그래프 실행이 종료됩니다.


### **StateGraph**

`StateGraph` 클래스는 그래프를 정의하는 주요 클래스입니다. 사용자 정의 `State` 객체를 매개변수로 사용하여 그래프를 구성합니다.


**그래프 컴파일하기**

그래프를 빌드하려면 먼저 상태(state)를 정의하고, 그 다음 노드와 엣지를 추가한 후 컴파일합니다. 그렇다면 그래프를 컴파일하는 것이 무엇이며 왜 필요한 것일까요?

컴파일은 매우 간단한 단계입니다. 그래프의 구조에 대한 몇 가지 기본적인 검사를 수행합니다(고아 노드가 없는지 등). 또한 실행 시 인수(checkpointers 및 breakpoints와 같은)를 지정할 수 있는 곳이기도 합니다. 그래프를 컴파일하려면 단순히 `.compile` 메서드를 호출하면 됩니다:

```python
graph = graph_builder.compile(...)
```

그래프를 사용하기 전에 반드시 그래프를 컴파일해야 합니다.


## State

그래프를 정의할 때 처음 해야 할 일은 그래프의 상태(State)를 정의하는 것입니다. 상태는 그래프의 스키마와 상태에 대한 업데이트를 적용하는 방법을 지정하는 리듀서 함수로 구성됩니다. 상태의 스키마는 그래프의 모든 노드와 엣지의 입력 스키마가 되며, `TypedDict` 또는 Pydantic 모델일 수 있습니다. 모든 노드는 상태에 대한 업데이트를 발생시키며, 이 업데이트는 지정된 리듀서 함수를 사용하여 적용됩니다.

물론입니다! 다음은 요청하신 "Schema"와 "Multiple schemas" 섹션의 번역입니다:

---

**스키마**

그래프의 스키마를 지정하는 주된 문서화된 방법은 `TypedDict`를 사용하는 것입니다. 그러나 우리는 기본값과 추가 데이터 검증을 추가하기 위해 Pydantic `BaseModel`을 그래프 상태로 사용하는 것도 지원합니다.

기본적으로 그래프는 동일한 입력 및 출력 스키마를 가집니다. 이를 변경하려면 명시적인 입력 및 출력 스키마를 직접 지정할 수도 있습니다. 이는 많은 키가 있고 일부는 명시적으로 입력용이고 다른 일부는 출력용인 경우에 유용합니다. 사용 방법은 [이 노트북](#)을 참조하세요.

**여러 스키마**

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


## **리듀서**

리듀서는 노드에서 상태로 업데이트가 적용되는 방식을 이해하는 데 중요합니다. 상태의 각 키는 고유한 리듀서 함수를 가집니다. 명시적으로 리듀서 함수가 지정되지 않은 경우, 해당 키에 대한 모든 업데이트가 덮어쓰는 것으로 간주됩니다. 리듀서에는 몇 가지 유형이 있으며, 기본 리듀서 유형부터 시작합니다:

**기본 리듀서**

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

### 메시지와 함께 작업하기

**왜 메시지를 사용하나요?**

대부분의 최신 LLM 제공업체는 입력으로 메시지 목록을 수락하는 채팅 모델 인터페이스를 제공합니다. 특히 LangChain의 `ChatModel`은 입력으로 메시지 객체의 목록을 수락합니다. 이러한 메시지는 `HumanMessage`(사용자 입력)나 `AIMessage`(LLM 응답)와 같은 다양한 형태로 제공됩니다. 메시지 객체에 대한 자세한 내용은 [이 개념 가이드](https://langchain-ai.github.io/langgraph/concepts/low_level/#default-reducer)를 참조하세요.

**그래프에서 메시지 사용하기**

많은 경우, 이전 대화 기록을 그래프 상태에 메시지 목록으로 저장하는 것이 유용합니다. 이를 위해, 메시지 객체 목록을 저장하는 키(채널)를 그래프 상태에 추가하고 리듀서 함수로 주석을 달 수 있습니다(아래 예시의 `messages` 키 참조). 리듀서 함수는 상태 업데이트마다 상태에서 메시지 객체 목록을 업데이트하는 방법을 그래프에 알려주는 데 중요합니다. 리듀서를 지정하지 않으면, 모든 상태 업데이트는 마지막에 제공된 값으로 메시지 목록을 덮어씁니다. 기존 목록에 메시지를 단순히 추가하려면 `operator.add`를 리듀서로 사용할 수 있습니다.

그러나 그래프 상태에서 수동으로 메시지를 업데이트하고 싶을 수도 있습니다(예: 인간이 개입하는 경우). `operator.add`를 사용하면 수동 상태 업데이트가 기존 메시지 목록에 추가되기 때문에 기존 메시지를 업데이트하는 대신 새로운 메시지가 추가됩니다. 이를 피하기 위해, 메시지 ID를 추적하고 기존 메시지를 덮어쓸 수 있는 리듀서가 필요합니다. 이를 달성하기 위해, 미리 정의된 `add_messages` 함수를 사용할 수 있습니다. 새로운 메시지의 경우 기존 목록에 단순히 추가되지만, 기존 메시지에 대한 업데이트도 올바르게 처리합니다.

**직렬화**

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

