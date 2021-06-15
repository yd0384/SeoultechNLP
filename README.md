# Top 20 NewsGroup Classification 성능향상 시키기
결론부터 말하면 성능 향상에 실패했다. 어떤 방법들을 시도해 봤는지 소개하겠다.

## 목표
Top 20 NewsGroups 분류 문제에 대한 성능을 75% 이상으로 올리는 것

## 사용한 도구
tensorflow2 : 딥러닝을 위해 사용
sklearn : dataset과 일부 함수들
re, nltk : 텍스트 전처리
numpy

## 프로세스
setup
데이터셋 불러오기 : headers, footers, quotes를 사용하면 뉴스그룹을 더 쉽고 정확하게 분류할 수 있지만 이를 사용하는 것은 우리가 원하는 바가 아니기 때문에 제거해서 불러와야 함
데이터셋 분할 : validation dataset을 생성해서 어떤 모델을 선택할지에 활용
텍스트 전처리, 토큰화 : 텍스트에서 불필요한 부분을 제거하고, lemmatize, stopwords 제거를 거쳐서 토큰화, 이때 tokenizer는 오직 training dataset만 사용해서 fit
임베딩에 사용할 glove 불러오기
모델 학습
결과 도출

## 모델 아키텍쳐
![image](https://user-images.githubusercontent.com/25950908/122035613-9a169500-ce0d-11eb-8f73-96ca7173ddf1.png)

RNN 모델 중 Long-Term dependency 문제를 해결하고 학습 시간이 LSTM보다 빠른 GRU를 선택, 자연어 문제에서 일반적으로 성능이 좋은 Bidirectional layer 사용
Dropout과 BatchNormalization을 통해 일반화 성능을 향상시키려고 노력

![image](https://user-images.githubusercontent.com/25950908/122035968-ff6a8600-ce0d-11eb-90d7-f74673cab372.png)

CNN 모델, 1D Convolution layer를 사용

## 결과 해석
EarlyStopping, ModelCheckpoint callback을 사용해 이른 validation loss가 가장 낮은 모델을 저장했고, validation loss가 5번동안 낮아지지 않으면 학습을 종료하도록 했다.
validation 데이터셋에 대해서는 70%에 근접한 정확도를 보였는데 이를 마지막에 evaluate했을때 더 낮은 정확도를 가졌다.
embedding layer를 glove를 사용한 weight로 세팅하고 freeze하고 학습했다.
이후 학습이 종료되었을 때 더 낮은 learning late로 embedding layer까지 finetuning을 진행했는데 성능이 소폭 향상했다.

|finetuning|RNN|CNN|
|---|---|---|
|x|0.6260|0.6192|
|o|0.6366|0.6294|
두 분류기의 예측 값을 합쳐서 측정한 정확도는 0.6547이었다.
과제에서 balanced_accuracy_score를 사용했기 때문에 이를 측정해봤더니 0.6414였다.

![image](https://user-images.githubusercontent.com/25950908/122035362-5b80da80-ce0d-11eb-9d06-e3720518d9c1.png)

RNN 모델 학습 과정

![image](https://user-images.githubusercontent.com/25950908/122035464-75bab880-ce0d-11eb-8a1b-a72bb16fec83.png)

CNN 모델 학습 과정

## 왜 잘 안됐을까?

### 비슷한 주제?

![image](https://user-images.githubusercontent.com/25950908/122038291-67ba6700-ce10-11eb-81c6-6595edfe81ee.png)

이 데이터셋에는 같은 대분류를 갖는 주제들이 존재한다. confusion matrix를 보면 같은 대주제를 가진 것들이 오분류가 된 경우가 많다. 이게 어려움중 하나일 수도 있겠다.

### Sequence의 역할?

과제에서 TF-IDF를 사용했을 때 70%정도의 정확도를 가졌다.
혹시 이런 주제를 분류하는 작업에는 단어의 순서보다 단어의 빈도 수를 가지고 학습 하는 것이 더 좋았을까?

혹시 이 주제로 높은 성능을 올린 학생이 있다면 보고 내가 놓친 부분이 무엇인지 학습해야겠다.
