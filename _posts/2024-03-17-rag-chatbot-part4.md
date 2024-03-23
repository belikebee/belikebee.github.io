---
author: blb
title: "[RAG] langchain활용 RAG 챗봇 구축기 part 4 /4"
date: 2024-03-17 22:00:00 +0900
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
 
### 1. 문서 검색을 입맛대로 업데이트
- 우선적으로 유사도 기반 결과 리턴을 위한 리트리버 정의

```python
retriever = vectorstore.as_retriever(
    search_type="similarity_score_threshold",
    search_kwargs={"k":k, "score_threshold": 0.6}
)

docs = retriever.get_relevant_documents(f"{query}")
## -> NotImplementedError
## milvus/pinecone에서는 정상적으로 동작 안하는 듯 하다.
```

- 따라서 threshold를 이용해서 문서를 검색하기 위해 chain을 상속받아 문서 검색 함수를 수정하기로 결정

- chain을 수정하기 전에 문서 검색을 잘하기 위한 조건 선정
  1) EXCEL, CSV에서 우선 검색을 한다.
  2) 최신 순부터 검색한다.
  3) 기간에 대한 검색을 가능하게 한다.
  4) 검색된 문서의 유사도 스코어를 확인하여 일정 수준 이하의 제외한다.
  5) k 개보다 문서가 적은 경우 추가 검색을 진행한다.
  6) 일치하는 문서가 0개인 경우 fallback 메시지를 출력한다.

- retriever가 아닌 vectorstore로 구축해야해서 "VectorDBQAWithSourcesChain"을 활용하려 했지만, 비동기를 지원하지않아 "RetrievalQAWithSourcesChain"을 상속받아 수정 진행

