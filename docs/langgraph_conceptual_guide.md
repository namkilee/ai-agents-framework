[![LangGraph Conceptual](https://img.shields.io/badge/LangGraph-Conceptual-blue?logo=langgraph)](https://langchain-ai.github.io/langgraph/concepts/)


# Conceptual Guide

이 가이드는 LangGraph 프레임워크와 더 넓은 AI 애플리케이션의 주요 개념을 설명합니다. 

개념 가이드를 자세히 읽기 전에 적어도 빠른 시작 가이드를 먼저 읽어보는 것을 추천합니다. 빠른 시작 가이드를 통해 여기서 논의될 개념을 이해하는 데 도움이 되는 실질적인 배경 지식을 제공받을 수 있습니다.

개념 가이드는 단계별 지침이나 특정 구현 예제를 다루지 않습니다 — 이러한 내용은 튜토리얼 및 How-to 가이드에 있습니다. 자세한 참고 자료는 API 참조를 확인하세요.


## LangGraph


### High Level

- [Why LangGraph?](./why_langgraph.md): LangGraph 의 개요와 그 목표.


### Concepts

- [LangGraph Glossary](./langgraph_glossary.md): LangGraph 워크플로우는 서로 다른 구성 요소를 표현하는 노드와 노드들 간의 정보 흐름을 표현하는 엣지로 이루어진 그래프 형태로 설계되어 있습니다. 이 가이드는 LangGraph 그래프 기본 요소와 관련된 주요 개념을 개괄적으로 설명합니다.
- [Common Agentic Patterns](./agent_architectures.md): 에이전트는 더 복잡한 문제를 해결하기 위해 스스로 흐름을 선택하도록 LLM을 사용합니다! 에이전트는 대다수의 LLM 애플리케이션에서 주요 구성 요소입니다. 이 가이드는 다양한 에이전트 아키텍처 유형과 이를 애플리케이션의 흐름을 제어하는 데 사용하는 방법에 대해 설명합니다.
- [Multi-Agent Systems](./multi_agent_systems.md): 복잡한 LLM 애플리케이션은 종종 애플리케이션의 서로 다른 부분을 각각 책임지는 여러 에이전트로 나눌 수 있습니다. 이 가이드는 다중 에이전트 시스템을 구축하는 일반적인 패턴을 설명합니다.
- [Breakpoints](./breakpoints.md): breakpoints는 특정 지점에서 그래프 실행을 일시 중지할 수 있게 합니다. 중단점을 사용하여 디버깅 목적을 위해 그래프 실행을 단계별로 진행할 수 있습니다.
- [Human-in-the-loop](./human_in_the_loop.md): LangGraph 애플리케이션에서 사람과의 상호작용을 통합하는 다양한 방법을 설명합니다.
- [Time travel](./time_travel.md): Time travel은 LangGraph 애플리케이션에서 과거의 행동을 replay하여 대체 경로를 탐색하고 문제를 디버그할 수 있게 합니다.
- [Persistence](./persistence.md): LangGraph는 체크포인터를 통해 구현된 내장된 영속성 계층을 가지고 있습니다. 이 영속성 계층은 human-in-the-loop, memory, time travel 및 fault-tolerance와 같은 강력한 기능을 지원하는 데 도움이 됩니다.
- [Memory](./memory.md): AI 애플리케이션에서 메모리는 과거 상호작용에서 정보를 처리, 저장 및 효과적으로 recall 할 수 있는 능력을 의미합니다. 메모리를 통해 에이전트는 피드백을 학습하고 사용자의 선호도에 적응할 수 있습니다.
- [Streaming](./streaming.md): 스트리밍은 LLM을 기반으로 한 애플리케이션의 응답성을 향상시키는 데 필수적입니다. 완전한 응답이 준비되기 전에 점진적으로 출력을 표시함으로써, 특히 LLM의 지연 문제를 다룰 때 사용자 경험(UX)을 크게 향상시킵니다.
- [Functional API(beta)](./functional_api.md): LangGraph에서 개발을 위한 Graph API(StateGraph)의 대안입니다.
- [FAQ](https://langchain-ai.github.io/langgraph/concepts/faq/): LangGraph에 대한 자주 묻는 질문입니다.


## LangGraph Platform

LangGraph 플랫폼은 오픈 소스 LangGraph 프레임워크를 기반으로 구축된, production 환경에서 에이전트 애플리케이션을 배포하기 위한 **상업적 솔루션**입니다.

LangGraph 플랫폼의 몇 가지 다른 배포 옵션은 [이 가이드](./deployment_options.md)에서 설명합니다.

> **Tip**
> - LangGraph는 MIT 라이선스가 적용된 오픈 소스 라이브러리로, 커뮤니티를 위해 이를 유지하고 성장시키는 데 전념하고 있습니다.
> - LangGraph 플랫폼을 사용하지 않고도 오픈 소스 LangGraph 프로젝트를 사용하여 자신의 인프라에 LangGraph 애플리케이션을 배포할 수 있습니다.


### High Level

- [Why LangGraph Platform?](./why_langgraph_platform.md): LangGraph 플랫폼은 LangGraph 애플리케이션을 배포하고 관리하기 위한 명확한 방식(opinionated way)을 제공합니다. 이 가이드는 LangGraph 플랫폼의 주요 기능과 개념에 대한 개요를 제공합니다.
- [Deployment Options](./deployment_options.md): LangGraph 플랫폼은 네 가지 배포 옵션을 제공합니다: Self-Hosted Lite, Self-Hosted Enterprise, bring your own cloud(BYOC), Cloud SaaS. 여기에서는 이러한 옵션들 간의 차이점과 각 플랜에서 사용 가능한 옵션을 설명합니다.
- Plan: LangGraph 플랫폼은 세 가지 다른 플랜을 제공합니다: Developer, Plus, Enterprise. 이 가이드는 이러한 옵션들 간의 차이점과 각 플랜에서 사용할 수 있는 배포 옵션, 가입 방법을 설명합니다.
- Template Applications: LangGraph를 사용하여 애플리케이션을 신속하게 구축하는 데 도움이 되는 참고 애플리케이션입니다.


### Components

LangGraph Platform은 LangGraph 애플리케이션의 배포 및 관리를 지원하기 위해 함께 작동하는 여러 구성 요소로 구성됩니다:

- [LangGraph Server](./langgraph_server.md): LangGraph 서버는 백그라운드 처리부터 실시간 상호작용까지 다양한 에이전트 애플리케이션 사용 사례를 지원하도록 설계되었습니다.
- [LangGraph Studio](./langgraph_studio.md): LangGraph 스튜디오는 LangGraph 서버에 연결하여 애플리케이션을 로컬에서 시각화, 상호작용 및 디버깅할 수 있는 특수화된 IDE입니다.
- [LangGraph CLI](./langgraph_cli.md): LangGraph CLI는 로컬 LangGraph와 상호작용하는 데 도움이 되는 command-line 인터페이스입니다.
- [Python/JS SDK](./langgraph_sdk.md): Python/JS SDK는 배포된 LangGraph 애플리케이션과 프로그래밍 방식으로 상호작용할 수 있는 방법을 제공합니다.
- [Remote Graph](../how_to/how_to_interact_with_the_deployment_using_remotegraph.md): 원격 그래프(Remote Graph)를 사용하면 로컬에서 실행되는 것처럼 배포된 LangGraph 애플리케이션과 상호작용할 수 있습니다.


### LangGraph Server

- [Application Structure](./application_structure.md): LangGraph 애플리케이션은 하나 이상의 그래프, LangGraph API Configuration 파일(langgraph.json), 종속성을 지정하는 파일과 환경 변수를 포함합니다.
- [Assistants](./assistant.md): Assistants는 LangGraph 애플리케이션의 다양한 구성을 저장하고 관리하는 방법입니다.
- [Web-hooks](./langgraph_server.md#webhooks): 웹훅은 실행 중인 LangGraph 애플리케이션이 특정 이벤트 시 외부 서비스에 데이터를 전송할 수 있게 합니다.
- [Cron Jobs](./langgraph_server.md#cron-jobs): 크론 작업은 LangGraph 애플리케이션에서 특정 시간에 작업을 예약하여 실행하는 방법입니다.
- [Double Texting](./double_texting.md): 이중 메시지는 그래프가 완료되기 전에 사용자가 여러 메시지를 보낼 때 LLM 애플리케이션에서 흔히 발생하는 문제입니다. 이 가이드는 LangGraph Deploy를 사용하여 이중 메시지를 처리하는 방법을 설명합니다.
- [Authentication & Access Control](./authentication_and_access_control.md): LangGraph 플랫폼을 배포할 때 인증 및 접근 제어 옵션에 대해 배울 수 있습니다.


### Deployment Options

- Self-Hosted Lite: 연간 최대 100만 노드 실행까지 무료로 사용할 수 있는 제한된 버전의 LangGraph 플랫폼으로, 로컬 또는 셀프 호스팅 방식으로 실행할 수 있습니다.
- Cloud SaaS: LangSmith의 일부로 호스팅됩니다.
- Bring Your Own Cloud: 인프라 관리를 저희가 하지만, 모든 인프라는 사용자의 클라우드에서 실행됩니다.
- Self-Hosted Enterprise: 완전히 사용자가 관리합니다.

