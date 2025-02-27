[![LangGraph How to](https://img.shields.io/badge/LangGraph-How_to-yellow?logo=langgraph)](https://langchain-ai.github.io/langgraph/how-tos/pass_private_state/)


# How to pass private state between nodes

어떤 경우에는 그래프의 메인 스키마와는 관련이 없지만 로직에 중간에 필요한 정보를 노드 간에 교환하고 싶을 수 있습니다. 이 비공개 데이터는 전체 입력/출력과 관련이 없고 특정 노드 간에만 공유되어야 합니다.

이 가이드에서는 세 개의 노드 (node_1, node_2 및 node_3)로 구성된 그래프를 생성하는 예제를 사용할 것입니다. 비공개 데이터는 첫 번째 및 두 번째 노드 (node_1 및 node_2) 간에 전달하고, 세 번째 노드 (node_3)는 전체 상태만을 접근하는 방법을 설명합니다.

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

# 그래프의 전체 상태 (공개 상태로 노드 간에 공유됨)
class OverallState(TypedDict):
    a: str

# node_1에서 출력된 비공개 데이터는 전체 상태의 일부가 아닙니다
class Node1Output(TypedDict):
    private_data: str

# 비공개 데이터는 node_1과 node_2 간에만 공유됩니다
def node_1(state: OverallState) -> Node1Output:
    output = {"private_data": "node_1에서 설정됨"}
    print(f"node_1에 진입:\n\t입력: {state}.\n\t출력: {output}")
    return output

# node_2의 입력은 node_1 이후 사용 가능한 비공개 데이터만 요청합니다
class Node2Input(TypedDict):
    private_data: str

def node_2(state: Node2Input) -> OverallState:
    output = {"a": "node_2에서 설정됨"}
    print(f"node_2에 진입:\n\t입력: {state}.\n\t출력: {output}")
    return output

# node_3은 전체 상태에만 접근하며 (node_1의 비공개 데이터에는 접근하지 않음)
def node_3(state: OverallState) -> OverallState:
    output = {"a": "node_3에서 설정됨"}
    print(f"node_3에 진입:\n\t입력: {state}.\n\t출력: {output}")
    return output

# 상태 그래프 빌드
builder = StateGraph(OverallState)
builder.add_node(node_1)  # node_1은 첫 번째 노드입니다
builder.add_node(node_2)  # node_2는 두 번째 노드이며 node_1의 비공개 데이터를 사용합니다
builder.add_node(node_3)  # node_3은 세 번째 노드이며 비공개 데이터를 보지 않습니다
builder.add_edge(START, "node_1")  # 그래프를 node_1로 시작합니다
builder.add_edge("node_1", "node_2")  # node_1에서 node_2로 전달합니다
builder.add_edge("node_2", "node_3")  # node_2에서 node_3로 전달합니다 (전체 상태만 공유됨)
builder.add_edge("node_3", END)  # node_3 이후 그래프 종료
graph = builder.compile()

# 초기 상태로 그래프 호출
response = graph.invoke(
    {
        "a": "시작 시 설정됨",
    }
)

print()
print(f"그래프 호출의 출력: {response}")
```
API Reference: StateGraph | START | END

```
node_1에 진입:
    입력: {'a': '시작 시 설정됨'}.
    출력: {'private_data': 'node_1에서 설정됨'}
node_2에 진입:
    입력: {'private_data': 'node_1에서 설정됨'}.
    출력: {'a': 'node_2에서 설정됨'}
node_3에 진입:
    입력: {'a': 'node_2에서 설정됨'}.
    출력: {'a': 'node_3에서 설정됨'}

그래프 호출의 출력: {'a': 'node_3에서 설정됨'}
```
