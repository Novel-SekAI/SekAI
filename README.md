# 세계관·AI (SekAI)

> **소설 세계관 보조 AI** — 작가가 소설 기획 브리프를 입력하면 AI가 인물·관계·세력·법칙·연표로 구조화된 세계관을 생성하고, 작가가 이를 시각적으로 탐색·편집할 수 있게 하는 웹 서비스.

이 저장소는 SekAI의 **부모(모노) 저장소**로, 모든 팀 레포를 Git 서브모듈로 묶어 관리합니다.

---

## 제품 소개

작가가 기획 브리프 한 편을 제출하면 AI가 세계관을 다섯 가지 뷰로 생성합니다.

| 뷰 | 설명 | 생성 주체 |
|----|------|-----------|
| 인물 관계도 | 시점(Era)별 인물 간 관계 그래프 | AI |
| 세력 구도 | 시점별 세력 간 역학 시각화 | AI |
| 법칙표 | 세계 규칙(마나 체계 등) 백과사전 | AI |
| 타임라인 | 사건 연대기 | AI |
| 티어표 | 인물·능력·종족의 영향력 정리 | 사용자 |

> "기획만 있으면, 세계관은 AI가 세운다. 그리고 그 세계는 당신이 손볼 수 있는 지도(map)로 남는다."

자세한 내용은 [PRD](./docs/01-기획/PRD.md)를 참고하세요.

---

## 저장소 구성 (서브모듈)

| 디렉터리 | 저장소 | 역할 | 포트 / 노드 |
|----------|--------|------|-------------|
| [`frontend/`](https://github.com/Novel-SekAI/frontend) | Novel-SekAI/frontend | 웹 클라이언트 — 세계관 생성·편집 UI (Vite) | 80 (nginx) / CPU |
| [`backend/`](https://github.com/Novel-SekAI/backend) | Novel-SekAI/backend | REST API 오케스트레이터 — 인증·프로젝트·세계관 리소스 (Spring Boot) | 8080 / CPU |
| [`ai/`](https://github.com/Novel-SekAI/ai) | Novel-SekAI/ai | AI 추론 서버 — 브리프 → 구조화 세계관 JSON (vLLM/TGI self-host) | 8000 / **GPU** |
| [`design/`](https://github.com/Novel-SekAI/design) | Novel-SekAI/design | 디자인 시스템 — 토큰·컴포넌트·UI 킷의 단일 출처 | — |
| [`docs/`](https://github.com/Novel-SekAI/docs) | Novel-SekAI/docs | 기획·설계·컨벤션 문서 | — |

### 시스템 구성

```
[Frontend] ──REST──▶ [Backend] ──동기 호출──▶ [AI 추론 서버]
  Vite/nginx           Spring Boot                vLLM (GPU)
                       ├─ PostgreSQL 16 (영속화)
                       └─ Redis 7 (세션 저장소)
```

- 인증은 쿠키 세션 기반(Redis), 생성 요청은 BE가 AI 서버에 위임 후 응답을 검증·영속화합니다(동기, 스트리밍 미사용).
- AI 서버는 stateless — DB·인증·클라이언트를 알지 못하고 계약의 요청/응답만 처리합니다.

---

## 시작하기

### 클론 (서브모듈 포함)

```bash
git clone --recurse-submodules https://github.com/Novel-SekAI/SekAI.git
```

이미 일반 클론을 받았다면:

```bash
git submodule update --init --recursive
```

### 서브모듈 최신화

```bash
# 각 서브모듈을 원격 기본 브랜치 최신으로 갱신
git submodule update --remote --merge

# 부모 레포에 갱신된 포인터 커밋
git add <서브모듈 경로>
git commit -m "chore: 서브모듈 포인터 갱신"
```

각 서비스의 실행 방법은 해당 서브모듈의 README와 [`docs/03-컨벤션/CONTAINER.md`](./docs/03-컨벤션/CONTAINER.md)를 참고하세요.

---

## 문서 가이드

문서는 [`docs/`](./docs)에 목적별로 정리되어 있습니다. 폴더명 앞 숫자는 읽는 순서입니다.

| 폴더 | 내용 | 주요 문서 |
|------|------|-----------|
| [01-기획](./docs/01-기획) | 무엇을 왜, 언제까지 만드는가 | [PRD](./docs/01-기획/PRD.md) · [WBS](./docs/01-기획/WBS.md) · [GANTT](./docs/01-기획/GANTT.md) |
| [02-설계](./docs/02-설계) | 팀 간 인터페이스·데이터 계약 | [API](./docs/02-설계/API.md) · [AI-BE-CONTRACT](./docs/02-설계/AI-BE-CONTRACT.md) · [DATA-MODEL](./docs/02-설계/DATA-MODEL.md) · [BE-ARCHITECTURE](./docs/02-설계/BE-ARCHITECTURE.md) |
| [03-컨벤션](./docs/03-컨벤션) | 협업 규칙·자동화 | [BRANCH-STRATEGY](./docs/03-컨벤션/BRANCH-STRATEGY.md) · [COMMIT-CONVENTION](./docs/03-컨벤션/COMMIT-CONVENTION.md) · [CICD](./docs/03-컨벤션/CICD.md) · [ENVIRONMENT](./docs/03-컨벤션/ENVIRONMENT.md) · [CONTAINER](./docs/03-컨벤션/CONTAINER.md) |

---

## 개발 규칙 요약

- **브랜치 전략**: GitHub Flow — 상세는 [BRANCH-STRATEGY.md](./docs/03-컨벤션/BRANCH-STRATEGY.md)
- **커밋 컨벤션**: [COMMIT-CONVENTION.md](./docs/03-컨벤션/COMMIT-CONVENTION.md)
- **시크릿 관리**: 프론트엔드 `VITE_*` 변수는 번들에 포함되므로 시크릿 금지 — [ENVIRONMENT.md](./docs/03-컨벤션/ENVIRONMENT.md)
