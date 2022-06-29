# AMEX lightBGM quickstart
1. 이 노트북은 가독성보다 메모리 효울성 최적화를 추구하는 방향으로 쓰여졌다. 
그 부분을 중점적으로 볼 것.  
예를 들어 train 데이터와 test 데이터를 한꺼번에 읽는 대신 train 데이터를 읽어와 feature engineering을 한 후,  
test 데이터를 읽어와 feature engineering을 한다.  
또한 feature engineering 과정에 사용한, 머신러닝 학습에는 필요없는 데이터프레임은 바로 지운다.  
메모리를 많이 사용하는 groupby 대신 .iloc[mask_array, columns]를 사용한다. 
2. 어떤 feature engineering을 했는지, 특히 한 customer당 (다른 시간을 나타내는)여러 로우가 있는 데이터를 어떻게 한 customer 당 한 로우가 있는 데이터로 만들었는지 중점적으로 볼 것.  
한 customer를 나타내는 여러 시간의 data 중 max, min, mean, last 값을 뽑아 feature로 만들었다.  
(각 aggregation 마다 칼럼을 선별하여 적용하였는데 그 기준은 정확히 말하고 있지 않다.)  
