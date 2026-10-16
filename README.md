# 유튜브 분야 상승 예측 프로젝트

![status](https://img.shields.io/badge/status-final%20presentation-2F5D62)
![model](https://img.shields.io/badge/model-BiGRU-355E63)
![task](https://img.shields.io/badge/task-Top--N%20category%20forecast-C2A88D)
![scope](https://img.shields.io/badge/scope-core%2010%20categories-7DA39D)

최근 12주 반응 시계열, Google Trends, 캘린더 변수를 함께 사용해 핵심 10개 유튜브 분야 중 다음 4주 동안 더 올라갈 가능성이 높은 Top-5 분야를 예측한 딥러닝 프로젝트입니다.

이 저장소는 결과만 올려 둔 공간이 아니라, 문제를 어떻게 다시 정의했는지, 데이터 정합성 문제를 어떻게 풀었는지, 왜 BiGRU를 선택했는지, 그리고 최종 발표와 Q&A까지 어떻게 정리했는지를 함께 남겨 둔 작업 기록입니다.

## 30초 요약

| 항목 | 내용 |
|---|---|
| 목표 | 핵심 10개 유튜브 분야 중 향후 4주 상승 가능성이 높은 Top-5 분야 예측 |
| 입력 | 최근 12주 카테고리 시계열, Google Trends, 캘린더 변수, category embedding |
| 모델 | RNN 계열의 표준 시계열 딥러닝 모델인 BiGRU |
| 출력 | 상승 확률, 순위 상승 확률, 최종 Top-N 선별 |
| 최종 Top-5 | 반려동물, 먹방, 경제, 브이로그, 교육 |

## 팀 구성과 내 역할

수업 과제로 진행한 3인 팀 프로젝트입니다.

| 담당 | 실제 기여 |
|---|---|
| 나 | 프로젝트 아이디어를 제시하고 YouTube API 기반 데이터를 수집했습니다. Codex와 함께 변수·모델 코드 작업을 진행했습니다. |
| 팀원 1 | 발표 자료를 준비하고 발표했습니다. |
| 팀원 2 | 데이터 수집을 함께 도왔습니다. |

코드와 파생변수, 모델 구현은 생성형 AI(Codex)와 함께 진행했습니다. 저는 아이디어와 수집 데이터를 제공했고 코드를 직접 타이핑하지 않았습니다. 이 문서의 변수 설명은 저장소의 코드를 다시 읽고 계산식을 풀어 쓴 것입니다.

## 이 저장소에 남긴 과정

- 개별 영상에서 카테고리 상승 예측으로 문제를 다시 정의한 과정
- YouTube API 데이터 수집과 주차·분야 단위 데이터 정리
- 발표 3일 전 분석 중 랭킹 구조의 모델명을 보고 딥러닝 과제 조건과 맞지 않음을 깨달아, 머신러닝 분류·랭킹 구조에서 BiGRU로 전환한 과정
- AI와 함께 만든 변수·모델의 계산 기준, 결과와 한계를 설명한 기록

## 바로 보기

- 발표 자료: [딥러닝 발표 PPT.pdf](./딥러닝%20발표%20PPT.pdf)
- 예상 질문 표: [QnA_Report.pdf](./QnA_Report.pdf)
- 발표용 답변 정리: [PRESENTATION_QNA.md](./PRESENTATION_QNA.md)
- 모델 설명: [FINAL_MODEL.md](./FINAL_MODEL.md)
- 결과 정리: [RESULTS.md](./RESULTS.md)
- 프로젝트 스토리: [PROJECT_JOURNEY.md](./PROJECT_JOURNEY.md)
- 영어 문서 시작점: [PROJECT_JOURNEY_EN.md](./PROJECT_JOURNEY_EN.md)

## 읽는 사람별 추천 동선

- 처음 들어온 사람: `README → 발표 PPT → RESULTS`
- 교수님이나 기술 검토자: `README → FINAL_MODEL → DATA_VARIABLE_GUIDE → EVALUATION_GUIDE`
- 프로젝트 스토리가 궁금한 사람: `README → PROJECT_JOURNEY → TROUBLESHOOTING → PRESENTATION_QNA`

## 프로젝트 흐름

```mermaid
flowchart LR
    A["문제 인식<br/>무슨 영상이 뜨는가"] --> B["문제 재정의<br/>어떤 분야가 앞으로 올라갈까"]
    B --> C["데이터 수집<br/>YouTube API + Google Trends + Calendar"]
    C --> D["전처리와 주차 정렬<br/>ISO week 통일과 category-week 생성"]
    D --> E["파생변수 생성<br/>momentum, rolling mean, competition, opportunity"]
    E --> F["BiGRU 학습<br/>최근 12주 흐름과 category embedding 반영"]
    F --> G["Top-5 예측<br/>상승 가능성이 높은 핵심 분야 선별"]
    G --> H["발표와 Q&A 정리<br/>결과 해석과 활용 시나리오 정리"]
```

## 문제 정의

처음에는 개별 영상의 virality를 맞히는 방향으로 접근했습니다. 하지만 실제로는 영상 하나의 초기 반응만으로 미래 관심 흐름을 설명하기 어렵다고 판단했고, 질문을 어떤 영상이 잘 될까에서 어떤 분야가 앞으로 더 주목받을까로 바꾸었습니다.

최종 문제 정의는 아래와 같습니다.

> 최근 12주간의 유튜브 반응 데이터, Google Trends, 캘린더 변수를 활용하여 핵심 10개 유튜브 분야 중 다음 4주 동안 상승 가능성이 높은 Top-5 분야를 예측한다.

## 데이터와 변수

이 프로젝트는 세 종류의 데이터를 함께 사용했습니다.

- YouTube Data API 기반 영상 메타데이터와 반응 지표
- 카테고리별 주간 추세 데이터
- Google Trends와 공휴일, 연휴, 방학, 시험기간 같은 캘린더 변수

대표 파생변수는 `rolling_4week_mean`, `momentum_ratio`, `competition_score`, `opportunity_score`이며, 정답 라벨은 `rise_label`, `rank_up`으로 구성했습니다.

자세한 기준은 아래 문서에 정리했습니다.

- [DATA_VARIABLE_GUIDE.md](./DATA_VARIABLE_GUIDE.md)
- [DATA_PIPELINE_CODE_INDEX.md](./DATA_PIPELINE_CODE_INDEX.md)

## 모델과 선택 이유

최종 모델은 RNN 계열에서 GRU를 양방향으로 확장한 BiGRU입니다. 5월 18일자 발표 준비 문서에는 이전 머신러닝 단계가 HistGradientBoostingClassifier 기반 상승 여부 분류와 HGB·Random Forest·ExtraTrees(0.5/0.3/0.2) 혼합 랭킹으로 기록되어 있습니다. 발표 3일 전 분석 과정에서 랭킹 구조에 사용된 모델 이름을 확인하다가, 딥러닝 수업 과제에 머신러닝 모델을 사용하고 있다는 사실을 깨달았습니다. 과제의 딥러닝 모델 사용 조건을 놓친 것이었고, 시간 흐름을 입력에 반영할 수 있다는 Codex의 제안을 바탕으로 BiGRU로 전환했습니다. 다른 모델보다 우수하다는 동일 조건 비교 실험을 거쳐 선택한 것은 아닙니다.

모델은 다음 순서로 작동합니다.

1. 카테고리별 최근 12주 시계열과 외부 변수를 입력으로 받습니다.
2. category embedding으로 분야별 고유 특성을 함께 반영합니다.
3. BiGRU가 최근 흐름의 방향성을 학습합니다.
4. 상승 확률과 순위 상승 확률을 함께 산출해 최종 Top-N을 선별합니다.

모델 구조 설명은 [FINAL_MODEL.md](./FINAL_MODEL.md)에 정리했습니다.

## 핵심 결과

### 분류 성능

| Metric | Score |
|---|---:|
| Accuracy | 0.767 |
| Balanced Accuracy | 0.798 |
| Precision | 0.944 |
| Recall | 0.739 |
| F1-score | 0.829 |
| ROC AUC | 0.845 |

### Top-5 선별 성능

| Metric | Score |
|---|---:|
| Precision@5 | 0.900 |
| Recall@5 | 0.801 |
| HitRate@5 | 1.000 |
| NDCG@5 | 0.881 |

### 최종 Top-5 예측 분야

1. 반려동물
2. 먹방
3. 경제
4. 브이로그
5. 교육

여기서 중요한 점은 이 결과가 지금 가장 큰 분야를 그대로 고른 것이 아니라, 최근 반응 흐름과 검색 관심도, 상대 순위 변화를 함께 반영해 앞으로 더 올라갈 가능성이 있는 분야를 선별한 결과라는 점입니다.

결과 해석은 [RESULTS.md](./RESULTS.md), 지표 의미는 [EVALUATION_GUIDE.md](./EVALUATION_GUIDE.md)에서 볼 수 있습니다. 위 평가는 핵심 10개 분야의 테스트 윈도우 30개에 대한 결과입니다. 독립된 외부 데이터나 이전 머신러닝 구조 대비 우위를 검증한 수치로 해석하지 않습니다.

## 이번 프로젝트에서 어려웠던 점

- `year_week` 기준이 파일마다 달라 ISO week로 다시 맞춰야 했습니다.
- Google Trends 수집 결과를 바로 모델에 넣기 어려워 별도 정리 파이프라인이 필요했습니다.
- 모든 분야를 한 번에 다루면 분포가 불안정해 활동량과 연속성이 충분한 핵심 10개 분야로 문제를 다시 정의했습니다.
- 발표 3일 전 분석 중 랭킹 구조의 모델명을 보고 과제 조건과의 불일치를 깨달아 머신러닝 분류·랭킹 구조에서 BiGRU로 전환했습니다. 모델 종류와 테스트 일정을 팀에 더 일찍 공유하지 못한 점도 돌아볼 부분입니다.

이 과정을 더 자세히 적어 둔 문서는 [PROJECT_JOURNEY.md](./PROJECT_JOURNEY.md)와 [TROUBLESHOOTING.md](./TROUBLESHOOTING.md)입니다.

## 문서 읽는 순서

1. [README.md](./README.md)
2. [딥러닝 발표 PPT.pdf](./딥러닝%20발표%20PPT.pdf)
3. [PROJECT_JOURNEY.md](./PROJECT_JOURNEY.md)
4. [FINAL_MODEL.md](./FINAL_MODEL.md)
5. [RESULTS.md](./RESULTS.md)
6. [PRESENTATION_QNA.md](./PRESENTATION_QNA.md)

## 저장소 안 핵심 파일

```text
.
├─ 딥러닝 발표 PPT.pdf
├─ QnA_Report.pdf
├─ youtube_trend_project_pipeline.ipynb
├─ train_active_category_rank_bigru.py
├─ run_core10_top_predictions.py
├─ make_paper_visualization_suite.py
├─ PROJECT_JOURNEY.md
├─ FINAL_MODEL.md
├─ RESULTS.md
├─ PRESENTATION_QNA.md
├─ DATA_VARIABLE_GUIDE.md
├─ EVALUATION_GUIDE.md
└─ docs/assets/
```

## 공개 자료 범위

발표 자료의 이름과 학번을 제거하고, 노트북의 저장 출력과 개인 PC 경로를 제거했습니다. 원본과 이전 머신러닝 단계 노트북은 비공개 보관합니다. 공개 노트북은 코드 열람용이며 원본 입력 데이터가 포함되어 있지 않아 전체 실행에는 별도 수집이 필요합니다.

## 실행과 재현

```bash
pip install -r requirements.txt
```

실행 순서와 재현 범위는 아래 문서를 참고하면 됩니다.

- [RUN_PROJECT.md](./RUN_PROJECT.md)
- [REPRODUCIBILITY.md](./REPRODUCIBILITY.md)

## 한 줄 정리

이 저장소는 유튜브 핵심 10개 분야의 향후 4주 상승 가능성을 최근 12주 시계열과 외부 변수, BiGRU 기반 딥러닝 모델로 예측하고, 그 과정을 발표 자료와 Q&A까지 포함해 정리한 프로젝트 기록입니다.