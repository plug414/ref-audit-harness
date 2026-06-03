---
name: domestic-verification
description: "RISS 5단계 방법론으로 국내(한국어) 학술문헌을 검증하는 스킬. 영문 핵심단어 분리 → 한글 키워드 → 저널명 조합 → KCI 목차대조 → 인간위임 순서로 진행한다. 국내문헌 검증, RISS 검색, 한국 논문 확인 요청 시 사용."
---

# 국내문헌 검증 스킬 (RISS 5단계)

테스트리뷰 1~14의 시행착오에서 정제된 국내 학술문헌 검증 방법론.

## 검색 플로우

```
인용 영문 제목 확인
       ↓
[1순위] 영문 핵심 단어 3~5개 → RISS 검색 (저널명 제외)    ← 1회, 저비용
       ↓ 발견? → 상세 확인 → CONFIRMED/CITATION_ISSUE
       ↓ 미발견?
[2순위] 한글 주제 키워드 → RISS 검색 (저널명 제외)         ← 1회, 저비용
       ↓ 발견? → 상세 확인
       ↓ 미발견?
[3순위] 한글 키워드 + 저널명/학교명 → RISS 검색            ← 1회, 저비용
       ↓ 발견? → 상세 확인
       ↓ 미발견?
[4순위] 저널 목차 전수 대조
       ├─ KCI 등재 저널 → KCI landing 자동 확인 (1회)
       ├─ 서버 렌더링 저널 사이트 → 자동 확인 (1회)
       └─ 불가? → ★ 인간에게 위임 (가이드 제공)
           ↓
       발견? → CONFIRMED
       미발견? → FABRICATED 확정 (인간 확인 후에만)
```

## 검색어 구성법

### 예시
```
원본: "The effect of virtual influencer characteristics on cosmetic purchase intention"

Step 1: 관사/전치사/접속사 제거
  → effect virtual influencer characteristics cosmetic purchase intention

Step 2: 일반 학술용어 제거 (effect, study, research, impact, analysis)
  → virtual influencer characteristics cosmetic purchase intention

Step 3: 핵심 명사 3~5개
  → virtual influencer cosmetic purchase intention
```

### 주의
- "study", "research", "effect" 등은 노이즈 — 제거
- 고유명사/전문용어는 유지: TTCT, bigkinds, AIGC, uncanny valley
- 최소 3개, 최대 5개

## RISS 검색 URL
```
https://www.riss.kr/search/Search.do?detailSearch=false&searchGubun=true&query={검색어}
```
WebFetch로 결과 파싱 가능 (서버 사이드 렌더링). 상세 페이지: `https://www.riss.kr/link?id=A숫자` (학술지) 또는 `T숫자` (학위논문).

## 발견 후 서지 대조

RISS 상세 페이지에서 반드시 확인:
1. 저자 이니셜 일치
2. 저널명 일치 (유사 저널 주의: 한국미용학회지 ≠ 대한미용학회지)
3. 권호·연도 일치 (1년 차이 허용)
4. 페이지 범위 일치
5. RISS 영문 제목과 인용 영문 제목의 의미적 일치

## 실패 패턴 & 교훈

| 패턴 | 사례 | 대응 |
|------|------|------|
| 저널명 오류 | B-8: 한국미용학회지→대한미용학회지 | 1차 검색에서 저널명 제외 |
| 번역 용어 차이 | B-10: 사실성≠실제감 | 영문 키워드 우선 |
| 영문 제목 의역 | A-4: commercials→ads | 핵심 명사 분리 |
| 잘못된 저널에서 대조 | B-8: Vol.29에서 찾아 Vol.19 오판 | RISS 키워드 검색으로 실제 저널 먼저 확인 |

## 인간 위임 가이드 템플릿

자동 확인 불가 시 아래 형식으로 가이드를 생성한다:

```
수동확인 요청: [{항목 번호}]
■ 자동 검색 경과:
  - 1순위 RISS 영문 키워드 "{키워드}" → {결과}
  - 2순위 RISS 한글 키워드 "{키워드}" → {결과}
  - 3순위 RISS 한글+저널명 "{키워드}" → {결과}
  - 자동 목차 확인 시도 → {불가 사유}

■ 확인 대상:
  - 저널: {저널명}
  - 권호: {Vol.X No.Y}
  - 연도: {YYYY}
  - 찾는 논문: "{영문 제목}"
  - 저자: {저자}

■ 권장 검색 방법:
  1. {저널 사이트} 또는 DBpia에서 해당 저널 검색
  2. {Vol.X No.Y} 호를 선택하여 목차 열기
  3. 해당 논문이 목차에 있는지 확인
  4. 없으면 → FABRICATED 확정
```

## 출력
`_workspace/03_domestic_results.json`

상세 레퍼런스 파일은 `references/riss-search-patterns.md` 참조 (필요 시).
