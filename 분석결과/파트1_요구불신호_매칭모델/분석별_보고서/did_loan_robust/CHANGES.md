# CHANGES — 에러 수정 기록 (사양 변경 없음)

SPEC.md(커밋 48b2056) 이후 실행 중 생긴 에러만 고쳤다. 추정식·표본·판정 기준은 바꾸지 않았다.

| # | 에러 | 수정 | 결과 영향 |
|---|---|---|---|
| 1 | py3.11에서 parquet 읽기 실패 (`ImportError: Unable to find a usable engine`) | `py -3.11 -m pip install pyarrow` | 없음 (환경) |
| 2 | L2의 pyfixest 교차 확인에서 FE 제거가 수렴하지 않음 (`ValueError: Demeaning failed after 10000 iterations`) | `run.py`의 `pf.feols(...)`에 `fixef_maxiter=100000` | 없음 (보고 수치는 자체 계산, pyfixest는 확인용) |
| 3 | 그림 글꼴(Malgun Gothic)에 U+2212(−) 기호 없음 | `plot.py`의 U+2212를 "-"로 바꿈 | 없음 (표시만) |
