# LLM 기반 RAG(Retrieval-Augmented Generation) 실습 프로젝트

한국 세금 문서를 활용하여 다양한 RAG 파이프라인을 구현하고 비교하는 학습 프로젝트입니다. LangChain 추상화 방식과 직접 구현 방식, 그리고 Chroma와 Pinecone 두 가지 벡터 데이터베이스를 비교합니다.

## 프로젝트 구조

```
llm_app/
├── 1.langchain_llm_test.ipynb              # LangChain + OpenAI 기본 연동 테스트
├── 2.rag_with_chroma.ipynb                 # LangChain + Chroma를 이용한 RAG
├── 3.rag_without_langchain_with_chroma.ipynb # LangChain 없이 직접 구현한 RAG + Chroma
├── 4.rag_with_pinecone.ipynb               # LangChain + Pinecone을 이용한 RAG (쿼리 변환 포함)
├── tax_docs/                               # 세금 관련 문서 (docx)
│   ├── tax_with_markdown.docx
│   └── tax_with_table.docx
├── chroma/                                 # Chroma 벡터 DB 저장소 (로컬)
└── .env                                    # API 키 설정 (OpenAI, LangSmith, Pinecone)
```

## 노트북 설명

### 1. LangChain LLM 기본 테스트

- `ChatOpenAI` 모델 초기화 및 기본 호출
- 환경변수(`.env`) 기반 API 키 관리

### 2. LangChain + Chroma RAG

- **문서 로드**: `Docx2txtLoader`로 docx 파일 로딩
- **텍스트 분할**: `RecursiveCharacterTextSplitter` (chunk_size=1000, overlap=200)
- **임베딩**: `OpenAIEmbeddings` (text-embedding-3-large)
- **벡터 저장소**: Chroma (로컬 영속 저장)
- **질의응답**: `RetrievalQA` 체인 + LangSmith Hub의 RAG 프롬프트

### 3. LangChain 없이 직접 구현한 RAG + Chroma

- **문서 로드**: `python-docx`로 직접 파싱
- **텍스트 분할**: `tiktoken` 기반 토큰 단위 분할 (1500 토큰)
- **벡터 저장소**: Chroma API 직접 사용
- **LLM 호출**: OpenAI 클라이언트 직접 사용
- LangChain 추상화 없이 각 단계를 명시적으로 구현

### 4. LangChain + Pinecone RAG (쿼리 변환)

- **벡터 저장소**: Pinecone 클라우드 DB
- **쿼리 변환**: 사용자 질문의 용어를 문서에 맞게 변환 (예: "사람" → "거주자")
- **체인 파이프라인**: 쿼리 변환 → 문서 검색 → 답변 생성 (LCEL 문법)

## 주요 비교 포인트

| 비교 항목       | 노트북 2      | 노트북 3      | 노트북 4            |
| --------------- | ------------- | ------------- | ------------------- |
| **프레임워크**  | LangChain     | 직접 구현     | LangChain           |
| **벡터 DB**     | Chroma (로컬) | Chroma (로컬) | Pinecone (클라우드) |
| **텍스트 분할** | 문자 기반     | 토큰 기반     | 문자 기반           |
| **쿼리 최적화** | 없음          | 없음          | 용어 변환 체인      |

## 사용 기술

- **LLM**: OpenAI GPT-4
- **임베딩**: OpenAI text-embedding-3-large
- **벡터 DB**: ChromaDB, Pinecone
- **프레임워크**: LangChain, LangChain Hub
- **문서 처리**: python-docx, docx2txt, tiktoken

## 환경 설정

### 1. pyenv 가상환경 설정

```bash
# pyenv 설치 (macOS)
brew install pyenv pyenv-virtualenv

# 쉘 설정 (~/.zshrc)에 아래 추가
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc
source ~/.zshrc

# Python 설치 및 가상환경 생성
pyenv install 3.10.20          # 원하는 Python 버전 설치
pyenv virtualenv 3.10.20 llm_app  # 가상환경 생성
pyenv local llm_app              # 프로젝트 디렉토리에 자동 활성화 설정 (.python-version 파일 생성)
```

### 2. API 키 설정

`.env` 파일에 아래 API 키를 설정합니다:

```
OPENAI_API_KEY=your_openai_api_key
LANGCHAIN_API_KEY=your_langsmith_api_key
PINECONE_API_KEY=your_pinecone_api_key
```

### 3. 패키지 설치

```bash
pip install \
    langchain \              # LLM 애플리케이션 프레임워크
    langchain-openai \       # LangChain의 OpenAI 연동 모듈 (ChatOpenAI, OpenAIEmbeddings)
    langchain-chroma \       # LangChain의 Chroma 벡터 DB 연동 모듈
    langchain-pinecone \     # LangChain의 Pinecone 벡터 DB 연동 모듈
    chromadb \               # Chroma 벡터 데이터베이스 (로컬 임베딩 저장/검색)
    openai \                 # OpenAI API 클라이언트 (LangChain 없이 직접 호출 시 사용)
    python-docx \            # Word(.docx) 문서 파싱 (python-docx 라이브러리)
    docx2txt \               # Word(.docx) 문서에서 텍스트 추출 (LangChain 로더용)
    tiktoken \               # OpenAI 토크나이저 (토큰 기반 텍스트 분할에 사용)
    langchainhub \           # LangSmith Hub에서 프롬프트 템플릿 가져오기
    python-dotenv            # .env 파일에서 환경변수 로드
```

### 4. 실행

각 노트북을 순서대로 실행합니다.
