---
title: "[Kaggle] ICR - Identifying Age-Related Conditions"
image: 
  path: /assets/img/content/pro4/1.png
  thumbnail: /assets/img/content/pro4/1.png
---

**요약**

- 익명화된 개인 건강 정보를 통해 질병 진단을 예측하는 대회
- 피험자가 이러한 질환 중 하나로 진단되었는지 여부를 예측하는 binary classification 문제

**역할**

- PCA를 사용하여 차원 축소하여 평가 지표 값 확인
- AutoML으로 하이퍼파라미터 찾는 역할

성과

- Leaderboard Score : 0.46
- Cross Validation Score : 0.338992

**기간**

- 2023.06 ~ 2023.08

**GitHub 링크**

[https://github.com/darkhairlove/Kaggle_ICR](https://github.com/darkhairlove/Kaggle_ICR)

- [대회 링크](https://www.kaggle.com/competitions/icr-identify-age-related-conditions)


# 대회 배경

- 현재 노화는 전 세계 인구의 주요 사망 원인 중 하나로 다양한 질병과 관련이 있습니다.
- 노화와 관련된 질병을 예방하고 치료하는 새로운 방법을 찾고 있습니다.
- 빅데이터와 인공지능은 노화와 관련된 질병을 예방하고 치료하는 데 도움이 될 수 있습니다.

# 진단을 예측하기 위해서

## 데이터

- `train.csv`
    - `Id` 각 관측값에 대한 고유 식별자입니다.
    - `AB-GL` 익명화된 56개의 건강 특성. categorical인 `EJ`를 제외하고 모두 숫자입니다.
    - `class` binary target : `1`은 피험자가 세 가지 조건 중 하나로 진단받았음을 나타내고, `0`은 진단받지 않았음을 나타냅니다.
- `test.csv` : 피험자가 두 `class` 각각에 속할 확률을 예측하는 것
- `greeks.csv` - 훈련 집합에만 사용할 수 있는 보조 metadata
    - `Alpha` : 연령 관련 조건이 있는 경우 해당 유형을 식별합니다.
        - `A` : 연령 관련 조건이 없습니다. 클래스 `0`에 해당합니다.
        - `B, D, G` : 세 가지 연령 관련 조건. 클래스 `1`에 해당합니다.
    - `Beta`, `Gamma`, `Delta` : 세 가지 실험 특성입니다.
    - `Epsilon` : 이 피험자에 대한 데이터가 수집된 날짜입니다. 테스트 세트의 모든 데이터는 훈련 세트가 수집된 후에 수집되었습니다.

## EDA

- 결측치 확인 결과
    
    ```python
    BQ       60
    CB        2
    CC        3
    DU        1
    EL       60
    FC        1
    FL        1
    FS        2
    GL        1
    ```
    
- Class 별로 `greeks.csv`에 있는 `Beta`, `Gamma`, `Delta` 의 분포를 Bar Plot으로 확인
    
    ![__results___25_0.png](/assets/img/content/pro4/2.png)
    
- 각 Cloumn 별로 regplot을 확인 → 상관관계가 있는 Cloumn과 없는 Cloumn을 확인
    
    ![__results___26_0.png](/assets/img/content/pro4/3.png)
    
- 히트맵을 확인하여 Correlation 확인

![__results___28_0.png](/assets/img/content/pro4/4.png)

![스크린샷 2023-08-04 오후 4.29.59.png](/assets/img/content/pro4/5.png)

## Preprocessing & Feature Engineering

### Preprocessing features

- `KNNImputer`
    - 결측치 보간, ‘EJ’ column은 제외하고 사용
        
        ```python
        imp = KNNImputer()
        X_train_1 = imp.fit_transform(X_train)
        X_test_1 = imp.transform(X_test)
        
        tmp1 = pd.DataFrame(columns=X_train.columns, data = X_train_1)
        X_train = pd.concat([tmp1], axis=1)
        
        tmp2 = pd.DataFrame(columns=X_test.columns, data = X_test_1)
        X_test = pd.concat([tmp2], axis=1)
        ```
        
    - `KNNImputer` 사용한 이유
        
        각 컬럼의 데이터는 익명화된 건강 특성을 의미하고, 이것이 숫자로 변환된 상태입니다.
        따라서 각각의 숫자가 단순한 숫자가 아닌 특성을 나타내는 의미가 있기 때문에, 결측치를 보간하는 방법을 숫자의 특성이 반영될 방법을 사용해야 합니다.
        
        `KNNImputer` 는 KNN(K-Nearest Neighbor)알고리즘을 사용하여 결측치를 보간하는 방법으로 가까운 이웃의 수를 정하고 그 이웃들을 이용하여 결측치를 채우는 방식입니다. 즉, KNN 알고리즘은 거리가 중요합니다.
        

### Feature Engineering

- `Label Encoding`
    - ‘EJ’ column의 unique 값 A, B를 각각 0과 1로 변환
        
        ```python
        train['EJ'] = train['EJ'].replace({'A': 0, 'B': 1})
        test['EJ' ]= test['EJ'].replace({'A': 0, 'B': 1})
        ```
        
    - greeks의 ’Epsilon’ 제외한 나머지 cloumn에 LabelEncoder 적용
        - `sklearn`의 preprocessing 함수 중 LabelEncoder
            
            ```python
            greeks['Epsilon']= 0
            
            cols = greeks.columns
            
            for colin cols:
                le = LabelEncoder()
                greeks[col] = le.fit_transform(greeks[col])
            ```
            
    - 나머지 ‘Id’ 및 ‘Class’ column 을 제외한 나머지 feature들은 모두 numeric한 값
- `StandardScaler`
    - `sklearn`으로 Data Scaling
        
        ```python
        sc = StandardScaler() # MinMaxScaler or StandardScaler or RobustScaler
        X_train[numeric_columns] = sc.fit_transform(X_train[numeric_columns])
        X_test[numeric_columns] = sc.transform(X_test[numeric_columns])
        ```
        
    - `StandardScaler`을 사용한 이유
        
        거리 기반의 모델링을 할 때, 상대적으로 범위가 넓은 변수가 있다면, 거리 계산하는 과정에서 더 많은 기여를 하게 되어 중요한 변수 또는 영향력이 높은 변수로 인식됩니다. 이러한 방지를 막기 위해 적용했습니다. `StandardScaler`는 평균이 0, 분산이 1인 정규분포를 띄도록 데이터를 표준화하는 것입니다.
        
- `Oversampling`
    - `greeks.Alpha`를 기준으로 `SMOTE`
        
        ```python
        X_over, y_over = SMOTE(random_state=42).fit_resample(train, greeks.Alpha)
        ```
        
    - 오버샘플링 전 shape 체크:  `(617,)` → 오버샘플링 후 shape 체크:  `(2036,)`
        
        ![스크린샷 2023-08-04 오후 4.56.12.png](/assets/img/content/pro4/6.png)
        
    - `Oversampling` 으로 `SMOTE` 사용한 이유
        
        낮은 비율로 존재하는 클래스의 데이터를 K-NN 알고리즘을 활용하여 새롭게 생성하는 방법입니다. 이에 따라 과적합 발생 가능성이 단순 무작위 방법보다 적습니다. 따라서 과적합 발생 가능성을 가장 낮추기 위해 `SMOTE` 기법을 사용했습니다.
        
- PCA, VIF
    - PCA : 차원 축소와 변수 추출 기법으로 변수의 개수가 많아 새로운 변수를 가지고 모델링하는 방법
        
        ```python
        from sklearn.decomposition import PCA
        pca = PCA(n_components=35)
        pca.fit(X_train)
        X_train = pca.transform(X_train)
        X_test  = pca.transform(X_test)
        # 29번째까지 Explained Variance의 누적합이 0.861220 
        ```
        
        `X_train.shape (2036, 29)`
        
        ![스크린샷 2023-08-01 오후 11.32.26.png](/assets/img/content/pro4/7.png)
        
    - VIF : 다중회귀모델에서 독립 변수 간 상관관계가 있는지 보는 척도입니다. VIF가 10이 넘으면 다중공선성이 있다고 판단이 되고 5를 넘으면 주의가 필요합니다. 다중공산성(Multicollinearity)는 독립변수 간 상관관계가 있다는 것입니다.
        
        ```python
        def check_vif(df):
            vifs = [variance_inflation_factor(df, i) for i in range(df.shape[1])]
            vif_df = pd.DataFrame({"features":df.columns, "VIF" : vifs})
            vif_df = vif_df.sort_values(by="VIF", ascending=False)
            remove_col = vif_df.iloc[0, 0]
            top_vif = vif_df.iloc[0, 1]
            return vif_df, remove_col, top_vif
        
        top_vif = 100
        
        while(top_vif > 5):
             vif_df, remove_col, top_vif = check_vif(X_train)
             print(vif_df)
             print(remove_col, top_vif)
             if top_vif < 5:
                 break
        		 X_train = X_train.drop(columns=remove_col)
        X_test = pd.DataFrame(data=X_test, columns=X_train.columns)
        ```
        
        `X_train.shape (2036, 51)`
        
    - 최종 모델에 PCA, VIF를 사용하지 않은 이유
        
        Column을 줄이기 위해 사용했지만, balancedlogloss가 좋아지지 않아 사용하지 않습니다.
        기본 모든 데이터를 사용했을 때와 PCA와 VIF를 사용했을 때 비교한 결과입니다.
        
        |              | XGB      | LGBM     | CatBoost | RandomForest |
        | ------------ | -------- | -------- | -------- | ------------ |
        | 일반(Defult) | 1.000901 | 0.670095 | 0.890632 | 0.636166     |
        | VIF          | 0.924561 | 0.678577 | 0.966972 | 0.814292     |
        | PCA          | 1.246885 | 0.992418 | 1.348672 | 1.272332     |

## Modeling

### Cross Validation : StratifiedKFold 사용

- KFold : 교차검증을 위해서 가장 일반적으로 사용되는 방식
    
    ```python
    kf = KFold(n_splits = 5, shuffle = True, random_state = 721)
    ```
    
- StratifiedKFold : 데이터셋을 학습/검증 셋으로 나눌 때 데이터셋의 class별 비율을 동일하게 가져가도록 하는 것
    
    ```python
    kf = StratifiedKFold(n_splits = 5, shuffle = True, random_state = 721)
    
    for train_index, valid_index in kf.split(X_train, y_train):
    ```
    
- MultilabelstratifiedKFold: multi-label을 갖는 데이터들의 비율을 일정하게 나눌 수 있습니다.
`greeks`에서 ‘Alpha’, ‘Beta’, ‘Gamma’, ‘Delta’ columns  기준.
    
    ```python
    kf = MultilabelStratifiedKFold(n_splits = 5, shuffle = True, random_state = 721)
    
    for train_index, valid_index in kf.split(train, greeks.iloc[:,1:-1]):
    ```
    
- StratifiedKFold을 사용한 이유
    
    각 모델을 비교한 결과 balancedlogloss가 StratifiedKFold 좋아서 선택하였습니다.
    
    |                           | XGB      | LGBM     | CatBoost | RandomForest |
    | ------------------------- | -------- | -------- | -------- | ------------ |
    | KFold                     | 0.746435 | 0.661613 | 0.916079 | 1.195992     |
    | StratifiedKFold           | 0.559826 | 0.585273 | 1.000901 | 1.128134     |
    | MultilabelstratifiedKFold | 0.661613 | 0.712506 | 1.119652 | 1.162062     |

### Ensemble : CV Stacking

- 개별 모델이 예측한 데이터를 다시 training set으로 사용해서 학습
- 모델 개선 후 최종 개별 모델 및 메타 모델
    - 개별 모델
        - LGBMClassifier
        - CatBoostClassifier
        - HistGradientBoostingClassifier
        - RandomForestClassifier
    - 메타 모델
        - LGBMClassifier
- 원본 학습 데이터 Shape: `(2036, 56)` 원본 테스트 데이터 Shape: `(5, 56)` 
→ Stacking 학습 데이터 Shape: `(2036, 4)` Stacking 테스트 데이터 Shape: `(5, 4)`

### Hyperparameter

- [Optuna](https://github.com/darkhairlove/Kaggle_ICR/blob/main/Hyperparameter/Optuna_Automl.ipynb) 🔗
- [Flaml](https://github.com/darkhairlove/Kaggle_ICR/blob/main/Hyperparameter/Flaml_Automl.ipynb) 🔗

# 결과

- Leaderboard Score : 0.46
- Cross Validation Score : 0.338992

# 한계점

- Ensemble 기법을 다양하게 사용하지 못한 것이 한계점이고, 이 대회에서 Automl 중 하나인 autogluon을 사용하지 못하게 되어서 아쉬웠습니다.
- `greeks` 데이터를 다양하게 활용하지 못한 것이 한계점이고 다른 참가자의 코드를 참고했을 때, feature를 선별하는 이유에 대한 근거를 찾기 어려웠습니다.

# 느낀점

- Kaggle로 대회를 처음이었지만, 함께 Jupyter notebook을 공유해서 공동 작업하기 수월했습니다.
- 제한이 많이 되는 대회인 만큼 주어진 데이터와 모델을 구성하는 것이 어려웠습니다.
- 평가 지표인 balanced logarithmic loss가 어떤 것인지 알아 갈 수 있던 계기가 되었습니다.