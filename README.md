# RUNuts Dashboard

> **라이브**: https://kapehorn-dev.github.io/runuts-dashboard
>
> ⚠️ **2026-04-18 기준 보안 재설계 진행 중** — 곧 회사 PC 자체 호스팅 + Google Workspace OAuth 인증으로 전환 예정. URL 변경 가능성 있음.

RUNuts VR 게임의 데이터 분석 대시보드입니다. 마케팅 데이터(Discord/SNS) + 게임 내 행동 데이터(BigQuery)를 통합해 시각화하고, 플레이테스트 분석 보고서를 제공합니다.

---

## 현재 구조

```
runuts-dashboard/
├── index.html          ← 대시보드 메인 (단일 HTML + Chart.js)
├── reports/            ← 플레이테스트 분석 보고서 (독립 HTML)
│   ├── reports.json    ← 보고서 메타데이터
│   ├── PT08_2026-03.html
│   └── PT09_2026-04.html
└── README.md
```

이 레포는 GitHub Pages 호스팅용 사본입니다. **직접 수정하지 않으며**, [runuts-metrics](https://github.com/kapehorn-dev/runuts-metrics) 레포의 `dashboard/` 폴더에서 작업한 결과물을 복사합니다.

---

## 대시보드 기능

### 메인 화면 (`index.html`)

- **개요 / 디스코드 / 플레이테스트 / 리텐션 / 유저 / SNS / 보고서 / 정보 / 설정** 9개 탭
- 사이드바 네비게이션
- KST/UTC 시간대 토글
- LSD/RUNuts 유저 그룹 필터
- 다크/라이트 테마

### 보고서 시스템 (`reports/`)

플레이테스트 단위 분석 보고서. 각 보고서는 **독립 HTML 파일**로 작동합니다 (Chart.js CDN + 정적 데이터).

**현재 게시된 보고서:**
- **PT09 (9차 Playtest)** — 2026-04-05~06 KST
  - EDA 형태 (10섹션, 판정 라벨 없이 관찰/검토사항/유보 톤)
  - 2개/3개 플테 비교 칩 토글
  - 유저 3분할 분석 (리텐션 / 복귀 / 신규)
  - Golden Nuts 경제 + 퀘스트/도전과제/테크트리
  - Discord × 인게임 크로스 플랫폼 분석
- **PT08 (8차 Playtest)** — 2026-03-21~23

---

## 데이터 흐름

```
[Discord 서버] → [Discord Bot, 회사 PC Docker]
                                  ↓
[Unity Analytics] → [BigQuery]    ↓
                       ↓          ↓
          [sync_game_data.py]   [sheets_client]
                       ↓          ↓
                 [Google Sheets (16개 + Game User Summary 4개 탭)]
                       ↓
                 [대시보드 (이 레포)]
```

**데이터 소스 (현재):**
- Google Sheets "웹에 게시" CSV — 클라이언트에서 직접 fetch
- BigQuery — 직접 접근 X (sync_game_data.py가 시트로 옮김)

**보안 재설계 후 (예정):**
- 회사 PC Nginx + OAuth2 Proxy + Sheets API Flask
- Google Workspace(@kapehorn.io) 로그인 필수
- Google Sheets 공개 게시 해제

---

## 배포 방법 (현재)

```bash
# 1. runuts-metrics 레포에서 수정
cd ~/Documents/Projects/RUNuts\ Project\ Managing/Cowork_Outputs/Marketing_Metrics_System
# 작업 후 commit + push
git add dashboard/ && git commit -m "메시지" && git push origin main

# 2. 이 레포로 복사
cp dashboard/index.html ~/runuts-dashboard/
cp -r dashboard/reports/ ~/runuts-dashboard/reports/
cd ~/runuts-dashboard
git add . && git commit -m "메시지" && git push
```

push 후 1~2분 내 GitHub Pages 자동 반영.

---

## 관련 레포

| 레포 | 용도 |
|---|---|
| [runuts-metrics](https://github.com/kapehorn-dev/runuts-metrics) | Discord Bot + 자동화 스크립트 + 대시보드 소스 |
| **runuts-dashboard** (이 레포) | GitHub Pages 호스팅용 사본 |

---

## 기술 스택

- HTML/CSS/JavaScript (vanilla)
- [Chart.js 4.4.1](https://www.chartjs.org/) (CDN)
- [chartjs-chart-matrix](https://github.com/kurkle/chartjs-chart-matrix) — 코호트 히트맵
- 폰트: [Pretendard](https://github.com/orioncactus/pretendard) (CDN)

---

## 문의

Jack (jack.lim@kapehorn.io)
