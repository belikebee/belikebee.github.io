---
author: blb
title: "[RAG] langchain활용 RAG 챗봇 구축기 part 1 /4"
date: 2024-03-14 22:00:00 +0900
categories: [RAG, CHATBOT, OPENAI, LANGCHAIN, AI]
tags: [RAG]
render_with_liquid: false
toc: true
comments: true

---

### 0.프로젝트 정의 및 개발 환경

- 프로젝트 정의 : 1900년대 문서들부터 최신 문서까지 약 1,000건의 문서를 기반으로한 RAG 구축

- 개발 환경 : Ubuntu

- 활용 프레임워크 : Langchain
- 활용 LLM : OpenAI API
- Vector Store : MILVUS(chromadb, pinecone)

- 데이터 형태 : Source로 사용될 문서는 1건의 csv 문서, 1건의 excel문서와 PDF 문서들

----

### 1. Vector Store구축


```python
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain_community.document_loaders import DataFrameLoader
from langchain_community.document_loaders.csv_loader import CSVLoader

from langchain.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
```

text_splitter 같은 경우는 ["일등박사"](https://drfirst.tistory.com/entry/langchain%EA%B3%B5%EB%B6%80-Input-%ED%85%8D%EC%8A%A4%ED%8A%B8%EA%B0%80-%EB%84%88%EB%AC%B4-%EA%B8%B8%EB%95%8C-Text-Spitter-feat-RecursiveCharacterTextSplitter)님께서 정리해주신 문서를 참고하여 선정하였음


- VectorStore는 가장 처음 테스트는 ChromaDB로 시작해서 Pinecone을 갔다가 데이터는 폐쇄망에 보관되야하는 이슈를 만나 Milvus로 최종 결정
- Milvus는 docker-compose로 간단하게 구축 가능하며 GPU를 활용할 수 도 있는 장점 보유
  - 빠른 구축을 위해 둘의 차이는 테스트 해보지 않았음!
  - CPU만 : https://milvus.io/docs/install_standalone-docker.md
  - GPU도 : https://milvus.io/docs/install_standalone-docker-compose-gpu.md
```python
# 사실 어떤 Vector Store를 선택하든 embedding값을 저장하는 구조는 동일하다.
from langchain_community.vectorstores import Milvus

# 최근 공개된 gpt3 기반 embedding 모델 적용
# small 사이즈 기반 구축 진행
size, chunk_size, overlap_size = "small", 1000, 100

embeddings = OpenAIEmbeddings(model=f"text-embedding-3-{size}")

vectorstore = Milvus(
    embeddings,
    # dataname과 embedding모델의 size, chunk_size, overlap_size등을 기록한 collection name 설정
    collection_name=f"{dataname}_{size}_{chunk_size}_{overlap_size}",
    # docker-compose로 구축한 milvus에 대한 default config
    connection_args={"host": "localhost", "port":"19530"},
)
```

- csv 문서들은 row 단위로, pdf파일은 chunk_size 단위로 분리
- vectorstore.add_documnets를 통한 collection에 embedding값 저장
- 참고사항으로는 collection에 처음들어간 데이터의 metadata구조를 뒤에 추가는 데이터들도 따라야하는 구조
- 모두 같은 형태의 데이터를 추가한다면 문제가 없겠지만 라이브러리들이 가져온 metadata는 데이터별로 차이가 있어서 어느 정도 metadata의 schema 구축 필요
- 초기에는 기본적인 내용으로 metadata를 구축했으나 추후에 metadata를 활용하여 고도화를 시도하기에 전체 데이터 및 검색 기준에 활용될 수 있는 metadata를 구축하는 것이 좋을 것으로 판단 됨
  - 우선 일반적인 구축으로 작성하고 다음 편에서 추가 예정

```python
# 일반적으로 loader들을 이용하게 되면 page_content에 embedding 대상의 값들이 저장된다.
# collection에 embeddings 후 vector에 저장된다.

########## add documents from Excel
import pandas as pd

excel_df= pd.read_excel(f"{excel_file}.xlsx", index_col=0)

excel_loader = DataFrameLoader(excel_df, "임베딩에 사용될 컬럼명")
excel_data = excel_loader.load()

# metadata 통일을 위한 작업
for doc in excel_data:
  # 데이터 출처 확인용 default로 파일명이 정의되어 있긴함
  doc.metadata["source"] = f"{excel_filename}.xlsx"
  # 날짜기반 검색용
  doc.metadata["date"] = f"{date_from_data}"
  # PDF가 대부분이라 page정보가 추가될 예정이므로 excel의 row를 page로 할당
  # 크게 유의미하지 않은 데이터
  doc.metadata["page"] = f"{excel_row_num}"
  # “임베딩에 사용될 컬럼명”의 데이터만 page_content로 추출
  # 추가적인 설명으로 생성이 가능하도록 최종 문서 업데이트
  doc.page_content = f”{doc.page_content} - {해당 content의 설명}”

# insert된 결과를 확인하기 위한, results는 insert된 pk 값들이 list로 리턴된다.
results = vectorstore.add_documnents(excel_data)

########## add documents from csv
csv_loader = CSVLoader(file_path=f"{csv_filename}.csv")
csv_data = csv_loader.load()

"""
#  csv 파일 구조
#  컬럼1   | 컬럼2
#  데이터1 | 데이터2
#  데이터3 | 데이터4

print(csv_data[0])
Document(page_content='\ufeff{컬럼1}: {데이터1}\n컬럼2}: {데이터2}', metadata={'source': '{csv_filename}.csv', 'row': 0})
"""

# metadata 통일을 위한 작업
for doc in csv_data:
  doc.metadata["source"] = f"{csv_filename}.csv"
  doc.metadata["date"] = f"{date_from_data}"
  doc.metadata["page"] = f"{csv_row_num}"

results = vectorstore.add_documnents(csv_data)

########## add documents from pdf
pdf_loader = PyPDFLoader(f"{pdf_file}.pdf")
pdf_data = pdf_loader.load()
text_splitter = RecursiveCharacterTextSplitter(chunk_size=chunk, chunk_overlap=overlap)
pdf_data = text_splitter.split_documents(pdf_data)

# metadata 통일을 위한 작업
for doc in pdf_data:
  doc.metadata["source"] = f"{pdf_filename}.pdf"
  doc.metadata["date"] = f"{date_from_data}"
  # split할 때 default로 정의
  # doc.metadata["page"] = f"{pdf_row_num}"
 
results = vectorstore.add_documents(pdf_data)

```


### 2. RAG 테스트

```python
# RAG Chain
from langchain.chains import RetrievalQAWithSourcesChain
from langchain_openai import ChatOpenAI

# RAG prompt
from langchain import hub
prompt = hub.pull("rlm/rag-prompt")

chain_type_kwargs = {"prompt": prompt}

k = 10
retriever = vectorstore.as_retriever(search_kwargs={"k": k})

# model_name으로 "gpt-4-turbo-preview" 사용시 가장 최신 turbo 모델을 가져온다.
llm = ChatOpenAI(model_name="gpt-4-turbo-preview", temperature=0)  
chain = RetrievalQAWithSourcesChain.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever = retriever,
    chain_type_kwargs=chain_type_kwargs,

)

res = chain(question)
print(res['answer'])

```

### 3. 테스트 결과
- 어느 정도 잘한다.
- 어느 정도만 잘한다.
- 일관된 형태의 응답이 나오지 않는다.
- 나중에 넣은 데이터를 더 잘 찾는 경향이 있는 것 같다.
- 최근 데이터는 주 단위, 월 단위로 쪼개져 있는데, 과거 데이터는 1년이 통합으로 구성되어 있어 과거 데이터에서 잘 찾아오는 경향이 있다.
- 아직 문서 외에 질문에 대해 방어는 잘 하지 못한다.

기본적인 형태로 구축하니 빠른 테스트가 가능했지만, 필요한 성능엔 상당히 미흡한 형태

### 4. 추가 고도화 방향
1) 프롬프트 업데이트
2) 데이터 구조 업데이트
3) 리트리버 업데이트

