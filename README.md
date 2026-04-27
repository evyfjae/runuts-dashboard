# RUNuts Dashboard

> **라이브**: https://kapehorn-dev.github.io/runuts-dashboard
>
> ⚠️ **2026-04-18 기준 보안 재설계 진행 중** — Google Sheets 공개 게시 노출 이슈 발견. 곧 회사 PC 자체 호스팅 + Google Workspace OAuth 인증으로 전환 예정. URL 변경 가능성 있음.

---

## 이 대시보드는 무엇인가요?

RUNuts VR 게임의 데이터 분석 대시보드입니다. 두 종류 데이터를 통합 시각화합니다:

1. **마케팅 데이터** — Discord 서버 성장, YouTube/TikTok 지표, 유저 퍼널, 테스터 파이프라인
2. **인게임 데이터** — Unity Analytics 이벤트, 모드별 체류, 경제(Golden Nuts), 퀘스트/도전과제, 코호트 잔존

또한 플레이테스트별 **상세 분석 보고서**를 제공합니다 (현재 8차/9차 게시).

---

## 구조

```
runuts-dashboard/
├── index.html              ← 대시보드 메인 (단일 HTML + Chart.js)
├── reports/                ← 플레이테스트 분석 보고서 (독립 HTML)
│   ├── reports.json        ← 보고서 메타데이터
│   ├── PT08_2026-03.html   ← 8차 Playtest
│   └── PT09_2026-04.html   ← 9차 Playtest (EDA 포맷, 최신)
└── README.md
```

