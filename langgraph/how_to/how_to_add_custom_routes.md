[![LangGraph How to](https://img.shields.io/badge/LangGraph-How_to-yellow?logo=langgraph)](https://langchain-ai.github.io/langgraph/how-tos/http/custom_routes/)


# How to add custom routes

다음은 귀하가 요청한 내용을 자연스럽게 한국어로 번역한 것입니다:

---

### 사용자 정의 경로 추가 방법¶

LangGraph 플랫폼에서 에이전트를 배포할 때, 서버는 런(run) 및 스레드(thread) 생성, 장기 메모리 저장소와 상호작용, 구성 가능한 어시스턴트 관리, 기타 핵심 기능(모든 기본 API 엔드포인트 참조)을 위한 경로를 자동으로 노출합니다.

Starlette 앱(FastAPI, FastHTML 및 기타 호환 앱 포함)을 직접 제공함으로써 사용자 정의 경로를 추가할 수 있습니다. langgraph.json 설정 파일에 앱 경로를 지정하여 LangGraph 플랫폼에 이를 알릴 수 있습니다. (예: `"http": {"app": "path/to/app.py:app"}`).

사용자 정의 앱 객체를 정의하면 원하는 모든 경로를 추가할 수 있으며, /login 엔드포인트 추가부터 단일 LangGraph 배포에서 전체 풀스택 웹앱을 작성하는 것까지 가능합니다.

아래는 FastAPI를 사용하는 예제입니다.

---

### Python 전용
현재 우리는 `langgraph-api>=0.0.26` 이상의 Python 배포에서만 사용자 정의 인증 및 권한 부여를 지원합니다.

---

### 애플리케이션 생성¶

기존 LangGraph 플랫폼 애플리케이션에서 시작하거나 CLI를 사용하여 템플릿에서 새로운 애플리케이션을 생성할 수 있습니다.

```bash
langgraph new --template=new-langgraph-project-python my_new_project
```

LangGraph 프로젝트를 생성한 후, 아래와 같은 앱 코드를 추가하십시오:

```python
# ./src/agent/webapp.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/hello")
def read_root():
    return {"Hello": "World"}
```

---

### langgraph.json 구성¶

다음 코드를 `langgraph.json` 파일에 추가하세요. 경로가 위에서 생성한 `app.py` 파일을 올바르게 가리키는지 확인하십시오.

```json
{
  "dependencies": ["."],
  "graphs": {
    "agent": "./src/agent/graph.py:graph"
  },
  "env": ".env",
  "http": {
    "app": "./src/agent/webapp.py:app"
  }
  // 기타 구성 옵션 (예: 인증, 저장소 등).
}
```

---

### 서버 시작¶

로컬에서 서버를 테스트하세요:

```bash
langgraph dev --no-browser
```

브라우저에서 `localhost:2024/hello`로 이동하면(2024는 기본 개발 포트) `{"Hello": "World"}`를 반환하는 `hello` 엔드포인트를 확인할 수 있습니다.

---

### 기본 엔드포인트 덮어쓰기

앱에서 생성한 경로는 시스템 기본값보다 우선순위가 높아지므로, 기본 엔드포인트의 동작을 덮어쓰거나 재정의할 수 있습니다.

---

### 배포¶

위 설정 그대로 관리되는 LangGraph 클라우드나 자체 호스팅 플랫폼에 앱을 배포할 수 있습니다.

---

### 다음 단계¶

사용자 정의 경로를 배포에 추가한 후, 유사한 기법을 사용하여 서버 동작을 추가로 사용자 정의할 수 있습니다. 예를 들어, 사용자 정의 미들웨어 및 수명 이벤트를 정의할 수 있습니다.

---

더 궁금한 점이 있다면 말씀해주세요! 😊
