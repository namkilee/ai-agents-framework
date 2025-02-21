[![AutoGen Core](https://img.shields.io/badge/AutoGen-Core-7fba00.svg?logo=data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAMgAAADICAYAAACtWK6eAAAACXBIWXMAAA7EAAAOxAGVKw4bAAACMklEQVR42u3aIU4DURiF0VsyqoKEqqaSsIO6LgBSh6jEsg7CEroEEnSDaJpQVYfCoFCILgAaNgA7mD6SeYY5R/8ZcTOfmkkAAAAAAACA3hmUHH1fn78mGZurc5vTp4/btoO7bRZJlqaqYn5/mbe2g6bwQeMkE3t2blRwM7R9NUff/xMbgUBAICAQEAgIBAQCAgGBgEAAgYBAQCAgEBAICAQEAgIBgYBAAIGAQEAgIBAQCAgEBAICAYEAAgGBgEBAICAQEAgIBAQCAgGBAAIBgYBAQCAgEBAICAQEAgIBBAICAYGAQEAgIBAQCAgEBAICAQQCAgGBgEBAICAQEAgIBAQCCAQEAgIBgYBAQCAgEBAICAQEAggEBAICAYGAQEAgIBD4R5rCu02Skbk691Jws0+yMlUVBxMAAAAAAAB9NSi6evxaJBmaq3P73Jzt2g5+nnORZGaqKtaDq3y2HZR+SV8mmdizc6skuyM3syQPpqpimrQH4l8sEAgIBAQCAgGBgEBAICAQEAggEBAICAQEAgIBgYBAQCAgEBAIIBAQCAgEBAICAYGAQEAgIBBAICAQEAgIBAQCAgGBgEBAICAQQCAgEBAICAQEAgIBgYBAQCCAQEAgIBAQCAgEBAICAYGAQEAggEBAICAQEAgIBAQCAgGBgEAAgYBAQCAgEBAICAQEAgIBgYBAAIGAQEAgIBAQCAgEBAICgZ5oCu/mf7il3KHgZp1kaqoq3k0AAAAAAAD01S9QnR3xthR2owAAAABJRU5ErkJggg==)](https://microsoft.github.io/autogen/stable/user-guide/core-user-guide/core-concepts/architecture.html)


# Agent Runtime Environments

기본적으로 프레임워크는 에이전트 간 통신을 용이하게 하고, 에이전트의 신원과 라이프사이클을 관리하며, 보안 및 프라이버시 경계를 강제하는 런타임 환경을 제공합니다.

이 프레임워크는 두 가지 유형의 런타임 환경을 지원합니다: 독립 실행형 및 분산형. 두 유형 모두 다중 에이전트 애플리케이션을 구축하기 위한 공통 API 세트를 제공하므로 에이전트 구현을 변경하지 않고도 전환할 수 있습니다. 각 유형은 여러 가지 구현을 가질 수도 있습니다.


## Standalone Agent Runtime

독립 실행형 런타임은 모든 에이전트가 동일한 프로그래밍 언어로 구현되고 동일한 프로세스에서 실행되는 단일 프로세스 애플리케이션에 적합합니다. Python API에서 독립 실행형 런타임의 예는 [`SingleThreadedAgentRuntime`](https://microsoft.github.io/autogen/stable/reference/python/autogen_core.html#autogen_core.SingleThreadedAgentRuntime)입니다.

![Standalone Runtime](../asset/architecture_standalone.svg)

여기에서 에이전트는 런타임을 통해 메시지를 통해 통신하며, 런타임은 에이전트의 라이프사이클을 관리합니다.

개발자는 라우팅된 에이전트, AI 모델 클라이언트, AI 모델 도구, 코드 실행 샌드박스, 모델 컨텍스트 저장소 등 제공된 구성 요소를 사용하여 에이전트를 신속하게 구축할 수 있습니다. 또한, 처음부터 자신의 에이전트를 구현하거나 다른 라이브러리를 사용할 수도 있습니다.


## Distributed Agent Runtime

분산형 런타임은 에이전트가 다른 프로그래밍 언어로 구현되고 다른 기계에서 실행될 수 있는 다중 프로세스 애플리케이션에 적합합니다.

![Distributed Runtime](../asset/architecture_distributed.svg)

위의 다이어그램에서 보듯이 분산형 런타임은 호스트 서비스와 여러 작업자로 구성됩니다. 호스트 서비스는 작업자 간의 에이전트 통신을 용이하게 하고 연결 상태를 유지합니다. 작업자는 에이전트를 실행하고 게이트웨이를 통해 호스트 서비스와 통신합니다. 작업자는 실행하는 에이전트를 호스트 서비스에 광고하고 에이전트의 라이프사이클을 관리합니다.

에이전트는 독립 실행형 런타임에서와 동일한 방식으로 작동하여 개발자가 에이전트 구현을 변경하지 않고도 두 런타임 유형 간에 전환할 수 있습니다.

이 설명이 도움이 되었기를 바랍니다. 더 알고 싶은 점이 있나요?