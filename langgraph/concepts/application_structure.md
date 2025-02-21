[![LangGraph Conceptual](https://img.shields.io/badge/LangGraph-Conceptual-blue?logo=langgraph)](https://langchain-ai.github.io/langgraph/concepts/application_structure/)


# Application Structure

<br>

## Overview

LangGraph 애플리케이션은 하나 이상의 그래프, LangGraph API 구성 파일(`langgraph.json`), 종속성 파일, 선택적으로 환경 변수를 지정하는 `.env` 파일로 구성됩니다.

이 가이드는 LangGraph 애플리케이션의 전형적인 구조를 보여주며 LangGraph 플랫폼을 사용하여 LangGraph 애플리케이션을 배포하는 데 필요한 정보를 설정하는 방법을 설명합니다.

<br>

## Key Concepts

LangGraph 플랫폼을 사용하여 배포하려면 다음 정보를 제공해야 합니다:

- [LangGraph API 설정 파일](#configuration-file) (`langgraph.json`)은 애플리케이션에 사용할 종속성, 그래프, 환경 변수를 지정합니다.
- [그래프](#graphs)는 애플리케이션의 로직을 구현합니다.
- 애플리케이션을 실행하는 데 필요한 [종속성](#dependencies) 파일입니다.
- 애플리케이션 실행에 필요한 [환경 변수](#environment-variables)입니다.

<br>

## File Structure

아래는 Python 및 JavaScript 애플리케이션의 디렉토리 구조 예시입니다:

**Python (requirements.txt)**

```
my-app/
├── my_agent # 모든 프로젝트 코드는 여기 있습니다
│   ├── utils # 그래프 유틸리티
│   │   ├── __init__.py
│   │   ├── tools.py # 그래프 도구
│   │   ├── nodes.py # 그래프 노드 함수
│   │   └── state.py # 그래프 상태 정의
│   ├── __init__.py
│   └── agent.py # 그래프를 구성하는 코드
├── .env # 환경 변수
├── requirements.txt # 패키지 종속성
└── langgraph.json # LangGraph 구성 파일
```

> **Note** \
> LangGraph 애플리케이션의 디렉토리 구조는 사용하는 프로그래밍 언어와 패키지 관리자에 따라 달라질 수 있습니다.

<br>

## Configuration File

`langgraph.json` 파일은 LangGraph 애플리케이션을 배포하는 데 필요한 종속성, 그래프, 환경 변수 및 기타 설정을 지정하는 JSON 파일입니다. 

이 파일은 다음 정보를 지정할 수 있습니다:

| 키             | 설명                                                                                                     |
|----------------|----------------------------------------------------------------------------------------------------------|
| `dependencies` | **Required.** LangGraph API 서버의 종속성 배열입니다. 종속성은 다음 중 하나일 수 있습니다: (1) `"."`, 로컬 Python 패키지를 찾습니다, (2) 앱 디렉토리의 `./local_package`에 있는 `pyproject.toml`, `setup.py` 또는 `requirements.txt`, (3) 패키지 이름 |
| `graphs`       | **Required.** 컴파일된 그래프나 그래프를 만드는 함수가 정의된 경로 매핑입니다. 예: <ul><li> **Complied Graph**: `./your_package/your_file.py:variable`, 여기서 `variable`은 `langgraph.graph.state.CompiledStateGraph`의 인스턴스입니다.</li><li> **Function**: `./your_package/your_file.py:make_graph`, 여기서 `make_graph`는 구성 딕셔너리(`langchain_core.runnables.RunnableConfig`)를 가지는 `langgraph.graph.state.StateGraph` / `langgraph.graph.state.CompiledStateGraph`의 인스턴스 함수입니다.</li></ul> |
| `env`          | `.env` 파일 경로 또는 환경 변수와 그 값을 나타낼 수 있습니다.                                                |
| `python_version` | 3.11 또는 3.12. 기본값은 3.11입니다.                                                                    |
| `pip_config_file` | pip config 파일 경로입니다.                                                                              |
| `dockerfile_lines` | 부모 이미지에서 가져온 후 Dockerfile에 추가할 라인의 배열입니다.                                        |

> **Tip** \
> LangGraph CLI는 기본적으로 현재 디렉토리의 `langgraph.json` 구성 파일을 사용합니다.


### Examples

**Python**

* 종속성에는 사용자 정의 로컬 패키지와 `langchain_openai` 패키지가 포함됩니다.
* 단일 그래프는 `./your_package/your_file.py` 파일에서 변수 `variable`을 통해 로드됩니다.
* 환경 변수는 .env 파일에서 로드됩니다.

```json
{
    "dependencies": [
        "langchain_openai",
        "./your_package"
    ],
    "graphs": {
        "my_agent": "./your_package/your_file.py:agent"
    },
    "env": "./.env"
}
```

<br>

## Dependencies

LangGraph 애플리케이션은 다른 Python 패키지나 JavaScript 라이브러리에 의존할 수 있습니다(애플리케이션이 작성된 프로그래밍 언어에 따라 다름).

종속성을 올바르게 설정하려면 일반적으로 다음 정보를 지정해야 합니다:

1. 디렉토리내에 종속성을 지정하는 파일(예: requirements.txt, pyproject.toml 또는 package.json).
2. [LangGraph configuratino file](#configuration-file)내에 LangGraph 애플리케이션을 실행하는 데 필요한 종속성이 적힌 `dependencies` 키.
3. 추가적인 바이너리 또는 시스템 라이브러리는 [LangGraph configuratino file](#configuration-file)내의 `dockerfile_lines` 키를 사용하여 지정 가능.

<br>

## Graphs

배포된 LangGraph 애플리케이션에서 사용할 그래프를 지정하려면 LangGraph 구성 파일의 `graphs` 키를 사용합니다.

구성 파일에서 하나 이상의 그래프를 지정할 수 있습니다. 각 그래프는 이름(고유해야 함)과 다음 중 하나의 경로로 구분됩니다:
(1) 컴파일된 그래프 또는 (2) 그래프를 생성하는 함수가 정의된 경로.

<br>

## Environment Variables
로컬에서 배포된 LangGraph 애플리케이션을 사용할 때, LangGraph 구성 파일의 `env` 키에 환경 변수를 구성할 수 있습니다.

프로덕션 배포의 경우, 일반적으로 배포 환경에서 환경 변수를 구성하고자 할 것입니다.
