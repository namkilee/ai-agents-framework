[![LangGraph Conceptual](https://img.shields.io/badge/LangGraph-Conceptual-blue?logo=langgraph)](https://langchain-ai.github.io/langgraph/concepts/langgraph_platform/)


# LangGraph Platform


## Overview

LangGraph Platform은 오픈 소스 LangGraph 프레임워크를 기반으로 구축된, production 환경에서 에이전트 애플리케이션을 배포하기 위한 상업적 솔루션입니다.

LangGraph Platform은 LangGraph 애플리케이션의 개발, 배포, 디버깅 및 모니터링을 지원하기 위해 함께 작동하는 여러 구성 요소로 구성됩니다:

- [LangGraph Server](./langgraph_server.md): 서버는 에이전트 애플리케이션을 배포하기 위해 명확한 API와 아키텍처를 제공합니다. 이를 통해 서버 인프라 개발에 신경 쓰지 않고 에이전트 로직 구현에 집중할 수 있습니다.
- [LangGraph Studio](./langgraph_studio.md): LangGraph 스튜디오는 LangGraph 서버에 연결하여 애플리케이션을 로컬에서 시각화, 상호작용 및 디버깅할 수 있는 특수화된 IDE입니다.
- [LangGraph CLI](./langgraph_cli.md): LangGraph CLI는 로컬 LangGraph와 상호작용하는 데 도움이 되는 command-line 인터페이스입니다.
- [Python/JS SDK](./langgraph_sdk.md): Python/JS SDK는 배포된 LangGraph 애플리케이션과 프로그래밍 방식으로 상호작용할 수 있는 방법을 제공합니다.
- [Remote Graph](../how_to/how_to_interact_with_the_deployment_using_remotegraph.md): 원격 그래프(Remote Graph)를 사용하면 로컬에서 실행되는 것처럼 배포된 LangGraph 애플리케이션과 상호작용할 수 있습니다.

![lg_platform](../asset/lg_platform.png)

LangGraph Platform은 배포 옵션 가이드에서 설명된 몇 가지 다른 배포 옵션을 제공합니다.

## Why Use LangGraph Platform?

LangGraph 플랫폼은 LLM 애플리케이션을 프로덕션으로 배포할 때 발생하는 일반적인 문제를 처리하여 서버 인프라를 관리하는 대신 에이전트 로직에 집중할 수 있게 합니다.

- **스트리밍 지원**: 에이전트가 더욱 정교해짐에 따라 토큰 출력 및 중간 상태를 사용자에게 스트리밍하는 것이 유리해집니다. 스트리밍 없이 사용자는 피드백 없이 잠재적으로 긴 작업을 기다려야 합니다. LangGraph 서버는 다양한 애플리케이션 요구에 최적화된 여러 스트리밍 모드를 제공합니다.
- **백그라운드 실행**: 처리 시간이 길어지는 에이전트의 경우(예: 몇 시간), 열린 연결을 유지하는 것이 비실용적일 수 있습니다. LangGraph 서버는 백그라운드에서 에이전트 실행을 지원하며 실행 상태를 효과적으로 모니터링하기 위해 폴링 엔드포인트와 웹훅을 모두 제공합니다.
- **긴 실행 지원**: 기본 서버 설정은 긴 시간 동안 완료되는 요청을 처리할 때 종종 타임아웃이나 중단 문제를 겪습니다. LangGraph 서버의 API는 정기적인 하트비트 신호를 보내 장기 프로세스 동안 예상치 못한 연결 종료를 방지하여 이러한 작업을 견고하게 지원합니다.
- **버스트 처리**: 특히 실시간 사용자 상호작용이 있는 애플리케이션에서는 많은 요청이 동시에 서버에 몰리는 "버스트" 요청 로드를 경험할 수 있습니다. LangGraph 서버는 작업 큐를 포함하여 많은 로드에도 요청이 손실 없이 일관되게 처리되도록 합니다.
- **이중 메시지**: 사용자 주도 애플리케이션에서는 사용자가 여러 메시지를 빠르게 보낼 수 있는 경우가 많습니다. 이러한 "이중 메시지"는 적절히 처리되지 않으면 에이전트 흐름을 방해할 수 있습니다. LangGraph 서버는 이러한 상호작용을 처리하고 관리하는 내장 전략을 제공합니다.
- **체크포인터 및 메모리 관리**: 지속성이 필요한 에이전트의 경우(예: 대화 메모리), 견고한 저장 솔루션을 배포하는 것이 복잡할 수 있습니다. LangGraph 플랫폼은 최적화된 체크포인터와 메모리 저장소를 포함하여 세션 간 상태를 관리하고 사용자 정의 솔루션 없이 이를 처리합니다.
- **인간-중심 루프 지원**: 많은 애플리케이션에서는 사용자가 에이전트 프로세스에 개입할 방법을 필요로 합니다. LangGraph 서버는 인간-중심 루프 시나리오를 위한 특수 엔드포인트를 제공하여 에이전트 워크플로우에 수동 감독을 통합하는 과정을 단순화합니다.

LangGraph 플랫폼을 사용하면 이러한 문제를 완화하는 견고하고 확장 가능한 배포 솔루션에 액세스할 수 있으며, 이를 수동으로 구현하고 유지 관리하는 노력을 절약할 수 있습니다. 이를 통해 배포 인프라 문제를 해결하는 대신 효과적인 에이전트 행동을 구축하는 데 집중할 수 있습니다.

더 궁금한 점이 있으시면 언제든지 말씀해 주세요! 😊
