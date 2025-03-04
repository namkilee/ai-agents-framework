[![LangGraph Conceptual](https://img.shields.io/badge/LangGraph-Conceptual-blue?logo=langgraph)](https://langchain-ai.github.io/langgraph/concepts/platform_architecture/)


# LangGraph Platform Architecture

![langgraph_platform_architecture](../asset/langgraph_platform_deployment_architecture.png)

<br>

## How to use Postgres

Postgres는 LangGraph Platform의 모든 사용자 및 실행 데이터에 대한 영구 저장 계층입니다. 이는 체크포인트와 서버 리소스(스레드, 실행, 어시스턴트 및 crons)를 저장합니다. 더 많은 정보는 [여기](./persistence.md)에서 확인하세요

<br>

## How to use Redis

Redis는 각 LangGraph Platform 배포에서 Server 와 Queue Worker 간의 통신을 위해 사용되며, 일시적인 메타데이터를 저장합니다. 자세한 내용은 아래를 참조하세요. 사용자나 실행 데이터는 Redis에 저장되지 않습니다.


### Communication

LangGraph Platform의 모든 runs은 각 container의 백그라운드 worker pool에 의해 실행됩니다. run 의 일부 기능(예: 취소 및 출력 스트리밍)을 사용하기 위해선 서버와 특정 run 을 처리하는 worker 간의 양방향 통신 채널이 필요합니다. 우리는 Redis를 사용하여 이러한 통신을 처리합니다.

1. 새 run 이 생성되면 worker 를 깨우기 위한 메커니즘으로 Redis 목록을 사용합니다. 이 목록에는 센티널 값(sentinal vlaue)만 저장되며, 실제 run 정보는 저장되지 않습니다. Worker 는 Postgres에서 run 정보를 검색합니다.
2. Redis 문자열과 Redis PubSub 채널의 조합은 서버가 특정 worker에게 작업 취소 요청을 전달하는 데 사용됩니다.
3. Redis PubSub 채널은 Worker가 작업을 처리하는 동안 에이전트의 스트리밍 출력을 브로드캐스트하는 데 사용됩니다. 서버의 열려있는 `/stream` 요청은 해당 채널을 구독하며, 도착하는 이벤트를 응답으로 전달합니다. Redis에는 어떠한 이벤트도 저장되지 않습니다.


### Ephemeral metadata*

LangGraph Platform에서 run은 특정 오류(현재 실행 중 발생하는 일시적인 Postgres 오류만 해당)를 위해 재시도될 수 있습니다. 재시도 횟수를 제한하기 위해(현재 실행당 3회로 제한됨) Redis 문자열에 시도 횟수를 기록합니다. 이는 ID 외의 실행 관련 정보를 포함하지 않으며, 짧은 시간 후에 만료됩니다.


