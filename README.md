🏙️ apt- — 서울 아파트 지수 파생상품 모의거래소 (MVP)

서울 아파트 가격지수를 기초자산으로 한 최초의 지수형 파생상품 실험
오프체인 데이터 → 오라클 → 모의거래소까지, 완전 재현 가능한 금융 데이터 인프라 데모

🚀 프로젝트 개요

**apt-**는 한국 부동산 시장의 “가격 헷지 수단 부재” 문제를 해결하기 위한
서울 아파트 가격지수 기반 파생상품(MVP) 입니다.

📈 “부동산 시장에도 나스닥 선물처럼, 지수를 거래할 수 있다면?”

이 프로젝트는 3단계 파이프라인으로 구성됩니다 👇

레이어	설명	결과물
🏗️ Index	일/시간 단위 나우캐스트 지수 산출	index_timeseries.csv
🔗 Oracle	다중 소스 → 미디안 → 윈저 → TWAP → 커밋·리빌 로그	oracle_feed.csv
💱 Exchange	지수를 기초로 한 무기한·월물 파생 모의거래	funding_log.csv
💡 왜 필요한가?

🇰🇷 한국 부동산엔 ‘롱/숏 대칭’ 시장이 없다.

📊 가격지수 산출은 불투명하고 시차가 크다.

🔍 데이터·오라클·상품 설계를 결합해 “투명한 지수 시장”을 제시한다.

이 MVP는 실거래가 없는 완전 모의 환경이지만,
“지수를 거래 가능한 상품으로 전환하는” 핵심 구조를 그대로 구현합니다.

🧱 시스템 아키텍처
graph TD
  A[데이터 수집 (raw)] --> B[지수 산출 (Index Layer)]
  B --> C[오라클 집계 (Oracle Layer)]
  C --> D[모의거래/정산 (Exchange Layer)]
  D --> E[출력 CSV (index/oracle/funding)]

⚙️ 빠른 시작 (Quickstart)
# 1️⃣ 환경 설정
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# 2️⃣ 전체 파이프라인 실행
python index/nowcast.py
python oracle/aggregator.py
python exchange/funding.py

# 3️⃣ 결과 확인
ls data/processed/
# index_timeseries.csv / oracle_feed.csv / funding_log.csv

# 4️⃣ 테스트
pytest -q


💾 결과 파일

파일	설명
index_timeseries.csv	일간 서울 아파트 나우캐스트 지수
oracle_feed.csv	오라클 집계(미디안·윈저·TWAP)
funding_log.csv	무기한 선물 펀딩 로그
🧩 핵심 개념 요약
개념	설명
앵커링 (Anchoring)	월간 공식지수로 일간 지수의 기준을 보정해 드리프트를 제어
stderr (표준오차)	지수 추정의 불확실성(±오차범위)
펀딩비 (Funding Rate)	선물과 현물 괴리를 줄이는 정산 메커니즘
Commit–Reveal	해시 커밋 후 원본 공개로 오라클 조작 방지 증명
📂 폴더 구조
apt-
├─ docs/              # 전체 설계 문서
├─ data/              # 원천→중간→산출 데이터
├─ index/             # 지수 계산 코드
├─ oracle/            # 오라클 파이프라인
├─ exchange/          # 거래소 로직(펀딩, 마크가격)
├─ tests/             # 테스트 코드
├─ Makefile           # 실행 단축 명령
└─ README.md          # 현재 문서

🧭 개발 로드맵
단계	내용
S0	리포 구조 / 더미 파이프라인 / 문서 정리
S1	미디안·윈저·TWAP + 펀딩 로직 단위테스트
S2	헤도닉 + 칼만 + 월간 앵커링
S3	커밋·리빌 자동화(GitHub Actions) / 데모 노트북
S4	리팩토링 + README/UI 다듬기
🧠 원칙

🧩 Reproducible — 누구나 클론 후 make run 으로 동일 결과 재현

🕵️ Transparent — 오라클 로그와 커밋·리빌 증명 공개

🧱 Modular — Index·Oracle·Exchange 독립 작동

🌱 Small First — 더미로 출발, 이후 실데이터/모델로 교체

📜 라이선스 & 데이터 정책

License: MIT (자유 사용/수정/배포)

Data: 샘플·모의데이터만 포함 (실데이터는 외부 저장소 관리)

💬 FAQ

❓ 월간 지수를 왜 쓰나요?
일간 예측치는 장기적으로 기준이 떠오를 수 있어,
월 1회 공식지수를 “앵커”로 삼아 기준을 맞춥니다.

❓ 현물 지수가 왜 필요한가요?
선물은 항상 “현물+기대치”로 가격이 형성됩니다.
현물이 있어야 펀딩·차익거래·가격발견이 가능합니다.

❓ 변동성이 너무 낮지 않나요?
월간 지수는 작지만, 일간화 + 레버리지로 충분한 변동폭을 확보합니다.

🤝 기여 (Contributing)

이슈는 명확한 재현경로와 함께 작성해주세요.

PR은 작은 단위로, 테스트·문서 포함을 권장합니다.

아이디어 제안은 /docs/ 내 roadmap.md 에 논의 기록을 남겨주세요.

🏁 비전

한국형 부동산 선물거래소로 나아가는 첫 걸음.
데이터·AI·파생상품이 연결된 새로운 금융 인프라의 시작점입니다.
