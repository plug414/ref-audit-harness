---
name: doi-verification
description: "DOI 기반 학술 참고문헌 정합성 검증 스킬. Crossref API로 메타데이터를 대조하고 DOI landing을 확인하여 상태를 판정한다. DOI 검증, Crossref 확인, 참고문헌 DOI 체크 요청 시 사용."
---

# DOI 검증 스킬

DOI가 존재하는 참고문헌에 대한 1차 정합성 검증 절차.

## 검증 플로우

```
각 참고문헌에 대해:
  ├─ DOI 있음?
  │   ├─ Yes → [Step 1] DOI Resolve
  │   │         ├─ 200/302 → [Step 2] Crossref 메타 대조
  │   │         │             ├─ 정합 → CONFIRMED
  │   │         │             └─ 불일치 → CITATION_ISSUE (구체적 불일치 항목 기록)
  │   │         └─ 404/403 → DOI_RESOLVE_FAIL → [Step 2] Crossref만으로 판정
  │   └─ No → [Step 3] Crossref 제목 검색으로 DOI 탐색
  │            ├─ 발견 → DOI 채워서 Step 1로
  │            └─ 미발견 → needs_further_check
  └─ 피어리뷰 여부 판정 [Step 4]
```

## Step 1: DOI Resolve
```
WebFetch: https://doi.org/{doi}
```
- redirect 경로 확인 (302→200이 정상)
- 최종 landing 페이지의 메타데이터 스캔

## Step 2: Crossref 메타데이터 대조
```
WebFetch: https://api.crossref.org/works/{doi}
```

대조 항목:
| 인용 필드 | Crossref 응답 경로 | 허용 오차 |
|----------|-------------------|----------|
| title | message.title[0] | 단어 유사도 0.8+ |
| author | message.author[].family | 이니셜 수준 일치 |
| year | message.published-print.date-parts[0][0] 또는 message.created.date-parts[0][0] | ±1년 |
| journal | message.container-title[0] | 약어/풀네임 허용 |
| volume | message.volume | 정확 일치 |
| issue | message.issue | 정확 일치 |
| pages | message.page | 형식 차이 허용 (123-45 vs 123-145) |

### 제목 유사도 계산
단어 단위 Jaccard 유사도: `|A ∩ B| / |A ∪ B|`
- stop words (the, a, of, in, on, for, and, or) 제거 후 비교
- 0.8 이상 → 일치
- 0.5~0.8 → 부분 일치 (CITATION_ISSUE + 상세 기록)
- 0.5 미만 → DOI_MISMATCH (심각)

## Step 3: Crossref 제목 검색 (DOI 없는 항목)
```
WebFetch: https://api.crossref.org/works?query.title={title}&rows=3
```
- 상위 3건의 title을 인용 제목과 비교
- 유사도 0.8+ 항목의 DOI를 채택하여 Step 1 진행
- 매칭 실패 → `needs_further_check` 분류

## Step 4: 피어리뷰 여부
| 조건 | 판정 |
|------|------|
| Crossref type: "journal-article" | peer-reviewed |
| Crossref type: "proceedings-article" | peer-reviewed |
| arXiv/SSRN/bioRxiv DOI | preprint |
| Crossref type: "book-chapter" | peer-reviewed |
| 기타/불명 | unknown |

## 로그 템플릿
```
① DOI 추출 → ② Crossref /works/{doi} 조회 → ③ title sim {값} → ④ {정합/불일치 항목}
① DOI resolve(302→200) → ② landing 메타 확인 → ③ 권호/연도 정합
① 제목 기반 Crossref 검색 → ② 상위 매칭 sim {값} → ③ {판정}
① DOI resolve 실패(404) → ② Crossref 메타 없음 → ③ needs_further_check
```

## 출력
`_workspace/02_doi_results.json` — 스키마는 doi-verifier 에이전트 정의 참조.
