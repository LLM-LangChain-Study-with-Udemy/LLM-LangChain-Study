
코드를 실행하기 앞서, import한 모듈에 대해 하나씩 알아보자.


```python
import os

from langchain.document_loaders import TextLoader
from langchain.text_splitter import CharacterTextSplitter
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain import OpenAI
import pinecone
from langchain.chains import RetrievalQA
```


## Text Loader

이 링크로 따라들어가보면: https://github.com/langchain-ai/langchain/blob/master/libs/langchain/langchain/document_loaders/text.py

text.py에 변경이 있다.

```python
from langchain_community.document_loaders.text import TextLoader

__all__ = ["TextLoader"]
```

document loader는 텍스트를 LLM이 소화하기 쉽게 처리해주는 함수다.
예를 들어 whatsapp_chat.py 코드를 보면 (https://github.com/langchain-ai/langchain/blob/aa1c7a56a9c8cee42d183daec4a434b2327fa151/libs/community/langchain_community/document_loaders/whatsapp_chat.py)

왓츠앱의 텍스트를 발신, 수신자, 발신 날짜 등을 추출하기 위해 정규표현식을 사용하고 그 다음에 한 문장씩 concatenate하는 방식이다.


```python
class WhatsAppChatLoader(BaseLoader):
    """Load `WhatsApp` messages text file."""

    def __init__(self, path: str):
        """Initialize with path."""
        self.file_path = path

    def load(self) -> List[Document]:
        """Load documents."""
        p = Path(self.file_path)
        text_content = ""

        with open(p, encoding="utf8") as f:
            lines = f.readlines()

        message_line_regex = r"""
            \[?
            (
                \d{1,4}
                [\/.]
                \d{1,2}
                [\/.]
                \d{1,4}
                ,\s
                \d{1,2}
                :\d{2}
                (?:
                    :\d{2}
                )?
                (?:[\s_](?:AM|PM))?
            )
            \]?
            [\s-]*
            ([~\w\s]+)
            [:]+
            \s
            (.+)
        """
        ignore_lines = ["This message was deleted", "<Media omitted>"]
        for line in lines:
            result = re.match(
                message_line_regex, line.strip(), flags=re.VERBOSE | re.IGNORECASE
            )
            if result:
                date, sender, text = result.groups()
                if text not in ignore_lines:
                    text_content += concatenate_rows(date, sender, text)

        metadata = {"source": str(p)}

        return [Document(page_content=text_content, metadata=metadata)]
```

LangChain의 장점은 왓츠앱, 구글 드라이브, 노션, pdf 등 웬만한 종류의 문서에 대해 
이미 가장 쉽게 처리할 수 있는 기능을 다 만들어놓았다는 점이다.


## Text Splitters

생각보다 복잡한 로직이 숨어있으니 코드를 다 뜯어볼 생각은 하지 말자.
청크의 적절한 크기를 정하는 것도 어려운 문제이기 때문이다.

그 중에 가장 단순한 방법은 글자 수로 청크를 나누는 것이다.

### Split by character

공식 문서: https://python.langchain.com/docs/modules/data_connection/document_transformers/character_text_splitter

```python
from langchain.text_splitter import CharacterTextSplitter

text_splitter = CharacterTextSplitter(
    separator="\n\n",
    chunk_size=1000,
    chunk_overlap=200,
    length_function=len,
    is_separator_regex=False,
)
```

chunk overlap역할이 중요하다. 
맥락이나 의미를 훼손하지 않고 텍스트를 청크로 나눌 수 있게 한다.


## 임베딩 모델

당연하지만, 텍스트 임베딩도 컴퓨터 리소스가 드는 작업이기 때문에
임베딩 모델(API 호출)을 사용할 때 돈을 내야 한다.

실습에서는 OpenAI의 임베딩 모델 text-embedding-ada-002를 사용하는데,
text-embedding-ada-002은 비용이 꽤나 저렴하기 때문에 쓸 만하다.

그 외에도 Cohere, Hugging Face, OpenAI 등 다양한 회사에서 임베딩 모델을 제공하는데
어차피 인터페이스는 똑같으니 아무거나 써도 문제는 발생하지 않는다.



## Pinecone

요즘 가장 많이 사용하는 벡터 스토어라고 한다.


## RetrievalQA

![2024-01-05_13-10-37](https://github.com/LLM-LangChain-Study-with-Udemy/LLM-LangChain-Study/assets/36873797/edb6f092-84a4-40a8-b9a2-5a47116472ee)

LangChain의 강점은 길고 복잡한 RAG 과정을 추상화해서 간편한 단계로 줄여준다는 점이다.