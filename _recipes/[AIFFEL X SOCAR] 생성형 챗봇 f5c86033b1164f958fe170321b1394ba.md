---
title: "[AIFFEL X SOCAR] 생성형 챗봇"
image: 
  path: /assets/img/content/pro1/1202.jpg
  thumbnail: /assets/img/content/pro1/1.jpg
---

**요약**

- 현재 쏘카의 고객문의 챗봇을 개선하기 위해, 관련 데이터로 학습한 생성형 챗봇을 만들어 사용자 편의를 높임
- NLP로 사용자의 문의를 해석하고 답변을 생성하는 부분을 구현
- CV, Image2Text를 통해 고객이 올린 사진을 인식하여 문제에 대한 분류를 구현
- 고객과 채팅, 업로드된 이미지를 기반으로 필요한 답을 생성하는 챗봇 구현

**역할**

- 쏘카 메모 데이터에서 **키워드를 추출**하고 **생성형 모델 KoDialoGPT 구현**
- 쏘카 홈페이지에서 FAQ **데이터 수집**

**성과**

- 생성형 챗봇에 필요한 데이터를 KoBART로 정제하고 FAQ데이터 수집하여 
Perplexity 평가 지표 29 → 2로 낮아짐

**시기**

- 2022.12 ~ 2023.2

**링크**

