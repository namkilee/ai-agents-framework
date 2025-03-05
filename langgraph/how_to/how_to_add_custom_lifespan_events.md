[![LangGraph How to](https://img.shields.io/badge/LangGraph-How_to-yellow?logo=langgraph)](https://langchain-ai.github.io/langgraph/how-tos/http/custom_lifespan/)


# How to add custome lifespan events

LangGraph Platform에서 에이전트를 배포하는 경우 서버가 시작될 때 데이터베이스 연결과 같은 리소스를 초기화하고 서버가 종료될 때 적절히 닫히도록 해야 할 필요가 있습니다. Lifespan events는 서버의 시작 및 종료 시퀀스에 연결하여 이러한 중요한 설정 및 정리 작업을 처리할 수 있도록 합니다.

이 작업은 [custom routes 를 추가](./how_to_add_custom_routes.md)하는 것과 동일한 방식으로 작동하며, FastAPI, FastHTML 등과 같은 Starlette 호환 앱을 제공하기만 하면 됩니다.

아래는 FastAPI를 사용한 예제입니다.

> **Python 전용**  
> 현재 `langgraph-api>=0.0.26` 이상의 Python 에서만 사용자 정의 lifespan events를 지원합니다.

<br>

## Create app

기존 LangGraph Platform 애플리케이션에서 시작한다면, 아래 lifespan 코드를 `webapp.py` 에 추가하세요. 만약 새로 시작한다면, CLI를 사용하여 템플릿에서 새로운 애플리케이션을 생성할 수 있습니다.

```bash
langgraph new --template=new-langgraph-project-python my_new_project
```

LangGraph 프로젝트를 생성한 후, 아래와 같은 앱 코드를 추가하십시오:

```python
# ./src/agent/webapp.py
from contextlib import asynccontextmanager
from fastapi import FastAPI
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

@asynccontextmanager
async def lifespan(app: FastAPI):
    # 예를 들어...
    engine = create_async_engine("postgresql+asyncpg://user:pass@localhost/db")
    # 재사용 가능한 세션 팩토리 생성
    async_session = sessionmaker(engine, class_=AsyncSession)
    # 앱 상태에 저장
    app.state.db_session = async_session
    yield
    # 연결 정리
    await engine.dispose()

app = FastAPI(lifespan=lifespan)

# 필요에 따라 사용자 정의 경로 추가 가능.
```

<br>

## Configure `langgraph.json`

다음 코드를 `langgraph.json` 파일에 추가하세요. 위에서 생성한 `webapp.py` 파일 경로 확인하세요.

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

<br>

## Start sever

로컬에서 서버를 테스트하세요:

```bash
langgraph dev --no-browser
```

서버가 시작될 때 시작 메시지가 출력되고, `Ctrl+C`로 종료하면 정리 메시지가 출력되는 것을 확인할 수 있습니다.

<br>

## Deploying

위 설정 그대로 관리되는 LangGraph 클라우드나 자체 호스팅 플랫폼에 앱을 배포할 수 있습니다.

<br>

## Next steps

Lifespan events를 추가한 후, 유사한 기법을 사용하여 사용자 정의 경로 또는 미들웨어를 추가하여 서버 동작을 더욱 사용자 정의할 수 있습니다.
