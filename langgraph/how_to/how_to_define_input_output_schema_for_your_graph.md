[![LangGraph How to](https://img.shields.io/badge/LangGraph-How_to-yellow?logo=langgraph)](https://langchain-ai.github.io/langgraph/how-tos/input_output_schema/)


# How to define input/output schema for your graph

기본적으로 `StateGraph`는 단일 스키마를 가지며, 모든 노드는 해당 스키마를 사용하여 통신합니다. 그러나 그래프를 위해 별도의 입력 및 출력 스키마를 정의하는 것도 가능합니다.

별도의 스키마가 지정되더라도 노드 간에는 내부 스키마를 여전히 사용됩니다. 입력 스키마는 입력이 예상되는 구조와 일치하며, 출력 스키마는 정의된 출력 스키마에 따라 내부 데이터를 필터링하여 관련 정보만 반환합니다.

이 예제에서는 입력 및 출력 스키마를 정의하는 방법을 살펴보겠습니다.

<br>

## Setup
먼저 필요한 패키지를 설치합니다:
```python
%%capture --no-stderr
%pip install -U langgraph
```

<br>

## Define and use the graph

```python
from langgraph.graph import StateGraph, START, END
from typing_extensions import TypedDict

# 입력 스키마 정의
class InputState(TypedDict):
    question: str

# 출력 스키마 정의
class OutputState(TypedDict):
    answer: str

# 입력 및 출력을 모두 포함하는 전체 스키마 정의
class OverallState(InputState, OutputState):
    pass

# 입력을 처리하고 답변을 생성하는 노드 정의
def answer_node(state: InputState):
    return {"answer": "bye", "question": state["question"]}

# 입력 및 출력 스키마를 지정하여 그래프 빌드
builder = StateGraph(OverallState, input=InputState, output=OutputState)
builder.add_node(answer_node)
builder.add_edge(START, "answer_node")
builder.add_edge("answer_node", END)
graph = builder.compile()

# 입력과 함께 그래프 호출 및 결과 출력
print(graph.invoke({"question": "hi"}))
```
API Reference: StateGraph | START | END

```python
{'answer': 'bye'}
```

invoke의 결과가 출력 스키마만 포함하고 있는 것을 확인할 수 있습니다.
