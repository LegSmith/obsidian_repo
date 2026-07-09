- [[#1. Tool Calling 개념 이해]]
    - [[#1.1. Tool Calling 이란?]]
- [[#2. LangChain 에서 제공하는 내장 도구(Tool)]]

---

## 1. Tool Calling 개념 이해

### 1.1. Tool Calling 이란?

- LLM 이 ==**외부 기능이나 데이터에 접근**==할 수 있게 해주는 메커니즘
- **==LLM의 한계(최신 정보 부족, 특정 작업 수행 불가 등등)를 극복==**하는 방법이다
- 실시간으로 데이터에 접근, 특수 기능 수행, 정확성 향상 등 LLM 과 외부 도구 연동이 필요하다.

    ![[image 28.png|image 28.png]]

    - 예를 들어 자연어로 LLM 에 질의를 전송하면 ==**LLM 은 외부도구(곱셉툴)을 이용**==한다.
    - 이 때 LLM은 사용자 질의에서 Tool 로 보낼 `arguments` 를 뽑아내서 전달하고 해당 **==Tool은 arguments 를 통해 기능을 수행 후 결과를 LLM 에 return==** 한다.
    - 그리고 LLM 은 return 값을 정제하거나 후처리를 통해 사용자에게 알려준다.

---

## 2. LangChain 에서 제공하는 내장 도구(Tool)


---

## 관련 문서

- 상위 목차: [[AI 에이전트]]
- 이전 문서: [[AI Agent 정리(Google)]]
