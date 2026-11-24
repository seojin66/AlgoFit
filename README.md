# 알고핏 (AlgoFit) — 코딩 문제 풀이 이력 분석과 LLM을 활용한 맞춤형 학습 지원 시스템

한양여자대학교 빅데이터과 2026 졸업작품
팀: 김서진, 양지원, 장서은, 홍연의 · 지도교수: 하효동 · 산업체 멘토: 커리어케어 이효영 상무 

## 1. 개요

정답률만으로는 코딩 학습자가 **실력에 비해 어느 분야가 약한지** 알기 어렵습니다(진단 대상 8,502명 중 79.3%가 정답률 90% 이상).
이 프로젝트는 Codeforces 공개 풀이 기록(제출 17,607,999건)을 분석해 다음을 만들었습니다.

1. **약한 분야 진단** — 체스 실력 점수 방식인 Elo 레이팅을 학습에 적용한 방법(Pelánek, 2016)으로 문제마다 맞힐 확률 P를 계산합니다.
   - P = 1 / (1 + e^−(a + b × (문제 난이도 − 실력 점수) / 400)), a = 2.79, b = −0.63 (2023년 7~12월 기록 908,985개로 로지스틱 회귀)
   - 분야별로 P를 더한 '예상 맞힘 수'와 실제 맞힌 수를 비교해 z ≤ −2면 약한 분야, z ≥ +2면 잘하는 분야로 판정
   - 결과: 약한 분야가 있는 학습자 1,101명, 예측 정확도(AUC) 평소 실력 점수 0.743 vs 공식 점수 0.722
2. **학습 유형 진단** — 풀이 습관 지표 9개로 K-means를 돌려 4가지 유형(로직 점검형 · 도전적 시행착오형 · 효율 개선 필요형 · 문법 실수·빠른 재제출형)으로 나눔 (엘보우·실루엣으로 4개 결정)
3. **알고핏 웹 서비스** — 진단 결과에 맞춰 AI 코치(OpenAI gpt-4o-mini)가 도움 정도와 단계별 힌트를 줌 (정답 코드는 주지 않음)
4. **재사용 가능한 진단 알고리즘** — 채점 결과 이름표만 바꿔 일본 사이트 AtCoder 기록에도 적용

## 2. 목차 (폴더 구성과 실행 순서)

```
capstone/                    진단 알고리즘
├── engine.py                공통 계산(맞힐 확률·예상 개수·z·학습 유형 배정)
├── step0_prepare.py         ① 원본 제출 기록 정리 (중복·채점 오류 제외, 분야 10개로 묶기)
├── step1_periods_model.py   ② 기간 3개로 나누고 a, b 구하기 (로지스틱 회귀)
├── step3_diagnosis.py       ③ 8,502명 약한/잘하는 분야 진단
├── step4_learning_type.py   ④ 풀이 습관 9개 → K-means 학습 유형 4가지
├── step5_new_user.py        ⑤ 새 학습자 진단
├── step6_validate.py        ⑥ 진단에 쓰지 않은 다음 기간 기록으로 검증
├── step7_results.py         ⑦ 결과 표·요약
├── step8_web_llm_data.py    ⑧ 웹 서비스용 사용자별 진단 파일 만들기
├── model/model_settings.json  a, b, 판정 기준, K-means 군집 중심 등 설정값
├── output/                  결과 파일(보고서·결과 표 등, 용량이 큰 중간 파일은 제외)
└── generic/                 다른 사이트에 다시 쓰기(AtCoder 적용, 공식 점수 없이 재진단 검증)
분석결과물/                  졸업작품 분석 결과물 (10월 첫 분석 노트북·결과와 LLM 웹 서비스)
├── 0_먼저_읽어주세요_파일안내.md   파일 안내
├── 1_분석코드_Colab용.ipynb · 2_분석코드_실행결과포함.ipynb · 3_분석코드_파이썬.py   분석 노트북·코드
├── 5_분석방법_단계별_안내.docx · 7_재사용_알고리즘_개발계획.docx · 8_기준값_비교실험_결과.docx
├── 분석결과/                결과표·대시보드·그림·사용자별 진단 파일(학습자 ID 익명)
└── 6_LLM_학습도우미_스트림릿/   알고핏 웹 서비스 (Streamlit + OpenAI)
    ├── 학습도우미_app.py     웹 앱 본체 · 실행 방법은 실행방법.md
    ├── env_예시.txt          API 키 설정 예시 (.env로 복사해 키 입력, 키는 올리지 않음)
    ├── UI설계서_알고핏.docx · 화면캡처/ · 시연영상/
    └── 점검_테스트.py        앱 동작 점검
발표자료/                    최종 발표 자료·최종 산출물(PDF)·시연 영상(30초)
대시보드/                    분석 대시보드 2개 (내려받아 브라우저로 열기)
├── 대시보드1_알고핏_재사용진단.html
└── 대시보드2_첫분석_Codeforces.html
문서/                        분석 결과 요약·LLM 학습도우미 설명·사례별 진단·재사용 알고리즘·UI 설계서(docx)
figures/                     발표·보고서 그래프와 그리는 코드(make_figures.py)
```

※ 발표·산출물의 숫자는 `capstone/output/`(최종 진단)과 `capstone/generic/output/`(AtCoder·공식 점수 없이 재진단) 결과 파일 기준입니다. `분석결과물/분석결과/`는 10월 첫 분석 결과라 일부 숫자가 다를 수 있습니다.

실행 순서: 아래 데이터셋을 내려받아 `capstone/step0_prepare.py`의 `DATA_FILE` 경로를 맞춘 뒤 step0 → step1 → step3 → step4 → step6 → step7 → step8 순서로 실행합니다.
웹 앱: `cd 분석결과물/6_LLM_학습도우미_스트림릿 && pip install -r requirements.txt && streamlit run 학습도우미_app.py` (OpenAI API 키 필요)

## 3. 출처

- 데이터: denkCF, *UsersCodeforcesSubmissionsEnd2024*, Hugging Face Datasets (CC BY 4.0) — 용량 문제로 이 저장소에는 원본을 넣지 않았습니다.
- 문제 정보: Codeforces API (problemset.problems) · AtCoder Problems API (kenkoooo.com/atcoder)
- Pelánek, R. (2016). Applications of the Elo rating system in adaptive educational systems. *Computers & Education*, 98, 169–179.
- Piech, C. et al. (2015). Deep Knowledge Tracing. *NeurIPS 2015*.
- Kasneci, E. et al. (2023). ChatGPT for good? On opportunities and challenges of large language models for education. *Learning and Individual Differences*, 103, 102274.
- 도구: Python 3.12, pandas, NumPy, SciPy, scikit-learn, Matplotlib, Streamlit, python-docx, OpenAI API (gpt-4o-mini)
