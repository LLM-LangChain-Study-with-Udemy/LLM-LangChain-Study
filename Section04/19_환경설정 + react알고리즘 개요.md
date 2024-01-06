# [udemy] LangChain

## Chap4. ReAct 에이전트의 특별한 기능 자세히 알아보기

- 에이전트를 통해 텍스트를 입력으로 받은 다음 정확한 도구를 선택해 실행한 다음, 응답을 큐레이팅 → 구체화
- 랭체인의 리엑트 에이전트 프레임워크 내부를 살펴보자.
- → 리액트 알고리즘에서 일어나는 모든 움직이는 부분을 이해하자.
- 

initialize agent 함수를 이용해서 에이전트를 정의함.

tool list, LLM을 받아서 에이전트 유형에 Zero-shot ReAct 설명을 부여했음. 

### agent 실행기 구현하기

agent 실행기 → while loop

- llm이 답변을 정의했다고 판단할 때 멈추거나, 기본적으로 이터레이션 횟수나 보내려는 메시지 수를 제한하려고 할 때 멈춤.
- while loop → LLM 호출 → 응답 해석 → 사용할 도구 선택 및 실행 → 실행 반복

![image](https://github.com/LLM-LangChain-Study-with-Udemy/LLM-LangChain-Study/assets/142489787/bd733cf8-e090-4c82-ac18-eea09feff528)


- parse : 텍스트 구문 분석
- LLM이 선택한 도구로부터 응답을 반환하기에 충분한 정보가 있으면 Answer 반환, 없다면 Agent로 다시 돌아감.

### 환경설정하기

```bash
mkdir react-langchain
cd react-langchain

pipenv shell

pipenv install langchain openai black python-dotenv
# gpt 3.5를 사용할거라서 블랙포매팅 툴과 파이썬 패키지를 설치해줌.
```

파이참 or vscode를 실행시킨 다음, ‘.env’ 파일을 만들어줌 ⇒ OPENAI_API_KEY로 저장할 환경변수를 갖게 됨.

다음 `main.py` 파일을 만듦.

이를 위해 python-dotenv 패키지를 사용할것임

```python
def get_text_length(text:str)->int :
	return len(text)

if __name__ == '__main__':
	print("Hello ReAct LangChain!")
	print(get_text_length(text="Dog"))
```

이 함수를 랭체인 도구에 캐스트 하고, 나중에 에이전트가 이 도구를 사용할 능력을 갖추도록 만들것임.

`도구 데코레이터`를 사용하면 함수를 받아 랭체인 도구로 전환할 수 있음.

- tool decorator는 랭체인 유틸리티 함수인데, 이 함수를 가져와서 랭체인 도구를 만들 수 있음. 함수의 이름, 즉 인수로 받는 내용을 플러그인 함.

랭스미스의 랭체인 허브를 이용할 것임. (마켓플레이스)

- 리액트 에이전트를 위한 프롬프트를 포함해 , 잘 만들어진 프롬프트들을 공유함. 
- 랭스미스의 랭체인 허브를 이용하기 위해선 회원가입을 해야함.

[smith.langchain.com/hub](https://smith.langchain.com/hub)

- 사용할 도구를 결정할 때는 에이전트의 이터레이션을 참조함.
- 본 강의는 react paper 추론 제정을 구현한 것임.
- 본 강의에서는 헤리슨체이스가 사용한 프롬프트를 이용하였음.