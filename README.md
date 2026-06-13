# LG전자 VS본부 생산 데이터 분석 - 제품 이상 여부 판별 AI 모델

## 과제 개요
LG전자 VS본부 Sub Assembly Line의 생산 공정 데이터를 활용하여 제품의 불량 발생 여부를 예측하는 AI 이진 분류 모델 개발

## 배경
차량용 디스플레이(Cluster + CID) 생산 공정에서 발생하는 다양한 불량(기포, Crack, Misalignment 등)을 사전에 예측하고, 데이터 기반 품질 관리 체계 구축을 목표로 함

## 생산 프로세스
Sub Assembly Line의 4개 주요 공정 데이터를 활용

| 공정 | 설명 |
|------|------|
| **Dam** | Resin 외측 도포 및 반경화 |
| **Fill1 / Fill2** | Resin 내측 도포 (1차, 2차) |
| **AutoClave** | 진공 챔버 탈포 (미세 기포 제거) |

## 데이터
- **입력(X)**: 공정별 측정 및 설비 파라미터 (좌표, 속도, 시간, 토출량, 온도 등)
- **출력(Y)**: `Normal` / `AbNormal` 이진 분류

## 주요 단계

### 1. 데이터 전처리

- 결측치 비율이 높은 열 제거
- 공정별로 중복 입력된 동일 열 통합 및 상수 열 제거
- 범주형 변수 OrdinalEncoder로 수치 인코딩

### 2. 규칙 기반 이상 탐지

모델 추론 전, 아래 조건을 만족하는 데이터는 **규칙 기반으로 AbNormal 판정**

| 조건 | 내용 |
|------|------|
| 레시피 불일치 | 공정 간 레시피 번호가 다른 경우 |
| 생산 수량 불일치 | 공정 간 생산 수량이 다른 경우 |
| 장비 혼용 | Equipment #1, #2가 혼재된 경우 |

### 3. 모델 학습

**클래스 불균형 처리**
- 언더샘플링으로 Normal : AbNormal = 4 : 1 비율로 조정

**장비별 분리 학습**
- 규칙 기반 필터링 후 Equipment #1 / #2 데이터를 분리하여 각각 독립적으로 모델 학습

**하이퍼파라미터 튜닝**
- GridSearchCV (5-fold CV, f1_weighted 기준) 로 각 모델 최적 파라미터 탐색

**앙상블**
- RandomForest, XGBoost, SVM, LightGBM 4개 모델을 Soft Voting으로 앙상블
- 장비별로 각각 Voting 앙상블 모델 구성 (model1_voting, model2_voting)

### 4. 추론 파이프라인

```
입력 데이터
    │
    ├─ 규칙 조건 해당? ──► AbNormal
    │
    ├─ Equipment #1? ──► model1_voting으로 예측
    │
    └─ Equipment #2? ──► model2_voting으로 예측
```

## 성능 지표
- **Accuracy**
- **F1-score** (`AbNormal` 기준)

## 사용 라이브러리

```
pandas, numpy, scikit-learn, xgboost, lightgbm, matplotlib, seaborn
```