이 레포는 GitHub Pages 호스팅용 사본입니다. **직접 수정하지 않으며**, [runuts-metrics](https://github.com/kapehorn-dev/runuts-metrics) 레포의 `dashboard/` 폴더에서 작업한 결과를 복사해 배포합니다.

---

## 대시보드 메인 (index.html)

### 9개 탭

| 탭 | 내용 |
|---|---|
| **개요** | KPI 요약, 최근 플테 비교, 멤버 성장 추이 |
| **디스코드** | 가입/이탈/메시지 추이, 유입 경로 분석 |
| **플레이테스트** | 플테별 참여자, 일별 DAU, 세션/이벤트 차트 |
| **리텐션** | 코호트 히트맵, 잔존 커브, 이탈 분석, FTUE 퍼널, 플테 비교 레이더 |
| **유저** | 전체 유저 테이블, 필터(그룹/상태/테스터), 실시간 검색, 상세 패널 |
| **SNS** | YouTube/TikTok/UGC Archive |
| **보고서** | 플테별 분석 보고서 카드 목록 + 뷰어 |
| **정보** | KPI 정의서, 그룹 분류 설명 |
| **설정** | 데이터 URL, 테마, 비밀번호 |

### 글로벌 기능

- **시간대 토글**: KST/UTC (기본값 KST)
- **그룹 토글**: RUNuts만 / 전체(LSD 포함)
- **다크/라이트 테마**: 사용자 선호 저장
- **반응형**: 모바일 / 태블릿 / 와이드 모니터 대응

---

## 보고서 시스템 (reports/)

플레이테스트 단위 분석 보고서. 각 보고서는 **독립 HTML 파일**로 작동합니다 (Chart.js CDN + 정적 데이터, 인쇄/공유 용이).

### 현재 게시된 보고서

**PT09 — 9차 Playtest** (2026-04-05~06 KST)
- **EDA 형태** (10섹션, 판정 라벨 없이 관찰/검토사항/유보 톤)
- **2개/3개 플테 비교 칩 토글**
- **유저 3분할 분석** — 리텐션(28명) / 복귀(30명) / 신규(10명)
- **모드별 체류시간** — 플레이어 단위 합산 (Sky Tower, Circuit, Mountain, Casual, Infection, Horror)
- **Golden Nuts 경제** — 유입(퀘스트/도전과제) / 소비(코스메틱/테크트리) / 잔액 분포
- **퀘스트 / 도전과제 / 테크트리** 상세 달성 현황
- **Discord × 인게임 크로스 플랫폼** — 메시지 활동량과 인게임 체류의 상관

**PT08 — 8차 Playtest** (2026-03-21~23)
- 초기 형태 (참여자, 잔존율, 모드별 이용)

### 보고서 메타데이터

`reports/reports.json` 파일이 보고서 목록을 관리합니다:

```json
[
  {
    "id": "PT09",
    "title": "9차 Playtest 보고서",
    "playtest": "9차 Playtest",
    "period": "2026-04-05 ~ 2026-04-06 KST",
    "date": "2026-04-17",
    "summary": "...",
    "file": "PT09_2026-04.html",
    "status": "published"
  }
]
```

대시보드의 **보고서 탭**이 이 JSON을 읽어 카드 목록을 렌더링합니다.

---

## 데이터 흐름

```
[Discord 서버]
     ↕ (실시간 이벤트)
[Discord Bot — 회사 PC Docker, 24/7]
     ↕ (Google Sheets API, 서비스 계정)
[Google Sheets — 중앙 저장소 (16개 시트)] ←── [GitHub Actions cron 00:00/11:00 KST]
     ↑
[BigQuery (events_with_users VIEW)] ──→ [sync_game_data.py] ──→ [Game User Summary 시트 (4개 탭)]
     ↓
[대시보드 (이 레포, GitHub Pages)]
     ↑ Google Sheets 공개 CSV (현재) → 회사 PC 인증 프록시 (재설계 후)
```

### 현재 데이터 소스

대시보드는 Google Sheets의 **"웹에 게시" 공개 CSV**를 클라이언트에서 직접 fetch합니다.
서버 없이 정적 HTML만으로 동작합니다.

| 시트 | 용도 |
|---|---|
| User Master | 유저 1,170명, Discord/Meta/PlayFab 매칭 |
| Discord Daily | 일별 멤버 변동 480일+ |
| Health Snapshot | 일일 건강도/등급 분포 |
| User Activity Log | 유저별 일일 메시지/보이스 |
| YouTube/TikTok/UGC | SNS 지표 |
| Tester Pipeline | 테스터 신청→승인 |
| KPI Dashboard | 핵심 지표 일별 |
| Funnel Tracker | 월간 퍼널 |
| Game Users | 405명 BigQuery 게임 데이터 |
| Playtest Summary/Daily/Event Breakdown | 플테별 집계 |

### 보안 재설계 후 (예정)

```
[팀원 브라우저]
     ↓ HTTPS
[Tailscale Funnel] *.ts.net
     ↓
[회사 PC Docker]
  ├─ Nginx (auth_request 게이트)
  ├─ OAuth2 Proxy (Google SSO, @kapehorn.io 제한)
  ├─ Sheets API Flask (서비스 계정으로 Sheets API)
  └─ 대시보드 정적 파일
```

**변경 효과:**
- Google Sheets 공개 게시 해제 (URL 노출 불가)
- @kapehorn.io 이메일 보유자만 접근
- 서비스 계정만 데이터 접근, 클라이언트는 인증된 요청만 가능
- 상세: [runuts-metrics 레포](https://github.com/kapehorn-dev/runuts-metrics) 참조

---

## 자동화 구성

### 실행자 3명

| 실행자 | 위치 | 역할 |
|---|---|---|
| **Discord Bot** | 회사 PC(JINNY) Docker | 실시간 이벤트 감지 + 매일 자정 집계 |
| **GitHub Actions** | GitHub 서버 (cron) | 스냅샷 + YouTube/KPI/퍼널/UGC 업데이트 |
| **Jack (수동)** | - | TikTok, Server Insights CSV, YouTube retention |

### 시간순 자동화

| 시간 (KST) | 실행자 | 동작 |
|---|---|---|
| 실시간 | Bot | 가입/이탈/역할변경/메시지 감지 |
| **00:00** | Bot `daily_summary()` | 일일 집계, Health Snapshot, User Activity Log |
| **00:00** | GitHub Actions | 멤버 수 스냅샷 (확인용) |
| **11:00** | GitHub Actions | YouTube 통계, KPI 재계산, UGC 감지, PlayFab 매칭 |
| **매월 1일** | GitHub Actions | 월간 퍼널 집계 |
| 플테 후 | Jack (수동) | `sync_game_data.py` 실행 → BigQuery 게임 데이터 시트 적재 |

---

## 배포 방법

```bash
# 1. runuts-metrics 레포에서 수정
cd ~/Documents/Projects/RUNuts\ Project\ Managing/Cowork_Outputs/Marketing_Metrics_System
# 작업 후 commit + push
git add dashboard/ && git commit -m "메시지" && git push origin main

# 2. 이 레포로 복사 + 배포
cp dashboard/index.html ~/runuts-dashboard/
cp -r dashboard/reports/ ~/runuts-dashboard/reports/
cd ~/runuts-dashboard
git add . && git commit -m "메시지" && git push
```

push 후 1~2분 내 GitHub Pages 자동 반영.

---

## 기술 스택

- **HTML / CSS / JavaScript** (vanilla, 빌드 시스템 없음)
- **[Chart.js 4.4.1](https://www.chartjs.org/)** (CDN) — 모든 차트
- **[chartjs-chart-matrix](https://github.com/kurkle/chartjs-chart-matrix)** — 코호트 히트맵
- **폰트**: [Pretendard](https://github.com/orioncactus/pretendard) Variable (CDN)
- **호스팅**: GitHub Pages (무료, 서버리스)
- **데이터**: Google Sheets Published CSV (현재) → 회사 PC API (예정)

### 의존성 없는 클라이언트

빌드/번들러/Node.js 불필요. HTML 파일을 브라우저로 열기만 하면 동작합니다.

---

## 관련 레포

| 레포 | 용도 |
|---|---|
| [runuts-metrics](https://github.com/kapehorn-dev/runuts-metrics) (Private) | Discord Bot 코드 + 자동화 스크립트 + 대시보드 소스 + 데이터 인벤토리 |
| **runuts-dashboard** (이 레포, Public) | GitHub Pages 호스팅용 사본 |

---

## 비용

| 항목 | 비용 |
|---|---|
| GitHub Pages | $0 |
| Google Sheets | $0 |
| BigQuery | $0 (무료 할당량 내) |
| YouTube Data API | $0 (10,000 unit/일 내) |
| Tailscale | $0 (Personal Free, 6 seats / 무제한 기기) |
| 회사 PC | $0 (전기세만, 회사 부담) |
| **총** | **$0 / 월** |

---

## 문의

Jack (jack.lim@kapehorn.io)
