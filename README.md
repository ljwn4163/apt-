apt- — 서울 아파트 지수 파생상품 모의거래소 (MVP)

한 줄 소개
서울 아파트 가격지수를 기초자산으로 하는 지수형 파생상품(현물 단위·무기한·월물) 의 모의거래소 데모입니다.
지수 산출(오프체인) → 오라클 집계 → 거래소 로직(마크가격·펀딩)까지 재현 가능한 최소 파이프라인을 제공합니다.

왜 이것을 만드나

헷지 수단 부재: 한국 부동산엔 롱·숏 대칭의 지수형 파생이 사실상 없습니다.

투명성 부족: 가격지수 산출·전송 과정이 불투명하면 거래 신뢰가 떨어집니다.

MVP 목표: 소규모/오프체인으로도 지수 → 오라클 → 파생 정산의 핵심을 검증합니다.

시스템 개요 (3 레이어)

지수 레이어 (Index)

실거래/호가 등 다중 신호를 가정해 일(日) 단위 나우캐스트 지수를 만듭니다.

월 1회 발표되는 공식 월간지수로 앵커링(드리프트 보정) 하여 장기 기준을 맞춥니다.

불확실성(표준오차, stderr)을 함께 공개합니다.

오라클 레이어 (Oracle)

다중 소스 → 미디안 → 윈저라이즈(상·하위 꼬리 절단) → TWAP(시간가중평균) 로 깨끗한 피드를 만듭니다.

커밋·리빌(Commit-Reveal) 로그로 사후조작 방지를 증빙합니다.

거래소 레이어 (Exchange)

SAIT: 지수 1포인트=1 단위(회계단위)

SAIT-PERP: 무기한 스왑의 펀딩비 로직(괴리 시 롱↔숏 정산)

월물 선물(Stub): 만기 현금결제 가정(구조만 제공)

산출물(Artifacts)

data/processed/index_timeseries.csv

timestamp, index_value, stderr

일자별(또는 시간별) 지수와 불확실성

data/processed/oracle_feed.csv

timestamp, median, winsor, twap

오라클 집계 결과(거래소 참조가)

data/processed/funding_log.csv

timestamp, mark_price, index_value, basis, funding_rate

마크가격·펀딩 정산 기록

폴더 구조
apt-
├─ README.md
├─ LICENSE
├─ .gitignore
├─ Makefile
├─ requirements.txt
├─ docs/
│  ├─ 00-overview.md           # 개요
│  ├─ 01-index-spec.md         # 지수 사양/앵커링/오차
│  ├─ 02-oracle-spec.md        # 미디안·윈저·TWAP·커밋리빌
│  ├─ 03-exchange-spec.md      # 마크가격·펀딩·상품 정의
│  ├─ 04-data-sources.md       # 데이터 출처/스키마
│  ├─ 05-risk-notes.md         # 리스크 목록
│  └─ 06-roadmap.md            # 개발 로드맵
├─ data/
│  ├─ raw/                     # 원천/모의 입력
│  ├─ interim/                 # 중간 산출물(비커밋)
│  └─ processed/               # 데모 최종 출력(CSV/Parquet)
├─ index/                      # 지수 계산
├─ oracle/                     # 오라클 파이프라인
├─ exchange/                   # 거래소 로직
└─ tests/                      # 간단 테스트

빠른 시작 (Quickstart)

아래 명령은 환경 준비와 파이프라인 실행을 위한 것입니다.
코드 구현 없이도 데모 CSV가 생성되는지 재현할 수 있어야 합니다.

# 1) 환경 구성
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# 2) 지수 → 오라클 → 펀딩 순서 실행
python index/nowcast.py
python oracle/aggregator.py
python exchange/funding.py

# 3) 결과 확인
ls -l data/processed/
# index_timeseries.csv / oracle_feed.csv / funding_log.csv 가 보이면 OK

# 4) 테스트(스키마 검사)
pytest -q

설계 핵심 개념 (요약)

앵커링(Anchoring):
우리 일간 지수의 월평균을 공식 월간지수에 맞추는 스케일 보정.
→ 장기 드리프트(누적 편차) 방지, 기준선 정렬.

stderr (표준오차):
지수 추정의 불확실성 폭. 예: stderr=0.4면 ±0.4포인트 수준의 오차범위.

펀딩비(Funding):
선물 마크가격−현물 지수의 괴리를 줄이는 동적 정산 금리.
괴리가 +면(롱 과열) 롱이 숏에게 지급, −면 반대.

Commit-Reveal:
계산 당시 해시(commit) 를 먼저 남기고, 나중에 원본(reveal) 공개해
사후조작이 없었음을 검증.

원칙

재현성: 새로 클론해도 Quickstart만으로 동일 결과를 얻음

투명성: 오라클 경로와 커밋·리빌 로그 공개

모듈성: 각 레이어는 독립 실행, CSV/Parquet로 연결

Small First: 더미로 시작 → 실제 데이터/모델로 쉽게 교체

로드맵 (요약)

S0: 리포 스켈레톤 + 문서 + 더미 파이프라인

S1: 미디안/윈저/TWAP, 펀딩 로직 최소 단위테스트

S2: 헤도닉 + 칼만 + 월간 앵커링 적용

S3: 커밋·리빌 자동화(GitHub Actions) & 데모 노트북

S4: 리팩토링, 테스트 ≥10, 스크린샷, README 정리

자세한 내용은 docs/ 참조.

데이터 & 라이선스

데이터: 데모 재현을 위해 소용량 CSV만 저장소에 포함합니다.
대용량/민감 원천은 외부 저장소를 권장합니다.

라이선스: MIT (자유 사용/수정/배포, 보증 없음)

기여(Contributing) & 이슈

버그/개선 제안은 GitHub Issues 를 이용하세요.

PR은 작은 단위로, 재현 가능한 변경(테스트/문서 포함)을 선호합니다.

FAQ

Q. 왜 ‘월간’ 공식지수를 쓰나요?
A. 일간 나우캐스트는 예측치라 장기 드리프트가 생깁니다. 월 1회 공식 값으로 기준점을 맞춰 신뢰도를 높입니다.

Q. 선물만 할 거면 현물 지수가 왜 필요하죠?
A. 선물의 기초자산은 현물 지수입니다. 선물-현물 괴리를 펀딩/차익 거래가 줄이며 가격발견이 이뤄집니다.

Q. 변동성이 작지 않나요?
A. 월간 지수는 작지만 일간화 + 레버리지로 트레이딩 감각을 확보합니다. 기관 헷지엔 작은 변동성도 충분히 유의미합니다.