### 2. RetrievalQAWithSourcesChain 업데이트
- 문서를 가져오는 부분과 결과를 생성하는 부분 업데이트 진행
** [RetrievalQAWithSourcesChain](https://github.com/langchain-ai/langchain/blob/master/libs/langchain/langchain/chains/qa_with_sources/retrieval.py) 참조

```python
# 아래 함수들을 수정하기 위해 추가 import
from langchain_core.callbacks import (
    AsyncCallbackManagerForChainRun,
    CallbackManagerForChainRun,
)
import datetime

# Threshold 기준으로 필터링하기 위한 함수 정의
# 디폴트로 0.6 정의
def _filtering_docs(self, docs, threshold=0.6):
   
    return [ doc for doc, score in docs if score<threshold ]

# 문서를 가져오는 function
def _get_docs(
    self, inputs: Dict[str, Any], *, run_manager: CallbackManagerForChainRun
    ) -> List[Document]:
    question = inputs[self.question_key]

    # 선택된 문서들의 집합
    results: List[Document] = list()    

    # vectorstore를 통해 cosine distance 기준 정렬 function 사용
    # 앞서 정의한 문서 검색을 잘하기 위한 1번 조건 적용
    docs = self.retriever.vectorstore.similarity_search_with_score(question, k=k, expr=f"category == 'EXCEL' or  category == 'CSV'")
   
    results.extend(self._filtering_docs(docs, threshold))

    # 특정 기간 순으로 조회할 리스트
    days = [30, 60, 90, 365]
    # 앞서 metadata에 추가한 date를 활용하기 위해 동일한 포맷(YYYYMMDD)의 시간값을 정의 // 숫자의 크기로 날짜를 비교하기 위하여 YYYYMMDD 형태를 사용
    dates = [datetime.datetime.strftime(datetime.datetime.now() - datetime.timedelta(days=day), "%Y%m%d") for day in days ]
   
    for date in dates:
        # k개의 문서를 못 채웠을 경우
        if len(results) < k:
            # PDF 문서와 시간을 활용하여 검색
            # 기간별 PDF 데이터 우선 조회
            docs = self.retriever.vectorstore.similarity_search_with_score(question, k=k, expr=f"category == 'PDF' and date>'{date}'")
            results.extend(self._filtering_docs(docs, threshold))
        else:
            break
   
    for doc in results :
        # EXCEL과 CSV는 앞서 검색을 위해 작성한 full_content와 page_content를 변경해준다.
        if doc.metadata['category'] == "EXCEL" or doc.metadata['category'] == "CSV":
            doc.page_content = doc.metadata['full_content']

    return self._reduce_tokens_below_limit(results)
   
    # llm 생성 부분
    def _call(
        self,
        inputs: Dict[str, Any],
        run_manager: Optional[CallbackManagerForChainRun] = None,
    ) -> Dict[str, str]:
        _run_manager = run_manager or CallbackManagerForChainRun.get_noop_manager()
        accepts_run_manager = (
            "run_manager" in inspect.signature(self._get_docs).parameters
        )

        # _get_docs 부분 수정으로 특정 조건에 의해 검색된 문서 리턴
        if accepts_run_manager:
            docs = self._get_docs(inputs, run_manager=_run_manager)
        else:
            docs = self._get_docs(inputs)  # type: ignore[call-arg]


        # docs에 threshold를 넘는 문서가 없을경우 그냥 fallback 멘트를 리턴
        if len(docs) == 0 :
            result[self.answer_key] = "요청하신 질문과 관련된 내용을 문서에서 찾을 수 없습니다. 질문을 변경하시거나 사이트를 참고해주시기 바랍니다."
            result[self.sources_answer_key] = []
            return result
       
        # 그대로 사용
        answer = self.combine_documents_chain.run(
            input_documents=docs, callbacks=_run_manager.get_child(), **inputs
        )
        answer, sources = self._split_sources(answer)
        result: Dict[str, Any] = {
            self.answer_key: answer,
            self.sources_answer_key: sources,
        }
        if self.return_source_documents:
            result["source_documents"] = docs
        return result
```

- 기본 함수를 통해 추론을 진행하다가 비동기 함수를 활용했을때 시간이 많이 단축되는 것을 확인하고 비동기 함수부분을 수정
- 동기, 비동기 테스트 결과
  - 각 10개의 동일한 질문 입력  
  - 동기 방식 : 288.59 초  
  - 비동기 방식 : 73.40 초

```python
# 문서를 가져오는 비동기 function
async def _get_docs(
    self, inputs: Dict[str, Any], *, run_manager: CallbackManagerForChainRun
    ) -> List[Document]:
    question = inputs[self.question_key]

    # 선택된 문서들의 집합
    results: List[Document] = list()    

    # 비동기 적용을 위해 asimilarity_search_with_score로 변경
    docs = await self.retriever.vectorstore.asimilarity_search_with_score(question, k=k, expr=f"category == 'EXCEL' or  category == 'CSV'")
   
    results.extend(self._filtering_docs(docs, threshold))

    # 특정 기간 순으로 조회할 리스트
    days = [30, 60, 90, 365]
    # 앞서 metadata에 추가한 date를 활용하기 위해 동일한 포맷(YYYYMMDD)의 시간값을 정의 // 숫자의 크기로 날짜를 비교하기 위하여 YYYYMMDD 형태를 사용
    dates = [datetime.datetime.strftime(datetime.datetime.now() - datetime.timedelta(days=day), "%Y%m%d") for day in days ]
   
    for date in dates:
        # k개의 문서를 못 채웠을 경우
        if len(results) < k:
            # PDF 문서와 시간을 활용하여 검색
            # 기간별 PDF 데이터 우선 조회
            docs = await self.retriever.vectorstore.asimilarity_search_with_score(question, k=k, expr=f"category == 'PDF' and date>'{date}'")
            results.extend(self._filtering_docs(docs, threshold))
        else:
            break
   
    for doc in results :
        # EXCEL과 CSV는 앞서 검색을 위해 작성한 full_content와 page_content를 변경해준다.
        if doc.metadata['category'] == "EXCEL" or doc.metadata['category'] == "CSV":
            doc.page_content = doc.metadata['full_content']

    return self._reduce_tokens_below_limit(results)
   
    # llm 생성 부분
    async def _acall(
        self,
        inputs: Dict[str, Any],
        run_manager: Optional[CallbackManagerForChainRun] = None,
    ) -> Dict[str, str]:
        _run_manager = run_manager or CallbackManagerForChainRun.get_noop_manager()
        accepts_run_manager = (
            "run_manager" in inspect.signature(self._get_docs).parameters
        )

        # _aget_docs 부분 수정으로 특정 조건에 의해 검색된 문서 리턴
        if accepts_run_manager:
            docs = await self._aget_docs(inputs, run_manager=_run_manager)
        else:
            docs = await self._aget_docs(inputs)  # type: ignore[call-arg]


        # docs에 threshold를 넘는 문서가 없을경우 그냥 fallback 멘트를 리턴
        if len(docs) == 0 :
            result[self.answer_key] = "요청하신 질문과 관련된 내용을 문서에서 찾을 수 없습니다. 질문을 변경하시거나 사이트를 참고해주시기 바랍니다."
            result[self.sources_answer_key] = []
            return result
       
        # 그대로 사용
        answer = await self.combine_documents_chain.arun(
            input_documents=docs, callbacks=_run_manager.get_child(), **inputs
        )
        answer, sources = self._split_sources(answer)
        result: Dict[str, Any] = {
            self.answer_key: answer,
            self.sources_answer_key: sources,
        }
        if self.return_source_documents:
            result["source_documents"] = docs
        return result
```

- 수정하다가 보니 뭔가 langchain없이 구축하는것도 비슷하지 않을까란 생각이 문득 들었지만 우선 빠르게 chain을 활용할 수 있기에 구축


### 3. 테스트 결과
- 이전보다 확실하게 최신 결과부터 검색해오기 시작하고, 원하는 정보에 대해서 잘 응답하기 시작
- 과거의 대량 데이터 기반으로 생성되는 결과에 대해서 억제가 가능하고 일정 부분 이전의 결과는 참고하지 않도록 수정
- 생각보다 정확하고, 적합한 결과를 생성하는 것 확인
- 유사도 거리에 따른 결과를 여러번 테스트 진행
    - 0.6이하의 문서만 활용했을 때 보다 1.1이하의 문서만 활용한 결과가 더 나은 것 확인
- 추가적으로 threshold도 반복문을 적용하여 유사도 거리 0.6이하인 문서들과 1.1이하의 문서들을 검색해서 활용하는 방법도 적용 가능
- 급하게 구축한 RAG지만 원하는 수준의 성능 확보
- 다만 프롬프트 유출과 같은 공격에 대해서는 chatgpt와 동일하게 내성이 없어 추가적인 보완이 필요
 
### 4. 추가 고도화 방향  
1) ~~프롬프트 업데이트~~  
2) ~~데이터 구조 업데이트~~
3) ~~리트리버 업데이트~~

### 5. 추가적인 테스트
- 서비스로 오픈되었을 때의 스트레스 테스트 필요
  - 쿠버네티스로 구성하여 부하에 대한 유연성, 내구성 확보
  - Locust 프레임워크 활용 테스트 진행
