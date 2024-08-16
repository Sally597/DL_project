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

프로젝트 진행 순서
---
![Blank diagram2](https://github.com/user-attachments/assets/28f462b8-b19d-43b9-8eb0-2adf7e18ae26)

데이터 수집 & 소개
---

데이터 전처리
---

Mediapipe
---
문제1. 사람을 인식하는 속도가 지나치게 느림&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   문제2. 사람이 아닌 키오스크가 인식되는 문제




문제1. 사람을 인식하는 속도가 지나치게 느림
![image](https://github.com/user-attachments/assets/68ff6e6c-a25f-41b5-a823-77f5e7b9167f)
- 영상이 시작되고 10초가 지났는데도 인식이 안됨
  
문제2. 사람이 아닌 키오스크가 인식되는 문제
![image](https://github.com/user-attachments/assets/9fcdbce1-6889-42a4-8d8e-9069471f0e95)

YOLOv8 & Mediapipe
---
해결:  YOLOv8로 사람을 탐지한 후, MediaPipe로 Landmark로 지정
![image](https://github.com/user-attachments/assets/77779b92-5a5a-4352-8814-ff417703d1d0)
- YOLOv8 + MediaPipe 함께 사용 시 영상 시작과 동시에 인식

![image](https://github.com/user-attachments/assets/cce2ec79-959b-4fa5-b633-82bdcf554b7c)
- 키오스크가 아닌 사람을 인식

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
- Mean Validation Accuracy: 85.52%, Test Accuracy: 91.43%
![image](https://github.com/user-attachments/assets/1801e892-b9f8-43a0-b26c-4c312160d079) ![image](https://github.com/user-attachments/assets/3b8783a5-9ad7-4603-9647-1d512ec98625) ![image](https://github.com/user-attachments/assets/dea921a9-2b0f-483c-9943-13fba02bda3d)|

모델 평가
---

비즈니스 효과
---

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
