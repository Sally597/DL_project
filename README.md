# Deep Learning Project
Introduction
---
무인매장 고객 행동 탐지 모델 개발

프로젝트 개요
---
1. 팀원: 강민지, 김예림, 김이영
2. 수행기간: 2024.07.18 ~ 2024.08.08 (4주)
   
프로젝트 소개
---
<b>배경</b>
   - 무인매장의 성장: 2019년 대비 2023년에는 신규 가맹점 수가 3.8배 증가, 무인 매장의 카드 이용 금액은 4.81배 증가
   ![늘어나는 무인 매장](https://github.com/user-attachments/assets/776e0858-0198-4472-a655-db088a04469a)
   - 무인매장의 절도 증가: 2021년에는 월 평균 351건의 절도가 발생했으나, 2022년 1월부터 6월까지 절도는 월 평균 471건 발생 -> 무인매장 내 절도가 빈번해짐
   - 무인매장의 범죄 양상: 무인매장에서 가장 빈번하게 일어나느 범죄 유형 1위는 절도이며 2위는 손괴 (키오스크 파손)

<b>시사점</b>
   - 무인매장은 성장하는 사업인 만큼 관련 범죄에 대한 대응 방안이 마련되어 있지 않음
   - 무인매장의 범죄 양상을 파악하기 위해서는 절도 뿐만 아니라 파손을 고려해야함

프로젝트 진행 순서
---
![Blank diagram2](https://github.com/user-attachments/assets/72a0beef-195f-43e4-bcb6-0fdaa57b2d9b)



데이터 수집 & 소개
---
AI Hub의 실내 편의점/무인매장 고객 이상행동, 이상행동 데이터를 활용

<b>고객행동 유형</b>
   - 구매행동 데이터: '구매'를 사용
   - 이상행동 데이터: '파손', '절도'만 사용

<b>매장 종류</b>
   - 무인매장에 해당하는 SMA (소형무인매장 A), SMB (소형무인매장 B) 만 추출

<b>영상 추출</b>
   - 다른 각도에서 촬영한 동일한 영상이 존재 (영상별로 3-4개)
   - 중복된 영상이 포함되는 것을 방지하기 위해, 촬영 시간을 고려하여 각도별로 랜덤하게 영상을 추출
   - ![image](https://github.com/user-attachments/assets/c08dafc5-17a1-407a-8aa6-56d9dc13df2c)

<b>클립 추출</b>
   - 원본 영상의 길이는 총 1분이지만, 구매, 절도, 파손이 명확하게 드러나는 부분만 20-30초 가량 추출
   - 영상 길이가 모두 동일하지 않은 문제가 발생했으나, 딥러닝 시 padding을 통해 영상 길이를 모두 동일하게 맞춰줌

데이터 전처리
---

Mediapipe
---
![image](https://github.com/user-attachments/assets/90af49cb-dc06-4df8-ac46-4630f3052659)



YOLOv8 & Mediapipe
---
해결:  YOLOv8로 사람을 탐지한 후, MediaPipe로 Landmark로 지정
<br>
<br>
![image](https://github.com/user-attachments/assets/cb81ed9c-ce26-4c3d-bee7-3698a305d06e)



하이퍼파라미터 튜닝
---
1. Data Split
   - Train Test Split → 학습 전 Train, Test Set을 8:2로 split
   - K-fold Cross Validation → K-fold Cross Validation을 이용하여 (cv=5) Train Set을 다시 Train, Validation Set으로 나눔

2. 데이터 증강 (Train Set에만 적용)
   - 1. x, z 좌표 반전 : x 좌표는 좌우 대칭, z 좌표는 깊이 반전
     2. 노이즈 추가 : 평균이 0이고 표준편차가 0.1인 정규분포에서, 무작위 값을 추출하여 더함
     3. 무작위 시프트 : 각 좌표 값에 -0.01에서 0.01 사이의 무작위 값을 더함

3. 오버피팅 방지
   - 1. 양방형 LSTM : 시퀀스를 양쪽 방향으로 처리
     2. 배치 정규화 : 각 배치마다 평균과 표준편차를 이용하여 데이터를 정규화
     3. 학습률 스케일링 : 학습 과정에서 학습률을 동적으로 조절

4. 최적 파라미터 → learning_Rate:0.0000275, batch_size:32,  num_epochs: 250, hidden_size: 256, num_layers: 2, dropout: 0.2

최종 모델
---
- Mean Validation Accuracy: **85.52%**, Test Accuracy: **91.43%**
![image](https://github.com/user-attachments/assets/1801e892-b9f8-43a0-b26c-4c312160d079) ![image](https://github.com/user-attachments/assets/3b8783a5-9ad7-4603-9647-1d512ec98625) ![image](https://github.com/user-attachments/assets/dea921a9-2b0f-483c-9943-13fba02bda3d)|

모델 평가
---

•	최종 모델을 Test Set의 Landmark CSV 파일에 적용하여 최종 예측 라벨 도출

•	해당 라벨과 예측 확률을 영상에 고정하여 시각화

![image](https://github.com/user-attachments/assets/0afd8000-597b-49b4-936d-9726a37a6074)

비즈니스 효과
---

실시간 고객 행동 탐지 및 분류를 통해 절도 및 파손 발생 시 신속하게 대응하여 매장 내 손실을 줄이고, 관리자가 매장 상태를 실시간으로 파악할 수 있어 운영 효율성을 높일 수 있음

이슈 및 트러블슈팅
---
01. Mediapipe 객체탐지
   - 설명: Mediapipe만을 사용했을 때 객체탐지가 완벽하게 되지 않음. 키오스크를 사람으로 인식하는 경우도 있음.
   - 해결: YOLOv8과 Mediapipe를 함께 사용해 문제 해결
02. FPS
   - 설명: "구매"데이터와 "절도"&"파손"데이터의 FPS가 맞지 않아 데이터 불균형 문제가 발생.
   - 해결: "구매"데이터를 "절도"&"파손"데이터와 같은 3fps로 맞춤.  
03. 영상 길이
   - 설명: 영상들을 각 행동 구간별로 추출해 길이가 맞지 않음. RNN모델은 고정된 입력 크기를 필요로 하기 때문에 오류 발생
   - 해결: max padding을 사용해 데이터의 길이를 100프레임으로 맞춤. 
04. 오버피팅
   - 설명: 오버피팅으로 인해 validation loss가 발산.
   - 해결: 양방형 LSTM, 배치 정규화, 학습률 스케일링을 사용해 오버피팅 문제를 방지.
