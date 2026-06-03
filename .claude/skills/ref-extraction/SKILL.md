---
name: ref-extraction
description: "논문 원고(PDF/HWP/DOCX)에서 참고문헌 목록을 구조적으로 추출하고, DOI·언어·문헌유형을 사전 분류한다. 논문 레퍼런스 추출, 참고문헌 파싱, bibliography parsing, 레퍼런스 리스트 정리 요청 시 사용."
---

# 레퍼런스 추출 스킬

논문 원고에서 참고문헌 섹션을 식별·파싱하여 구조화된 JSON으로 출력한다.

## 추출 워크플로우

### Step 1: 파일 읽기
- PDF: Read 도구로 직접 읽기
- HWP: Read 도구로 시도. 실패 시 사용자에게 텍스트 복사 요청
- DOCX: Read 도구로 직접 읽기
- 텍스트 직접 입력: 그대로 사용

### Step 2: 참고문헌 섹션 식별
- "참고문헌", "References", "Bibliography", "인용문헌" 등의 헤딩을 찾는다
- 헤딩이 없으면 번호+저자 패턴(`[1]`, `1.`, `1)` 등)으로 시작점 추정

### Step 3: 항목별 파싱
각 항목에서 추출할 필드:
- **id**: 원문 번호 (재번호 금지)
- **raw_text**: 원문 그대로
- **authors**: 저자 (Last, F. 또는 성이름 형태)
- **year**: 발행 연도 (4자리 숫자 탐지)
- **title**: 논문/책 제목
- **journal**: 저널명 또는 출판사
- **volume/issue/pages**: 권호/페이지
- **doi**: DOI (`10.xxxx/` 패턴 탐지)

### Step 4: 언어·유형 자동 분류
| 분류 | 기준 |
|------|------|
| 언어: ko | 제목에 한글 포함 |
| 언어: zh | 제목에 中文 포함 |
| 언어: en | 위 해당 없음 |
| 유형: journal | 저널명+권호 존재 |
| 유형: book | 출판사명+ISBN 또는 단행본 패턴 |
| 유형: thesis | 학위논문 키워드 (석사/박사/dissertation) |
| 유형: conference | proceedings/학회 키워드 |
| 유형: web | URL만 존재 |

### Step 5: 요약 통계 생성
- DOI 보유/미보유 건수
- 언어별 건수
- 유형별 건수

## 출력
`_workspace/01_references.json` — 스키마는 ref-extractor 에이전트 정의 참조.

## 주의사항
- 원문 번호 체계를 절대 변경하지 않는다
- 파싱 불확실 항목은 `parse_uncertain: true`로 표시하고 raw_text 보존
- DOI가 본문에 없어도 `https://doi.org/` 링크가 있으면 추출
