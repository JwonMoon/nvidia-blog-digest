# nvidia-blog-digest

NVIDIA 개발자 블로그(https://developer.nvidia.com/blog)의 최신 글을 매일 자동 수집해 Claude Code로 한국어 요약을 생성하고 GitHub Pages로 게시하는 도구.

## 동작 방식

1. GitHub Actions가 매일 06:00 KST에 실행 (`.github/workflows/digest.yml`)
2. `scripts/fetch_posts.py` — RSS 피드에서 새 글 목록 수집 (`state/seen.json`과 대조)
3. `scripts/fetch_article.py` — 본문 텍스트 추출 (trafilatura)
4. `scripts/run_digest.py` — 글마다 `claude -p`로 한국어 요약 생성
5. `scripts/build_digest.py` — `digests/YYYY-MM-DD.md` 생성, `index.md` 갱신
6. 변경 사항 커밋 & 푸시 → GitHub Pages 자동 반영

## 최초 설정

1. GitHub에 저장소 생성 후 푸시
2. 로컬에서 `claude setup-token` 실행 → 발급된 토큰을 repo secret `CLAUDE_CODE_OAUTH_TOKEN`으로 등록
3. Settings → Pages → Source: `main` 브랜치 루트
4. Actions 탭에서 `Daily NVIDIA blog digest` 워크플로를 `workflow_dispatch`로 1회 수동 실행해 초기 콘텐츠 생성

## 로컬 실행

```bash
pip install -r requirements.txt
python scripts/run_digest.py
```

실패한 글은 `state/seen.json`에 기록되지 않아 다음 실행 때 자동 재시도된다.
