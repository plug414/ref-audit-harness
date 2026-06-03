# ref-audit-harness

논문 참고문헌(레퍼런스)의 진위여부를 자동 검증하고 HTML 보고서를 생성하는 Claude Code 하네스.

## 구조

```
.claude/
├── agents/                    # 전문 에이전트 5개
│   ├── ref-extractor.md       # 참고문헌 추출·구조화
│   ├── doi-verifier.md        # DOI/Crossref 1차 검증
│   ├── domestic-verifier.md   # 국내문헌 RISS 5단계 검증
│   ├── intl-verifier.md       # 영문/중문 문헌 검증
│   └── ref-report-writer.md   # HTML 보고서 생성
├── skills/                    # 스킬 6개
│   ├── ref-audit/             # 오케스트레이터 (진입점)
│   ├── ref-extraction/        # 추출 절차
│   ├── doi-verification/      # DOI 검증 절차
│   ├── domestic-verification/ # 국내문헌 검증 절차
│   ├── intl-verification/     # 국제문헌 검증 절차
│   └── ref-report/            # 보고서 생성 절차
└── settings.json
```

## 워크플로우

```
논문 파일 (PDF/HWP/DOCX)
    ↓
[ref-extractor] → 참고문헌 추출·구조화
    ↓
[doi-verifier] → DOI/Crossref 1차 검증
    ↓
    ├── 국문 → [domestic-verifier] RISS 검증
    └── 영문/중문 → [intl-verifier] 출판사/Scholar 검증
                ↓
[ref-report-writer] → HTML 보고서
```

## 사용법

1. 이 레포를 프로젝트 디렉토리에 clone (또는 `.claude/` 폴더를 복사)
2. Claude Code에서 논문 파일을 제공하고 "레퍼런스 검증해줘" 요청

## 검증 방법

| 에이전트 | 검증 경로 | 대상 |
|---------|----------|------|
| doi-verifier | Crossref API + DOI landing | DOI 보유 문헌 |
| domestic-verifier | RISS 5단계 (영문키워드→한글키워드→저널조합→KCI목차→인간위임) | 국내 학술문헌 |
| intl-verifier | 출판사 landing / Google Scholar / 参考网 / CNKI 간접 | 영문·중문 문헌 |

## 판정 코드

| 코드 | 의미 |
|------|------|
| CONFIRMED | 문헌 존재 + 서지 정합 |
| CITATION_ISSUE | 문헌 존재하나 서지 불일치 |
| FABRICATED | 부존재 확정 (인간 확인 후에만) |
| MANUAL_REVIEW | 자동 검증 불가 → 수동 확인 필요 |
| UNVERIFIABLE | 접근 제한 (CNKI 캡차 등) |

## 요구사항

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [oh-my-claudecode (OMC)](https://github.com/Yeachan-Heo/oh-my-claudecode) 권장

## 배경

14회의 테스트리뷰 경험에서 축적된 검증 방법론을 체계화한 하네스입니다.
