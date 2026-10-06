| 항목 | 요구불 | 운전자금 | 거치식 | 적립식 | 계정 간 | 근거 |
|---|---|---|---|---|---|---|
| 데이터 파일 | df_ready.csv (대구·경북, 2023-01~2025-12) | df_ready.csv (대구·경북, 2023-01~2025-12) | df_ready.csv (대구·경북, 2023-01~2025-12) | df_ready.csv (대구·경북, 2023-01~2025-12) | 같음 | run_harmonized.py DATA, 로그 `df_ready 대구·경북` 줄 |
| 표본 규칙 | `요구불예금잔액` > 0인 달 ≥ 1회 | `여신_운전자금대출잔액` > 0인 달 ≥ 1회 | `거치식예금잔액` > 0인 달 ≥ 1회 | `적립식예금잔액` > 0인 달 ≥ 1회 | 다름 — SPEC(계정별 보유 이력) | run_account: groupby max > 0 |
| 표본 법인 수 (전체 / 노출) | 10,723 / 1,026 | 9,541 / 981 | 843 / 175 | 666 / 142 | 다름 — SPEC(계정별 표본) | 로그 `=====` 줄 |
| 법인×월 (0 이하 잔액 행) | 266,360 (19,892) | 238,314 (16,805) | 23,584 (8,865) | 20,037 (8,101) | 다름 — SPEC | 로그 |
| h=6 추정 법인 수 | 9,336 | 8,310 | 768 | 637 | 다름 — SPEC(t+h·t−1 관측 조건) | results_all.csv |
| FE 구성 | 법인 FE + 지역×연월 FE (hlib.fit → partial_out(firm, regym)) | 법인 FE + 지역×연월 FE (hlib.fit → partial_out(firm, regym)) | 법인 FE + 지역×연월 FE (hlib.fit → partial_out(firm, regym)) | 법인 FE + 지역×연월 FE (hlib.fit → partial_out(firm, regym)) | 같음 | hlib.fit |
| 통제항 | 노출 × 영업일수 전년동월차 (SB) | 노출 × 영업일수 전년동월차 (SB) | 노출 × 영업일수 전년동월차 (SB) | 노출 × 영업일수 전년동월차 (SB) | 같음 | hlib.run_common SB |
| 충격 변수 | xkey='보정' → 보정YoY ÷ 100 | xkey='보정' → 보정YoY ÷ 100 | xkey='보정' → 보정YoY ÷ 100 | xkey='보정' → 보정YoY ÷ 100 | 같음 | C1 호출 인자(기본값) |
| 노출 변수 | expo_col='외환노출' (df_ready `exposed`) | expo_col='외환노출' (df_ready `exposed`) | expo_col='외환노출' (df_ready `exposed`) | expo_col='외환노출' (df_ready `exposed`) | 같음 | Panel 기본값, C1은 Panel(s, col) |
| 종속변수 식 | ln(잔액_{t+h}+1) − ln(잔액_{t−1}+1), 정확히 그 달 관측 (hold=False, positive=False) | ln(잔액_{t+h}+1) − ln(잔액_{t−1}+1), 정확히 그 달 관측 (hold=False, positive=False) | ln(잔액_{t+h}+1) − ln(잔액_{t−1}+1), 정확히 그 달 관측 (hold=False, positive=False) | ln(잔액_{t+h}+1) − ln(잔액_{t−1}+1), 정확히 그 달 관측 (hold=False, positive=False) | 같음 | C1 호출 인자(기본값) |
| 가중치 | weights=None (1) | weights=None (1) | weights=None (1) | weights=None (1) | 같음 | C1 호출 인자 |
| 윈저 | winsor=True → h마다 1%/99%, 가중치 없는 분위 | winsor=True → h마다 1%/99%, 가중치 없는 분위 | winsor=True → h마다 1%/99%, 가중치 없는 분위 | winsor=True → h마다 1%/99%, 가중치 없는 분위 | 같음 | C1 호출 인자(기본값) |
| 시차 범위 | -6~12, 18개, −1 제외=True | -6~12, 18개, −1 제외=True | -6~12, 18개, −1 제외=True | -6~12, 18개, −1 제외=True | 같음 | HS = [h for h in range(-6, 13) if h != -1], results_all.csv |
| 실패한 시차 | 0개 (로그 C1 줄 18개) | 0개 (로그 C1 줄 18개) | 0개 (로그 C1 줄 18개) | 0개 (로그 C1 줄 18개) | 같음 | 로그 |
| SE 방식 | 법인·연월 이중 군집 (CGM, 각 차원 G/(G−1)·(n−1)/(n−k)) | 법인·연월 이중 군집 (CGM, 각 차원 G/(G−1)·(n−1)/(n−k)) | 법인·연월 이중 군집 (CGM, 각 차원 G/(G−1)·(n−1)/(n−k)) | 법인·연월 이중 군집 (CGM, 각 차원 G/(G−1)·(n−1)/(n−k)) | 같음 | hlib.fit |
| p 자유도 | t(G_min−1), G_min 23~35 (= 월 수); 재계산 최대차 1e-15 | t(G_min−1), G_min 23~35 (= 월 수); 재계산 최대차 2e-15 | t(G_min−1), G_min 23~35 (= 월 수); 재계산 최대차 3e-16 | t(G_min−1), G_min 23~35 (= 월 수); 재계산 최대차 6e-16 | 같음 (G_min 값은 시차별 월 수라 SPEC대로 다름) | β3·SE·법인·월로 재계산 |
| Holm 범위 | h=1~12 (12칸), 재계산 최대차 1e-15 | h=1~12 (12칸), 재계산 최대차 0e+00 | h=1~12 (12칸), 재계산 최대차 7e-16 | h=1~12 (12칸), 재계산 최대차 1e-15 | 같음 | p_holm 재계산 |
| pyfixest 교차 확인 | h=-6,-2,0,6,12 | h=-6,-2,0,6,12 | h=-6,-2,0,6,12 | h=-6,-2,0,6,12 | 같음 | PF_C1 = {-6, -2, 0, 6, 12} |
| 사전추세 방식 | LP h=−6~−2 결합 χ²(5) p 0.917 · 검산 2e-15 · 원본 방식 N 184,746 | LP h=−6~−2 결합 χ²(5) p 0.866 · 검산 2e-15 · 원본 방식 N 165,562 | LP h=−6~−2 결합 χ²(5) p 0.151 · 검산 2e-16 · 원본 방식 N 16,969 | LP h=−6~−2 결합 χ²(5) p 0.560 · 검산 9e-16 · 원본 방식 N 14,643 | 방식 같음 (N은 SPEC대로 다름) | pretrend.csv, 로그 |
| 결과 출처 | harmonized run_common | harmonized run_common | harmonized run_common | harmonized run_common | 같음 | results_all.csv `출처` |
