# kpu-data-science-competition-2025
kaist-postech-unist-데이터사이언스 경진대회 2025

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
  - FEM 시뮬레이션 결과 (x1~x255, y1~y255, p1~p255)  
  - Class(Good/NG)

- **과제 구성:**  
  1) **불량 예측(Classification)**  
  2) **의사결정 최적화(Top-K 기반 Decision Making)**  

---

## Repository Structure

```plaintext
kpu-data-science-competition-2025/
│
├── README.md
│
├── src/
│   ├── preprocessing/
│   ├── modeling/
│   │   └── best_model.ipynb       # 리더보드 최고점 모델
│   └── utils/
│
├── submission/
│   └── final.csv                 # 최종 제출 파일
│
└── figures/
    └── pipeline.png              # 모델링 파이프라인 다이어그램
