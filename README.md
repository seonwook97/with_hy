# 🔡 NLP(Sentiment Analysis) 프로젝트
<br>

## 🔎 주제

### 📰➡💸 **KOSPI 상위 10개 종목 뉴스 헤드라인 감성분석을 통한 주가 변동성 예측 가능성 연구**

![기획서 배경](https://user-images.githubusercontent.com/92377162/151172664-9693323f-641c-452d-baba-fdaa98da088d.jpg)

<br>

## 🎯 목적

- **주가를 예측하는 것은 기술적, 경제적 요인 뿐만 아니라 심리적 요인 또한 고려해야 함**
- **심리적 요인은 주로 종목관련 SNS, 뉴스 등에 대한 대중들의 감성지수를 통해 파악 가능**
- **뉴스 정보의 감성분석을 통해 감성의 분류와 주가 변동성의 분류가 어느정도까지 매칭될 수 있는지 파악**
<br>

## 🛣️ 프로젝트 진행 방식

### ✅ 주식 종목 선정

- **2021년 시가총액이 큰 기업 중 계열사 제외 총 10개의 상위 기업 선정**
    - EX) 카카오, 카카오게임즈, 카카오뱅크, 카카오페이 → 카카오
<br>

### ✅ 데이터 수집 및 라벨링

- **FinanceDataReader를 통한 종목별 주가 데이터 수집**
    - FinanceDataReader는 한국 주식 가격, 미국주식 가격, 지수, 환율, 암호화폐 가격, 종목 리스팅 등 금융 데이터 수집 라이브러리
<br>

- **Selenium을 통한 네이버 금융 페이지의 종목뉴스검색 항목을 통해 종목별 뉴스 헤드라인 크롤링**
    - 네이버 금융의 종목뉴스는 다양한 언론사의 기사로 구성되어 있어 특정 언론사의 편향성을 최대한 방지하고자 함
    - 증권 홈페이지의 종목뉴스검색이라 최대한 종목과 관련된 기사로 필터링 된다고 생각하여 진행
    - 2011.12.31 ~ 2021.12.31 종목별 네이버 금융 종목 기사 뉴스 헤드라인 수집
    - 종목별로, 일자별로 기사의 개수가 모두 다르기 때문에 1page에 있는 일별 최근 기사 기준 20개를 크롤링
    - 일별 기사가 20개가 안되는 경우 -> 기사가 총 10 ~ 20개일 경우는 10개만, 5 ~ 10개는 5개, 1 ~ 5개는 1개만 크롤링 진행
<br>

- **각 뉴스 헤드라인의 감성을 라벨링 한 기준**
    - 일반화된 라벨링을 할 수 없기 때문에 당일의 주가 변동성의 상승/하락을 문장의 긍정/부정이라 가정하고 라벨링
    - 주가 변동성의 상승/하락은 당일 종가 - 전일 종가의 부호에 따라 분류
    - 당일 종가: 27000원, 전일 종가: 25000원 → 주가 변동 = +2000원 > 0 → 상승(1)
    - 당일 종가: 24000원, 전일 종가: 25000원 → 주가 변동 = -1000원 < 0 → 하락(0)
<br>

### ✅ 데이터 EDA & 전처리

- **감성 분류 → 주가 변동성 분류와의 연관성을 연구하기 위한 목적이기 때문에 주가 변동별 EDA 진행**
    - 종목별, 연도별, 종목 연도 별 뉴스 헤드라인 개수, 길이 확인
    - 특정 종목의 상승/하락과 무관하다고 판단되는 헤드라인 제거 (3000여개)
        - ex) [사진],<표>,[인사],[부고],[프로필],[오늘의 메모] 등
<br>
    
- **형태소 분석 전 text 전처리 기준(정규표현식 사용)**
    - 구두점
        - 구두점으로 표기된 형태는 공백으로 replace 처리하면서 전처리 진행
            - ex) …, .., ·
    - 한자
        - 헤드라인에 표기된 한자 자체만으로도 뜻이 있는 경우
            - ex) 국가명(韓, 美, 中), 대통령 성씨(朴, 文), 기관명(銀, 中企), 종목(株)
    - 영어
        - 영어가 포함된 헤드라인은 M&A, R&D, GDP 등 주요 경제 용어가 다수 포함되어 있어 띄어쓰기 기준으로라도 포함 진행
    - 숫자
        - 숫자는 ❌
        - 경제 관련 기사에서 숫자는 헤드라인 논조에서 중요한 역할을 담당하지만 단위, 시간 등에 따라 다른 숫자를 명확한 기준으로 전처리할 수 없기에 제외
    - 특수문자
        - ↓ , ↑ 와 같은 경우, 특수문자 자체로도 의미가 형성되므로 하락과 상승으로 전처리
    - 기타
        - 헤드라인에 “미리보는” “이데일리” 등 언론사나 상승/하락과 무관한 지표 제거
        - 다중 공백 제거
<br>

