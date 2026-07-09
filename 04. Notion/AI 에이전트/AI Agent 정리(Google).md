Google 에서 발표한 AI Agent에 대한 PDF 파일을 정리한 문서이다. 아래는 원본 문서이다.  
[https://www.youtube.com/watch?v=HujQhD8J2LQ](https://www.youtube.com/watch?v=HujQhD8J2LQ)

![[google_agent.pdf]]

- [[#1. Agent 란?]]
    - [[#1.1. Agent 개념 및 구조]]
    - [[#1.2. Orchestration ]]
- [[#2. Models vs Agent]]
- [[#3. Agent 가 동작하는 방식]]
    - [[#3.1. ReAct(Reasoning + Acting)]]
    - [[#3.2. CoT(Chain of Thought)]]
    - [[#3.3. ToT(Tree of Thoughts)]]
- [[#4. Tools]]
    - [[#4.1. Extensions]]
    - [[#4.2. Functions]]
- [[#5. Data stores(RAG)]]
    - [[#5.1. RAG 개념 및 구조]]
- [[#6. 요약]]
    - [[#6.1. Tools 요약]]

---

## 1. Agent 란?

### 1.1. Agent 개념 및 구조

- 기존의 AI는 사람이 AI 에게 요구를 하여 답변을 받는 구조였다.
    - 예) 요약해줘, 설명해줘, 등등
- 하지만 Agent란 사람의 간섭이 없이 자기 스스로 여러 단계를 거치며 생각을 하며 맞는 결정을 해서 결과물을 도출해낸다.

    ![[image 27.png|image 27.png]]


### 1.2. Orchestration

- `Agent`가 어떻게 **==정보(information)을 받아들이고 어떻게 생각을 해서 다음 결정을 내리는지 구조들과 논리들이 짜여져 있는 계층==**이다.

---

## 2. Models vs Agent

|**구분**|**Models**|**Agents**|
|---|---|---|
|데이터 소스|사전 학습 데이터만 사용|학습 데이터 + 외부 지식 연결|
|처리 방식|단순 질문/답변 도출|여러 단계를 거쳐 생각하고 결정|
|Tools 연동|별도 Tools 연동 없음|외부 Tools 연동|
|로직 구조|단순 응답 로직|`ReAct`, `CoT`, `ToT` 를 활용하여 추론(생각)하는 로직|

---

## 3. Agent 가 동작하는 방식

크게 `Chain of Thought(CoT)` , `Tree of Thoughts(ToT)`, `Reasoning + Acting(ReAct)` 가 있다.

### 3.1. ReAct(Reasoning + Acting)

- **개념** : `ReAct` 는 `Reasoning(추론)` 과 `Acting(행동)`을 결합한 프레임워크로, **==모델이 문제를 해결하는 과정에서 추론과 행동을 반복적으로 수행==**
    - 모델이 먼저 추론(생각 단계)를 하고, 이 추론에 기반한 행동(DB 쿼리, 정보검색 등)을 실행한다.
    - 이 ==**과정을 반복하여 최종 해결책에 도달**==한다.
- **사용사례** : _==주로 외부도구(API, DB)와 상호작용이 필요한 문제 해결상황에서 사용==_된다.
- **장점** : 모델이 외부 데이터를 동적으로 가져오며 다단계 문제를 해결할 수 있음.


### 3.2. CoT(Chain of Thought)

- **개념** : `CoT` 는 모델이 문제를 해결할 때, **==답변을 바로 도출하지 않고 단계별 추론 과정을 "생각을 말로 표현" 하듯이 작성하게 만드는 프롬프팅 기법==**이다.
- **작동방식** : ==**모델이 중간 추론 단계를 거쳐 최종 답을 도출하도록 유도**==한다.
    - 예: "엘리스가 사과 3개를 갖고 있고, 존에게 2개를 주었다면 몇개가 남나요?" → "앨리스는 처음에 사과를 3개 가지고있다. 2개를 존에게 준다. 3-1=1 따라서 엘리스는 사과 1개가 남는다"
- **사용사례** : 수학 문제, 논리적추론, 답변 과정이 명화개향 하는 상황에 적합
- **장점** : 모델이 중간 과정을 거쳐 생각을 명확히 정희하므로 정확도가 향상된다.


### 3.3. ToT(Tree of Thoughts)

- **개념** : 의사결정 트리에서 영감을 받은 프레임워크로, **==모델이 문제를 해결하는 과정에서 여러 가능성을 나무 구조로 분기하여 탐색==**한다.
    - 각각의 `Branch(가지)`는 **==문제를 해결하는 하나의 잠재적 경로==**를 나타낸다.
    - 다양한 경로를 탐색한 후, 정확도나 일관성 기준에 따라 최적의 경로를 선택
- **사용 사례** : 다양한 해결책이나 전략을 고려해야 하는 작업에서 유용
- **장점** : 단일 경로에 지나치게 의존하지 않고, _==다양한 해결책을 탐색하여 최적의 답을 도출==_

---

## 4. Tools

`Google` 에서는 `Tools` 를 `Extensions` 와 `Functions` 로 나누었다.

### 4.1. Extensions

![[image 1 17.png|image 1 17.png]]

- 비행편을 예약한다고 하면 `Agent` 가 `Google Flights API` 를 호출하여 예약을 하게 된다.
- 사용자가 Agent 에게 "미국으로 가는 비행편을 예약하고 싶어" 라고 했을 때 **==Agent 는 출발지를 모르기 때문에 에러가 발생==**하게 된다.
- 이 때, 이런 실패를 대비해서 `Extension` 을 만든다.

    ![[image 2 15.png|image 2 15.png]]

- `Extension` 은 `Agent` 가 _==API 를 호출하기 전에 중간단계에서 데이터를 정리하여 API 호출을 실패하지 않도록하게 한다.==_
    1. Agent 한테 API 요청을 할 떄 예시를 주어 훈련을 시킨다.
    2. Agent 에게 필수 파라미터나 변수들이 뭐가 필요한지 알려준다.
- `Extension` 은 한개가 아니라 여러개를 쌓을 수 있다.

    ![[image 3 14.png|image 3 14.png]]

- `Extensions` 를 만드는 예시 코드이다.

    ```python
    import vertexai
    import pprint

    PROJECT_ID = "YOUR_PROJECT_ID"
    REGION = "us-central1"

    vertexai.init(project=PROJECT_ID, location=REGION)

    from vertexai.preview.extensions import Extension

    extension_code_interpreter = Extension.from_hub("code_interpreter")
    CODE_QUERY = """Write a python method to invert a binary tree in O(n) time."""

    response = extension_code_interpreter.execute(
        operation_id="generate_and_execute",
        operation_params={"query": CODE_QUERY}
    )

    print("Generated Code:")
    pprint.pprint(response['generated_code'])

    # The above snippet will generate the following code.
    def invert_binary_tree(root):
        """
        Inverts a binary tree.
        Args:
            root: The root of the binary tree.
        Returns:
            The root of the inverted binary tree.
        """
        if not root:
            return None

        # Swap the left and right children recursively
        root.left, root.right = invert_binary_tree(root.right), invert_binary_tree(root.left)

        return root

    # Example usage:
    # Construct a sample binary tree
    root = TreeNode(4)
    root.left = TreeNode(2)
    root.right = TreeNode(7)
    root.left.left = TreeNode(1)
    root.left.right = TreeNode(3)
    root.right.left = TreeNode(6)
    root.right.right = TreeNode(9)

    # Invert the binary tree
    inverted_root = invert_binary_tree(root)
    ```


### 4.2. Functions

- ==**특정 작업을 수행하곤 필요에 따라 재사용할 수 있는 독립적인 코드 모듈**==을 뜻한다.
- 평소에 자주 사용하는 논리나 공식같은 것을 함수로 만들어놔서 호출하여 쓰게 된다.
- 이러한 Functions 는 Agent가 쓰게 된다.
    - 실시간 API 호출은 하지 않는다.
    - `Functions` 는 클라이언트 측에서 실행하고, `Extensions` 은 서버측에서 실행된다.
- 그림을 보면 Functions 은 Agent 에게 어떤 함수들이 있고 사용자가 어떤 함수를 원하는지 Agent에게 알려주기만 하고 실제로 API 호출은 하지 않는다.

    ![[image 4 12.png|image 4 12.png]]

- Agent 가 요청을 받았을 때 **==Function 으로 데이터를 정리 후 다시 클라이언트 측으로 보내서, 클라이언트 측에서 직접 API 를 호출==**하는 방식이다.

    ![[image 5 11.png|image 5 11.png]]

- Functions 이 필요한 경우는 다음과 같다.
    - API 호출을 **==Agent가 실행되는 구조 밖에서 호출==**하고자 할때
    - API 호출을 **==Agent가 직접 호출할 수 없는 경우==**(로그인을 해야할 때, 인증/인가에 제한이 있을 때 등)
    - **==API 호출을 받고 대기==**해야 하는 경우
    - Agent 자체가 **==API 호출을 받고 추가적으로 Agent가 알아야하는 정보나 로직이 있다면==** 직접 호출보다 Functions 을 이용하는게 좋다.

---

## 5. Data stores(RAG)

Agent가 외부의 툴 뿐만 아니라 미리 지정한 자료(Docs , Website, Structured Data 등등)에 접근한다.

### 5.1. RAG 개념 및 구조

- 대표적으로 Python 에서 ChromaDB(VectorDB)를 통해 데이터를 저장해 RAG 서치를 하여 사용자에게 답변을 하는 경우가 있다.

    ![[image 6 9.png|image 6 9.png]]


---

## 6. 요약

### 6.1. Tools 요약

|**실행**|**Extensions**|**Function Calling**|**Data Stores**|
|---|---|---|---|
|**사용 사례**|**Agent-Side 실행**|**Client-Side 실행**|**Agent-Side 실행**|
|주요 용도|개발자가 API 엔드포인트와의 상호작용을 제어하려는 경우|보안 또는 인증 제한이 에이전트가 API를 직접 호출하는 것을 방지하는 경우|개발자가 다음 데이터 형식으로 검색 증강 생성(RAG)을 구현하려는 경우|
|활용 시나리오|기본 제공되는 확장 기능 (예: Vertex Search, Code Interpreter 등)을 활용할 때 유용|실시간 API 호출을 방지하는 시간 제약 또는 작업 순서 제약 (예: 배치 작업, 사람의 검토가 필요한 경우 등)|미리 인덱싱된 도메인과 URL로 된 웹사이트 콘텐츠|
|특징|다단계 계획 및 API 호출 (예: 다음 에이전트 작업은 이전 작업/API 호출의 출력에 의존함)|인터넷에 노출되지 않거나 Google 시스템에서 접근할 수 없는 API|PDF, Word Docs, CSV, 스프레드시트 등의 구조화된 데이터|
|추가 기능|||관계형 / 비관계형 데이터베이스|
|데이터 형식|||HTML, PDF, TXT 등의 비정형 데이터|

## 관련 문서

- 상위 목차: [[AI 에이전트]]
- 다음 문서: [[LangChain ToolCalling - Agent 활용법]]
