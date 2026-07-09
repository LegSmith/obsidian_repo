## 1. Python 설치

- Python 공식 홈페이지에서 최신 버전을 다운로드 받으면 된다.
    
    > [!info] Download Python  
    > The official home of the Python Programming Language  
    > [https://www.python.org/downloads/](https://www.python.org/downloads/)  
    

## 2. 가상환경(Miniconda) 설치

- 가상환경에는 여러 종류들이 있지만 지금은 `Miniconda` 를 이용한다.
- 위 블로그에서 기본적인 설치를 한다(추후 캡쳐)
    
    > [!info] Miniconda 로 가상환경 만들기. #1 [Miniconda 설치와 실행]  
    > 1.  
    > [https://swmakerjun.tistory.com/80](https://swmakerjun.tistory.com/80)  
    
- 설치가 됐다면 관리자 권한으로 실행한다(충돌을 막기위해)
    
    ![[image 16.png|image 16.png]]
    
- `conda config --add channels conda-forge` 명령어를 입력한다
    
    - 위 명령어는 Conda 에서 패키지를 설치할 때, `conda-forge` 라는 특별한 저장소를 추가하는 명령어이다.
    - `conda-forge` 는 더 다양한 프로그램과 최신 버전을 제공해주는 유명한 저장소(채널)이다.
    
    ![[image 1 7.png|image 1 7.png]]