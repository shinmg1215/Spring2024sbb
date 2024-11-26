![dsadadsad](https://github.com/user-attachments/assets/1eb39734-7933-413a-8f11-a6e5421c4566)

# MobileBERT를 활용한 pc 게임 평점 분석

<img src="https://img.shields.io/badge/pytorch-%23EE4C2C.svg?&style=for-the-badge&logo=pytorch&logoColor=white" /> <img src="https://img.shields.io/badge/pycharm-%23000000.svg?&style=for-the-badge&logo=pycharm&logoColor=white" />
<img src="https://img.shields.io/badge/python-%233776AB.svg?&style=for-the-badge&logo=python&logoColor=white" />

## 1.개요

![62e7ef17 (1)](https://github.com/user-attachments/assets/efed0641-5065-43e9-90cd-3ee948610948)

### 1.1 문재 정의

밸브 코퍼레이션이 개발하고 운영 중인 세계 최대 규모의 전자 소프트웨어 유통망.
스팀 클라이언트를 통해 여러 가지 게임을 구입, 관리할 수 있으며, 채팅, 방송 및 다양한 커뮤니티 기능을 통해 다른 유저들과 소통할 수 있다. 약 5만 개가 넘는 게임들이 있다. 그래서 리뷰를 분석해보겠다.

### 1.2 데이터 및 모델 개요 

스팀 게임리뷰 데이터셋을 토대로 분석을 진행해보겠다. 그러나 이미 분석된 https://github.com/yeon0306/steam_game/blob/main/README.md 에 보면 알수 있듯이 학습데이터의 품질이 매우 낮은편이어서, 이 데이터를 활용하기 위해서는 데이터 라벨링을 다시 할 필요가 있다.

## 2. 데이터

## 2.1 탐색적 데이터 분석 

| | review |label|
|-|----------|---|
|0|노래가 너무 적음|0|
|1|돌겠네 진짜. 황숙아, 어크 공장 그만 돌려라. 죽는다.|0|
|2|막노동 체험판 막노동 하는사람인데 장비를 내가 사야돼 뭐지|1|
|3|차악!차악!!차악!!! 정말 이래서 왕국을 되찾을 수 있는거야??|1|
|..|...|...|...|
|1998|짜고치는 고스톱의 좆병신 섭 인간들 모임|0|
|1999|현실 돈 아니라고 맨날 올인 박는 애들 대가리를 망치로 쳐야됌.|1|
|2000|친구들이랑 하면 정말 재미 있어요 알피지 요소가 있으며 던전을 클리어 해나가는 게입 입니다. 곳곳에 함정과 숨은길이 있으며 하나의 던전 마지막에 보스가 존재하여 그 몬스터를 잡아야 맵이 클리어 됩니다.|1|

2000개의 리뷰가 있으며 의문은 -1, 부정은 0, 긍정은 1로 구성되어있다.

|-|건수|
|-|----|
|긍정|879|
|부정|1540|
|계|1925|

부정적인 데이터가 조금 더 많다.

### 2.2 학습 데이터 구축
라벨링을 다시하기 위해서 2,000건 정도를 다시 라벨링을 했다. 0, 1, -1로 구분해본 결과 -1 인 애매모호한 것이 75건, 0, 부정 은 1540건, 1 긍정 은 879건이 있었다. 전반적으로 (76% 정도의 정확도를 보였다)

애매모호한 -1 라벨링의 예시는(관람객 vs 호랑이 불가	-1, 이거 어뜨게 한굳어함?	-1) 같은 게임 리뷰가 아닌 논 외의 리뷰를 -1 로 정했다.
기존 라벨과 새로 단 라벨이 다른 예시(1	이게임은 정말 좋은 게임 ㅋㅋㅋ LOVE GAME	0,  1	배경화면 보려고 게임별로 안하게됨	0)
전반적으로 데이터 자체가 오류가 많은 것을 확인했다.



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


