[![LangGraph Conceptual](https://img.shields.io/badge/LangGraph-Conceptual-blue?logo=langgraph)](https://langchain-ai.github.io/langgraph/concepts/agentic_concepts/)


# Agent architectures

많은 LLM(대규모 언어 모델) 애플리케이션은 LLM 호출 전후에 특정한 제어 흐름을 구현합니다. 예를 들어, RAG(문서 검색 및 생성)는 사용자 질문과 관련된 문서를 검색한 후, 해당 문서를 LLM에 전달하여 모델의 응답을 제공된 문서 맥락에 기반을 두도록 합니다.

고정된 제어 흐름을 하드코딩하는 대신, 더 복잡한 문제를 해결하기 위해 LLM 시스템이 자체적인 제어 흐름을 선택할 수 있게 하고 싶을 때가 있습니다. 이것이 [에이전트](../blog/what_is_an_ai_agent.md)의 한 정의입니다: <span style="color: red;">**Agent 는 애플리케이션의 제어 흐름을 결정하기 위해 LLM을 사용하는 시스템**</span>입니다.

LLM이 애플리케이션을 제어하는 방법에는 여러 가지가 있습니다:
- LLM이 두 가지 가능한 경로 중 하나를 선택할 수 있습니다.
- LLM이 여러 도구 중 어떤 도구를 호출할지 결정할 수 있습니다.
- LLM이 생성된 답변이 충분한지 또는 더 많은 작업이 필요한지 결정할 수 있습니다.

결과적으로, LLM에게 다양한 수준의 제어권을 제공하는 여러 유형의 에이전트 아키텍처가 있습니다.
![agent_types](../asset/agent_types.png)


## Router

라우터는 LLM이 사전 정의된 옵션 세트로 부터 단일 단계를 선택하도록 하는 **에이전트 아키텍처**입니다. 이 아키텍처는 LLM이 단일 결정을 내리고 제한된 옵션 세트에서 특정 출력을 생성하는 데 중점을 두므로 상대적으로 제한된 수준의 제어를 가집니다. 라우터는 이를 달성하기 위해 몇 가지 다른 개념을 사용합니다.


### Structed Output

LLM(대규모 언어 모델)에서 구조화된 출력은 응답 시 따라야 할 특정 형식이나 스키마를 제공함으로써 작동합니다. 이는 도구 호출과 유사하지만 더 일반적인 개념입니다. 도구 호출은 일반적으로 미리 정의된 함수를 선택하고 사용하는 것을 포함하지만, 구조화된 출력은 형식화된 응답이 필요한 모든 상황에 활용될 수 있습니다. 구조화된 출력을 달성하기 위한 일반적인 방법은 다음과 같습니다:

- **Prompt engineering**: 시스템 프롬프트를 통해 LLM이 특정 형식으로 응답하도록 지시합니다.
- **Output parsers**: LLM 응답에서 구조화된 데이터를 추출하기 위해 후처리를 사용합니다.
- **Tool calling**: 일부 LLM의 내장 도구 호출 기능을 활용하여 구조화된 출력을 생성합니다.

구조화된 출력은 라우팅에 필수적입니다. 이는 LLM의 결정을 시스템이 신뢰할 수 있게 해석하고 실행할 수 있도록 보장하기 때문입니다. 구조화된 출력에 대해 더 자세히 알아보려면 [이 방법 가이드](https://langchain-ai.github.io/langgraph/concepts/low_level/#default-reducer)를 참조하세요.


## Tool calling agent

라우터는 LLM이 단일 결정을 내릴 수 있게 하지만, 더 복잡한 에이전트 아키텍처는 두 가지 주요 방식으로 LLM의 제어를 확장합니다:

1. **Multi-step decision making**: LLM이 단일 결정 대신 연속적인 결정을 내릴 수 있습니다.
2. **Tool access**: LLM이 다양한 도구를 선택하고 사용할 수 있습니다.

ReAct는 확장된 기능들을 통합하여 세 가지 핵심 개념을 결합한 범용 에이전트 아키텍처입니다. 이 아키텍처는 다음과 같은 요소를 중심으로 구성됩니다:

1. **Tool Calling**: LLM(대규모 언어 모델)이 다양한 도구를 선택하고 사용할 수 있도록 지원합니다.
2. **Memory**: 이전 단계에서 얻은 정보를 저장하고 이를 활용할 수 있게 합니다.
3. **Planning**: 목표를 달성하기 위해 다단계 계획을 생성하고 이를 실행할 수 있는 능력을 제공합니다.

이러한 architecture는 단순한 라우팅을 넘어, 여러 단계에 걸쳐 동적으로 문제를 해결할 수 있게 만들어 더 복잡하고 유연한 에이전트가 동작할 수 있도록 합니다. `create_react_agent` 를 사용하여 구현할 수 있습니다.


### Tool calling

Tool은 에이전트가 외부 시스템과 상호작용할 때 유용합니다. 외부 시스템(예: API)은 일반적으로 자연어가 아닌 특정 입력 스키마나 페이로드를 요구합니다. 예를 들어, API를 도구로 바인딩할 때 모델은 필요한 입력 스키마를 인식합니다. 모델은 사용자의 자연어 입력에 따라 tool 을 선택하여 호출하며, tool의 required schema에 맞는 출력을 반환합니다.

많은 LLM 제공자가 (chat model을 통해) tool calling을 지원하며, LangChain에서 도구 호출 인터페이스는 간단합니다: Python 함수에 `ChatModel.bind_tools(function)`을 전달하기만 하면 됩니다.

![tool call](../asset/tool_call.png)


### Memory

메모리는 에이전트가 문제 해결의 여러 단계를 통해 정보를 유지하고 활용할 수 있게 해주는 중요한 요소입니다. 메모리는 다음과 같은 다양한 규모에서 작동합니다:

- **Short-term memory**: 시퀀스의 초기 단계에서 얻은 정보를 접근할 수 있게 합니다.
- **Long-term memory**: 대화의 과거 메시지와 같은 이전 상호작용에서 정보를 기억할 수 있게 합니다.

LangGraphs는 애플리케이션의 메모리 관리 방식을 세부적으로 설정할 수 있습니다.

- **`State`**: 유지해야 할 메모리의 구조를 정의하는 사용자 지정 스키마를 의미합니다.
- **`Checkpointer`**: 다양한 상호작용에서 모든 단계의 에이전트 상태(State)를 저장하는 메커니즘을 의미합니다.

이 유연한 접근 방식은 특정 에이전트 아키텍처 요구에 맞춰 메모리 시스템을 맞춤화할 수 있게 해줍니다. 그래프에 메모리를 추가하는 방법에 대한 실용적인 가이드는 [이 튜토리얼](https://langchain-ai.github.io/langgraph/concepts/agentic_concepts/#structured-output)을 참조하세요.

효과적인 메모리 관리는 에이전트가 맥락을 유지하고, 과거 경험에서 배우며, 시간이 지남에 따라 더 정확한 결정을 내릴 수 있게 해줍니다.


### Planning

ReAct 아키텍처에서는 LLM이 while-loop에서 반복적으로 호출됩니다. 각 단계에서 에이전트는 호출할 도구와 그 도구에 대한 입력을 결정합니다. 그런 다음 도구가 실행되고, 출력은 LLM에 <span style="text-decoration: underline; cursor: pointer;" title="'관찰값(observation)'이라는 용어는 주로 강화 학습(RL)이나 에이전트 시스템에서 사용되며, 에이전트가 환경과 상호작용하며 얻는 정보를 관찰값으로 정의합니다. LLM 기반 시스템에서도 이 개념을 차용하여, 도구 실행 결과를 관찰값으로 간주하고 이를 다음 단계의 입력으로 사용하는 구조를 따릅니다">관찰값(observations)</span>으로써 피드백됩니다. 에이전트가 사용자 요청을 해결하기 위해 충분한 정보를 얻었다고 결정하면 while-loop가 종료됩니다.


### ReAct implementation

[이 논문](https://arxiv.org/abs/2210.03629)과 `create_react_agent` 구현 사이에는 몇 가지 차이점이 있습니다:

* 첫째, 우리는 tool-calling 을 사용하여 LLM이 도구를 호출할 수 있게 하지만, 논문에서는 prompting + raw output 을 parsing 하는 방법을 사용했습니다. 논문 작성 당시에는 도구 호출 기능이 없었기 때문인데, 도구 호출 기능이 더 좋고 신뢰할 수 있습니다.
* 둘째, 우리는 LLM에 프롬프트를 메시지 형식으로 제공하지만, 논문에서는 문자열 형식을 사용했습니다. 논문 작성 당시 LLM은 메시지 기반 인터페이스를 제공하지 않았고, 현재는 메시지 기반 인터페이스만 제공합니다.
* 셋째, 논문에서는 모든 도구의 입력을 단일 문자열로 요구했는데, 이는 당시 LLM의 성능 한계로 인해 주로 단일 입력만 생성할 수 있었기 때문입니다. 우리의 구현은 여러 입력이 필요한 도구도 사용할 수 있게 합니다.
* 넷째, 논문에서는 한 번에 하나의 도구만 호출하는 것을 다루었는데, 이는 당시 LLM의 성능 한계 때문입니다. 우리의 구현은 동시에 여러 도구를 호출할 수 있습니다.
* 마지막으로, 논문에서는 도구를 호출하기 전에 명시적으로 "생각(Thought)" 단계를 생성하도록 LLM에 요청했지만, 우리의 구현에서는 LLM의 성능이 향상되어 기본적으로 이를 생략합니다. 물론 필요에 따라 프롬프트를 추가할 수 있습니다.


## Custom agent architectures

Router와 Tool calling 에이전트(예: ReAct)는 일반적이지만, [cutomizing agent architectures](../blog/why_you_should_outsource_your_agentic_infrastructure_but_own_your_cognitive_architecture.md)는 특정 작업에 대해 더 나은 성능을 발휘할 수 있습니다. LangGraph는 맞춤형 에이전트 시스템을 구축하기 위한 몇 가지 강력한 기능을 제공합니다:


### Human-in-the-loop

인간의 개입은 특히 민감한 작업에서 에이전트의 신뢰성을 크게 향상시킬 수 있습니다. 다음이 포함될 수 있습니다:

- 특정 작업 승인(Approving)
- 에이전트 상태 업데이트를 위한 피드백 제공(Providing feedback)
- 복잡한 의사 결정 과정에서의 지침 제공(Offering guidance)

완전한 자동화가 불가능하거나 바람직하지 않은 경우, Human-in-the-loop 이 필수적입니다. 자세한 내용은 [human-in-the-loop guide](./human_in_the_loop.md)를 참조하세요.


### Parallelization

병렬 처리는 효율적인 다중 에이전트 시스템과 복잡한 작업에 필수적입니다. LangGraph는 Send API를 통해 병렬 처리를 지원하여 다음을 가능하게 합니다:

- 여러 상태의 동시 처리
- 맵-리듀스(map-reduce)와 같은 작업의 구현
- 독립적인 하위 작업의 효율적인 처리

구현 예제는 [map-reduce tutorial](../how_to/how_to_create_map_reduce_branchs_for_parallel_execution.md)을 참조하세요.


### Subgraphs

[Subgraphs](./langgraph_glossary.md#Subgraphs)는 특히 [multi-agent systems](./multi_agent_systems.md)처럼 복잡한 에이전트 아키텍처를 관리하는 데 필수적입니다. 서브그래프는 다음을 허용합니다:

- 개별 에이전트를 위한 독립적인 상태 관리
- 에이전트 팀의 계층적 구성
- 에이전트와 메인 시스템 간 제어된 communication

서브그래프는 상태 스키마의 overlapping keys를 통해 부모 그래프와 통신합니다. 이를 통해 유연하고 모듈화된 에이전트 설계를 할 수 있습니다. 구현 세부 사항은 [subgraph how-to guide](../how_to/how_to_use_subgraphs.md)를 참조하세요.


### Reflection

Reflection 메커니즘은 다음을 통해 에이전트의 신뢰성을 크게 향상시킬 수 있습니다:

- 작업 완료 및 정확성 평가
- 반복적 개선을 위한 피드백 제공
- 자기 수정 및 학습 가능

Reflection은 종종 LLM 기반 시스템에서 결정론적 방법으로 사용할 수 있습니다. 예를 들어, 코딩 작업에서 컴파일 오류는 피드백으로 작용할 수 있습니다. 이 접근 방식은 [LangGraph를 사용한 자기 수정 코드 생성](https://langchain-ai.github.io/langgraph/concepts/agentic_concepts/#reflection) 비디오에서 시연됩니다.


이러한 기능을 활용함으로써 LangGraph는 복잡한 워크플로를 처리하고, 효과적으로 협력하며, 성능을 지속적으로 개선할 수 있는 정교하고 작업별 맞춤형 에이전트 아키텍처를 생성할 수 있습니다.
