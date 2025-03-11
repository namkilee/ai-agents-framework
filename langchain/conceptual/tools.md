[![LangChain Conceptual](https://img.shields.io/badge/LangChain-Conceptual-blue?logo=langchain)](https://python.langchain.com/docs/concepts/tools/)


# Tools


## Overview
​
LangChain에서 **tool** 추상화는 함수의 **이름**, **설명** 및 **예상 argument**를 정의한 스키마를 가진 Python **function** 입니다.

**Tools**는 [tool calling]()을 지원하는 [chat models]()에 전달될 수 있으며, 모델은 특정 입력값과 함께 특정 함수를 요청할 수 있습니다.

<br>

## Key concepts

* Tools는 함수와 해당 스키마를 캡슐화하여 채팅 모델에 전달될 수 있는 방식입니다.
* `@tool` 데코레이터를 사용해 tool를 생성하면, 다음과 같은 과정을 간소화할 수 있습니다:
  * tool의 이름, 설명 및 예상 argument를 자동으로 추론하며, 사용자 지정도 지원합니다.
  * 아티팩트(예: 이미지, 데이터프레임 등)를 반환하는 tool 정의.
  * 모델로부터 스키마의 입력 argument를 숨기는 **Injected tool arguments**.

<br>

## Tool interface

Tool 인터페이스는 `Runnable Interface`의 하위 클래스인 `BaseTool` 클래스에 정의되어 있습니다.

Tool 스키마와 관련된 주요 속성:

* `name`: tool의 이름.
* `description`: tool가 수행하는 작업에 대한 설명.
* `args`: tool의 argument를 위한 JSON 스키마를 반환하는 속성.

Tool과 연관된 함수를 실행하기 위한 주요 메서드:

* `invoke`: 주어진 argument를 사용해 tool를 호출합니다.
* `ainvoke`: 비동기적으로 주어진 argument를 사용해 tool를 호출합니다. [비동기 프로그래밍](./async_programming_with_langchain.md)을 사용합니다.

<br>

## Create tools using the `@tool` decorator

Tool 생성 시 가장 권장되는 방법은 [`@tool`](https://python.langchain.com/api_reference/core/tools/langchain_core.tools.convert.tool.html) 데코레이터를 사용하는 것입니다. 이 데코레이터는 tool 생성 과정을 간소화하며, 대부분의 경우에 사용해야 합니다. 함수를 정의한 후, 이를 `@tool`로 데코레이션하여 Tool Interface를 구현하는 tool을 생성할 수 있습니다.

```python
from langchain_core.tools import tool

@tool
def multiply(a: int, b: int) -> int:
   """두 숫자를 곱합니다."""
   return a * b
```
API Reference: `tool`

Tool 생성에 대한 자세한 내용은 [이 가이드](../how_to/how_to_create_tools.md)를 참조하세요.

> **NOTE**  
> LangChain에서는 tool를 생성하는 몇 가지 다른 방법도 제공합니다. 예: `BaseTool` 클래스의 하위 클래스를 만들거나 `StructuredTool`을 사용하는 방법. 이러한 방법은 [사용자 지정 tool 생성 가이드](../how_to/how_to_create_tools.md)에 설명되어 있지만, 대부분의 경우 `@tool` 데코레이터를 사용하는 것이 권장됩니다.

<br>

## Use the tool directly

tool을 정의한 후에는 함수 호출을 통해 직접 사용할 수 있습니다. 예를 들어, 위에서 정의한 `multiply` tool을 사용하려면 다음과 같이 호출하면 됩니다:

```python
multiply.invoke({"a": 2, "b": 3})
```


### Inspect

Tool의 스키마와 기타 속성을 확인할 수도 있습니다:

```python
print(multiply.name)  # multiply
print(multiply.description)  # Multiply two numbers.
print(multiply.args)
# {
#   'type': 'object',
#   'properties': {'a': {'type': 'integer'}, 'b': {'type': 'integer'}},
#   'required': ['a', 'b']
# }
```

> **NOTE**  
> 미리 구성된 LangChain 또는 LangGraph 의 `create_react_agent`를 사용하는 경우, tool과 직접 상호작용할 필요가 없을 수도 있습니다. 그러나 디버깅 및 테스트를 위해 tool 사용 방법을 이해하는 것은 유용할 수 있습니다. 또한 사용자 지정 LangGraph workflow를 만들 때 tool를 직접 사용하는 것이 필요할 수 있습니다.

<br>

## Configuring the schema

`@tool` 데코레이터는 tool의 스키마를 구성하기 위한 추가 옵션을 제공합니다(예: 이름, 설명 수정 또는 함수의 doc-string을 분석하여 스키마를 추론).

자세한 내용은 `@tool`의 API reference를 확인하고, 예제를 보려면 [이 가이드](../how_to/how_to_create_tools.md)를 확인하세요.

<br>

## Tool artifacts

Tools는 모델이 호출할 수 있는 유틸리티이며, 출력이 모델에 다시 전달되도록 설계되어 있습니다. 하지만 때로는 tool의 실행 결과로 생성된 아티팩트를 체인 또는 에이전트의 다운스트림 구성 요소에 제공하고 싶으면서도, 이를 모델 자체에는 노출하지 않으려는 경우가 있습니다. 예를 들어, tool이 사용자 지정 객체(데이터프레임 또는 이미지)를 반환하는 경우, 이 출력에 대한 메타데이터를 모델에 전달하면서도 실제 출력을 모델에 전달하지 않을 수 있습니다. 동시에, 이러한 전체 출력을 다른 tool 등에서 접근할 수 있게 하고 싶을 수도 있습니다.

```python
@tool(response_format="content_and_artifact")
def some_tool(...) -> Tuple[str, Any]:
    """특정 작업을 수행하는 tool."""
    ...
    return 'Message for chat model', some_artifact
```

자세한 내용은 [이 가이드](../how_to/how_to_return_artifacts_from_a_tool.md)를 확인하세요.

<br>

## Special type annotations

Tool의 함수에서 런타임 동작을 설정하기 위해 사용할 수 있는 몇 가지 특수 타입 annotation이 있습니다.

아래 타입 annotation은 tool 스키마에서 해당 argument를 제거하게 됩니다. 이는 모델에 노출되거나 모델이 제어해서는 안 되는 argument에 유용하게 사용될 수 있습니다.

* `InjectedToolArg`: `.invoke` 또는 `.ainvoke`를 사용하여 런타임에 수동으로 값을 삽입해야 합니다.
* `RunnableConfig`: `RunnableConfig` 객체를 tool에 전달합니다.
* `InjectedState`: LangGraph의 전체 상태를 tool에 전달합니다.
* `InjectedStore`: LangGraph 스토어 객체를 tool에 전달합니다.

또한, 문자열과 함께 `Annotated` 타입을 사용하여 tool의 스키마에 노출될 argument에 대한 설명을 제공할 수도 있습니다.

- `Annotated[..., "string literal"]` -- tool 스키마에 노출될 argument에 대한 설명을 추가합니다.


### `InjectedToolArg`

런타임 시 tool에 특정 argument를 전달해야 하지만, 이 argument를 모델이 직접 생성하지 않도록 해야 하는 경우가 있습니다. 이를 위해 `InjectedToolArg` annotation을 사용하면, 특정 매개변수를 tool 스키마에서 숨길 수 있습니다.

예를 들어, 런타임에 동적으로 `user_id`를 tool에 삽입해야 하는 경우는 다음과 같이 구성할 수 있습니다:

```python
from langchain_core.tools import tool, InjectedToolArg

@tool
def user_specific_tool(input_data: str, user_id: InjectedToolArg) -> str:
    """입력 데이터를 처리하는 tool."""
    return f"User {user_id} processed {input_data}"
```
API Reference: tool | InjectedToolArg

`InjectedToolArg`로 `user_id` argument를 annotation하면, LangChain은 이 argument를 tool 스키마의 일부로 노출하지 않도록 처리합니다.

자세한 내용은 [이 가이드](../how_to/how_to_pass_run_time_values_to_tools.md)를 참조하세요.


### `RunnableConfig`
`RunnableConfig` 객체를 사용해 tool에 사용자 지정 런타임 값을 전달할 수 있습니다.

Tool 내에서 `RunnableConfig` 객체에 접근하려면 함수 시그니처에서 `RunnableConfig` annotation을 사용할 수 있습니다:

```python
from langchain_core.runnables import RunnableConfig

@tool
async def some_func(..., config: RunnableConfig) -> ...:
    """특정 작업을 수행하는 tool."""
    # config를 사용해 작업 수행
    ...

await some_func.ainvoke(..., config={"configurable": {"value": "some_value"}})
```
API Reference : RunnableConfig

`config`는 tool 스키마의 일부가 아니며, 런타임에 적절한 값으로 삽입됩니다.

> **NOTE**  
> Python 3.9/3.10 환경에서 비동기 작업 시 `RunnableConfig` 객체를 수동으로 하위 호출에 전달해야 할 수도 있습니다. 
> 
> 자세한 내용은 [Propagation RunnableConfig 가이드](./runnable_interface.md#propagation-of-runnableconfig)를 참조하거나 Python 3.11로 업그레이드하세요.

---

### `InjectedState`
`InjectedState`에 대한 자세한 내용은 해당 [문서](https://langchain-ai.github.io/langgraph/reference/prebuilt/#langgraph.prebuilt.tool_node.InjectedState)를 참조하세요.

### `InjectedStore`
`InjectedStore`에 대한 자세한 내용은 해당 [문서](https://langchain-ai.github.io/langgraph/reference/prebuilt/#langgraph.prebuilt.tool_node.InjectedState)를 참조하세요.

<br>

## Best practices

모델에서 사용될 도구를 설계할 때 다음을 염두에 두세요:  

* 이름이 잘 지어지고, 정확히 문서화되며, 올바르게 타입 힌트가 지정된 도구는 모델이 더 쉽게 사용할 수 있습니다.  
* 단순하고 좁은 범위로 설계된 도구는 모델이 올바르게 사용하는 데 더 용이합니다.  
* 도구 호출 API를 지원하는 채팅 모델을 사용해 도구의 이점을 최대화하세요.  

<br>

## Toolkit

LangChain에는 "툴킷"이라는 개념이 있습니다. 이는 특정 작업을 위해 함께 설계된 도구들을 묶어주는 매우 간단한 추상화입니다.  

### Interface

모든 툴킷에는 `get_tools` 메서드가 있으며, 이를 통해 도구 목록을 반환받을 수 있습니다. 예를 들어:  

```python
# 툴킷 초기화
toolkit = ExampleToolkit(...)

# 도구 목록 가져오기
tools = toolkit.get_tools()
```  


