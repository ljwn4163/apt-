🏙️ APT — 서울 아파트 지수 기반 파생상품 모의거래소 (MVP)

Ⅰ. 프로젝트 개요

APT 프로젝트는 서울 아파트 가격지수를 기반으로 한
지수형 파생상품(현물·무기한 선물·월물) 의 모의거래 환경(MVP) 을 구축하기 위한 연구·개발 프로젝트입니다.

본 시스템은 지수 산출 → 오라클 데이터 집계 → 파생 정산 및 거래 시뮬레이션으로 이어지는
데이터 기반 금융 인프라의 최소 구조를 구현합니다.

Ⅱ. 추진 배경 및 필요성

시장 문제 인식:

국내 부동산 시장은 지수 기반의 투명한 가격 발행 체계가 미흡합니다.

실거래 중심의 비주기적 공시체계로 인해 시장 참여자의 가격 예측 및 헷지 수단이 제한적입니다.

기술적 목표:

실거래·호가 등 다양한 신호를 활용한 나우캐스트(단기예측) 지수 산출

다중 데이터 소스의 신뢰도를 보정하는 오라클 파이프라인 구축

산출된 지수를 기초자산으로 한 파생상품 정산 및 거래 구조 시뮬레이션

정책적 의의:

향후 공공·민간 기관이 부동산 지수 파생상품을 설계하거나
시장 리스크 관리 체계를 구축할 수 있는 기초모델을 제공합니다.

Ⅲ. 시스템 구성
구분	주요 기능	설명
1. 지수(Index Layer)	일·시간 단위 지수 산출	서울 아파트 가격의 단기 변동을 추정하는 나우캐스트 지수 산출 및 표준오차 공개
2. 오라클(Oracle Layer)	데이터 집계 및 정제	다중 데이터 소스의 미디안·윈저라이즈·TWAP 계산 및 커밋·리빌(Commit-Reveal) 로그 생성
3. 거래소(Exchange Layer)	파생 정산 및 모의거래	지수와 오라클 가격 간 괴리를 바탕으로 펀딩비 산출, 무기한 선물(PERP) 시뮬레이션 수행

Ⅳ. 기술적 특징

지수 산출 방식 (Index Calculation)

헤도닉 회귀모형을 기초로 면적·연식·입지 등을 통제한 후
이동평균 또는 칼만필터(Kalman Filter)를 적용하여 일간 지수를 추정합니다.
월간 공표지수(KB·부동산원 등)에 앵커링(Anchoring) 하여 장기 드리프트를 제어합니다.

오라클 파이프라인 (Oracle Aggregation)

다중 출처의 가격신호를 수집하여 미디안 계산 후,
이상치 제거를 위한 윈저라이즈(Winsorization) 및 TWAP(Time Weighted Average Price) 을 수행합니다.
커밋·리빌(Commit–Reveal) 로그를 통해 데이터 조작 방지 및 검증 가능성을 확보합니다.

파생상품 정산 로직 (Exchange Simulation)

오라클의 TWAP 가격을 마크가격으로 활용하며,
funding_rate = clamp(k * (mark_price - index_value) / index_value)
수식을 통해 롱·숏 간 펀딩 정산을 시뮬레이션합니다.

Ⅴ. 데이터 출력 구조
파일명	주요 내용	설명
index_timeseries.csv	timestamp, index_value, stderr	일간 지수 및 표준오차
oracle_feed.csv	timestamp, median, winsor, twap	오라클 집계 결과
funding_log.csv	timestamp, mark_price, index_value, basis, funding_rate	펀딩비 정산 기록

Ⅵ. 사용 방법
# 가상환경 생성 및 패키지 설치
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# 1. 지수 산출
python index/nowcast.py

# 2. 오라클 집계
python oracle/aggregator.py

# 3. 펀딩비 계산
python exchange/funding.py

# 4. 테스트 수행
pytest -q

Ⅶ. 개발 원칙

재현성(Reproducibility)

동일한 환경에서 항상 동일한 결과가 재현되도록 데이터·코드를 구조화합니다.

투명성(Transparency)

오라클 커밋·리빌 로그를 공개하여 산출 과정의 신뢰성을 확보합니다.

모듈성(Modularity)

각 레이어(Index·Oracle·Exchange)는 독립적으로 실행 및 검증이 가능합니다.

단계적 확장성(Scalability)

초기에는 더미데이터 기반으로 시작하되, 실거래·실시간 데이터로 확장 가능하도록 설계합니다.

