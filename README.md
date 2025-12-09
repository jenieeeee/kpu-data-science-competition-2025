# 제5회 KAIST-POSTECH-UNIST 데이터사이언스 경진대회  
### **과제: 타이어 불량 예측 및 시험 생산 의사결정 모델링**

---

## 프로젝트 개요

- **대회 목적:**  
  시뮬레이션·설계·공정 파라미터 기반으로 타이어의 불량(NG) 여부를 예측하고,  
  이를 기반으로 시험 생산을 진행할지 여부(True/False)를 결정하여 **Net Profit을 최대화**하는 모델 개발.

- **데이터 구성:**  
  - 설계 스펙 (Mass_Pilot, Width, Aspect, Inch 등)  
  - 공정 변수 (Proc_Param1 ~ ParamN)  
  - FEM 시뮬레이션 결과 (x1 ~ x255, y1 ~ y255, p1 ~ p255)  
  - Class(Good/NG)

- **과제 구성:**  
  1) **불량 예측**  
  2) **의사결정 최적화**  

---
## 참여 목적

- 실제 제조 도메인(타이어 생산)의 **시뮬레이션·설계·공정 파라미터 기반 불량 예측** 문제 경험
- 비용 구조가 포함된 **의사결정 최적화 문제** 를 접해보고 최적 전략 탐색
- Kaggle식 대회가 아닌 **비용 기반 평가 지표** 를 활용해 실제 현업 문제 접근 방식 체험

---

## 문제 요약

### **과제 1 — 불량 예측**

- 학습 데이터: `train.csv` (720행 × 799컬럼)
- 주요 피처
    - 설계 스펙 (Mass_Pilot, Width, Aspect, Inch 등)
    - 공정 변수(Proc_Param1 ~ ParamN)
    - FEM 시뮬레이션 결과 (x1~x255, y1~y255, p1~p255)
- 타겟: `Class` (Good / NG)

---

### **과제 2 — 시험 생산 의사결정**

`submission.csv`에서

- `probability`: Good일 확률
- `decision`: True/False

**구간별 True 개수 제한: 최대 200개 (L/N/P 동일 그룹 기준)**

---

### **평가 방식 (Net Profit 기반)**

- Good + True → **+1 보상**
- NG + True → **2000 패널티**
- Profit은 최소 0 이상만 반영

➡️ 즉, **Threshold 조정 + 구간별 Top-K 선택이 성능을 결정하는 핵심 전략**

---

## 접근 방식 요약

### **EDA + Feature Engineering**

- 타겟과 상관 높은 공정 파라미터 및 시뮬레이션 변수 분석
- 이상치 제거·스케일 정규화보다
    
    **타겟 인코딩 + 파생 변수(feature engineering)** 가 더 효과적
    
- FEM 변수(x, y, p 시리즈)에서 요약 통계 feature 생성

### **여러 모델 비교**

- RandomForest / XGBoost / CatBoost / LightGBM 비교
- **CatBoost + 피처엔지니어링 조합이 가장 성능 우수**

### **불균형 문제 대응**

- Class 비율 Good(613) / NG(107) → 약 1:6
- F1보다 **Expected Profit 기준 threshold 최적화**가 중요

### **최종 전략**

- 모델 예측 확률 기반
- L / N / P 구간별 **Top-K 방식으로 True 선택**
- L 구간과 P 구간은 동일한 확률 기반 선택 방식을 사용하며,
  두 구간 모두 같은 기준으로 True가 결정되도록 구성
- Profit 최대화 기준으로 threshold 최적화

---

## Repository Structure

```plaintext
kpu-data-science-competition-2025/
│
├── README.md
│
├── src/
│   └── modeling/
│       └── best_model.ipynb       # 리더보드 최고점 모델
│
├── submission/
│   └── final.csv                 # 최종 제출 파일
│
└── figures/
    └── pipeline.png              # 모델링 파이프라인 다이어그램

---

## 시도한 방법과 성능 비교

### **1) 시도한 방법**

- EDA → Feature Engineering → Multi-model comparison → Threshold tuning
- 타겟과 상관 높은 시뮬레이션 값 및 공정 변수 중심으로 새로운 feature 생성
- 기본 모델: RandomForest, XGBoost, CatBoost 모두 테스트
- 최종적으로 **CatBoost + Top-K 전략**이 가장 높은 리더보드 성능
- L 구간의 True sample이 거의 없어
    
    → **P 구간에서 선택된 True 값을 L에 그대로 복사**
    
- Global threshold가 아니라
    
    → **구간별 K값 최적화 방식으로 profit 증가**
    

---

### **2) 성능 비교**

- 피처 엔지니어링을 통해 시뮬레이션 변수를 요약(평균/표준편차/최소/최대 등)하여 사용했을 때
    
    → **리더보드 최고 점수 달성**
    
- CatBoost 단독보다
    
    → **"EDA 기반 피처 생성 + 타겟 인코딩 + 구간별 Top-K 전략"** 조합이 압도적 성능 향상
    

---

## 모델링 흐름 요약 (Pipeline)

1. **Data Load**
2. **EDA**: 분포 분석, 상관 분석, 결측 처리
3. **Feature Engineering**
    - 시뮬레이션 요약 통계(feature summary)
    - 범주형 파라미터 Target Encoding
4. **Train/Test Split**
5. **Model Training (CatBoost)**
6. **Predict Probability**
7. **Section-wise Top-K Selection (L, N, P)**
8. **submission.csv 생성**

---

## 개선 여지

- 변수 수가 매우 많아(799개) **AutoML / Feature Selection 적용 가능성**
- 시뮬레이션 데이터 구조 자체를
    
    CNN/Transformer로 직접 학습시키면 성능 향상 여지 있음
    
- 비용 기반 metric을 직접 loss로 사용하는 **custom loss 적용 가능**
- 구간별 최적 K값을 모델이 스스로 학습하도록 확장 가능

---

## 회고

- 이 문제는 단순 classification 문제가 아니라
    
    **비용 기반 의사결정 문제라는 점이 핵심**
    
- 모델 성능이 높아도 threshold 조정이 잘못되면 profit이 급락
- 결국 단일 모델 개선보다 **후처리 전략(Top-K 선택)** 이 성능을 결정
- 실제 제조업 도메인에서 ML이 어떻게 활용되는지 이해할 수 있었던 실전형 문제
