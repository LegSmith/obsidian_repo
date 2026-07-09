- [[#1. LLM 활용하여 답변을 생성하는 방법]]
    - [[#1.1. Ollama 설치]]
    - [[#1.2. Ollama 사용법]]
    - [[#1.3. 개발환경 세팅]]
    - [[#1.4. Deepseek r1 모델 사용]]
    - [[#1.5. llama3.2 모델 사용]]
    - [[#1.6. GPT 모델 사용]]

  

---

## 1. LLM 활용하여 답변을 생성하는 방법

대표적인 LLM으로는 `ChatGPT`, `Claude`, `HyperClova` 등의 유료 모델이 있으며, `Ollama`는 로컬 환경에서 실행 가능한 오픈소스 모델입니다.

### 1.1. Ollama 설치

- 우선 Ollama 공식 홈페이지에 사용중인 OS에 맞게 설치를 받는다. → [https://ollama.com/download](https://ollama.com/download)
    
    ![[image.png]]
    
- 압축파일을 실행하면 다음과 같다.
    
    ![[image 1.png]]
    
- install 을 누르면 자동으로 C 드라이브로 다운받아진다.

### 1.2. Ollama 사용법

- Ollama 에서 사용할 모델들은 Git에서 Clone 하듯이 간단하게 불러올 수 있다.
- Ollama 모델목록은 다음 링크에서 확인 가능하다 → [https://ollama.com/search](https://ollama.com/search)
- 이번 예제에서는 `deepseek-r1` 모델 중 1.5b 짜리를 사용한다.
    
    ![[image 2.png]]
    
    > 위 숫자들은 해당 _==**모델이 학습한 파라미터의 갯수를 뜻한다.  
    > **==_671b 은 약 6710억개의 파라미터이므로 용량이 400GB 정도이다.
    
- 우측에 체크된 명령어를 터미널에서 실행하면 된다.
    
    - 단, `run` 이 아닌 `pull` 로 바꿔서 실행하자 → `ollama pull deepseek-r1:1.5b`
    
    ![[image 3.png]]
    
    > 보통 모델을 받으면 C드라이브에 Ollama가 지정한 경로로 받아지게 되는데 다른 경로에 받고싶다면 다음과 같이 수정한다.  
    > 1. 시스템 환경 변수 편집 → 환경변수 → 사용자변수 에서 다음과 같이 추가해준다.  
    > 2. 변수이름은 `OLLAMA_MODELS` 로 지정하고 변수값에 다운받고 싶은 경로를 적어준다.
    > 
    > ![[image 4.png]]
    
      
    

### 1.3. 개발환경 세팅

- `Python3.11.9` 버전에 가상환경은 `venv` 를 사용한다.
- 우선 빈 프로젝트를 생성 후 `python -m venv ./env` 를 터미널에서 실행하여 가상환경을 프로젝트 경로에 생성한다.
- 그리고 프로젝트 경로에서 `env\Scripts\activate.bat` 을 실행하여 가상환경을 활성화한다.
- 그리고 Jupyter노트북사용을 위해 `chat.ipynb` 를 생성해준다.
    
    ![[image 5.png]]
    

### 1.4. Deepseek r1 모델 사용

- `Ollama` 에서 제공하는 모델을을 사용할려면 패키지 설치가 필요하다
    - `%pip install -q langchain-ollama`
- Deepseek r1 모델을 사용하기 위해 다음과 같이 코딩한다.
    
    ```python
    from langchain_ollama import ChatOllama
    
    llm = ChatOllama(model="deepseek-r1:1.5b") # Deepseek r1 추론모델을 사용
    
    llm.invoke("What time is Korean time?") # 추론모델이기 때문에 추론할 질문을 해주면 된다.
    ```
    
    ![[image 6.png]]
    
- 답변을 보면 `<think>` 같은게 있는데 추론모델이라서 저런 태그들이 뜬다.

### 1.5. llama3.2 모델 사용

- `llama3.2` 모델은 다음 링크에서 받는다 → [https://ollama.com/library/llama3.2](https://ollama.com/library/llama3.2)
    
    ![[image 7.png]]
    
- 똑같이 터미널에서 명령어를 실행하는데, `run` 이 아닌 `pull` 로 실행한다 !
    
    ![[image 8.png]]
    
- 그리고 모델명만 바꿔서 코딩하면 다음과 같다.
    
    ```python
    from langchain_ollama import ChatOllama
    
    # llm = ChatOllama(model="deepseek-r1:1.5b") # Deepseek r1 추론모델을 사용
    llm = ChatOllama(model="llama3.2:1b") # llama3.2 모델을 사용
    
    llm.invoke("What time is Korean time?") # 추론모델이기 때문에 추론할 질문을 해주면 된다.
    ```
    
    ![[image 9.png]]
    

### 1.6. GPT 모델 사용

- GPT를 사용하기 위해서는 패키지설치가 필요하다.
    - `%pip install -q langchain-openai`
- 그리고 GPT를 사용하기 위해서는 `API_KEY` 가 필요한데, API_KEY를 사용하는 방법은 두가지다.
    1. `OPENAI_API_KEY` 를 환경변수(`.env`)에 등록하여 사용
    2. `ChatOpenAI`에 직접 키를 등록
- 2번은 사용할 때마다 키를 적어서 등록해야하는 번거로움이 있기 때문에 1번을 추천한다.
- 프로젝트 루트경로에 `.env` 라는 파일을 생성한다.
- 그리고 `OPENAI_API_KEY=”api key”` 를 입력해주면 된다.
- 그리고 환경변수를 불러오는 패키지를 설치해야 한다.
    - `%pip install -q python-dotenv`
- 마지막으로 다음코드와 같이 코딩하면 된다.
    
    ```python
    from langchain_openai import ChatOpenAI
    from dotenv import load_dotenv
    
    load_dotenv() # 환경변수 불러오기
    
    llm = ChatOpenAI(model = "gpt-4o-mini")
    
    llm.invoke("지금 한국시간은?")
    ```
    
    ![[image 10.png]]
    

> 참고로 환경변수 키값이 `OPENAI_API_KEY` 인 경우는 _==**ChatGPT 기준**==_이므로 다른 유료 LLM 모델을 사용한다면 키값이 달라질 수 있다. _==**공식문서를 확인해보면 됨!**==_