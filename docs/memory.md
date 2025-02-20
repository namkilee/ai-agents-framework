<style>code { white-space: pre; overflow-x: auto; }</style>

[![LangGraph Conceptual](https://img.shields.io/badge/LangGraph-Conceptual-blue?logo=langgraph)](https://langchain-ai.github.io/langgraph/concepts/memory/)


# Memory

<br>

## What is Memory?

[메모리](../asset/cognitive_neuroscience_perspective_on_memory.pdf)는 사람들이 정보를 저장, 검색 및 활용하여 현재와 미래를 이해할 수 있게 하는 인지(congitive) 기능입니다. 예를 들어, 매번 동일한 내용을 반복해서 말해야 하는 동료와 함께 일하는 좌절감을 상상해 보세요! AI 에이전트가 수많은 사용자 상호작용을 포함한 더 복잡한 작업을 수행함에 따라, 메모리 기능을 가지는 것은 효율성과 사용자 만족도를 위해 매우 중요합니다. 메모리를 통해 에이전트는 피드백을 학습하고 사용자의 선호도를 적용할 수 있습니다. 이 가이드에서는 기억 범위에 따라 두 가지 유형의 메모리를 다룹니다:

**단기 메모리(Short-term memory)**, 또는 스레드 범위 메모리(thread-scoped memory)는 사용자와의 단일 대화 스레드 내에서 언제든지 호출할 수 있습니다. LangGraph는 에이전트 [상태(state)](./langgraph_glossary.md#state)의 일부분으로 단기 메모리를 관리합니다. 상태는 [체크포인터](./persistence.md#checkpoints)를 사용하여 데이터베이스에 영속되기 때문에 언제든지 스레드를 다시 시작할 수 있습니다. 단기 메모리는 그래프가 호출되거나 단계가 완료될 때 업데이트되며, 각 단계의 시작 시 상태가 읽혀집니다.

**장기 메모리(Long-term memory)** 는 대화 스레드 **전체**에 걸쳐 공유됩니다. 이는 언제든지, **어떤 스레드**에서든 호출할 수 있습니다. 메모리는 단일 스레드 ID 범위에 속해질 뿐만 아니라 사용자 정의 네임스페이스 범위 내에도 속해집니다. LangGraph는 장기 메모리를 저장하고 호출할 수 있는 [저장소](./persistence.md#memory-store)를 제공합니다.

두 가지 유형의 메모리는 모두 애플리케이션을 이해하고 구현하는 데 중요합니다.

![short_vs_long](../asset/short-vs-long.png)

<br>

## Short-term memory

단기 메모리는 애플리케이션이 단일 스레드 또는 대화 내에서 이전의 상호작용을 기억할 수 있게 합니다. 이메일이 단일 대화에서 메시지를 그룹화하는 것과 유사하게 스레드는 세션에서 여러 상호작용을 구성합니다.

LangGraph는 에이전트 상태의 일부로써 단기 메모리를 관리하며, 이는 스레드 범위의 체크포인트를 통해 저장됩니다. 이 상태(state)에는 일반적으로 대화 기록과 업로드된 파일, 검색된 문서 또는 생성된 아티팩트와 같은 다른 상태 데이터가 포함될 수 있습니다. 이를 그래프 상태에 저장함으로써 봇은 다른 스레드와 분리된 상태에서 주어진 대화의 전체 컨텍스트를 접근할 수 있습니다.

대화 기록이 단기 메모리를 나타내는 가장 일반적인 형태이므로, 다음 섹션에서는 메시지 목록이 길어질 때 대화 기록을 관리하는 기술에 대해 다룹니다. 높은 수준의 개념에 집중하고 싶다면,  장기 메모리을 참고하세요.


### Managing long conversation history

긴 대화는 오늘날의 LLM에게 도전 과제입니다. 전체 기록이 LLM의 context window에 전부 들어갈 수 없기에  복구 불가능한 오류가 발생할 수 있습니다. 설령 LLM 이 기술적으로 전체 컨텍스트 길이를 지원하더라도, 대부분의 LLM은 긴 컨텍스트에서 성능이 저하됩니다. 오래된 내용이나 주제와 벗어난 콘텐츠에 의해 "산만해지고(distracted)", 응답 시간이 느려지고 비용이 증가하는 문제가 발생합니다.

단기 메모리를 관리하는 것은 애플리케이션의 다른 성능 요구사항(지연 시간 및 비용)과 [정밀도와 재현율(precision & recall)](https://en.wikipedia.org/wiki/Precision_and_recall#:~:text=Precision%20can%20be%20seen%20as,irrelevant%20ones%20are%20also%20returned)의 균형을 맞추는 작업입니다. 항상 그렇듯이, LLM을 위해 정보를 어떻게 표현할지 비판적으로 고민하고 데이터를 검토하는 것이 중요합니다. 아래에서는 메시지 목록을 관리하기 위한 몇 가지 일반적인 기법을 다루며, 여러분의 애플리케이션에 가장 적합한 절충안을 선택할 수 있도록 충분한 맥락을 제공하고자 합니다.

- [Editing message lists](#editing-message-lists): 언어 모델에 전달하기 전에 메시지 목록을 다듬고 필터링하는 방법에 대해 생각합니다.
- [Summarizing past conversations](#summarizing-past-conversations): 메시지 목록을 필터링하는 것 외에 자주 사용하는 기술입니다.


### Editing message lists

Chat model 은 개발자 명령(시스템 메시지)과 사용자 입력(사용자 메시지)을 포함하는 메시지를 컨텍스트로 받습니다. 채팅 애플리케이션에서는 메시지가 사용자 입력과 모델 응답 간에 교대로 전송되어 시간이 지남에 따라 메시지 목록이 길어집니다. 컨텍스트 윈도우는 제한적이며, 토큰이 많은 메시지 목록은 비용이 많이 들기 때문에, 많은 애플리케아이션이 오래된 정보를 수동으로 제거하거나 버림는 것이 유리할 수 있습니다.

![message_filter](../asset/filter.png)

가장 직접적인 방식은 목록에서 오래된 메시지를 제거하는 것입니다([가장 최근에 사용되지 않은 캐시처럼](https://en.wikipedia.org/wiki/Page_replacement_algorithm#Least_recently_used)).

LangGraph에서 목록 내의 콘텐츠를 삭제하는 일반적인 방법은 특정 노드에서 목록의 일부를 삭제하게끔 업데이트를 반환하는 것입니다. 이 업데이트의 형식은 사용자가 정의할 수 있지만, 일반적으로 객체나 딕셔너리를 반환하여 유지할 값을 지정하는 방식이 많이 사용됩니다.

```python
def manage_list(existing: list, updates: Union[list, dict]):
    if isinstance(updates, list):
        # 일반적인 경우, 기존 기록에 새로 추가
        return existing + updates
    elif isinstance(updates, dict) and updates["type"] == "keep":
        # 업데이트 형식은 사용자가 정의 가능
        # 예를 들어, "DELETE" 문자열인 경우 전체 목록을 삭제하도록 설정 가능
        return existing[updates["from"]:updates["to"]]
    # 기타 업데이트 방식도 정의 가능

class State(TypedDict):
    my_list: Annotated[list, manage_list]

def my_node(state: State):
    return {
        # "my_list" 필드에 대해 마지막 5개의 값만 유지하도록 업데이트 반환
        "my_list": {"type": "keep", "from": -5, "to": None}
    }
```

위의 예제에서 LangGraph는 `my_list` 키가 업데이트 될 때마다 `manage_list` "리듀서" 함수를 호출합니다. 해당 함수에서 업데이트 방식을 결정합니다. 일반적으로 메시지는 기존 목록에 추가됩니다(대화는 점점 길어집니다); 그러나, 딕셔너리를 통해 상태의 특정 부분만 유지하도록 하는 기능도 추가했습니다. 이를 통해 오래된 메시지를 프로그래밍적으로 제거할 수 있습니다.

또 다른 일반적인 벙법은 삭제할 모든 메시지의 ID를 가진 "remove" 객체 목록을 반환하는 것입니다. LangChain 메시지와 `add_messages` 리듀서(또는 동일한 기본 기능을 사용하는 `MessagesState`)를 사용하는 경우 `RemoveMessage를` 사용하여 이를 수행할 수 있습니다.

```python
from langchain_core.messages import RemoveMessage, AIMessage
from langgraph.graph import add_messages
# ... 다른 import

class State(TypedDict):
    # add_messages는 기본적으로 기존 목록에 ID로 메시지를 삽입합니다.
    # RemoveMessage가 반환되면, 해당 ID의 메시지를 목록에서 삭제합니다.
    messages: Annotated[list, add_messages]

def my_node_1(state: State):
    # 상태의 `messages` 목록에 AI 메시지 추가
    return {"messages": [AIMessage(content="Hi")]}

def my_node_2(state: State):
    # 상태의 `messages` 목록에서 마지막 2개의 메시지를 제외한 모든 메시지 삭제
    delete_messages = [RemoveMessage(id=m.id) for m in state['messages'][:-2]]
    return {"messages": delete_messages}
```
API Reference: RemoveMessage | AIMessage | add_messages

위의 예에서, `add_messages` 리듀서는 my_node_1에서 보여준 대로 `messages` 키에 새 메시지를 [추가](./langgraph_glossary.md#serialization)할 수 있게 합니다. `RemoveMessage`를 보면 해당 ID의 메시지를 목록에서 삭제합니다(그리고 RemoveMessage는 폐기됩니다). LangChain 특정 메시지 처리에 대한 자세한 내용은 `RemoveMessag`e 사용 방법에 대한 [이 가이드](../how_to/how_to_delete_messages.md)를 참조하세요.


### Summarizing past conversations

위에서 보여준 메시지 삭제나 제거의 문제점은 메시지 큐를 조정하면서 정보를 잃을 수 있다는 점입니다. 이러한 이유로 일부 애플리케이션은 chat model 을 사용하여 메시지 기록을 요약하는 것처럼 정교한 방법을 사용하는 것이 유리합니다.

![summary](../asset/summary.png)

간단한 프롬프트와 오케스트레이션 로직을 사용하여 이를 달성할 수 있습니다. 예를 들어, LangGraph에서는 [MessagesState](./langgraph_glossary.md#working-with-messages-in-graph-state)를 확장하여 `summary` 키를 포함할 수 있습니다.

```python
from langgraph.graph import MessagesState

class State(MessagesState):
    summary: str
```

그런 다음, 기존 요약을 다음 요약의 컨텍스트로 활용하여 채팅 기록의 요약을 생성할 수 있습니다. `summarize_conversation` 노드는 `messages` 키 내에 일정 수의 메시지가 누적된 후 호출될 수 있습니다.

```python
def summarize_conversation(state: State):
    # 먼저, 기존 요약을 가져옵니다.
    summary = state.get("summary", "")

    # 요약 프롬프트 생성
    if summary:
        # 이전 요약이 있는 경우.
        summary_message = (
            f"This is a summary of the conversation to date: {summary}\n\n"
            "Extend the summary by taking into account the new messages above:"
        )
    else:
        summary_message = "Create a summary of the conversation above:"

    # 프롬프트를 기록에 추가
    messages = state["messages"] + [HumanMessage(content=summary_message)]
    response = model.invoke(messages)

    # 최근 2개의 메시지를 제외한 모든 메시지 삭제
    delete_messages = [RemoveMessage(id=m.id) for m in state["messages"][:-2]]
    return {"summary": response.content, "messages": delete_messages}
```

예제 사용에 대한 자세한 내용은 [이 가이드](../how_to/how_to_add_summary_of_the_conversation_history.md)와 LangChain Academy 과정의 모듈 2를 참조하세요.


### Knowing **when** to remove messages

대부분의 LLM은 최대 컨텍스트 윈도우(token)를 가지고 있습니다. 메시지를 잘라낼 시점을 결정하는 간단한 방법은 메시지 기록의 토큰 수를 세고, 제한에 가까워질 때마다 잘라내는 것입니다. 단순한 잘라내기는 직접 구현하기 쉽지만, 몇 가지 "주의할 점(gotchas)"이 있습니다. 일부 모델 API는 메시지 유형의 순서 제한(예: 사람 메시지로 시작해야 하며, 동일한 유형의 메시지가 연속해서 나올 수 없음 등)을 두기도 합니다.

LangChain을 사용하는 경우 `trim_messages` 유틸리티를 사용하여 목록에서 유지할 토큰 수와 경계를 처리하는 전략(예: 마지막 `max_tokens` 유지)을 지정할 수 있습니다.

아래는 예제입니다.

```python
from langchain_core.messages import trim_messages

trim_messages(
    messages,
    # 메시지의 마지막 <= n_count 토큰을 유지합니다.
    strategy="last",
    # 모델에 따라 조정하거나 사용자 정의 token_encoder를 전달해야 합니다.
    token_counter=ChatOpenAI(model="gpt-4"),
    # 원하는 대화 길이에 따라 조정하세요.
    max_tokens=45,
    # 대부분의 채팅 모델은 채팅 기록이 다음 중 하나로 시작하는 것을 기대합니다:
    # (1) HumanMessage 또는 (2) HumanMessage 다음에 오는 SystemMessage
    start_on="human",
    # 대부분의 채팅 모델은 채팅 기록이 다음 중 하나로 끝나는 것을 기대합니다:
    # (1) HumanMessage 또는 (2) ToolMessage
    end_on=("human", "tool"),
    # 보통 SystemMessage가 원래 기록에 존재하는 경우 이를 유지하고자 합니다.
    # SystemMessage에는 모델에 대한 특별한 지시가 있습니다.
    include_system=True,
)
```
API Reference: trim_messages

<br>

## Long-term mermory

LangGraph의 장기 메모리는 시스템이 다양한 대화나 세션 간에 정보를 유지할 수 있게 합니다. 스레드 범위의 단기 메모리와 달리, 장기 메모리는 사용자 정의 "namespaces" 내에 저장됩니다.


### Stroing memories

LangGraph는 JSON 문서 형식으로 장기 메모리를 [저장소](./persistence.md#memory-store)에 저장합니다. 각 메모리는 사용자 정의 네임스페이스(폴더와 유사)와 고유한 키(파일 이름과 유사) 아래에 구성됩니다. 네임스페이스에는 일반적으로 사용자나 조직 ID 또는 정보를 더 쉽게 다룰 수 있게 하는 기타 레이블이 포함됩니다. 이 구조는 메모리의 계층적 조직을 가능하게 합니다. 교차 네임스페이스 검색은 콘텐츠 필터를 통해 지원됩니다. 아래 예제를 참조하세요.

```python
from langgraph.store.memory import InMemoryStore

def embed(texts: list[str]) -> list[list[float]]:
    # 실제 임베딩 함수나 LangChain 임베딩 객체로 대체하세요.
    return [[1.0, 2.0] * len(texts)]

# InMemoryStore는 데이터를 in-memory 딕셔너리에 저장합니다. 프로덕션 사용 시 DB 기반 스토어를 사용하세요.
store = InMemoryStore(index={"embed": embed, "dims": 2})
user_id = "my-user"
application_context = "chitchat"
namespace = (user_id, application_context)

store.put(namespace, "a-memory", {
    "rules": [
        "User likes short, direct language",
        "User only speaks English & python"
    ],
    "my-key": "my-value",
})

# ID로 "memory" 가져오기
item = store.get(namespace, "a-memory")

# 이 네임스페이스 내에서 "memories" 검색, 콘텐츠 동등성 기준으로 필터링, 벡터 유사도 기준으로 정렬
items = store.search(namespace, filter={"my-key": "my-value"}, query="language preferences")
```


### Framework for thinking about long-term memory

장기 기억은 단일한 해결책이 없는 복잡한 도전 과제입니다. 그러나 아래의 질문들은 다양한 기술을 탐색하는 데 도움을 줄 수 있는 구조적 틀을 제공합니다:

**What is the type of memory**

사람은 [사실](https://en.wikipedia.org/wiki/Semantic_memory), [경험](https://en.wikipedia.org/wiki/Episodic_memory), [규칙](https://en.wikipedia.org/wiki/Procedural_memory)을 기억하는 데 메모리를 사용합니다. AI 에이전트도 동일한 방식으로 메모리를 활용할 수 있습니다. 예를 들어, AI 에이전트는 특정 사용자에 대한 사실을 기억하여 작업을 수행할 수 있습니다. [아래 섹션](#memory-types)에서는 여러 유형의 메모리에 대해 더 자세히 설명합니다.

**When do you want to update memories**  

메모리는 에이전트의 애플리케이션 로직의 일부로 업데이트될 수 있습니다(예: "핫 패스"에서). 이 경우, 에이전트는 일반적으로 사용자에게 응답하기 전에 사실을 기억하도록 결정합니다. 또는 메모리는 백그라운드 작업(백그라운드에서 비동기로 실행되며 메모리를 생성하는 로직)으로 업데이트될 수도 있습니다. 이러한 접근 방식 간의 트레이드오프는 [아래 섹션](#writing-memories)에서 설명합니다.

<br>

## Memory types

다양한 애플리케이션은 다양한 유형의 메모리를 필요로 합니다. 비록 완벽한 비유는 아니지만, [사람의 메모리 유형](../blog/type_of_memory.md)을 살펴보는 것은 유익할 수 있습니다. 일부 연구(예: [CoALA 논문](../asset/cognitive_architectures_for_language_agents.pdf))는 이러한 메모리 유형을 AI 에이전트에서 사용되는 것들과 매핑하기도 했습니다.

| Memory type  | What is Stored | Human Example         | Agent Example         |
|--------------|----------------|-----------------------|-----------------------|
| Semantic     | Facts          | Things I learned in school | Facts about a user |
| Episodic     | Facts          | Things I did          | Past agent actions    |
| Procedural   | Instrcutions   | Instincts or motor skills | Agent system prompt |


### Semantic Memory

[Semantic memory](https://en.wikipedia.org/wiki/Semantic_memory)는 사람과 AI 에이전트 모두에서 특정 사실과 개념을 저장함을 의미합니다. 사람에게는 학교에서 배운 정보와 개념 및 그 이해관계가 포함됩니다. AI 에이전트에서는 종종 과거 상호작용에서 얻은 사실이나 개념을 기억하여 애플리케이션을 개인화하는 데 사용됩니다.

> **Note** 
> "Semantic search"과 혼동하지 마세요. Semantic search는 "meaning"를 사용하여 유사한 콘텐츠를 찾는 기술(일반적으로 임베딩을 사용)입니다. Semantic memory는 심리학 용어로, 사실과 지식을 저장하는 것을 의미하는 반면, Semantic search은 정확한 일치 대신 의미에 기반하여 정보를 검색하는 방법입니다.

**Profile**

Semantic mermory는 다양한 방식으로 관리할 수 있습니다. 예를 들어, 메모리는 사용자, 조직 또는 기타 엔티티(에이전트 자체를 포함)에 대한 잘 정의되고 구체적인 정보를 지속적으로 업데이트하는 단일 "프로필"로 관리될 수 있습니다. 프로필은 일반적으로 도메인을 표현하기 위해 선택한 다양한 키-값 쌍으로 구성된 JSON 문서입니다.

프로필을 기억할 때마다, 각 프로필을 업데이트하고 있는지 확인하고 싶을 것입니다. 따라서 이전 프로필을 전달하고 모델에게 새 프로필을 생성하거나 기존 프로필에 적용할 일부 JSON 패치를 생성하도록 요청하고 싶을 것입니다. 그러나 프로필이 커질수록 오류가 발생하기 쉬워지므로, 이를 방지하기 위해 프로필을 여러 문서로 분리하거나 문서를 생성할 때 엄격한 디코딩을 적용하여 메모리 스키마의 유효성이 유지되도록 하는 것이 더 나을 수 있습니다.

![update_profile](../asset/update-profile.png)

**Collection**

반면에, 메모리를 시간이 지나면서 지속적으로 업데이트되고 확장되는 문서들의 컬렉션으로 관리할 수도 있습니다. 각각의 개별 메모리는 더 좁은 범위로 정의되고 생성하기 쉬워지며, 이는 시간이 지나도 정보를 잃을 가능성을 줄여줍니다. LLM이 새로운 정보를 기존 프로필과 결합하는 것보다 새롭게 생성하는 것이 더 쉽기 때문에, 문서 컬렉션 방식은 결과적으로 더 높은 정보 회수율을 제공하는 경향이 있습니다.

그러나 이 방식은 메모리 업데이트와 관련된 복잡성을 증가 시킵니다. 모델은 이제 리스트에서 기존 항목을 삭제하거나 업데이트해야 하며, 이는 까다로울 수 있습니다. 게다가 일부 모델은 과도한 삽입을 기본으로 하고, 다른 모델은 과도한 업데이트를 기본으로 할 수 있습니다. 이를 관리하는 한 가지 방법으로는 [Trustcall](https://github.com/hinthornw/trustcall) 패키지를 참고할 수 있으며, LangSmith와 같은 도구를 사용하는 것을 고려해볼 수 있습니다.

문서 컬렉션을 활용하면 리스트에서 메모리를 검색하는 작업에도 복잡성이 증가됩니다. 현재 `Store`는 semantic search와 content filtering을 모두 지원합니다.

마지막으로, 메모리 컬렉션을 사용하는 경우 모델에 포괄적인 컨텍스트를 제공하는 데 어려움이 있을 수 있습니다. 개별 메모리가 특정 스키마를 따르더라도, 이 구조가 메모리 간의 전체적인 맥락이나 관계를 포착하지 못할 수 있습니다. 그 결과, 이러한 메모리를 사용해 응답을 생성할 때, 통합된 프로필 접근 방식에서 더 쉽게 얻을 수 있는 중요한 맥락 정보를 모델이 놓칠 가능성이 있습니다.

![update_list](../asset/update-list.png)

메모리 관리 방식과 상관없이, 핵심은 에이전트가 semantic memories를 활용해 [응답](./retrieval_augmented_generation_rag.md)을 만든다는 점입니다. 이는 종종 더 개인화되고 관련성 높은 상호작용으로 이어집니다.