[https://github.com/darkhairlove/AIFFEL_SOCAR_ChatBot](https://github.com/darkhairlove/AIFFEL_SOCAR_ChatBot)


# 생성형 챗봇을 만들게 된 배경

- 현재 쏘카에서는 시나리오 베이스로 되어 있어 고객이 불편함을 느끼는 문제점이 있습니다.
    - [나무위키 (쏘카)](https://namu.wiki/w/%EC%8F%98%EC%B9%B4)
        
        ![Untitled](/assets/img/content/pro1/2.png)
        
- [생성형 기반에 챗봇](https://tech.scatterlab.co.kr/luda-gen-1/)이 연구되고 있는 상황입니다.
- 멀티모달(Multi-Modal)의 Image to Text을 응용할 수 있습니다.
- 상담 업무의 효율화와 비용 절감할 수 있습니다.

# 챗봇을 만들기 위해서

## 데이터 전처리 및 준비

- 쏘카 메모 데이터에 어떤 문의 사항이 많은지 파악하기 위해 **WordCloud 사용하여 데이터 EDA**를 했습니다.
    
    ```python
    categories = list(qkey_df['category'])
    
    # 한국어 불용어 사전을 정의
    stopwords = set(stop_words)
    
    # CREATE WORDCLOUD
    wc = WordCloud(
        font_path='/usr/share/fonts/truetype/nanum/NanumGothic.ttf',
        background_color='white',
        collocations=False,
        stopwords=stopwords,
        width=800, 
        height=400)
    wc.generate(' '.join(categories))
    
    # Plot wordcloud
    plt.figure(figsize=(15,10)) # plot size
    plt.imshow(wc) 
    plt.axis('off')             # axis delete
    plt.show()
    ```
    
- 쏘카 메모 데이터에서 정규 표현식을 사용하여 **문의 내용을 추출**했습니다.
    
    ```python
    # 정규표현식으로 궁금해용 추출 함수 정의
    def regularexp(string):
        # 비식별화된 내용입니다.
        regular = re.compile('(?<=퀘션[해사 ][퀘션사항 ][용항 ])(.*?)(?=[ㅁ확종긍])')
        string = string.replace(':', ' ').replace('\n', ' ').replace('  ', ' ')
    
        result1 = regular.search(string).group()
        result2 = re.sub('[^가-힣 ]+', ':', result1).split(':')
    
        list_query = []
        for sentence in result2:
            if sentence == '' or sentence == ' ': continue
            sentence = sentence.strip()
            list_query.append(sentence)
    
        return list_query
    
    # 궁금해용 있는 행 인덱스 추출 
    find_words = ['퀘션내용', 'ㅁ퀘션내용', '퀘션 내용',
                  'ㅁ퀘션', '퀘션사항', 'ㅁ퀘션사항', '퀘션 사항', '퀘션구분']  # 비식별화된 내용입니다.
    
    index = []
    for idx, row in df_memo.iterrows():
        memo = str(row[9]).replace(":", " ").split()
        for word in find_words:
            if word in memo:
                index.append(idx)
    
    # 문의내용 추출 후 query 컬럼에 넣기
    key_df = df_memo.loc[index]
    key_df = key_df.reset_index()
    
    key_df['query'] = 0
    
    for idx in range(len(key_df)):
        str1 = regularexp(key_df['memo'][idx])
        key_df['query'][idx] = str1
    ```
    
- 쏘카 홈페이지에서 [자주 묻는 질문 FAQ](https://socarhelp.zendesk.com/hc/ko/categories/360003929833) **데이터를 수집**했습니다.
    
    ```python
    df2 = pd.DataFrame(columns=['Q', 'A'])
    for i, v in zip(qn,an1):
        df2 = df2.append(pd.DataFrame([[i,v]], columns=['Q','A']), ignore_index=True)
    df2
    ```
    
    ![스크린샷 2023-03-02 오전 11.28.59.png](/assets/img/content/pro1/3.png)
    

## 키워드 추출 모델

- 전처리한 데이터로 `{부품 A : [’밀림’, ‘소음’, ‘고장’], 부품 B : [’마모’, ‘공기압경고등’]}` 과 같이 **매핑 테이블**을 만들었습니다.
    - 매핑 테이블을 사용한 이유
        
        주어진 쏘카 메모 데이터를 활용하기 위해서 dictionary 형태로 만들면 
        중복된 key일 경우, 그 key 안에 value로 다양한 단어를 넣을 수 있기 때문에 사용했습니다.
        
- 사용자가 입력한 문의 내용이 들어오면 **맞춤법 검사기**를 통과한 후, 형태소 분석기 **Mecab, Kkma**로 명사 추출 후 **불용어**을 제거했습니다. 마지막으로 **KeyBERT**을 사용해 키워드를 추출합니다.
    - 형태소 분석기 **Mecab, Kkma**를 사용한 이유
        
        Oka, Komorma, Mecab, Kkma와 같은 다양한 형태소 분석기를 사용한 결과, 쏘카에 관련된 명사가 잘 나오는 형태소 분석기가 Mecab과 Kkma이기 때문에 사용했습니다.
        
    - KeyBERT 모델을 사용한 이유
        
        텍스트 임베딩을 형성하는 단계에서 BERT를 사용하여 **N-gram 모델을 사용해 문서의 각 문장을 단어로 쪼갭니다.** 문서와 마찬가지로 쪼갠 이후 BERT를 통해 각 단어에 대한 임베딩 벡터를 추출한다.
        그로 인해 키워드와 문서 간 거리는 최소로 하고, 키워드와 키워드 간 거리는 최대로 하면서 원하는 키워드가 반영되어 추출되기 때문입니다.
        
        ![스크린샷 2023-02-23 오후 5.10.16.png](/assets/img/content/pro1/4.png)
        
        [KeyBERT 모델 GitHub](https://github.com/MaartenGr/KeyBERT)
        
- 추출한 키워드가 매핑 테이블에 있는 경우
    - 최소 두 개 단어 추출 → 매핑 테이블 values에서 단어 횟수 세기 → values들의 합산 값 비교 → values 상위 합산 값의 key를 반환
    - 반환된 key로 시나리오에 맞는 대답을 출력했습니다.
        
        ```python
        def count_values_keywords(keywords, user_sent, mapping_table_total):
            # (1) 카운트 딕셔너리 만든 후 매핑테이블 키들에 대해 0으로 초기화
            count_dict = {}
            for k in mapping_table_total.keys():
                count_dict[k] = 0
        
            # (2) 매핑테이블 각 Key의 value에 추출된 키워드가 몇 개 존재하는지 개수 세기
            for keyword in keywords:
                for key in mapping_table_total.keys():
                    count = 0
                    for value in mapping_table_total[key]:
                        if keyword == value:
                            count += 1
        
                    count_dict[key] = count_dict[key] + count
        
                # (3) 추출된 키워드가 카테고리 이름과 같다면 가중치로 1 추가해줌
                if keyword in count_dict.keys():
                    count_dict[keyword] = count_dict[keyword] + 1
            
            # (4) 카운트 딕셔너리 내림차순으로 정렬       
            sorted_dic = dict(sorted(count_dict.items(), key=lambda x: x[1], reverse=True))
        
            # (5) 카테고리 선택
            if max(sorted_dic.values()) == 0:    # 어떤 키워드도 매핑테이블에 존재하지 않는 경우
                print('최대값이 0 -> 키워드 존재하지 않는 경우를 수행')
                print('FastText 함수 호출')
                final_key = 'FastText 함수 결과값'
                return (final_key, _ )
            else:
                final_key, output_list = existing(sorted_dic, user_sent) 
                return (final_key, output_list)
        
        final_key, output_list = count_values_keywords(keywords, user_sent, mapping_table_total)
        print(final_key)
        ```
        
        ![스크린샷 2023-03-02 오후 12.26.58.png](/assets/img/content/pro1/5.png)
        
- 추출한 키워드가 매핑 테이블에 없는 경우
    - **FastText**을 매핑 테이블 value값으로 학습시킨 후, 사용자가 입력한 문장을 그대로 넣어 유사한 단어를 추출했습니다.

## 문제 발생

처음에는 키워드 추출 모델을 사용해서 시나리오 베이스 챗봇을 구상했습니다.
하지만 시나리오 베이스로 하면서 다음과 같은 문제점이 발생하였습니다.

1. 다양한 대답이 나오지 않았습니다. 
2. 현재 쏘카에서 사용하는 챗봇은 시나리오 기반이기 때문에 차별점이 없고, 비용 절감이 될 수 없는 문제점이 있습니다.

팀원과 회의를 한 결과, 생성형 모델 기반에 챗봇으로 가야 다양한 답변과 현재 쏘카에서 이용하는 챗봇과의 차별성이 있다는 점으로 문제를 해결했습니다.

## 생성형 모델

- [KoBART](https://github.com/darkhairlove/AIFFEL_SOCAR_ChatBot/tree/main/KoBART)
    
    문맥 파악 Encoder(BERT)를 통해 특정 분야의 답변 생성이 장점입니다.
    하지만 특정 문장(모든 글자가 같은 문장)에 대하여 다양한 답변에 부족합니다.
    
- [KoDialoGPT](https://github.com/darkhairlove/AIFFEL_SOCAR_ChatBot/tree/main/KoDialoGPT)
    
    모델의 응답이 하나의 질문에 여러 개의 답변이 있는 대화의 one-to-many 가 가능한 장점입니다.
    하지만 다음 문장도 같이 넣어 출력하는 길이의 제약이 있고, 데이터셋이 대화형이어야 합니다.
    
- 평가 지표
    - Perplexity
    특정 확률 모델이 실제로 관측되는 값을 얼마나 잘 예측하는지를 뜻합니다.
    Perplexity는 낮을수록 모델의 성능이 좋습니다.
    확률 모델이 다른 모델에 비해 얼마나 개선되었는지 평가할 때 사용합니다.
    토픽 모델링 기법이 얼마나 빠르게 수렴하는지 확인할 때 사용한다.
    - Perplexity를 사용한 이유
        
        모델 평가 시 활용한 테스트 데이터에서 KoBART, KoDialoGPT 두 모델이 얼마나 높은 정확도를 보였는지 알 수 있기 때문입니다.
        
    - BLEU
    카운트 기반 측정 방식입니다.
    n-gram 정밀도를 이용하여 겹치는 토큰 수를 측정합니다.
    단, 예측문장의 중복된 단어 수는 실제 문장의 중복단어 수를 초과하지 못합니다.

# 결과

| Perplexity                | KoBART | KoDialoGPT |
| ------------------------- | ------ | ---------- |
| QA 문맥 정제 + FAQ 데이터 | 5.258  | 2.041      |

![FAQ 질의 응답](/assets/img/content/pro1/6.gif)

FAQ 질의 응답

![시동 관련 문제](/assets/img/content/pro1/7.gif)

시동 관련 문제

# 한계점

- 사용자의 이전 질문에 대해 기억하지 못합니다.
- QA 문맥 정제 + FAQ 데이터의 양이 2,000개로 적어 FAQ, 시동 외에 다른 주제의 답변을 생성하지 못합니다.

# 느낀점

- 모델에 대한 논문을 찾아보고, 왜 사용하는지에 대한 근거를 찾아보는 경험이 되었습니다.
- 생성형 모델과 키워드 추출 모델을 함께 사용하여 챗봇을 구현해보고 싶습니다.