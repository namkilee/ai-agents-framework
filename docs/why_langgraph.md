[![LangGraph Conceptual](https://img.shields.io/badge/LangGraph-Conceptual-blue?logo=langgraph)](https://langchain-ai.github.io/langgraph/concepts/high_level/)


# Why LangGraph?

<br>

## LLM applications

LLM(대형 언어 모델)은 새로운 클래스의 애플리케이션에 지능을 내재화(embed)할 수 있게 합니다. LLM을 사용하는 애플리케이션을 만드는 데는 여러 가지 패턴이 있습니다. [워크플로우](../reference/building_effective_agents.md)는 LLM 호출 주변에 사전 정의된 코드로 경로를 구성하는 구조를 가지고 있습니다. LLM은 이러한 사전 정의된 코드 경로를 통해 제어 흐름을 제어할 수 있으며, 이를 일부에서는 "에이전틱 시스템(agentic system)"이라고 부르기도 합니다. 다른 경우에는 이러한 구조를 제거하여, 도구 호출을 통해 계획을 세우고 행동하며, 자신의 행동으로부터 피드백을 받아 추가적인 행동을 수행할 수 있는 자율 에이전트를 생성하는 것도 가능합니다.

![agent_workflow](../asset/agent_workflow.png)

<br>

## What LangGraph provids

LangGraph는 워크플로우나 에이전트의 기반이 되는 저수준의 인프라를 제공합니다. 프롬프트나 아키텍처를 추상화하지 않으며, 다음 세 가지 주요 이점을 제공합니다:


### Persistence

LangGraph는 [영속성(persistence)](./persistence.md) 계층을 제공하며, 이를 통해 다음과 같은 혜택을 얻을 수 있습니다:
- [Memory](./memory.md): LangGraph는 애플리케이션 상태의 다양한 측면을 저장하여 사용자 간 상호작용 및 대화 내용을 기억할 수 있습니다
- [Human-in-the-loop](./human_in_the_loop.md): 상태가 체크포인트되기 때문에 실행이 중단되고 재개될 수 있어 사람 입력을 통한 의사 결정, 검증 및 수정이 가능합니다.

### Streaming

LangGraph는 실행 과정 동안 워크플로우 / 에이전트 상태를 사용자(또는 개발자)에게 스트리밍하는 것도 지원합니다. LangGraph는 애플리케이션에 내장된 LLM 호출에서 이벤트(예: [도구 호출 피드백](../how_to/how_to_stream.md#updates)) 및 [토큰 스트리밍](../how_to/how_to_stream_llm_tokes_from_your_graph.md)을 모두 지원합니다.

### Debugging and Deployment

LangGraph는 [LangGraph Platform](./langgraph_platform.md)을 통해 애플리케이션을 테스트, 디버깅 및 배포하는 쉬운 접근성을 제공합니다. 여기에는 워크플로우 또는 에이전트를 시각화, 상호작용 및 디버깅할 수 있는 IDE인 스튜디오가 포함됩니다. 또한 다양한 배포 옵션도 포함됩니다.
