# 1. 데이터 다운로드
데이터 용량이 너무 커서 데이터 셋을 다운 받을 수 없었다. Discussion에서 이를 해결하는 방법들을 찾아 티스토리에 정리했다.  

용량이 큰 데이터를 다루는 방법  
1) 용량이 큰 데이터를 읽어오는 방법:  
https://spnz.tistory.com/144

2) 읽어온 데이터를 처리하는 방법:  
https://spnz.tistory.com/145
 
결정적으로 누군가가 데이터 중 모든 float type 칼럼에 artificial noise가 더해진 것을 발견했다.  
이를 제거하면 데이터 세트 사이즈를 줄일 수 있다.  
그 결과 188개의 float/categorical 피처를 95개의 np.int8/np.int16 피처와 93개의 np.float32 피처로 변환시킨 데이터 세트를 다운받았다.
https://www.kaggle.com/competitions/amex-default-prediction/discussion/328514

# 2. Initial EDA
일단 스스로 EDA를 수행, 전체적인 data 파악했다.  
그런데 칼럼이 190개로 너무 많아서 그 후 어떤 distribution을 봐야할지 잘 모르겠기에    
그래서 다음 노트북을 필사했다:  
https://www.kaggle.com/code/ambrosm/amex-eda-which-makes-sense

## 1) 스스로 수행한 EDA
* target 분포: 0이 1보다 훨씬 많으므로 이를 고려한 평가지표를 사용해야 한다.
* string columns: 'S_2'의 범위를 보면 2017년 3월 - 2018년 3월 총 13달 간의 기록이다.
customer_ID별 'S_2'의 분포를 살펴보면 customer의 85%는 13개 데이터가 있다. 2017년 3월 - 2018년 3월간 매월 하나씩 데이터가 있는 듯 하다.
나머지 15%는 일부 월에만 데이터가 있는 것으로 보인다. 
![image](https://user-images.githubusercontent.com/87534455/176506695-917b4e38-5765-4e86-9a44-669423a77f20.png)
* integer columns: 칼럼별로 unique한 값의 수를 보면 2 - 309가지 unique한 값을 가진다. 특히 많은  칼럼은 2, 3가지 값을 가진다. 
* 결측값 확인: 결측값이 많다.

## 2) Notebook: Amex EDA which makes sense
* target 분포: 0과 1의 분포를 확인하는 것 외에 결측값과 duplicate 값 존재 여부도 확인했다.  
앞에서 말했듯 0이 1보다 훨씬 많으므로 stratifiedKfold를 사용하는 것이 좋으며, 이런 편향된 분포에 맞는 평가지표를 사용해야 한다  
(이미 이 competition의 평가지표가 이를 고려한 평가지표다.).
* statement date 분포:
train 데이터와 test 데이터 statement date의 max와 min값을 확인했다.
두 기간이 겹치지 않으므로 economic cycle의 영향을 학습할 수 없다.  
그 다음 customer 별 마지막 statement date를 확인했다. train 데이터는 모두 2018년 3월에 마지막 statement를 받았고 test 데이터는 모두 2019년 4월이나 10월에 statement를 받았다.  
이전 discussion에서 사람들이 4월 data는 public leaderboard  평가에, 9월 data는 private leaderboard 평가에 사용된다는 것을 확인하였다.  

(모든 statement를 train, public lb, pribvate lb로 나눠서 날짜별 histogram을 그림)
![image](https://user-images.githubusercontent.com/87534455/176506911-4edeeea0-3967-4355-b628-faf3d8972e1f.png)

* NA값의 분포:'B_29'의 경우 2019년 6월부터는 모든 customer에 대한 데이터가 있지만 그 전에는 데이터가 1/10도 없다.  
즉, private leader board에만 많이 있는 피처이다. (피처를 아예 drop하는게 나을지 생각해야 한다.)
![image](https://user-images.githubusercontent.com/87534455/176506999-d0cfd150-e60e-4ec1-977d-49a1d28a8967.png)
* 카테고리형 데이터 분포: target=0과 target=1의 분포가 다르다. 즉, 모든 피처가 target에 대한 정보를 준다.  
카테고리는 최대 8가지. 원-핫 인코딩이 가능한 숫자다. 
* 연속형 데이터 분포: outlier가 있는 것을 알 수 있다. 진짜 outlier인지 rare event일 뿐인지는 알 수 없다.

## 요약  
일단 다음 4가지 포인트를 유의해서 데이터를 분석할 것:  
* target의 분포가 편향됨. strarified kfold 사용과 스케일링 고려. 
* time series 데이터를 어떻게 customer별 데이터로 만들지 다양한 방법을 고려. 
* 결측값을 어떻게 처리할지. 특히 time series에서 앞 time point에 결측값이 있는 데이터임을 인지하고 있을 것.  
(특히 'B_29' 피처는 거의 public lb에만 데이터가 있음. drop을 고려.)
* 카테고리형 피처 원-핫인코딩 고려.
 
