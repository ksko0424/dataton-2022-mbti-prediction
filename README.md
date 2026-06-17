# dataton-2022-mbti-prediction

2022 데이터톤 경진대회 (2022.02.23 ~ 2022.02.25) 대상 수상 프로젝트입니다.

- Task: 인터넷 사용자가 남긴 텍스트 기반 MBTI 유형 예측
- Final code: [`Final_code_team15.ipynb`](./Final_code_team15.ipynb)
- Award: 최종 정확도 순위 1위, 대상 수상

![Dataton 2022 grand prize](assets/header.png)

## 1. Problem

인터넷 사용자가 남긴 텍스트 데이터를 기반으로 별도의 MBTI 검사 없이 사용자의 MBTI 유형을 예측하는 문제입니다.

일반적인 접근은 MBTI를 `E-I`, `S-N`, `T-F`, `J-P`의 4개 지표로 나누어 각각 독립적인 binary classification 문제로 예측한 뒤 결과를 합치는 방식입니다. 하지만 데이터상에서 특정 지표 조합이 함께 나타나는 경향이 있다면, 이 방식은 유형 간 상관관계와 조합 정보를 충분히 활용하지 못할 수 있습니다.

따라서 본 프로젝트에서는 4개 지표를 따로 예측하는 방식과 16개 MBTI 유형을 한 번에 예측하는 방식을 비교하고, 최종적으로 16개 유형 동시 분류 접근을 사용했습니다.

## 2. Data Set

- Train Data: 74,357건
  - Columns: `MBTI`, 400개 단어
- Test Data: 9,337건
  - Columns: 400개 단어
- 데이터 셋 내 MBTI 분포는 균등하지 않습니다.

## 3. Data Preprocessing

처음 고려한 전처리는 토큰화, 표제어 추출, 벡터화입니다.

- Tokenization: 텍스트를 여러 token으로 나누는 과정
- Lemmatization: 단수/복수 등 단어 형태를 원형으로 변환하는 과정
- Vectorization: 모델이 텍스트를 처리할 수 있도록 수치 벡터로 변환하는 과정

주어진 데이터가 문장이 아니라 이미 단어 단위로 구성되어 있었기 때문에 별도 토큰화는 적용하지 않았습니다. 벡터화에는 빈도 기반 텍스트 표현 방식인 TF-IDF를 사용했습니다.

## 4. Modeling Strategy

### 4.1 Four binary classifiers

첫 번째 접근은 MBTI의 4개 지표를 각각 예측한 뒤 결과를 합치는 방식입니다.

- Validation accuracy: 약 70% 로지스틱 회귀 기준
- 한계: 4개 지표를 서로 독립적인 문제로 다루기 때문에, 유형 간 조합 정보가 손실될 수 있습니다.

### 4.2 Single 16-class classifier

두 번째 접근은 16개 MBTI 유형 전체를 하나의 multi-class classification 문제로 다루는 방식입니다.

비교 모델과 validation accuracy는 다음과 같습니다.

| Model | Accuracy |
|---|---:|
| Logistic Regression | 79.65% |
| Random Forest | 47.03% |
| Deep Learning | 69.19% |
| LinearSVC | 80.20% |
| LinearSVC + Bagging | 77.21% |

최종 모델은 단순히 validation accuracy만 기준으로 정한 것이 아니라, TF-IDF로 표현된 고차원 sparse text feature에 적합한 선형 결정경계를 만들 수 있고, 학습/추론 비용과 해석 가능성 측면에서도 대회 조건에 적합하다고 판단해 LinearSVC를 중심으로 구성했습니다.

## 5. Hyperparameter Tuning

LinearSVC의 주요 하이퍼파라미터인 `C` 값을 조정했습니다.

- Best validation setting: `C = 0.475`
- Final model: TF-IDF + LinearSVC

## 6. Result

- 최종 정확도 순위: 1위
- 최종 정확도: 93.01%
- 수상: 2022 데이터톤 경진대회 대상

## 7. Key Takeaways

- 16개 유형 동시 분류는 MBTI 지표 간 조합 정보를 보존한다는 문제 정의 측면에서 4개 독립 이진 분류보다 더 적합한 구조였습니다.
- TF-IDF와 LinearSVC 조합은 고차원 sparse text feature를 효율적으로 처리하면서도 재현성과 해석 가능성이 좋은 실용적인 baseline으로 동작했습니다.
- 클래스 불균형이 존재하므로 정확도뿐 아니라 class-wise metric, macro-F1, confusion matrix 등을 함께 확인하면 모델 해석을 더 강화할 수 있습니다.

## Related Repository

- Poster paper version: <https://github.com/ksko0424/mbti-text-classification>
