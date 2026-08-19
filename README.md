# nvidia-blog-digest

[NVIDIA 개발자 블로그](https://developer.nvidia.com/blog)의 최신 글을 매일 자동 수집해 Claude Code로 한국어 요약을 생성하고 GitHub Pages로 게시하는 도구.

**결과물 보기**: https://jwonmoon.github.io/nvidia-blog-digest/

## 동작 방식

### 매일 다이제스트 (`digest.yml`, 매일 06:00 KST)

1. `scripts/fetch_posts.py` — RSS 피드에서 새 글 목록 수집 (`state/seen.json`과 대조해 중복 제거)
2. `scripts/fetch_article.py` — 원문 페이지에서 본문 추출 (trafilatura, 이미지 포함 마크다운)
3. `scripts/run_digest.py` — 글마다 `claude -p`로 한국어 요약 생성 (형식: `prompts/summarize.md`)
4. `scripts/build_digest.py` — `digests/YYYY-MM-DD.md` 생성, `index.md` 갱신
5. 커밋 & 푸시 → GitHub Pages 자동 반영
6. 새 요약이 있는 날만 다이제스트 전문을 이메일로 발송

실패한 글은 `state/seen.json`에 기록되지 않아 다음 실행 때 자동 재시도된다.

### 주간 하이라이트 (`weekly.yml`, 매주 월요일 06:30 KST)

`scripts/build_weekly.py` — 지난 7일치 다이제스트를 Claude가 읽고 중요 글 3~5개를 선정해 `weekly/YYYY-MM-DD.md` 생성.

### 메일 테스트 (`test-email.yml`, 수동 실행 전용)

메일 설정 확인용. Actions 탭에서 수동 실행하면 테스트 메일 1통 발송.

## 설정

### 필요한 repo secrets

| 이름 | 값 | 용도 |
|---|---|---|
| `CLAUDE_CODE_OAUTH_TOKEN` | `claude setup-token`으로 발급한 토큰 | CI에서 Claude 요약 실행 |
| `MAIL_USERNAME` | Gmail 주소 | 메일 발신·수신 주소 |
| `MAIL_APP_PASSWORD` | [Google 앱 비밀번호](https://myaccount.google.com/apppasswords) 16자리 | Gmail SMTP 인증 (2단계 인증 필요) |

등록 위치: Settings → Secrets and variables → Actions.

### 카테고리 필터 (선택)

기본은 전체 글 수집. 특정 분야만 수집하려면 `.github/workflows/digest.yml`의 `DIGEST_CATEGORIES` 주석을 해제하고 수정 (쉼표 구분, 부분 일치):

```yaml
DIGEST_CATEGORIES: "Generative AI, Robotics, CUDA"
```

피드에서 관측되는 분류 예: `Agentic AI / Generative AI`, `Robotics`, `Simulation / Modeling / Design`, `Data Science`, `CUDA`, `Quantum Computing`, `Physical AI` 등.

### GitHub Pages

Settings → Pages → Source: `main` 브랜치 루트. `_config.yml`(Jekyll Cayman 테마)로 렌더링.

## 로컬 실행

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/python scripts/run_digest.py            # 전체 실행
DIGEST_MAX_POSTS=1 .venv/bin/python scripts/run_digest.py  # 글 1건만 테스트
.venv/bin/python scripts/build_weekly.py          # 주간 하이라이트 생성
```

## 문제 해결

- **Actions 실패 시**: 실패 run 로그에서 "Verify Claude auth" 단계 확인. `401` 또는 `Not logged in`이면 토큰 문제 — 로그의 `CLAUDE_CODE_OAUTH_TOKEN:` 줄이 비어 있으면 secret이 빈 값으로 저장된 것. `claude setup-token`으로 재발급 후 secret 갱신.
- **메일이 안 옴**: 새 요약이 생성된 날만 발송됨. 설정 자체 확인은 `test-email.yml` 수동 실행.
- **같은 글이 다시 처리됨**: 정상 — 이전 실행에서 요약 실패한 글은 재시도됨.
