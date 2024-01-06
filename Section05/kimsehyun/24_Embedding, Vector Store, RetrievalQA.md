## 개요

- Embeddings
- Vector Stores (Pinecone)
- RetrievalQA Chain
- LangChain document loaders
- LangChain text splitters

가장 먼저 살펴볼 건 document loader이다.

## Document Loader

![2024-01-05_11-38-45](https://github.com/LLM-LangChain-Study-with-Udemy/LLM-LangChain-Study/assets/36873797/56a4370e-c0a6-4db2-bedb-97d004f8850f)

LLM은 입력 데이터를 문서(document) 형식으로 받는다.
그리고 LangChain은 다른 third party(구글 드라이브, 노션, 위키피디아 등)에서 
문서(document)를 가져와 활용할 수 있게 해준다.

이걸 가능하게 하는 게 document loader이다. 일종의 wrapper라고 생각하자.



## LangChain Text Splitters

아까 LLM의 입력 데이터는 문서(document)라고 했다. 그럼 LLM에 문서를 통째로 넣는 걸까?
당연히 아니다. 문서를 작은 조각(chunk)으로 쪼개서 넣어야 한다. 
문서의 텍스트를 쪼개는 역할을 하는 게 text splitter이다.

하지만 문서마다 파일 유형이나 접근법이 다양하다. 
그리고 우리가 텍스트를 쪼갤 땐 "의미론적으로 관련이 있는(semantically related)" 것끼리 나누고 싶은데 이 기준이 또 텍스트 유형마다 다를 수밖에 없다.

그래서 text splitter를 정의하는 게 많이 복잡하다. 
LangChain 공식 docs에도 보면 여러 종류의 text splitter가 있다.


![2024-01-05_11-50-15](https://github.com/LLM-LangChain-Study-with-Udemy/LLM-LangChain-Study/assets/36873797/eb59dd5c-8792-4e7e-8475-9cbfa9aadc42)




## Retrieval Augmentation Generation (RAG)

원래 prompt에 context를 추가해서 LLM의 답변 정확도를 높이는 기술을 말한다.
Context는 prompt와 관련이 있는 텍스트를 말한다.

![Pasted image 20240105121528](https://github.com/LLM-LangChain-Study-with-Udemy/LLM-LangChain-Study/assets/36873797/22896eb3-c4a0-4bef-b0fa-620538409762)



어떤 책에 대한 질문을 LLM에 했다고 가정해보자.
질문(prompt)에 대답하기 위해 책의 모든 정보를 chunk로 잘라서 입력으로 넣으면
API 호출이 불필요하게 많이 발생할 것이다. 
그만큼 추론하는 시간도 증가하고 비용도 발생하니 이 방법은 비효율적이다.

반면에 prompt에 대한 정보가 특정 chunk에만 있다면 이 chunk만 LLM에 입력해주면 된다.
이때 prompt와 관련이 있는 chunk가 context에 해당한다.
그리고 prompt와 관련 있는 정보를 context로 추가해주었으니 prompt가 '증강(augmentation)'되었다고 할 수 있다.

LLM은 돈키호테에 대해 학습한 적이 없고 온라인에서 자동으로 검색을 할 수도 없지만,
context로 증강해주어서 이 추가된 지식을 바탕으로 올바른 답을 내릴 수 있게 된다.

그렇다면 컴퓨터는 특정 청크가 prompt와 관련이 있다는 걸 어떻게 판단하는 걸까?



## Text Embedding

딥러닝 NLP 모델에서 이미 많이 다룬 개념이라 자세히 설명하지는 않겠다.
사람이 아닌 기계가 텍스트를 처리할 수 있도록 벡터 공간으로 매핑한 기술을 의미한다.

텍스트 임베딩의 핵심은 다음과 같다.

1) 의미가 비슷한 단어, 절, 또는 문장끼리는 벡터 공간에서 서로 가까이 위치한다.
2) 의미가 유사한 정도를 벡터 간의 거리로 수치화한다.

텍스트를 임베딩한 결과(벡터)를 저장해놓은 데이터베이스를 **벡터 스토어(vector store)** 라고 한다.
대표적인 벡터 스토어로 **파인콘(pinecone)** 이 있다.


### 텍스트 임베딩을 통해 거리가 가까운 벡터를 context로 가져온다

다시 책 예시로 돌아가보자.
우리가 던진 질문(prompt)과 서로 관련이 있는 내용을 context로 증강해서 LLM에 입력으로 넣는다고 했다. 
만약 사전에 책을 청크로 잘라서 벡터로 임베딩해놓고, 이걸 어딘가에 저장해놓았다면
prompt와 거리가 가까운 벡터에 해당하는 내용도 금방 다시 불러올 수 있을 것이다.
그게 바로 context에 해당한다.


![Pasted image 20240105123926](https://github.com/LLM-LangChain-Study-with-Udemy/LLM-LangChain-Study/assets/36873797/2bf1b8ee-7566-44f3-b84a-36fdf375a731)


