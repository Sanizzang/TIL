### CNN?(합성곱 신경망)

이미지 처리할 때 많이 사용하는 뉴럴 네트워크 중 하나

- 시각적 이미지 분석에 가장 일반적으로 사용되는 인공신경망의 한 종류, 입력 이미지로부터 특징을 추출하여 입력 이미지가 어떤
  이미지인지 클래스를 분류함
- 이미지 및 비디오 인식, 추천 시스템, 이미지 분류, 의료 이미지 분석 및 자연어 처리에 응용됨

INPUT -> CONVOLUTION + RELU -> POOLING(채널 수를 유지 하면서 입력의 변화에 민감하지 않게금) -> FLATTEN(행렬을 일렬로 나열) -> SOFTMAX

학습을 할때는 RELU, CLASSIFICATION할떄는 SOFTMAX 함수를 사용

- 일반 신경망의 경우, 이미지 전체를 하나의 데이터로 입력하기 때문에 이미지의 특성을 찾지 못하고
  이미지의 위치가 약간 변경되거나 왜곡된 경우 올바른 성능을 기대할 수 없음 -> 데이터의 형상이 무시됨

- 합성곱 신경망(CNN)은 이미지를 하나의 데이터가 아닌, 여러 개로 분할하여 처리함

- 이미지가 왜곡되더라도 이미지의 부분적 특성을 추출할 수 있어 올바른 성능을 낼 수 있음

### CNN의 구조

- 기존의 신경만 구조
  : 인접하는 계층의 모든 뉴런이 결합된 완전연결로, Affine 계층으로 구성됨

- CNN의 구조
  : 신경망 구조에서 합성곱 계층(Convolutional Layer)과 풀링 계층(Pooling Layer)이 추가됨

### 합성곱 연산?

- 합성곱 계층에서의 합성곱 연산은 이미지 처리에서 말하는 필터(마스크) 연산에 해당됨
  - 2차원 입력 데이터가 들어오면, 필터(커널)의 윈도우가 일정 간격으로 이동해가며 연산이 적용됨
  - 입력과 필터에서 대응하는 원소끼리 곱한 후 그 총합을 구하면 그 결과가 출력됨
  - 이 과정을 모든 장소에서 수행하면 합성곱 연산의 출력이 완성됨

### 합성곱 연산의 편향

- 필터 적용 후 데이터에 편향(bias)이 더해짐
  - 완전연결 신경망에는 가중치 매개변수와 편향이 존재함
  - 합성곱 신경망에서는 이와 유사하게, 필터(가중치), 편향이 학습을 시킬 매개변수임
  - 편향은 항상 하나(1x1)의 값으로 존재하며 필터를 적용한 후 모든 원소에 더해짐
  - 즉, 합성곱 신경망을 통해 학습이 거듭되며 필터의 원소 값과 편향이 매번 갱신되는 것임

### 패딩(Padding) 처리

- 입력 데이터와 출력 데이터의 크기를 맞추기 위해 사용됨
  - 합성곱 연산을 수행하기 전에 입력 데이터 주변을 특정 값(주로 0을 사용: Zero padding)을 채우는 단계
  - 예를 들어 (4, 4) 입력 데이터에 (3, 3) 필터를 적용하면 출력은 (2, 2)가 되어, 입력보다 2만큼 줄어듦
  - 합성곱 연산을 여러번 반복해야 하는 심층 신경망에서 출력 크기가 1이 되어버릴 수 있기 떄문에 더 이상
    합성곱 연산을 적용할 수 없게 되는 문제가 발생할 수 있음
  - 따라서, 출력 크기를 조정하기 위해 패딩을 사용함

### 스트라이드(Stride)

- 필터를 적용하는 위치의 간격을 말함
  - 스트라이드가 커지면 출력 크기가 작아짐
  - 예를 들어 스트라이드를 2로 하면 필터를 적용하는 윈도우가 두 칸씩 이동함

### 풀링(Pooling) 연산이란?

- 세로, 가로 방향의 공간을 줄이는 연산으로, Sub-sampling이라고도 불림
  - 예를 들어 2x2 영역을 원소 하나로 집약하여 공간 크기를 줄임
  - 주로, 풀링의 윈도우 크기와 스트라이드 값은 같게 설정함
  - 최대 풀링(max pooling)은 대상 영역에서 최댓값을 취하고, 평균 풀링(average pooling)은 평균을 취함
  - 이미지 인식 분야에서는 주로 최대 풀링을 사용함

### 완전 연결층(Fully-connected layer)의 역할

- 추출된 특징 값을 기존 neural network에 넣어서 이미지를 분류함
  - 추출된 특징들은 2차원 자료를 다루지만 기존 neural network에 전달하기 위해선 1차원 자료로 바꿔줘야 하므로 Flatten 작업을 수행함

### 활성화 함수

- 소프트맥스(Softmax) 함수
  - 시그모이드 함수의 일반화된 형태인 활성화 함수
  - 다클래스 분류를 할 시 클래스 수 L과 같은 수의 유닛을 구성