[![AutoGen Core](https://img.shields.io/badge/AutoGen-Core-7fba00.svg?logo=data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAMgAAADICAYAAACtWK6eAAAACXBIWXMAAA7EAAAOxAGVKw4bAAACMklEQVR42u3aIU4DURiF0VsyqoKEqqaSsIO6LgBSh6jEsg7CEroEEnSDaJpQVYfCoFCILgAaNgA7mD6SeYY5R/8ZcTOfmkkAAAAAAACA3hmUHH1fn78mGZurc5vTp4/btoO7bRZJlqaqYn5/mbe2g6bwQeMkE3t2blRwM7R9NUff/xMbgUBAICAQEAgIBAQCAgGBgEAAgYBAQCAgEBAICAQEAgIBgYBAAIGAQEAgIBAQCAgEBAICAYEAAgGBgEBAICAQEAgIBAQCAgGBAAIBgYBAQCAgEBAICAQEAgIBBAICAYGAQEAgIBAQCAgEBAICAQQCAgGBgEBAICAQEAgIBAQCCAQEAgIBgYBAQCAgEBAICAQEAggEBAICAYGAQEAgIBD4R5rCu02Skbk691Jws0+yMlUVBxMAAAAAAAB9NSi6evxaJBmaq3P73Jzt2g5+nnORZGaqKtaDq3y2HZR+SV8mmdizc6skuyM3syQPpqpimrQH4l8sEAgIBAQCAgGBgEBAICAQEAggEBAICAQEAgIBgYBAQCAgEBAIIBAQCAgEBAICAYGAQEAgIBBAICAQEAgIBAQCAgGBgEBAICAQQCAgEBAICAQEAgIBgYBAQCCAQEAgIBAQCAgEBAICAYGAQEAggEBAICAQEAgIBAQCAgGBgEAAgYBAQCAgEBAICAQEAgIBgYBAAIGAQEAgIBAQCAgEBAICgZ5oCu/mf7il3KHgZp1kaqoq3k0AAAAAAAD01S9QnR3xthR2owAAAABJRU5ErkJggg==)](https://microsoft.github.io/autogen/stable/user-guide/core-user-guide/core-concepts/application-stack.html)


# Application Stack

AutoGen 코어는 다양한 멀티 에이전트 애플리케이션을 구축할 수 있도록 설계된 비의존적(unopinionated) 프레임워크입니다. 특정 에이전트 추상화나 멀티 에이전트 패턴에 종속되지 않으며, 유연하게 활용할 수 있습니다.

아래 다이어그램은 애플리케이션 스택을 보여줍니다.

![application_stack](../asset/application_stack.svg)

스택의 맨 아래에는 에이전트가 서로 통신할 수 있도록 지원하는 기본 메시징 및 라우팅 기능이 있습니다. 이 기능은 에이전트 런타임에 의해 관리되며, 대부분의 애플리케이션에서는 런타임에서 제공하는 고차원 API를 사용하여 상호작용하면 됩니다.(자세한 내용은 [이 가이드](./agent_and_agent_runtime.md)를 참고하세요.)

스택의 맨 위에서는 에이전트간 통신하는 메시지 유형을 개발자가 정의해야 합니다. 이 메시지 유형 세트는 에이전트가 따라야 하는 행동 계약(behavior contract)을 형성하며, 계약의 구현은 에이전트가 메시지를 처리하는 방식을 결정합니다. 행동 계약은 메시지 프로토콜이라고도 합니다. 동작 계약을 구현하는 것은 개발자의 책임입니다. 다중 에이전트 패턴은 이러한 행동 계약에서 비롯됩니다. (자세한 내용은 [이 가이드](./multi_agent_design_patterns.md)를 참고하세요.)


## An Example Application

코드 생성을 위한 다중 에이전트 애플리케이션의 구체적인 예를 고려해 봅시다. 이 애플리케이션은 세 개의 에이전트로 구성됩니다: Coder Agent, Executor Agent 및 Reviewer Agent. 아래 다이어그램은 에이전트 간의 데이터 흐름과 교환되는 메시지 유형을 보여줍니다.

![code_gen_example](../asset/code_gen_example.svg)

이 예에서 행동 계약은 다음으로 구성됩니다:
- **CodingTaskMsg**: 애플리케이션에서 Coder Agent로의 메시지
- **CodeGenMsg**: Coder Agent에서 Executor Agent로의 메시지
- **ExecutionResultMsg**: Executor Agent에서 Reviewer Agent로의 메시지
- **ReviewMsg**: Reviewer Agent에서 Coder Agent로의 메시지
- **CodingResultMsg**: Reviewer Agent에서 애플리케이션으로의 메시지

행동 계약(Behavior Contract)은 에이전트가 메시지를 처리하는 방식으로 구현됩니다. 예를 들어, Reviewer Agent는 `ExecutionResultMsg`를 수신하여 코드 실행 결과를 평가한 후, 승인 여부를 결정합니다. 만약 승인된다면, `CodingResultMsg`를 애플리케이션에 전송하고, 그렇지 않을 경우 `ReviewMsg`를 Coder Agent에게 보내 또 다른 코드 생성 단계를 진행합니다.

이런 행동 계약은 멀티 에이전트 패턴 중 하나인 *Reflection* 패턴의 사례입니다. 이 패턴에서는 생성된 결과를 다시 검토하고 추가 생성 단계를 거쳐 전체적인 품질을 향상시키는 과정을 포함합니다.
