# 가뭄 예측 시계열 모델링 — Drought Prediction

> 광동댐 수문, 기상 시계열 데이터를 기반으로 전통 시계열, 머신러닝, 딥러닝 모델을 비교하여 가뭄 지수를 예측하고, 데이터 분포와 클래스 불균형이 모델 성능에 미치는 영향을 정량적으로 분석한 프로젝트

---

## 프로젝트 개요

가뭄 지수(SPI6, MSWSI)를 예측, 분류하기 위해 전통 시계열 모델부터 딥러닝까지 단계적으로 실험했습니다. 단순히 성능이 높은 모델을 찾는 것을 넘어, 데이터 분포와 타깃 정의가 모델 성능을 어떻게 지배하는지를 정량적으로 분석하는 데 초점을 맞췄습니다.

## 데이터

- 수집 기간 - 2013-01-05 ~ 2024-12-28 (토요일 기준 주 단위 집계)
- 광동댐 수문 데이터 - 평균 저수율, 평균 유입량, 평균 방수량, 평균 용수공급량 (출처: wamis.go.kr)
- 태백시 기온 데이터 - 평균 강우량, 평균 기온 (출처: wamis.go.kr)
- 타깃 변수 - SPI6 (단기 가뭄 지수), MSWSI (종합 수자원 가뭄 지수) (출처: drought.go.kr)
- 파생변수 - 평균 유입량 - 평균 방수량 (다중공선성 완화 목적)
- 전처리 - RobustScaler 적용, VIF 점검으로 다중공선성 제거, ADF 검정으로 정상성 확인

## 담당 역할

전처리, EDA, 시각화는 팀 전체가 함께 진행했으며, 모델링은 아래 항목을 담당했습니다.

- SARIMA, SARIMAX - ACF/PACF 분석으로 차수(p, d, q, P, D, Q)를 직접 설계하고 외생 변수(수문, 기상) 결합 모델링 수행. 단기 예측 안정성 확인 및 장기 예측의 평탄화(Flattening) 한계를 통해 비선형 모델 도입의 기술적 근거 마련
- XGBoost - 슬라이딩 윈도우 기반 지도학습 데이터 변환 및 하이퍼파라미터 튜닝. 클래스 불균형 환경에서 Precision 확보의 한계를 정량적으로 분석
- RNN - 다중 분류(9클래스)에서 이진 분류(가뭄/비가뭄)로의 타깃 재정의 실험. 모델 구조 개선보다 데이터 분포와 타깃 설계가 성능에 더 지배적임을 수치로 증명

## 실험 구성

전통 시계열 모델(SARIMA, SARIMAX, VAR, VARMAX)로 베이스라인을 잡고 외생 변수의 영향력을 검증한 뒤, ML 모델(XGBoost, Random Forest, LightGBM, KNN, CNN)로 비선형 패턴을 학습했습니다. 마지막으로 RNN, LSTM으로 시간적 순차성을 직접 학습하며 타깃 정의(다중 분류 vs 이진 분류)와 가뭄 지수(SPI6 vs MSWSI)에 따른 성능 차이를 비교했고, 최종적으로 이진 분류 LSTM 모델로 2025년 가뭄을 예측했습니다.

- 02_classical_ts_models.ipynb - 전통 시계열 (SARIMA, SARIMAX, VAR, VARMAX)
- 03_ml_lightgbm.ipynb - LightGBM
- 03_ml_random_forest.ipynb - Random Forest
- 03_ml_knn_cnn.ipynb - KNN, CNN
- 03_ml_xgboost_classification.ipynb - XGBoost 분류
- 03_ml_xgboost_regression.ipynb - XGBoost 회귀
- 04_dl_lstm_mswsi.ipynb - LSTM MSWSI 예측
- 04_dl_lstm_mswsi_2025.ipynb - LSTM MSWSI 2025년 예측 (최종)
- 04_dl_lstm_spi6.ipynb - LSTM SPI6 예측
- 04_dl_lstm_spi6_2025.ipynb - LSTM SPI6 2025년 예측 (최종)
- 04_dl_rnn_2class.ipynb - RNN 이진 분류 (가뭄/비가뭄)
- 04_dl_rnn_3class.ipynb - RNN 3단계 분류
- 04_dl_rnn_9class.ipynb - RNN 9클래스 분류
- 04_dl_rnn_lag_features.ipynb - RNN 종속 이전 데이터 활용

## 최종 결과 (2025년 예측)

이진 분류(가뭄/비가뭄) LSTM 모델을 최종 채택하여 2025년 가뭄 지수를 예측했습니다.

- MSWSI - Accuracy 0.72 / 변동성이 큰 지수임에도 주요 가뭄 패턴 예측
- SPI6 - Accuracy 0.76 / 클래스 불균형으로 일부 상태 예측에 한계 존재

## 주요 인사이트

- 데이터 분포와 타깃 정의가 성능을 지배한다 - 모델 구조 개선보다 9클래스를 이진 분류로 재정의하는 것이 예측 안정성에 더 결정적인 영향을 미침
- 전통 시계열 모델은 설명력, 딥러닝은 반응성 - SARIMA, SARIMAX는 단기 예측과 계절성 파악에 유용하지만 장기 예측 시 평탄화 현상 발생, 비선형 모델 도입의 근거가 됨
- SPI6 vs MSWSI 예측 난이도 차이 - MSWSI는 주요 패턴 예측이 비교적 수월했으나, SPI6는 클래스 불균형이 심각해 데이터 증강이나 클래스 가중치 조정이 필수적임을 확인
- Feature Engineering > 모델 교체 - 종속 이전 데이터(lag features) 추가가 단순 모델 변경보다 더 큰 성능 향상을 가져옴

## 모델별 성능표

📊 전체 실험 결과 (전통 시계열, ML, DL 모델별 하이퍼파라미터 및 성능 수치): [성능표 링크](여기에-공유링크)

## 기술 스택

- Language - Python
- ML - Scikit-learn (RF, KNN, XGBoost, LightGBM)
- DL - TensorFlow / Keras (LSTM, RNN, CNN)
- 시계열 - statsmodels (SARIMA, SARIMAX, VAR, VARMAX)
- 데이터 처리 - Pandas, NumPy
- 시각화 - Matplotlib, Seaborn

## 실행 방법

```bash
pip install -r requirements.txt
# notebooks/ 폴더의 노트북을 번호 순서대로 실행
# 데이터는 data/광동댐_최종_VIF.csv 사용
```
