[![OpenAI Platform Assistants](https://img.shields.io/badge/OpenAI-Assistants-blue?logo=openai)](https://platform.openai.com/docs/assistants/overview)


# Assistants API overview
Assistants API 개요 - 베타
기본적인 도구와 통합 기능을 갖춘 AI Assistant 빌드

Assistants API를 통해 자신의 애플리케이션 내에서 AI Assistant를 구축할 수 있습니다. Assistant는 지침을 받아 모델, 도구, 파일을 활용해 사용자 질문에 답변할 수 있습니다. 현재 Assistants API는 세 가지 유형의 도구를 지원합니다: 코드 해석기(Code Interpreter), 파일 검색(File Search), 함수 호출(Function calling).

Assistants API의 기능은 Assistants playground를 통해 탐색하거나 Assistants API 퀵스타트에 따라 단계별 통합을 구축할 수 있습니다.

### Assistants 작동 방식
Assistants API는 다양한 작업을 수행할 수 있는 강력한 AI Assistant를 개발하도록 돕기 위해 설계되었습니다. Assistants API는 베타 버전이며, 우리는 기능을 계속 추가하고 있습니다. 피드백은 개발자 포럼에 공유해 주세요!

Assistant는 OpenAI의 모델을 특정 지침과 함께 호출하여 성격과 기능을 조정할 수 있습니다. Assistant는 여러 도구를 병렬로 액세스할 수 있습니다. 이러한 도구는 OpenAI에서 호스팅하는 도구(예: 코드 해석기, 파일 검색)일 수도 있고, 여러분이 직접 구축/호스팅하는 도구(함수 호출 방식)일 수도 있습니다.

Assistant는 지속 가능한 스레드(Thread)에 접근할 수 있습니다. 스레드는 메시지 기록을 저장하고 대화가 모델의 컨텍스트 길이에 맞지 않게 길어질 때 자동으로 잘라내어 AI 애플리케이션 개발을 단순화합니다. 스레드를 한 번 생성하고 사용자 응답에 따라 메시지를 추가하기만 하면 됩니다.

Assistant는 여러 형식의 파일에 접근할 수 있습니다. 도구를 사용할 때 Assistant는 파일(예: 이미지, 스프레드시트 등)을 생성하고 메시지에서 참조한 파일을 인용할 수도 있습니다.

### 객체 정의
| **OBJECT** | **WHAT IT REPRESENTS** |
|------------|-------------------------|
| **Assistant** | OpenAI의 모델을 사용하고 도구를 호출하는 목적 기반 AI |
| **Thread** | Assistant와 사용자 간의 대화 세션. 스레드는 메시지를 저장하고 모델의 컨텍스트에 맞게 내용을 자동으로 잘라냅니다. |
| **Message** | Assistant 또는 사용자가 생성한 메시지. 메시지는 텍스트, 이미지 및 기타 파일을 포함할 수 있습니다. 메시지는 스레드에 목록으로 저장됩니다. |
| **Run** | 스레드에서 Assistant를 호출하는 것. Assistant는 구성과 스레드의 메시지를 사용하여 도구와 모델을 호출해 작업을 수행합니다. 실행 과정에서 Assistant는 스레드에 메시지를 추가합니다. |
| **Run Step** | 실행 과정에서 Assistant가 수행한 단계를 자세히 나열한 것. Assistant는 실행 중 도구를 호출하거나 메시지를 생성할 수 있습니다. 실행 단계를 검토하여 Assistant가 최종 결과에 도달하는 방식을 내부적으로 분석할 수 있습니다. |

Assistants API에 대해 더 자세히 알고 싶으시면 [공식 문서](https://platform.openai.com/docs/assistants/overview)를 참조하세요.