- **Okt, Mecab의 형태소 분석기를 통해 EDA 진행**
    - OKT를 활용해 주가 상승/하락 별 품사(형용사, 부사, 동사, 명사 등) 빈도 수 확인
        - 주가 상승/하락 별 형용사, 부사 분석
        
        ![morphs](https://user-images.githubusercontent.com/92377162/151213827-e84468df-aaac-45b4-910c-1e62fe30966e.png)
        
    - Mecab을 통해 n-gram 별 주가 상승/하락 별 keyword 빈도 수 확인
        - 3-gram mecab_token (상승/하락) 분석
    
        ![ngram_up](https://user-images.githubusercontent.com/92377162/151214166-0000b99c-08cb-44a9-b05e-e7fd9e331ff8.png)
    
        ![ngram_down](https://user-images.githubusercontent.com/92377162/151214219-dc4d28a3-3914-4b7d-8dd5-49ccf7b4a5bf.png)
   
    - **시황**이라는 키워드가 포함된 헤드라인은 말그대로 현재 종목가의 상황을 중계하는 것으로 노이즈라 판단, 시황이 포함된 헤드라인 모두 제거 (8000여개)
<br>

- **Mecab 전처리**
    - mecab-ko-dic 품사태그에서 실질형태소 체언, 용언, 수식언 형태소만 사용
        
        ![mecab](https://user-images.githubusercontent.com/92377162/151214750-63df5f53-1d05-4afe-a4d4-fc2165157945.png)
        
        - 체언 : 일반명사, 고유명사, 의존명사, 수사, 대명사
            - ['NNG', 'NNP', 'NNB', 'NR', 'NP']
        - 용언 : 동사, 형용사, 긍정지정사, 부정지정사
            - ['VV', 'VA', 'VCP', 'VCN']
        - 수식언 : 관형사, 일반부사
            - ['MM', 'MAG']
        - 외국어: 영어만 포함
            - [’SL’]
        
        ※ 한 글자 token은 제거
<br>

- **단어 등장 빈도 EDA 후 전처리**
    - 전체 헤드라인 분포 확인 후 주가 변동 상승/하락으로 나누어 분포 확인
    - 상승/하락의 공통어 빈도수를 확인하면서 불용어 처리
    
    ![up](https://user-images.githubusercontent.com/92377162/151215185-63527e46-d9f3-4f39-8737-68ac3ac44ca7.png)

    ![down](https://user-images.githubusercontent.com/92377162/151215226-47fa3a4b-2911-495c-b4c9-342ad9abb5d2.png)
    
    - 상승/하락 헤드라인 상위 10개 token에서 공통으로 포함되는 용어를 불용어로 판단, 2번에 걸쳐 제거

<br>

### ✅ 머신러닝 & 딥러닝 분류기를 통한 뉴스 헤드라인의 감성지수 추출

- **뉴스 헤드라인 수집 후 머신러닝, 딥러닝 분류기를 통해 문장의 긍정, 부정 확률 값 추출**
    - tf-idf encoding 후 4개의 머신러닝 분류기(naive_bayes, sgd, lgbm, xgboost)에 대해 stacking후 stacking 모델을 통해 예측
    - Keras test_to_sequences encoding 후 LSTM, GRU를 이용한 이진 분류 문제를 수행하는 모델을 통해 예측
    - Hugging Face의 PyTorch BERT의 pretained 모델을 통해 예측
        - BERT의 기본 모델인 bert-base-multilingual-cased 사용
    - BERT 기반 한국어 분류기 딥러닝 모델 KoBERT 모델을 통해 예측
        - 기존 KoBERT 모델에 네이버 영화 평점 리뷰 데이터 추가 학습 진행
<br>

### ✅ 학습 모델을 통한 Test Set 평가

- **예측기로서 동작할 수 있는 Selection Function 생성**
    - 특정날짜의 주가 기사문장이 N개라고 했을 때, N개의 문장의 감성 분류스코어를 계산한 확률평균을 그날 주식 종가의 업다운을 결정하는 예측 값으로 사용
    - 문장 판별 확률의 평균으로 선택한 이유 -> 일반화된 테스트 데이터가 아님
<br>

- **10개의 각 종목별 예측 지표 확인**
    - 종목별 예측 정확도 비교를 위해 accuracy 확인
    - 주가의 변동성을 분류 예측하는 문제이며 문장 판별 확률의 평균을 사용했기 때문에 상승/하락의 precision 확인
    - 그 외 전체적인 지표를 확인하기 위해 confusion matrix, classification report 확인
<br>

![sample1](https://user-images.githubusercontent.com/92377162/151176773-eecf69e6-f89d-40e1-bfe5-30c46b21afd2.png)

<br>

## 🚧 프로젝트 한계 및 보완점

- **(크롤링 데이터 과정) 본문 학습 X, 주식 종목과 연관성이 낮다고 판단되는 기사 헤드라인 수집**
    - 모델 학습과 크롤링 과정에서 물리적 시간의 한계
        - 본문까지 활용한 자연어분석을 진행하여 평균을 내는 방식으로 develop 가능
    - 네이버 금융 뉴스 내 카카오 종목 검색 시 카카오와 연관이 낮은 헤드라인이 크롤링
        - “00제과 신제품 출시, **라이언과 니니즈**에 힘입어 매출 상승” — 카카오 종목과 관련없는 헤드라인(카카오 캐릭터에 대한 내용)에 대한 전처리에 대한 한계 존재
        - 본문 추가학습이나 종목과 관련있는 전처리 후 반복 크롤링 작업 수행 필요
<br>

- **(라벨링 과정) 특정 분야(금융, 유통, 바이오 등)에 대한 감성 분석의 한계**
    - 금융 뉴스 감성 분석은 관련 전문용어와 라벨링된 데이터가 존재하지 않으므로, 특정 종목에 대한 상승, 하락 지표로 임의로 선정함 (감성 라벨 지표 ❌)
    - 학습된 딥러닝 모델에선 특정 도메인에서 사용되는 전문 용어를 기반으로 학습이 진행되었기에 명확한 지표를 얻기가 어려움
<br>
    
- **(딥러닝 모델 구현 과정)  하이퍼파라미터 튜닝 최적화, 과적합 원인 미해결, 전체적인 모델 layer 구성, 다양한 embedding 기법 등**
    - 모델 구현에 앞서 기존 텍스트에 전처리하는 과정이 많이 소요
        - 모델에 대한 성능을 높이기 위해 하이퍼파라미터 튜닝 최적화를 진행할 수 있었으나, 낮은 test 평가 지표로 인해 전처리 과정에 집중됨
        - 모델에 대한 성능에 초점을 두고 layer 구성 및 다양한 embedding 기법을 조합하여 develop 가능
<br>

## 💻 개발 환경

<div align=left> 
   <img src="https://img.shields.io/badge/python-3776AB?style=for-the-badge&logo=python&logoColor=white"> 
   <img src="https://img.shields.io/badge/jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white">
   <img src="https://img.shields.io/badge/google colab-F9AB00?style=for-the-badge&logo=google colab&logoColor=white">
   <img src="https://img.shields.io/badge/selenium-43B02A?style=for-the-badge&logo=selenium&logoColor=white"> 
   <img src="https://img.shields.io/badge/scikit learn-F7931E?style=for-the-badge&logo=scikit learn&logoColor=white">
   <img src="https://img.shields.io/badge/tensorflow-FF6F00?style=for-the-badge&logo=tensorflow&logoColor=white">
   <img src="https://img.shields.io/badge/pytorch-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white"> 
   <br>
</div>
<br>

## 🔗 참고 문헌 및 링크

- KoBERT로 감성 분석을 해보자(Text Classification)
    - [https://tech-diary.tistory.com/31?category=952584](https://tech-diary.tistory.com/31?category=952584)
<br>

- 네이버 영화 평점 감성분석
    - [https://github.com/e9t/nsmc/](https://github.com/e9t/nsmc/)
<br>

- [NLP] 텍스트 분류와 감성(Sentiment)분석 구현하기
    - [https://techblog-history-younghunjo1.tistory.com/111?category=924148](https://techblog-history-younghunjo1.tistory.com/111?category=924148)
<br>

- 네이버 영화 리뷰 감성 분류하기(Naver Movie Review Sentiment Analysis)
    - [https://wikidocs.net/44249](https://wikidocs.net/44249)
<br>

- [Python, KoBERT] 다중 감정 분류 모델 구현하기 (huggingface로 이전 방법 O)
    - [https://hoit1302.tistory.com/159#6.앞으로의기대](https://hoit1302.tistory.com/159#6.%EC%95%9E%EC%9C%BC%EB%A1%9C%EC%9D%98%EA%B8%B0%EB%8C%80)
<br>

- KoBERT 네이버 영화 리뷰 감성분석 with Hugging Face
    - [https://complexoftaste.tistory.com/2](https://complexoftaste.tistory.com/2)
<br>

- SKTBrain - KoBERT 오픈소스
    - [https://github.com/SKTBrain/KoBERT](https://github.com/SKTBrain/KoBERT)
<br>

- 데이터 사이언티스트 한국어 자연어처리 방법
    - [https://blog.naver.com/reasoninlife/222574624867](https://blog.naver.com/reasoninlife/222574624867)
<br>

- 뉴스데이터를 활용한 주가 예측2 (감성분석)
    - [https://blog.naver.com/dalgoon02121/222051184805](https://blog.naver.com/dalgoon02121/222051184805)
<br>

- [Keras]기사 제목을 가지고 긍정 / 부정 / 중립으로 분류하는 모델 만들어보기
    - [https://somjang.tistory.com/entry/Keras기사-제목을-가지고-긍정-부정-중립-분류하는-모델-만들어보기#google_vignette](https://somjang.tistory.com/entry/Keras%EA%B8%B0%EC%82%AC-%EC%A0%9C%EB%AA%A9%EC%9D%84-%EA%B0%80%EC%A7%80%EA%B3%A0-%EA%B8%8D%EC%A0%95-%EB%B6%80%EC%A0%95-%EC%A4%91%EB%A6%BD-%EB%B6%84%EB%A5%98%ED%95%98%EB%8A%94-%EB%AA%A8%EB%8D%B8-%EB%A7%8C%EB%93%A4%EC%96%B4%EB%B3%B4%EA%B8%B0#google_vignette)





