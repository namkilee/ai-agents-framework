[![AutoGen Core](https://img.shields.io/badge/AutoGen-Core-7fba00.svg?logo=data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAMgAAADICAYAAACtWK6eAAAACXBIWXMAAA7EAAAOxAGVKw4bAAACMklEQVR42u3aIU4DURiF0VsyqoKEqqaSsIO6LgBSh6jEsg7CEroEEnSDaJpQVYfCoFCILgAaNgA7mD6SeYY5R/8ZcTOfmkkAAAAAAACA3hmUHH1fn78mGZurc5vTp4/btoO7bRZJlqaqYn5/mbe2g6bwQeMkE3t2blRwM7R9NUff/xMbgUBAICAQEAgIBAQCAgGBgEAAgYBAQCAgEBAICAQEAgIBgYBAAIGAQEAgIBAQCAgEBAICAYEAAgGBgEBAICAQEAgIBAQCAgGBAAIBgYBAQCAgEBAICAQEAgIBBAICAYGAQEAgIBAQCAgEBAICAQQCAgGBgEBAICAQEAgIBAQCCAQEAgIBgYBAQCAgEBAICAQEAggEBAICAYGAQEAgIBD4R5rCu02Skbk691Jws0+yMlUVBxMAAAAAAAB9NSi6evxaJBmaq3P73Jzt2g5+nnORZGaqKtaDq3y2HZR+SV8mmdizc6skuyM3syQPpqpimrQH4l8sEAgIBAQCAgGBgEBAICAQEAggEBAICAQEAgIBgYBAQCAgEBAIIBAQCAgEBAICAYGAQEAgIBBAICAQEAgIBAQCAgGBgEBAICAQQCAgEBAICAQEAgIBgYBAQCCAQEAgIBAQCAgEBAICAYGAQEAggEBAICAQEAgIBAQCAgGBgEAAgYBAQCAgEBAICAQEAgIBgYBAAIGAQEAgIBAQCAgEBAICgZ5oCu/mf7il3KHgZp1kaqoq3k0AAAAAAAD01S9QnR3xthR2owAAAABJRU5ErkJggg==)](https://microsoft.github.io/autogen/stable/user-guide/core-user-guide/core-concepts/agent-and-multi-agent-application.html)


# Agent and Multi-Agent Applications

에이전트는 메시지를 통해 통신하고, 자신의 상태를 유지하며, 수신한 메시지나 상태의 변경에 따라 작업을 수행하는 소프트웨어 엔터티입니다. 이런 작업은 에이전트의 상태를 수정하고 메시지 로그 업데이트, 새로운 메시지 전송, 코드 실행 또는 API 호출과 같은 외부와의 통신을 실행할 수 있습니다.

많은 소프트웨어 시스템은 상호 작용하는 독립 에이전트의 집합으로 모델링할 수 있습니다. 예를 들면 다음과 같습니다:
- 공장 바닥의 센서
- 웹 애플리케이션을 지원하는 분산 서비스
- 여러 이해 관계자가 관련된 비즈니스 워크플로우
- 코드 작성, 외부 시스템과의 인터페이스, 다른 에이전트와의 통신을 수행할 수 있는 언어 모델(GPT-4 등)로 구동되는 AI 에이전트

이렇게 여러 에이전트와 상호 작용하도록 구성된 시스템을 **다중 에이전트 애플리케이션**이라고 합니다. 

> **Note**  
> AI 에이전트는 일반적으로 소프트웨어 스택의 일부로 언어 모델을 사용하여 메시지를 해석하고, 추론하며, 작업을 실행합니다.


## Characteristics of Multi-Agent Applications

다중 에이전트 애플리케이션에서 에이전트는 아마도
- 동일한 프로세스 내 또는 동일한 기계에서 실행될 수 있습니다
- 다른 기계나 조직 경계를 넘어 운영될 수 있습니다
- 다양한 프로그래밍 언어로 구현되고 다른 AI 모델이나 지침을 사용할 수 있습니다
- 메시지를 통해 작업을 조정하여 공동의 목표를 향해 협력할 수 있습니다

각 에이전트는 독립적으로 개발, 테스트 및 배포할 수 있는 자립형 단위입니다. 이 모듈형 설계는 에이전트를 다양한 시나리오에서 재사용하고 더 복잡한 시스템으로 구성할 수 있게 합니다. 

에이전트는 본질적으로 **결합하여 동작 가능**하기에, 전체 시스템의 특정 기능이나 서비스에 기여하는 단순한 에이전트를 결합하여 복잡하고 적응 가능한 애플리케이션으로 만들 수 있습니다.
