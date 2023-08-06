---
title: "[AIFFEL] 브라질 Olist 이커머스 분석"
image: 
  path: /assets/img/content/pro3/1.png
  thumbnail: /assets/img/content/pro3/1.png
---

**요약**

- 브라질 Olist 이커머스 데이터를 이용해 다양한 시각화와 입점하려는 판매자에게 유용한 정보를 주는 목표

**역할**

- 브라질 지도에 배송 소요일, 평균 소비 금액, 상품의 리뷰 평점 사이의 관계를 시각화하는 역할

**성과**

- 판매자에게 100R$ 위주의 카테고리(제품)을 선택하고, 일회성 구매자를 고려하라는 정보 도출

**시기**

- 2022.09

**링크**

[https://github.com/darkhairlove/AIFFEL_DATATHON](https://github.com/darkhairlove/AIFFEL_DATATHON)



# 분석 배경

- Olist는 온라인 백화점이라 불리는 서비스입니다.
- 판매자들이 Olist에 상품정보를 등록하면 Olist는 그 상품정보를 여러 온라인 마켓사이트에 등록해주고 주문관리를 간편하게 만들어 줍니다.
- 온라인 마켓사이트에서 주문이 들어오면 즉시 판매자에게 알려주고 물류도 처리해줍니다.

# Olist 분석하기 위해서

## 데이터

- 2016년 10월부터 2년간의 주문, 상품, 고객, 판매자에 대한 기록입니다.
- 자세한 상품명과 회사명은 비식별화 처리되어 있습니다.
- 데이터 관계도

![olist_dataset.png](/assets/img/content/pro3/2.png)

## EDA

- 결측치 필터링 : 1603개 발견 → `TO10_dataframe = T10_dataframe.dropna(axis = 0)`

```python
order_id                    0
product_id                  0
order_item_id               0
product_category_name    1603
dtype: int64
```

- 포르투갈어 → 영어로 변경
- merge 작업
    
    ![스크린샷 2023-03-01 오후 2.53.40.png](/assets/img/content/pro3/3.png)
    
- 판매량 상위 10위 제품 시각화
    
    ![스크린샷 2023-03-01 오후 2.55.13.png](/assets/img/content/pro3/4.png)
    

## 지도 시각화

- 지역별 평균 소비금액, 상품 배송일과 상품의 리뷰 평점 사이의 관계를 시각화
- 지역별 총소비 금액(수익)
    
    ![스크린샷 2023-03-01 오후 2.56.31.png](/assets/img/content/pro3/5.png)
    
    지역별 총구매 금액 상파울루(SP)는 총구매금액이 많지만, 고객이 주문당 가격을 적게 지불합니다.
    
- 지역별 1인당 평균 구매가격
    
    ![스크린샷 2023-03-01 오후 3.01.26.png](/assets/img/content/pro3/6.png)
    
    브라질 남부 및 남동부 지역의 고객들은 북부 및 북동부 지역의 다른 고객들보다 평균 구매가격이 더 낮습니다.
    
- 지역별 평균 배송비와 총배송비

![지역별 평균 배송비](/assets/img/content/pro3/7.png)

지역별 평균 배송비

![지역별 총배송비](/assets/img/content/pro3/8.png)

지역별 총배송비

- 지역별 운송 비율
    
    배송비를 주문비로 나누면 `운송 비율`을 찾을 수 있습니다. 
    이 비율은 주문품을 배달하기 위해 사람이 지불해야 했던 제품 가격의 백분율입니다.
    
    예를 들어, 제품의 가격이 50.00이고 배송비가 10.00인 경우 운송 비율은 0.2 또는 20%입니다.
    
    높은 운송 비율은 고객들이 구매를 완료하지 못하게 할 가능성이 매우 높습니다. 물류비용 때문에 인구 밀도가 높은 지역에서는 낮은 운임 비율이, 희박한 지역에서는 높은 운송 비율이 예상됩니다.
    
    ![https://github.com/darkhairlove/AIFFEL_DATATHON/raw/main/images/geo7.png](/assets/img/content/pro3/9.png)
    
- 리우의 평균 배송일과 리뷰 스코어, 지연 비율

![리우의 평균 배송일](/assets/img/content/pro3/10.png)

리우의 평균 배송일

![리우의 리뷰 스코어](/assets/img/content/pro3/11.png)

리우의 리뷰 스코어

![리우의 지연 비율](https://github.com/darkhairlove/AIFFEL_DATATHON/raw/main/images/geo14.png)

리우의 지연 비율

![https://github.com/darkhairlove/AIFFEL_DATATHON/raw/main/images/geo13.png](/assets/img/content/pro3/12.png)

지도상에서 평균 배송일과 리뷰 평점이 관계가 있어 보여서 상관관계 계수를 계산했습니다. 
두 Feature 사이에 상관관계가 상당히 있어 인과관계를 분석할 필요가 있어 보입니다.

- 총정리
    
    ![https://github.com/darkhairlove/AIFFEL_DATATHON/raw/main/images/geo15.png](/assets/img/content/pro3/13.png)
    
    평균 배송비, 평균 배송일, 평균 배송 예측일과 배송일의 차이를 위와 같이 볼 수 있다.
    

## 고객 분석

- 시간에 따른 구매
    1. 브라질 전자상거래의 성장 추세가 있는지?
    2. 브라질 고객들은 어떤 요일에 온라인 구매를 하는지?
    3. 브라질 고객은 보통 몇 시(새벽, 아침, 오후 또는 밤)에 구매하는지?
        
        ![https://github.com/darkhairlove/AIFFEL_DATATHON/raw/main/images/geo16.png](/assets/img/content/pro3/14.png)
        
    
    위의 도표를 통해 다음과 같은 결론을 내릴 수 있습니다.
    
    - 브라질에서의 전자상거래는 시간이 지남에 따라 정말로 증가하는 추세를 가지고 있습니다.
    - 특정 월에 피크가 나타나는 계절성을 볼 수 있지만 일반적으로 고객이 이전보다 온라인으로 물건을 구매하는 경향이 더 높다는 것을 알 수 있습니다.
    - 월요일은 브라질 고객들이 선호하는 날이고 그들은 오후에 더 많이 사는 경향이 있습니다.

### 매출 대부분을 발생시킨 소비자와 판매자

- 총소비금액 기준으로 상위 몇 퍼센트의 소비자가 대부분의 소비를 했는지?
    - 일회성 소비자가 많은 현상과 상관있을 것 (충성소비자가 적음)

![https://github.com/darkhairlove/AIFFEL_DATATHON/raw/main/images/consumer1.png](/assets/img/content/pro3/15.png)

- 총판매금액 기준 상위 몇 퍼센트의 소비자가 대부분의 매출을 올렸는지?
    - 파레토 법칙을 거의 따르고 있습니다.

![https://github.com/darkhairlove/AIFFEL_DATATHON/raw/main/images/consumer2.png](/assets/img/content/pro3/16.png)

### 고객 RFM 분석

### **Recency :** 고객이 얼마나 최근에 구매했는가?

- (데이터셋의 마지막 날짜) - (고객의 마지막 구매일)
- 현상
    - 2016년 11월, 12월은 판매 기록이 없다.
    - 1년 이내에 구매 이력이 있는 고객이 많다.

![https://github.com/darkhairlove/AIFFEL_DATATHON/raw/main/images/consumer3.png](/assets/img/content/pro3/17.png)

### **Frequency** : 각 고객이 구매한 횟수

- 고객당 주문수(order_id)를 세어서 측정
- 1회 구매한 고객이 약 97%

![https://github.com/darkhairlove/AIFFEL_DATATHON/raw/main/images/consumer4.png](/assets/img/content/pro3/18.png)

### Monetary : 고객별 총구매금액

- 박스플롯
    - 적은 금액에 몰려있습니다.
    - 4만 헤알(R$), 11만 헤알(R$)을 소비한 아웃라이어가 존재합니다.
        
        ![https://github.com/darkhairlove/AIFFEL_DATATHON/raw/main/images/consumer5.png](/assets/img/content/pro3/19.png)
        
- 히스토그램
    - x축에 밑이 10인 log를 씌워서 히스토그램 확인
    - 평균가격대 (110~192 R$) 주변 가격이 높습니다.
    - 각 주별로 평균 가격대

![https://github.com/darkhairlove/AIFFEL_DATATHON/raw/main/images/consumer7.png](/assets/img/content/pro3/20.png)

![https://github.com/darkhairlove/AIFFEL_DATATHON/raw/main/images/consumer6.png](/assets/img/content/pro3/21.png)

- RFM score
    - ( Recency + Frequency + 2 * Monetary ) / 4로 계산

![https://github.com/darkhairlove/AIFFEL_DATATHON/raw/main/images/consumer8.png](/assets/img/content/pro3/22.png)

- RFM 점수 3D 시각화
    - 대부분 한번 구매하고, 1,000헤알 이하 금액대로 구매

![스크린샷 2023-03-01 오후 4.27.18.png](/assets/img/content/pro3/23.png)

- 소비자 Class별 R점수와 M점수
    - Class는 RFM 점수를 기준으로 4분할
    - R 점수, M 점수는 Recency, Monetary를 100점으로 환산했습니다.

![https://github.com/darkhairlove/AIFFEL_DATATHON/raw/main/images/consumer10.png](/assets/img/content/pro3/24.png)

### 시간에 따른 판매 추세

- 판매량이 Top3인 카테고리를 골라서 월별 판매량을 시각화했습니다.

### 침실&욕실물품

- 2018년부터는 안정적으로 600건 이상 판매되었습니다.
- 11월은 블랙프라이데이 영향으로 구매량이 대폭 증가했습니다.

![https://github.com/darkhairlove/AIFFEL_DATATHON/raw/main/images/consumer11.png](/assets/img/content/pro3/25.png)

### 헬스&뷰티

- 꾸준히 증가하는 추세를 보입니다.

![https://github.com/darkhairlove/AIFFEL_DATATHON/raw/main/images/consumer12.png](/assets/img/content/pro3/26.png)

### 스포츠&레저

- 6월에 매출이 떨어진 이유를 겨울이여서로 볼 수 있다.

![https://github.com/darkhairlove/AIFFEL_DATATHON/raw/main/images/consumer13.png](/assets/img/content/pro3/27.png)

# 결과

- 다양한 측면에서 시각화를 진행하였습니다.
- 대부분의 소비가 100~1,000 헤알(R$) 금액대에서 일어났습니다.
- 반복 구매자 비율은 약 2%로 매우 작았습니다.
- 배송일이 늦을수록 리뷰 평점이 낮은 경향이 있었습니다.
- 판매자에게 조언
    - 100 R$ 위주의 카테고리(제품)를 선택
    - 일회성 구매자를 고려

# 한계점

- 각 카테고리 별로 배송 소요일과 리뷰 평점 간의 관계을 도출하지 못했습니다.
- 브라질의 한 개의 State에서 배송 소요일이 긴 카테고리 제품과 짧은 카테고리 제품을 분석하지 못했습니다.
- Frequency 분석에서 2회, 3회 이상 구매한 사람들과 그 상품 판매자의 특징을 파악하지 못했습니다.

# 느낀점

- Geo Location 데이터를 이용해서 지도에 배송 소요일과 리뷰 평점을 시각화하는 법을 배웠습니다.
- 시간에 따른 구매를 보고 브라질 고객이 어떤 요일에 많이 구매하는지와 몇 시에 구매하는지 알 수 있었습니다.
- 하나의 타깃 예를 들어 1개의 state 또는 1개의 카테고리를 기준으로 EDA를 세밀하게 계획하는 것이 중요하다는 것을 배웠습니다.