Ⅷ. 개발 로드맵
단계	주요 목표	산출물
S0	리포지토리 구조 및 더미 파이프라인 구축	3개 CSV 파일
S1	오라클 로직(미디안·윈저·TWAP) 및 펀딩비 계산	펀딩 로그 산출
S2	헤도닉·칼만 기반 지수 산출 및 월간 앵커링	고도화된 지수 시계열
S3	커밋·리빌 자동화(GitHub Actions) 및 데모 노트북	로그·시각화 산출
S4	리팩토링·테스트 고도화 및 공개 문서 정리	최종 버전

Ⅸ. 라이선스 및 데이터 정책

소프트웨어 라이선스: MIT License

데이터 정책:

저장소에는 데모용 샘플 데이터만 포함됩니다.

실제 원천 데이터(실거래·매물 등)는 별도 저장소에서 관리해야 합니다.

Ⅹ. 기대 효과 및 향후 활용

시장 투명성 제고:
데이터 기반 지수형 상품 구조를 제시함으로써
부동산 시장의 가격발견 메커니즘을 고도화합니다.

정책·연구 활용:
공공기관 및 연구기관이 시장 리스크 관리, 자산지수 개발,
데이터 기반 정책 평가 등에 본 구조를 활용할 수 있습니다.

기술적 확장성:
추후 블록체인·AI 기반 지수형 파생상품(온체인 오라클, AI 예측지수 등)으로의 확장 가능성을 검증합니다.

📎 문의 및 기여

문의: 프로젝트 담당자 (예: contact@apt.dev)

기여: GitHub Issues 또는 Pull Request를 통해 개선 제안 가능


apt-/
│
├── README.md                     # 프로젝트 개요 (대표 문서)
├── LICENSE                       # MIT License
├── .gitignore                    # 불필요 파일 제외
├── requirements.txt              # 기본 의존성 (또는 pyproject.toml)
│
├── data/                         # 데이터 관련
│   ├── raw/                      # 원천 (비공개, .gitignore 처리)
│   ├── processed/                # 전처리된 공개용 (지수 계산용)
│   │   ├── seoul_apartment_prices.csv
│   │   ├── anchor_reference.csv
│   │   └── macro_indicators.csv
│   └── outputs/                  # 결과물 (지수 시계열 등)
│       └── index_timeseries.csv
│
├── src/                          # 핵심 코드 (모듈화)
│   └── apt/
│       ├── __init__.py
│       ├── data_io.py            # 데이터 입출력 (load_prices 등)
│       ├── index_calculator.py   # 앵커링 기반 지수 계산
│       ├── oracle_pipeline.py    # 금리/CPI 등 매크로 보정
│       ├── futures_simulation.py # 파생상품 시뮬레이션 (v0.2+)
│       ├── orderbook_engine.py   # 호가창 매칭 엔진 (v0.3)
│       ├── visualization.py      # 시각화 모듈 (v0.2)
│       └── utils.py              # 공용 함수, 상수 등
│
├── notebooks/                    # 분석/검증용 노트북
│   ├── 00_make_mock_data.ipynb   # 더미데이터 생성
│   ├── 01_index_validation.ipynb # 지수 검증
│   └── 02_macro_sensitivity.ipynb
│
├── tests/                        # 자동 테스트
│   ├── test_index_calculator.py
│   ├── test_data_io.py
│   └── test_oracle_pipeline.py
│
├── docs/                         # 프로젝트 문서화 (핵심)
│   ├── 00-overview.md            # 전체 개요
│   ├── 01-index/
│   │   ├── anchoring-method.md   # 앵커링 수식/원리
│   │   ├── data-spec.md          # 데이터 스펙/출처
│   │   └── validation.md         # 검증/백테스트
│   ├── 02-derivatives/
│   │   ├── futures-design.md     # 선물 설계
│   │   └── risk-metrics.md
│   ├── 03-systems/
│   │   ├── data-pipeline.md      # ETL 구조
│   │   └── oracle.md
│   ├── 04-research/
│   │   ├── adr/ADR-0001-anchoring-as-core-index.md
│   │   └── experiments/
│   ├── 05-governance/
│   │   ├── roadmap.md            # 버전 로드맵
│   │   ├── changelog.md
│   │   ├── contributing.md
│   │   └── data-policy.md
│   ├── 06-guides/
│   │   ├── quickstart.md
│   │   └── dev-setup.md
│   └── glossary.md
│
├── .github/                      # 깃허브 자동화
│   ├── workflows/
│   │   └── ci.yaml               # CI 자동 테스트
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   └── pull_request_template.md
│
└── scripts/                      # 유틸리티 (데이터/백테스트)
    ├── make_mock_data.py
    └── run_index_pipeline.py

