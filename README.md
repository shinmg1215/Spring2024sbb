![dsadadsad](https://github.com/user-attachments/assets/1eb39734-7933-413a-8f11-a6e5421c4566)

# MobileBERT를 활용한 pc 게임 평점 분석

<img src="https://img.shields.io/badge/pytorch-%23EE4C2C.svg?&style=for-the-badge&logo=pytorch&logoColor=white" /> <img src="https://img.shields.io/badge/pycharm-%23000000.svg?&style=for-the-badge&logo=pycharm&logoColor=white" />
<img src="https://img.shields.io/badge/python-%233776AB.svg?&style=for-the-badge&logo=python&logoColor=white" />

## 1.개요




### 1.1 문재 정의

밸브 코퍼레이션이 개발하고 운영 중인 세계 최대 규모의 전자 소프트웨어 유통망.
스팀 클라이언트를 통해 여러 가지 게임을 구입, 관리할 수 있으며, 채팅, 방송 및 다양한 커뮤니티 기능을 통해 다른 유저들과 소통할 수 있다. 약 5만 개가 넘는 게임들이 있다. 그래서 리뷰를 분석해보겠다.

![dsada](https://github.com/user-attachments/assets/ec9a151b-0eb2-4293-b9ea-f52f7c82d276)

### 1.2 데이터 및 모델 개요 

스팀 게임리뷰 데이터셋을 토대로 분석을 진행해보겠다. 그러나 이미 분석된 [깃허브](https://github.com/yeon0306/steam_game/blob/main/README.md) 에 보면 알수 있듯이 학습데이터의 품질이 매우 낮은편이어서, 이 데이터를 활용하기 위해서는 데이터 라벨링을 다시 할 필요가 있다.

## 2. 데이터
스팀 게임 리뷰 데이터셋은 깃허브 저장소에 수집되어 있는 [데이터](https://github.com/bab2min/corpus/tree/master/sentiment)가 있다. 이 데이터는 2020년 5월부터 6월까지 수집된 한국어 리뷰 10만 건으로 구성되어 있다. 이 데이터는 부정이 0 긍정이 1로 라벨링 되어 있는 데이터인데 앞서 언급한 데이터 분석 깃허브에서도 알 수 있듯이 라벨링의 품질이 저조하다고 판단된다. 따라서 이 프로젝트에서는 10만 건의 스팀 게임 리뷰 데이터셋에서 2천 건을 다시 라벨링하는 과정으로 데이터의 품질을 높이고자 한다.

### 2.1 탐색적 데이터 분석

|number| review |label|
|-|----------|---|
|0|노래가 너무 적음|0|
|1|돌겠네 진짜. 황숙아, 어크 공장 그만 돌려라. 죽는다.|0|
|2|막노동 체험판 막노동 하는사람인데 장비를 내가 사야돼 뭐지|1|
|3|차악!차악!!차악!!! 정말 이래서 왕국을 되찾을 수 있는거야??|1|
|..|...|...|...|
|1998|짜고치는 고스톱의 좆병신 섭 인간들 모임|0|
|1999|현실 돈 아니라고 맨날 올인 박는 애들 대가리를 망치로 쳐야됌.|1|
|2000|친구들이랑 하면 정말 재미 있어요 알피지 요소가 있으며 던전을 클리어 해나가는 게입 입니다. 곳곳에 함정과 숨은길이 있으며 하나의 던전 마지막에 보스가 존재하여 그 몬스터를 잡아야 맵이 클리어 됩니다.|1|

위 데이터는 라벨링을 다시 진행할 2,000건의 리뷰에 대한 내용이다. 2,000건의 리뷰 중 데이터가 없는 항목을 제외한 데이터는 1,973건이며, 언급한대로 부정은 0, 긍정은 1로 구성되어 있으며 긍부정 건수는 아래와 같다.  

|구분|건수|
|-|----|
|부정|1,002|
|긍정|971|
|계|1,973|

### 2.2 데이터 라벨링을 통한 학습 데이터 구축
라벨링 대상 데이터 2,000건에 대하여 아래와 같은 기준으로 다시 라벨링을 수행했다.
|라벨|내용|
|-|----|
|0|부정|
|1|긍정|
|-1|부정이나 긍정을 판단할 수 없는 애매모호 하거나 중립적인 문장|

라벨이 -1인 경우는 시각에 따라 약한 수준의 긍정이나 부정으로 나뉠 수 있겠으나, 학습 데이터의 명료성을 위해 라벨이 -1인 경우는 제외를 하고 학습 데이터를 구축했다. 직접 라벨링을 수행한 결과는 아래 표와 같다.

|라벨|건수|
|-|----|
|0|1,046|
|1|879|
|-1|47|
|계|1,972|

라벨링을 다시해본 결과 5% 정도의 오차가 있었으며, -1 라벨을 제외한 1,925건의 데이터를 학습데이터로 구축했다.

애매모호한 라벨의 예시는 아래와 같다.
|내용|기존 라벨|라벨|
|-|--|----|
|관람객 vs 호랑이 불가|1|-1|
|이거 어뜨게 한굳어함?|0|-1|

이미 부여되어 있던 라벨과 예측한 라벨이 다른 경우는 다음과 같다.
|내용|기존 라벨|라벨|
|-|--|----|
|배경화면 보려고 게임별로 안하게됨|1|0|
|꺼져라 고인물 쉑들|0|1|
|점점 내가 가해자이고 악마가 피해자 같다. 걍 떄려부셔야지 이히힣|1|0|
|.............열차+루트 같이 묶어서 파는게 좋을것 같네요 초보자들은 진짜 헤매거든요;;;;	|1|0|
|재미는 있는데 사람이 너무 없다|0|1|
||||


# 3. 재학습 결과

## 3.1 개발 환경
<img src="https://img.shields.io/badge/pycharm-000000?style=flat-square&logo=pycharm&logoColor=white"/> <img src="https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=Python&logoColor=white"/> <img src="https://img.shields.io/badge/torch-EE4C2C?style=flat-square&logo=pytorch&logoColor=white"/> <img src="https://img.shields.io/badge/pandas-150458?style=flat-square&logo=pandas&logoColor=white"/> <img src="https://img.shields.io/badge/numpy-013243?style=flat-square&logo=numpy&logoColor=white"/> <img src="https://img.shields.io/badge/transformers-81c147?style=flat-square&logo=transformers&logoColor=white"/>

## 3.2 학습 결과 그래프

![데이터](https://github.com/user-attachments/assets/819b14e7-9239-4b9a-a0e4-d550e96ff594)

<table>
  <tr align="center"><th></th><th></th><th>Epoch 1</th><th>Epoch 2</th><th>Epoch 3</th><th>Epoch 4</th></tr>
  <tr align="center"><th rowspan="2">학습데이터</th><td>평균 학습 오차</td><td>0.61</td><td>0.40</td><td>0.22</td><td>0.13</td></tr>
  <tr align="center"><td>검증 정확도</td><td>0.72</td><td>0.82</td><td>0.84</td><td>0.83</td></tr>
</table>

학습 데이터의 평균 학습 오차가 학습이 진행될 수록 감소되는 것을 볼 수 있다. Epoch 1에서 0.61이었지만 Epoch 4에서는 0.13로 감소하였으며 이는 모델이 학습 데이터에 대해 점점 더 효과적으로 학습되고 있다는 것을 알 수 있다. 
또한 검증 정확도도 에폭이 진행됨에 따라 검증 정확도가 0.72에서 0.83까지 상승하였다.

# 4. 결론 및 배운점
일단 [깃허브](https://github.com/yeon0306/steam_game/blob/main/README.md) 도움을 받은 분에게 감사하단말을 전한다 하면서 느낀점은 일단 라벨링이라는 작업 자체가 은근히 까다롭고 원본 데이터의 정학도가 (76%) 정도 된다 
원본 데이터를 다시 2000건 정도 라벨링을 다시 하나 하나 달았고 중간 중간 애매모호한 라벨들은 -1로 처리했다 2000건의 데이터의 긍, 부정 리뷰를 분석해본 결과 편균 오차가 0.61 에서 0.13 까지 줄어들었고 검증 정확도 는 0.72 에서 0.83 로 0.10 정도가 올랐다
0.9 이상이 나오지 않는 것은 게임 이름과 게임 용어가 쓰여진 리뷰 게임과 관련 없는 주제 등  때문일 가능성이라고 생각된다.
