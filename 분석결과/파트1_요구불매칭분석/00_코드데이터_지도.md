# 돈독 · 민영 파트 — 코드·데이터 지도와 재현 순서

- 작성: 민영 (돈독) / 2026-09-30
- 최종 정리 PDF: `돈독_민영파트_최종정리.pdf` (12쪽, 집계만) — 생성 코드 `build_final_pdf.py`
- **정리 원칙:** 파일은 옮기거나 이름을 바꾸지 않았다. 스크립트끼리 절대 경로로 참조하고, `outputs\` 아래 폴더는 각각 SPEC을 먼저 커밋한 git 저장소라 옮기면 재현 경로와 커밋 기록이 깨진다. 이 문서가 전체 지도다.
- **데이터 규칙:** `C:\test\data\`는 읽기만 한다(예외: 초기에 만든 `ext_bizday.csv`). 원자료와 그 파생 파일은 로컬 전용이며 외부 공유·GitHub 업로드 금지. 문서·PDF에는 법인 ID와 개별 잔액을 넣지 않는다.

## 1. 데이터 (`C:\test\data\`)

| 파일                                             | 내용                                                                                                              | 출처                                                  | 쓰는 코드                                                                 |
| ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------------- |
| `raw\(iM뱅크) 2026 교육용 법인 익명데이터.xlsx`  | 원자료. 365,988행 × 70열(월말 잔액, 월 흐름, 좌수·건수 구간값). 대출 실행·상환액 없음, 금액은 유효숫자 2자리 정도 | iM뱅크 교육용 (로컬 전용)                             | `code\analyze_im_bank.py`, `YoY.ipynb`                                    |
| `raw\K-stat 무역통계 - 한국무역협회 (1)/(2).xls` | 대구(1)·경북(2) 월별 수출입                                                                                       | 한국무역협회 K-stat                                   | `analyze_im_bank.py`, `YoY.ipynb`                                         |
| `processed\df_ready.csv`                         | 분석 패널(원자료 + 지역 수출 YoY `exp_yoy` + 노출 `exposed`). 전국 15,473곳 × 36개월, 이 중 대구·경북 11,036곳    | 팀 전처리 결과(생성 코드는 이 폴더에 없음, 로컬 전용) | 거의 모든 분석                                                            |
| `ext_region.csv`                                 | 대구·경북 월별 수출액                                                                                             | K-stat 정리본                                         | `code\matching\trigger.py`                                                |
| `ext_bizday.csv`                                 | 월별 영업일수 (자체 작성 달력)                                                                                    | `code\make_bizday.py`                                 | 포착률 재검증, 통합분석, trigger                                          |
| `workdays_2021_2025.csv`                         | 월별 영업일수·전년동월차 (팀 배포, 팀 레포 `data/external`과 동일)                                                | 팀 ④ 담당                                             | DID·harmonized·메커니즘·탐색                                              |
| `ext_prod.csv`, `panel_dg.parquet`               | 업종생산지수, 대구·경북 패널                                                                                      | 팀 파일                                               | `Downloads\segment12b.py`, `segment12c_recheck.py`, `lib.py` (표7 재확인) |

파생·중간 파일(로컬 전용):

- `분석결과\matching\trigger_stages.csv` — 지역×월 원YoY·보정YoY(= 원YoY − 4.458 × 영업일수차)·단계. `code\matching\trigger.py`가 만든다. 모든 사후 분석의 충격 변수.
- `C:\test\external\part3_work\` — 팀 파트3 코드를 수정 없이 돌린 미러(은행 파생 parquet 포함: `step1_loan_industry_panel.parquet`, `step3_psm_matched.parquet`). C2 매칭 변수와 파트3 재현에 쓴다.
- `C:\test\external\Don-Ddok_Data\` — 팀 GitHub 클론(원본, 수정하지 않음). `src/part2_deposit_industry`(예금), `src/part3_loan_industry`(여신).

## 2. 코드와 결과 (시간 순)

| 단계               | 코드                                                                                                                       | 결과                                                                                                      | 비고                                                             |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| 기초 분석          | `code\analyze_im_bank.py`, `YoY.ipynb`                                                                                     | –                                                                                                         | 원자료·K-stat 읽기, 초기 LP                                      |
| 포착률 달력 보정   | `code\financial_pressure_signals_adj.py`, `run_capture_step1.py`, `run_capture_adj.py`                                     | `분석결과\민영_포착률보정.md`                                                                             | Python 3.11 (linearmodels). 착시로 판명                          |
| 비대칭 재확인      | `code\asym_lp_adj.py`, `run_asym.py`                                                                                       | `분석결과\민영_비대칭재확인.md`                                                                           | 비대칭 없음 유지                                                 |
| 표7 재확인         | `Downloads\segment12c_recheck.py`                                                                                          | `분석결과\민영_표7재확인.md`                                                                              | 두 셀 탐색적                                                     |
| 영업일수 달력      | `code\make_bizday.py`                                                                                                      | `data\ext_bizday.csv`                                                                                     | 자체 작성                                                        |
| β1+β3 재검정       | `code\beta_sum_test.py` (+ `Downloads\lib.py`, `beta1_common.py` 원본은 수정 안 함)                                        | `분석결과\민영_β합검정*`                                                                                  | 사전등록                                                         |
| 매칭 모델 1~3단계  | `code\matching\trigger.py`, `persona.py`, `matcher.py`, `match_rules.yaml`, `run_stage3_update.py`, `build_summary_pdf.py` | `분석결과\matching\*.md`, `매칭모델_총정리.pdf`                                                           | 규칙은 모두 `match_rules.yaml`                                   |
| 추천 화면(데모 웹) | `code\matching\build_live_data.py` → `분석결과\matching\demo\index.html` (원본 `index_template.html`)                      | 웹 화면                                                                                                   | 매칭을 다시 돌리면 `build_live_data.py`만 다시 실행              |
| 인사이트 맵        | `code\insight_map.py`                                                                                                      | `분석결과\insight\`                                                                                       | 법인 클러스터 k=6                                                |
| 파트3 비교         | `code\compare_part3.py`                                                                                                    | `분석결과\비교_파트3\`                                                                                    | 영업일수 보정 효과                                               |
| 통합 분석          | `code\integrated_analysis.py` → `integrated_savings.py` → `build_integrated_pdf.py`                                        | `분석결과\통합분석\통합정리.pdf` (코드는 `요구불x여신_통합정리.pdf`로 저장, 현재 파일명은 `통합정리.pdf`) | 이 순서로 실행                                                   |
| 대출 강건성        | `outputs\did_loan_robust\run.py`, `plot.py`                                                                                | `REPORT.md`                                                                                               | git: 48b2056 → 0bc5a39 → d6b0bd8                                 |
| 요구불 DID         | `outputs\did_demand_deposit\run.py`(hlib 기반), `plot.py`                                                                  | `REPORT.md`                                                                                               | git: c8e48c3 → 8884970 → 8bb045d. 옛 코드 `run_old_code.py` 보존 |
| 공통 사양 재추정   | `outputs\harmonized\hlib.py`(run_common), `validate.py`, `run_harmonized.py`, `assemble.py`, `mde.py`, `audit.py`          | `REPORT.md`(부록 A 점검표, 부록 B MDE)                                                                    | git: 58502f3 → … → 8d0646f                                       |
| 메커니즘           | `outputs\mechanism_inflow\step0_describe.py`, `run.py`, `plot.py`                                                          | `REPORT.md`                                                                                               | git: 4e2298c → 865f896                                           |
| 탐색 분석          | `outputs\exploratory\step0_candidates.py`, `explore.py`, `make_summary.py`                                                 | `summary.md`, `tests_log.csv`                                                                             | git: 34b3f63                                                     |
| 업종 충격          | `outputs\industry_shock\build_mapping.py`, `stage1.py`, `stage1_import.py`                                                 | `REPORT.md`, `REPORT_IMPORT.md`                                                                           | HS4→KSIC 연결표. 수출·수입 모두 1단계에서 연결 약함, 2단계 미실행       |
| 기업별 외환 충격   | `outputs\firm_shock\step0.py`, `run.py`, `mde.py`, `judge.py`                                                              | `REPORT.md`                                                                                                | git: 12b17be → 9951e3c. h=6 확정 없음, 적립식 약한 증거               |
| 사전 MDE 판단      | `outputs\trade_extra_mde`, `outputs\trade_finance_mde`                                                                     | `prospect.md`, `prospect_by_h.md`                                                                          | 본 분석 전 판단용(회귀 없음). 전부 진행 안 함 또는 단위 주의            |
| 디벨롭 검토        | `outputs\devreview\bundle_a_data_integrity.py`                                                                             | `REPORT.md`                                                                                                | 최우선 검산 3건 + 문구 감사 + 묶음 A. 묶음 B~I는 다음 순서            |
| 최종 정리          | `outputs\FINAL\build_final_pdf.py`                                                                                         | `돈독_민영파트_최종정리.pdf`                                                                              | 이 폴더                                                          |

## 3. 공통 함수 `hlib.py`

- 원본: `outputs\harmonized\hlib.py`. `did_demand_deposit`에는 같은 파일(MD5 동일)을, `mechanism_inflow`·`exploratory`에는 §3용 옵션(`fe2`, `ctrl`, `dep_fn`, `need_two`)을 더한 복사본을 둔다. 기존 경로(C1)는 모두 같고, 각 폴더에서 harmonized 요구불 C1 h=6(β3 +0.363537, SE 0.113493)을 재현하는 검증을 먼저 돌린다.
- 추정: 법인 FE + 지역×연월 FE를 반복 없이 정확히 제거(FWL), 법인·연월 이중 군집(CGM), p = t(G_min − 1). 교대투영 결과와 기계 정밀도로 같음(`harmonized\fwl_vs_iter_check.csv`).

## 4. 재현 순서 (Python 3.11)

```
# 0) 충격 변수 (trigger_stages.csv)
py -3.11 C:\test\code\matching\trigger.py
# 1) 파트3 미러(C2 매칭 변수·파트3 재현에 필요): external\part3_work에서 팀 원본 step1_1~1_4, step3_2 실행
# 2) 사후 분석 (각 폴더에서)
py -3.11 -u outputs\did_loan_robust\run.py        ;  py -3.11 outputs\did_loan_robust\plot.py
py -3.11 -u outputs\did_demand_deposit\run.py     ;  py -3.11 outputs\did_demand_deposit\plot.py
cd outputs\harmonized
py -3.11 validate.py
py -3.11 -u run_harmonized.py 거치식 적립식 --industry
py -3.11 -u run_harmonized.py 운전자금 요구불 --models C1
py -3.11 -u run_harmonized.py 운전자금 --models C2,E23,EX,POS,HOLD,RAW
py -3.11 -u run_harmonized.py 요구불 --models C2,E23,EX,HOLD,RAW      # 요구불 POS는 did_demand_deposit A-양수에서 가져옴
py -3.11 assemble.py ; py -3.11 mde.py ; py -3.11 audit.py
py -3.11 -u outputs\mechanism_inflow\run.py       ;  py -3.11 outputs\mechanism_inflow\plot.py
py -3.11 -u outputs\exploratory\explore.py        ;  py -3.11 outputs\exploratory\make_summary.py
# 3) 최종 PDF
py -3.11 outputs\FINAL\build_final_pdf.py
```

필요 패키지(py3.11): pandas, numpy, scipy, matplotlib, reportlab, scikit-learn, pyarrow, pyfixest(교차 확인), tabulate(assemble 표), linearmodels(포착률 재검증).

## 5. 정리 대상으로 본 것 (지우지 않음)

- `__pycache__` — 2026-09-29에 6개 삭제(`code\`, `code\matching\`, `outputs\` 아래 4개, `.pyc`만 들어 있었음). 팀 레포 클론(`external\`)은 건드리지 않았다. 실행하면 다시 생기며 각 저장소의 `.gitignore`에 들어 있다.
- `분석결과\matching\demo\assets\hero_*`, `build_hero_layers.py` — 예전 첫 화면용, 현재 화면에서 쓰지 않음.
- 각 `outputs\*` 폴더의 `run_log.txt`·검산 스크립트(`chk`, `cmp_fwl.py`, `check_pos_13.py`)는 기록 목적이라 남겨 두었다.
