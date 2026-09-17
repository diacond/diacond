# 📂 Projects

신호림의 주요 프로젝트를 **주제 · 목적 · 동기 · 세부사항** 순서로 정리한 문서입니다.

| # | 프로젝트 | 한 줄 요약 | 주요 기술 | 형태 |
| --- | --- | --- | --- | --- |
| 1 | [하나케어](#1-하나케어-hanarocare) | 시니어 자산 보호 서비스 | Spring · MySQL · React · TypeScript | 팀 (6인) |
| 2 | [battery-soh-pipeline](#2-battery-soh-pipeline) | 배터리 수명(SOH/RUL) 예측 API | Python · scikit-learn · FastAPI · Docker · GitHub Actions | 개인 |
| 3 | [mstar-atr](#3-mstar-atr) | 레이더(SAR) 영상 표적 분류 API | Python · scikit-image · scikit-learn · FastAPI | 개인 |
| 4 | [QuantInvesting](#4-quantinvesting) | 이동평균 교차 전략 백테스팅 대시보드 | Python · vectorbt · Streamlit · Plotly | 개인 |
| 5 | [hanaro_first_hw](#5-hanaro_first_hw) | 게시판 DB 모델링 과제 | Next.js · TypeScript · Prisma · MySQL | 교육 과제 |
| 6 | [PracTest](#6-practest) | 알고리즘 문제 풀이 기록 | C++ | 개인 |
| 7 | [hana8](#7-hana8) | 디지털하나路 8기 학습 기록 | HTML · CSS · JavaScript · TypeScript | 교육 과정 |

---

## 1. 하나케어 (HanaroCare)

🔗 [HanaroCare/hanaro-care-main](https://github.com/HanaroCare/hanaro-care-main)

**주제**
시니어의 노후 지출을 정밀하게 설계하고 자산을 안전하게 지켜주는 시니어 자산 보호 서비스

**목적**
은퇴 이후 시니어가 자산을 계획적으로 쓰고, 상속까지 미리 준비할 수 있도록 돕는 금융 서비스를 만드는 것

**동기**
디지털하나路 8기 2차 팀 프로젝트로 진행했습니다. 고령화로 시니어의 자산 관리와 상속 준비 수요가 커지고 있다는 점에 주목했습니다.

**세부사항**
- 팀 구성: 6명 (자산 2 · 유저 1 · **상속 2** · 카드 1)
- **내 역할**
  - **상속 파트 풀스택 개발**: 요구사항 정리부터 백엔드 API, DB, 프론트엔드 화면까지 담당
  - **최종 발표**: 서비스 기획 의도와 구현 결과를 팀 대표로 발표
- 기술: Spring, MySQL, React, TypeScript
- 기능 소개, 아키텍처, ERD는 [저장소 README](https://github.com/HanaroCare/hanaro-care-main)에 정리되어 있습니다.

---

## 2. battery-soh-pipeline

🔗 [diacond/battery-soh-pipeline](https://github.com/diacond/battery-soh-pipeline)

**주제**
리튬이온 배터리의 충방전 기록으로 **SOH(현재 성능, %)** 와 **RUL(남은 수명, 사이클 수)** 을 예측하고, 결과를 API와 자연어 설명으로 제공하는 엔드투엔드 파이프라인

**목적**
EV·ESS 배터리의 수명과 안전성을 진단하는 서비스를 작은 규모로 재현하는 것입니다. 데이터 처리 → 모델 학습 → API 서빙 → 테스트/CI까지 전 과정을 직접 구현했습니다.

**동기**
배터리는 사용 조건에 따라 열화 속도가 달라서, 교체 시점을 미리 예측하면 정비 비용과 안전 위험을 줄일 수 있습니다. 실제 기업이 서비스로 운영하는 문제를 공개 데이터로 끝까지 풀어 보고 싶었습니다.

**세부사항**
- **데이터**: NASA PCoE 배터리 4개(B0005·B0006·B0007·B0018)의 충방전 기록, 외부 검증용 CALCE CS2 셀 4개
- **구조**: `.mat` 파싱 → 사이클 단위 피처 생성 → RandomForest 학습 → FastAPI `POST /v1/battery/diagnose`
- **검증 설계**
  - 무작위 분할에서 데이터 누수를 발견했습니다. 그래서 배터리 하나를 통째로 빼고 평가하는 **Leave-One-Battery-Out** 방식으로 바꿨습니다.
- **성능 개선**
  - 방전 곡선 형태 피처(전압 표준편차, 기울기, knee point 도달 시간)를 추가했습니다.
  - SOH 평균 MAE 0.021 → **0.008**, R² 0.924 → **0.982**
- **한계도 그대로 기록**
  - RUL은 평균이 개선됐지만 일부 배터리(B0006)는 오히려 나빠졌습니다. 그래서 API 응답에 "참고용" 안내를 함께 반환합니다.
  - 학습에 쓰지 않은 외부 배터리(CALCE)에서는 R²가 0.693으로 떨어졌습니다. 피처 중요도를 분석해 원인이 knee 피처의 스케일 차이임을 찾았습니다.
- **LLM 사용 원칙**
  - 위험 판단은 규칙 기반으로 처리하고, LLM은 결과를 문장으로 설명하는 역할만 맡겼습니다.
  - LLM 호출이 실패해도 규칙 기반 설명으로 자동 대체(fail-open)되어 서비스가 멈추지 않습니다.
- **MSA 변형**: 예측 서비스, 설명 서비스, 게이트웨이로 분리하고 docker-compose를 구성했습니다. 설명 서비스가 장애여도 예측 결과는 반환합니다.
- **테스트/CI**: pytest, GitHub Actions에서 데이터 다운로드부터 학습·테스트·외부 검증까지 전체를 재현합니다.
- 상세 문서: [README](https://github.com/diacond/battery-soh-pipeline) · [PROJECT_OVERVIEW](https://github.com/diacond/battery-soh-pipeline/blob/main/PROJECT_OVERVIEW.md) · [EXPERIMENT_LOG](https://github.com/diacond/battery-soh-pipeline/blob/main/EXPERIMENT_LOG.md)

---

## 3. mstar-atr

🔗 [diacond/mstar-atr](https://github.com/diacond/mstar-atr)

**주제**
합성개구레이더(SAR) 영상에서 군용 차량 10종을 자동으로 식별하는 표적 인식(ATR) 파이프라인

**목적**
레이더 정찰 영상을 사람이 한 장씩 판독하기 전에 기계가 1차 분류를 해 주는 서비스를 작게 재현하는 것입니다. 이미지를 올리면 예측 클래스, 확신도, 클래스별 확률을 돌려주는 API로 만들었습니다.

**동기**
배터리 프로젝트가 수치 예측(회귀) 문제였다면, 이번에는 이미지 분류 문제를 다뤄 보고 싶었습니다. 흔한 예제 대신 색 정보가 없고 노이즈가 많은 SAR 영상을 골랐고, 이 분야의 표준 벤치마크인 MSTAR 데이터셋으로 실험했습니다.

**세부사항**
- **데이터**: MSTAR 10클래스 (학습 17° 관측각 2,746장 / 시험 15° 관측각 2,425장)
- **구조**: 중앙 64×64 크롭 → HOG 특징 추출 → StandardScaler + RBF SVM → FastAPI `POST /v1/sar/classify`
- **실험으로 확인한 판단**
  - 크롭 크기를 128에서 64로 줄이자 정확도가 **78% → 87%** 로 올랐습니다. 배경 노이즈가 줄어든 효과입니다.
  - RandomForest는 60%에 그쳐, 고차원 연속 피처에 강한 SVM을 선택했습니다.
- **결과**: 테스트 정확도 **86.7%**
- **오류 분석**: 혼동행렬에서 BTR60↔BTR70, 2S1↔T62처럼 외형이 비슷한 차량끼리 가장 많이 혼동하는 것을 확인했습니다.
- **테스트/CI**: pytest, GitHub Actions에서 전체 파이프라인을 재현합니다.

---

## 4. QuantInvesting

🔗 [diacond/QuantInvesting](https://github.com/diacond/QuantInvesting)

**주제**
이동평균선 교차(골든크로스·데드크로스) 전략을 백테스트하는 퀀트 투자 대시보드 **QUANTMIND**

**목적**
주식과 코인의 과거 데이터로 매매 전략의 성과를 검증하고, 여러 종목을 한 화면에서 비교하는 것

**동기**
처음으로 퀀트 투자를 직접 구현해 본 프로젝트입니다. 감이 아닌 데이터로 전략을 검증하는 과정을 경험하고 싶었습니다.

**세부사항**
- **데이터**: yfinance로 주가를 수집합니다. 미국 주식, 코인(BTC-USD), 국내 주식(`.KS`)을 지원합니다.
- **전략 엔진** (`src/backtest.py`)
  - vectorbt로 단기·장기 이동평균선을 계산합니다.
  - 교차 시그널로 매수·매도하는 포트폴리오를 시뮬레이션합니다. 수수료도 반영합니다.
- **대시보드** (`app.py`, Streamlit + Plotly)
  - **단일 종목 분석**: 총 수익률, 최대 낙폭(MDD), 샤프 지수, 승률 KPI 카드와 성과 차트
  - **다중 종목 스크리너**: 종목별 성과표 정렬, 최신 매수·매도 시그널 표시, 누적 수익률 비교 차트
  - 이동평균 기간, 초기 자본, 수수료를 사이드바에서 조정할 수 있습니다.
  - 입력값 검증: 단기 기간이 장기 기간보다 길거나 날짜가 잘못되면 오류를 안내합니다.

---

## 5. hanaro_first_hw

🔗 [diacond/hanaro_first_hw](https://github.com/diacond/hanaro_first_hw)

**주제**
게시판 서비스(사용자 · 카테고리 · 게시글 · 댓글 · 좋아요)의 데이터베이스 모델링

**목적**
Next.js 프로젝트에 Prisma ORM과 MySQL을 연결하고, 관계형 스키마를 설계·적용하는 것

**동기**
디지털하나路 8기의 첫 번째 과제로 진행했습니다.

**세부사항**
- **스키마** (`prisma/schema.prisma`)
  - User, Category, Post, Comment, PostLike 5개 모델을 정의했습니다.
  - PostLike는 (사용자, 게시글) 복합 기본키를 써서 중복 좋아요를 막습니다.
  - 외래키에 `onDelete: Cascade`를 적용하고, 조회용 인덱스를 설계했습니다.
- **시드 데이터** (`prisma/seed.ts`): upsert로 카테고리, 사용자, 게시글을 중첩 생성합니다.
- **개발 환경**: Next.js 16, React 19, Tailwind CSS 4, Biome, Prisma 스크립트(db pull/push/generate/seed/migrate), docker-compose

---

## 6. PracTest

🔗 [diacond/PracTest](https://github.com/diacond/PracTest)

**주제**
백준과 프로그래머스 알고리즘 문제 풀이 기록

**목적**
자료구조와 알고리즘 기초를 꾸준히 다지고, 코딩 테스트에 대비하는 것

**동기**
문제 풀이를 꾸준히 기록으로 남기고 싶어 BaekjoonHub로 제출과 동시에 자동 커밋되도록 구성했습니다.

**세부사항**
- 언어: C++
- 풀이 수: **약 200문제**, 커밋 214개
  - 백준 83문제 (Bronze 58 · Silver 22 · Gold 3)
  - 프로그래머스 117문제 (Lv.0 42 · Lv.1 52 · Lv.2 16 · Lv.3 4 · Lv.4 3)
- 문제별 폴더에 풀이 코드와 문제 정보(README)를 함께 저장합니다.
- Gold 문제 예시: 오큰수(스택), 후위 표기식(스택), 별 찍기 - 10(재귀)

---

## 7. hana8

🔗 [diacond/hana8](https://github.com/diacond/hana8)

**주제**
디지털하나路 8기 교육 과정의 프론트엔드 기초 학습 기록

**목적**
HTML/CSS부터 JavaScript 핵심 개념, TypeScript까지 웹 개발 기초를 직접 코드로 익히는 것

**동기**
수업 내용과 과제를 한곳에 모아 복습할 수 있도록 정리했습니다. 동기들과 브랜치를 나눠 Pull Request로 병합하며 Git 협업 흐름도 연습했습니다.

**세부사항**
- `html/`: HTML·CSS 레이아웃 실습, Tailwind CSS 실습
- `js/trythis/`: 클로저, 호이스팅, 스코프, this, 프로토타입/OOP, 제너레이터, Promise, 태스크 큐, 구조 분해, Map/Set, 정규식, 깊은 복사 등 핵심 개념 실습 30여 개
- `ts/`: TypeScript 기초, 제네릭, 옵션 타입 실습과 tsconfig 설정
- Git: 브랜치 전략, PR 병합, 커밋 메시지 템플릿(`.gitmessage`) 실습
