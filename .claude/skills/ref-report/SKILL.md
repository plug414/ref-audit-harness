---
name: ref-report
description: "레퍼런스 검증 결과를 종합하여 출판 수준의 HTML 보고서를 생성한다. 요약카드 5종 + 상세 검증표 + 검증과정 로그 + 피어리뷰 분석 + 종합 코멘트를 포함. 검증 보고서 작성, 레퍼런스 리포트 생성, HTML 보고서 요청 시 사용."
---

# 레퍼런스 검증 보고서 스킬

테스트리뷰 1~14에서 정립된 표준 HTML 보고서 포맷으로 검증 결과를 렌더링한다.

## HTML 구조

### 기본 스타일
- 폰트: Pretendard, -apple-system, sans-serif
- 배경: #f5f5f7 (Apple-style light grey)
- 카드: #fff, border-radius: 12px, box-shadow
- 테이블: border-collapse, 12px 폰트, hover 효과

### 필수 섹션 (순서대로)

1. **헤더** — 제목 + 메타 정보 (논문명, 레퍼런스 수, 검증일)
2. **요약 카드** — 5칸 그리드 (전체/CONFIRMED/CITATION_ISSUE/FABRICATED/MANUAL_REVIEW)
3. **검증 방법론 안내** — info-box로 검증 경로 설명
4. **문제 요약** (있을 경우) — alert-box/warn-box로 주요 이슈
5. **상세 검증표** — 모든 레퍼런스 항목별 상세
6. **출처 신뢰도 분석** — 프리프린트 비율, 학회 수준, 비학술 출처
7. **종합 코멘트** — 심사 실무 권고
8. **푸터** — 검증일, 도구, 면책

### 상세 검증표 컬럼
| 컬럼 | 내용 | 너비 |
|------|------|------|
| # | 원문 번호 | 40px |
| 레퍼런스 | 원문 + 출처태그 + 유형태그 | auto |
| 판정 | 배지 (pass/issue/fail/grey) | 100px |
| 근거 | 검증 결과 핵심 요약 | auto |
| 검증 과정 | 단계별 로그 (issue-text) | auto |
| 피어리뷰 | peer-reviewed/preprint/unknown | 100px |

### 행 스타일링
| 판정 | 행 클래스 | 배경색 |
|------|----------|--------|
| CONFIRMED | (기본) | #fff |
| CITATION_ISSUE | row-issue | #fffdf5 |
| FABRICATED | row-fail | #fff5f5 |
| MANUAL_REVIEW | row-grey | #f9f9fb |
| UNVERIFIABLE | row-grey | #f9f9fb |

### 배지 스타일
```css
.badge-pass { background: #d1f2d9; color: #1b7a2d; }
.badge-issue { background: #fff3cd; color: #856404; }
.badge-fail { background: #f8d7da; color: #721c24; }
.badge-grey { background: #e5e5ea; color: #48484a; }
```

### 알림 박스
```css
.info-box { background: #eff6ff; border: 1px solid #bfdbfe; }  /* 정보 */
.warn-box { background: #fffbeb; border: 1px solid #fde68a; }  /* 주의 */
.alert-box { background: #fff5f5; border: 1px solid #fed7d7; } /* 경고 */
.critical-box { background: #fef2f2; border: 2px solid #fca5a5; } /* 심각 */
```

## 종합 코멘트 가이드

### 포함할 분석
1. **프리프린트 비율**: X/Y건 (Z%) — 30% 초과 시 warn-box
2. **비학술 출처**: 뉴스, 블로그, 보고서 등
3. **수정 필요 항목**: CITATION_ISSUE 건 목록 + 구체적 수정 사항
4. **AI 생성 의심 패턴** (해당 시):
   - 유사 DOI 오기 (끝자리만 다름)
   - 제목-DOI 매핑 오류
   - 저자/메타 체계적 불일치
5. **심사 실무 권고**: 수정 후 게재/수정 후 재심/게재불가 수준의 참고 의견

### 심사 권고 문구 예시
- 경미: "참고문헌 목록에 경미한 서지 오류가 확인됩니다. 수정 후 게재가 적합합니다."
- 중간: "참고문헌 목록에서 복수의 서지 오류와 프리프린트 과다 인용이 확인됩니다. 전수 정정 후 재검토가 필요합니다."
- 심각: "참고문헌 목록에서 체계적 오류 패턴(유사 DOI 오기, 제목-DOI 매핑 오류 등)이 확인됩니다. 이는 생성형 AI 보조 작성 과정에서 관찰되는 양상과 유사합니다. 전 참고문헌의 전수 정정 및 원문 근거 링크 제출을 요구합니다."

## 품질 체크 (생성 후 자동 수행)
1. 카드 숫자 합산 = 전체 건수
2. 표 행 수 = 전체 건수
3. 판정별 건수가 카드와 일치
4. 모든 항목에 피어리뷰 여부 기재
5. 검증 로그 누락 없음
6. HTML 문법 오류 없음 (태그 닫힘 확인)

## 출력
파일명: `{YYYY-MM-DD}_테스트리뷰{N}_레퍼런스검증.html`
저장 경로: 프로젝트 루트 (현재 작업 디렉토리)

상세 HTML 템플릿은 `references/html-template.md` 참조 (필요 시).
