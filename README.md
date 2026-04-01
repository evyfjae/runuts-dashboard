# RUNuts Marketing Dashboard

> **라이브**: https://evyfjae.github.io/runuts-dashboard

RUNuts 마케팅 메트릭스 대시보드. GitHub Pages로 호스팅되는 단일 HTML 파일입니다.

## 구조

```
runuts-dashboard/
└── index.html    ← 대시보드 전체 (HTML + CSS + JS)
```

`index.html`은 `runuts-metrics` 레포의 `dashboard/index.html`에서 복사됩니다.
이 레포에서 직접 수정하지 않습니다.

## 배포 방법

```bash
# runuts-metrics에서 수정 후
cp "/path/to/Marketing_Metrics_System/dashboard/index.html" ~/runuts-dashboard/index.html
cd ~/runuts-dashboard
git add index.html && git commit -m "메시지" && git push
```

push 후 GitHub Pages가 자동으로 배포합니다.

## 데이터 소스

대시보드는 Google Sheets "웹에 게시" CSV를 클라이언트에서 직접 읽습니다.
서버 없이 정적 HTML만으로 동작합니다.

## 관련 레포

| 레포 | 용도 |
|---|---|
| [runuts-metrics](https://github.com/evyfjae/runuts-metrics) | 자동화 스크립트 + Bot + 대시보드 소스 코드 |

## 문의

Jack (jack.lim@kapehorn.io)
