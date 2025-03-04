[![LangGraph Conceptual](https://img.shields.io/badge/LangGraph-Conceptual-blue?logo=langgraph)](https://langchain-ai.github.io/langgraph/concepts/langgraph_server/)


# LangGraph Server 

<br>

## Overview

LangGraph Server는 에이전트 기반 애플리케이션을 생성하고 관리하는 API를 제공합니다. 이 API는 특정 작업을 위해 구성된 에이전트(configured agents)인 [assistants](./assistant.md) 기반으로 만들어지며, 내장된 [persistence](./persistence.md)와 **작업 대기열(task queue)** 을 포함합니다. 이 다용도 API는 백그라운드 처리부터 실시간 상호작용에 이르는 다양한 에이전트 애플리케이션 사용 사례를 지원합니다.

<br>

## Key Features

LangGraph Platform은 에이전트 배포를 위한 검증된 방법론과 도구를 제공하여(incorporates best practices), 사용자가 에이전트 로직 개발에 집중할 수 있도록 지원합니다

- **Streaming endpoints**: [다양한 스트리밍 모드](./streaming.md)를 지원하는 엔드포인트입니다. 특히, 스트림 이벤트 간에 몇 분씩 간격이 생길 수 있는 장시간 실행되는 에이전트에서도 문제없이 작동하도록 설계되었습니다.

- **Background runs**: LangGraph Server는 어시스턴트를 백그라운드에서 실행할 수 있도록 지원하며, 실행 상태를 조회할 수 있는 엔드포인트와 실행 상태를 효과적으로 모니터링할 수 있는 웹훅을 제공합니다.

- **Support for long runs**: 어시스턴트를 실행을 위한 블로킹 엔드포인트는 정기적으로 하트비트 신호를 전송하여, 처리 시간이 오래 걸리는 요청에서도 예상치 못한 연결 종료를 방지합니다.

- **Task queue**: 요청이 갑작스럽게 몰리는 상황에서도 누락 없이 처리할 수 있도록 태스크 큐를 추가했습니다.

- **Horizontally scalable infrastructure**: LangGraph Server는 수평 확장이 가능하도록 설계되어, 필요에 따라 사용량을 유연하게 확장하거나 축소할 수 있습니다.

- **Double texting support**: 사용자와 그래프가 의도치 않게 상호작용하는 경우가 많습니다. 예를 들어, 첫 번째 메시지를 보낸 후 그래프가 실행을 완료하기 전에 두 번째 메시지를 보내는 상황(이를 "[이중 메시지(double texting)](./double_texting.md)"라고 부릅니다)에 대해 네 가지 처리 방식을 제공합니다.

- **최적화된 체크포인터**: LangGraph Platform은 LangGraph 애플리케이션에 최적화된 내장 [체크포인터(checkpointer)](./persistence.md#checkpoints)를 제공합니다.

- **Human-in-the-loop endpoints**: [Human-in-the-loop](./human_in_the_loop.md) 기능을 지원하기 위해 필요한 모든 엔드포인트를 제공합니다.

- **Memory**: LangGraph Platform은 스레드 수준의 지속성(위에서 언급한 [체크포인터](./persistence.md#checkpoints)) 외에도 내장 메모리 저장소를 제공합니다.

- **Cron jobs**: 정기적인 작업 스케줄링을 지원하여 데이터 정리나 배치 처리와 같은 작업을 애플리케이션 내에서 자동화할 수 있습니다.

- **Webhooks**: 애플리케이션이 외부 시스템에 실시간 알림과 데이터 업데이트를 전송할 수 있도록 하여, 타사 서비스와 쉽게 통합하고 특정 이벤트에 따라 작업을 트리거할 수 있습니다.

- **Monitoring**: LangGraph Server는 LangSmith monitoring platform과 원활하게 통합되어, 애플리케이션의 성능과 상태에 대한 실시간 인사이트를 제공합니다.

<br>

## What are you deploying?

LangGraph Server를 배포할 때 <span style="color: red;">**하나 이상의 그래프, 지속성을 위한 데이터베이스 및 작업 대기열**</span>을 배포합니다.


### Graph

LangGraph Server로 그래프를 배포할 때 [Assistant](./assistant.md)의 "청사진(blueprint)"을 배포합니다.

[Assistant](./assistant.md)는 특정 configuration setting과 결합된 그래프입니다. 동일한 그래프를 기반으로 다양한 용도를 위해 각각 다른 설정을 가진 여러 어시스턴트를 생성할 수 있습니다.

배포가 완료되면, LangGraph Server는 그래프의 기본 설정을 사용하여 각 그래프에 대한 기본 어시스턴트를 자동으로 생성합니다.

LangGraph Server API를 통해 어시스턴트와 상호작용할 수 있습니다.

> **Note**   
> 종종 그래프를 에이전트를 구현하는 도구로 생각하지만, 그래프가 반드시 에이전트를 구현해야 하는 것은 아닙니다. 예를 들어, 그래프는 단순한 챗봇을 구현할 수 있으며, 이는 단지 대화만 지원하고 애플리케이션의 제어 흐름에 영향을 미치지 않습니다. 실제로, 애플리케이션이 더 복잡해짐에 따라, 그래프는 여러 에이전트가 협력하여 작동하는 더 복잡한 흐름을 구현하는 경우가 많습니다.


### Persistence and Task Queue

LangGraph Server는 [영속성](./persistence.md)을 위한 데이터베이스와 작업 대기열을 활용합니다. 

현재 LangGraph Server의 데이터베이스로는 Postgres만 지원되며, 작업 대기열로는 Redis를 지원합니다. 

LangGraph 클라우드를 사용하여 배포하는 경우, 이러한 구성 요소는 자동으로 관리됩니다. LangGraph 서버를 자체 인프라에 배포하는 경우, 이러한 구성 요소를 직접 설정하고 관리해야 합니다. 

[배포 옵션 가이드](./deployment_options.md)를 검토하여 이러한 구성 요소가 어떻게 설정되고 관리되는지에 대한 자세한 내용을 확인하세요.

<br>

## Application Structure

LangGraph Server 애플리케이션을 배포하려면 배포할 그래프와 종속성, 그리고 환경 변수 같은 configuration settings 가 필요합니다. [애플리케이션 구조](./application_structure.md) 가이드를 읽어 LangGraph 애플리케이션을 배포하는 방법을 학습하세요.

<br>

## LangGraph Server API

LangGraph server API를 사용하면 [assistants](./langgraph_server.md#assistants), [threads](./langgraph_server.md#threads), [runs](./langgraph_server.md#runs), [cron jobs](./langgraph_server.md#cron-jobs) 등을 생성하고 관리할 수 있습니다. 

[LangGraph 클라우드 API](https://langchain-ai.github.io/langgraph/cloud/reference/api/api_ref.html#tag/assistants) 에서 API 엔드포인트 및 데이터 모델에 대한 자세한 정보를 제공합니다.


### Assistants

[어시스턴트](./assistant.md)는 특정 [graph](#graph)와 해당 그래프에 대한 [configuration](./langgraph_glossary.md#configuration) 설정을 함께 지칭하는 용어입니다.

어시스턴트는 [에이전트](./agent_architectures.md)의 saved configuration 이라고 생각할 수 있습니다.

에이전트를 만들 때 그래프 논리를 변경하지 않고, 빠르게 수정하는 경우가 흔히 발생합니다. 예를 들어, 프롬프트 또는 LLM 을 변경하는 것만으로도 에이전트의 동작에 상당한 영향을 미칠 수 있습니다. 어시스턴트는 이러한 에이전트 구성 변경을 쉽게 만들고 저장할 수 있는 방법을 제공합니다.


### Threads

스레드는 일련의 [runs](./langgraph_server.md#runs)에서 누적된 상태를 포함합니다. 특정 스레드에서 run이 수행되면, 어시스턴트의 기본 그래프 상태([state](./langgraph_glossary.md#state))가 해당 스레드에 저장(persist)됩니다.

스레드의 현재 상태와 과거 상태를 조회(retrieved)할 수 있습니다. 상태를 저장하려면 run을 수행하기 전에 스레드를 생성해야 합니다.

특정 시점에서 스레드의 상태를 [checkpoint](./persistence.md#checkpoints)라고 합니다. 체크포인트는 나중에 스레드의 상태를 복원하는 데 사용할 수 있습니다.

스레드와 체크포인트에 대한 자세한 내용은 [LangGraph 개념 가이드](./langgraph_glossary.md#persistence) 의 관련 섹션을 참조하세요.

LangGraph Cloud API에서 스레드 및 스레드 상태를 생성하고 관리하기 위한 여러 엔드포인트를 제공합니다. 자세한 내용은 [API 참조](https://langchain-ai.github.io/langgraph/cloud/reference/api/api_ref.html#tag/threads) 문서를 확인하세요.


### Runs

run은 [어시스턴트](./langgraph_server.md#assistants)를 호출하는 것 입니다. 각 run에는 own input, configuration 및 metadata 가 있을 수 있으며, 이는 기본 그래프의 실행 및 출력에 영향을 미칠 수 있습니다. run 선택적으로 [스레드](./langgraph_server.md#threads)에서 수행될 수 있습니다. 

LangGraph Cloud API에서 run의 생성 및 관리를 위한 여러 엔드포인트를 제공합니다. 자세한 내용은 [API 참조](https://langchain-ai.github.io/langgraph/cloud/reference/api/api_ref.html#tag/thread-runs/)를 참조하세요.


### Store

스토어는 모든 스레드에서 사용가능한 [key-value 저장소](./persistence.md#memory-store)를 관리하기 위한 API입니다. 

스토어는 LangGraph 애플리케이션에서 [메모리](./memory.md)를 구현하는 데 유용합니다.


### Cron Jobs

어시스턴트를 일정에 따라 실행하는 것이 유용한 상황이 많이 있습니다.

예를 들어, 일일 뉴스 요약 이메일을 보내는 어시스턴트를 매일 8:00 PM에 실행하려고 한다고 가정해 보세요. 

LangGraph 클라우드는 사용자 정의 일정에 따라 실행되는 크론 작업을 지원합니다. 사용자는 일정 및 어시스턴트 지정합니다. 이후, 지정된 일정에 따라 서버는 다음 작업을 수행합니다:

- 지정된 어시스턴트를 사용하여 새 스레드 생성
- 지정된 입력을 해당 스레드로 전송

스레드로 매번 동일한 입력을 전달하는것에 유념해야 합니다. 크론 작업 생성 방법에 대한 자세한 내용은 방법 [이 가이드](../how_to/cron_jobs.md)를 참조하세요. 

LangGraph 클라우드 API는 크론 작업 생성 및 관리를 위한 여러 엔드포인트를 제공합니다. 자세한 내용은 [API 참조](https://langchain-ai.github.io/langgraph/cloud/reference/api/api_ref.html#tag/runscreate/POST/threads/{thread_id}/runs/crons)를 참조하세요.


### Webhooks

웹훅은 LangGraph 클라우드 애플리케이션에서 외부 서비스로의 이벤트 기반 통신을 가능하게 합니다. 예를 들어, LangGraph 클라우드로 API 호출이 완료된 후 별도의 서비스에 업데이트를 수행하고 싶을 수 있습니다. 

많은 LangGraph 클라우드 엔드포인트는 `webhook` 매개변수를 전달받습니다. 이 매개변수가 POST 요청으로 전달되는 경우, LangGraph 클라우드는 실행 완료 시 요청을 보냅니다. 

자세한 내용은 [이 가이드](../how_to/use_webhooks.md)를 참조하세요.

