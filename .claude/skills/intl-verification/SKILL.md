---
name: intl-verification
description: "영문 및 중문 학술문헌 검증 스킬. 영문은 출판사 landing/Google Scholar/WebSearch, 중문은 参考网/豆瓣/CNKI 간접 경로를 사용한다. 프리프린트 판별과 정식 출판본 탐색도 수행. 영문 논문 검증, 중국어 문헌 확인, preprint check 요청 시 사용."
---

# 국제문헌(영문/중문) 검증 스킬

DOI 검증에서 미확인된 영문·중문 참고문헌의 존재와 서지 정합성 검증 방법론.

## 영문 문헌 검증

### 검증 순서
```
[Step 1] 제목 기반 WebSearch
  → "{정확한 논문 제목}" site:{추정 출판사}
  → 또는 일반 웹 검색으로 출판사 페이지 탐색
       ↓
[Step 2] 출판사 landing 확인
  → WebFetch로 메타데이터 스캔 (저자, 연도, 저널, DOI)
  → 서지 대조
       ↓
[Step 3] Google Scholar 검색 (Step 1-2 실패 시)
  → WebSearch: "{제목}" scholar
  → 인용 정보, 학술 DB 링크 확인
       ↓
[Step 4] 학회 proceedings 확인 (conference 유형)
  → ACM DL, IEEE Xplore, DBLP 등
```

### 주요 출판사 URL 패턴
| 출판사 | 도메인 | DOI prefix |
|--------|--------|-----------|
| Elsevier | sciencedirect.com | 10.1016 |
| Springer | link.springer.com | 10.1007 |
| IEEE | ieeexplore.ieee.org | 10.1109 |
| ACM | dl.acm.org | 10.1145 |
| Wiley | onlinelibrary.wiley.com | 10.1002 |
| Taylor & Francis | tandfonline.com | 10.1080 |
| SAGE | journals.sagepub.com | 10.1177 |
| MDPI | mdpi.com | 10.3390 |

## 중문 문헌 검증

### 검증 순서
```
[Step 1] 参考网 (fx361.com)
  → 중문 학술지 기사 검색
  → WebSearch: site:fx361.com "{중문 제목 키워드}"
       ↓
[Step 2] 豆瓣/京东 (단행본)
  → 단행본이면 ISBN 또는 제목으로 검색
  → WebSearch: site:douban.com OR site:jd.com "{서명}"
       ↓
[Step 3] CNKI 간접 확인
  → 직접 접근 불가 (캡차/인증)
  → WebSearch: site:cnki.net "{제목 키워드}" → 검색 결과에서 존재 여부 확인
       ↓
[Step 4] 百度学术
  → WebSearch: site:xueshu.baidu.com "{제목}"
```

### 중문 검증 주의사항
- CNKI 직접 접근은 거의 불가 → WebSearch 간접 확인이 주 경로
- 접근 제한 시 UNVERIFIABLE 처리 (FABRICATED 금지)
- 중문 ↔ 영문 제목이 다른 경우 흔함 → 저자+연도+저널로 보조 판단

## 프리프린트 판별

### arXiv/SSRN/bioRxiv 판별 기준
| 조건 | 판정 |
|------|------|
| arXiv/SSRN/bioRxiv 단독 출처 | `preprint(심사 미통과)` |
| 동일 제목 저널/학회 정식 출판본 확인됨 | `peer-reviewed(출판본 확인)` + 수정 권고 |
| 확인 근거 부족 | `unknown(추가 확인 필요)` |

### 정식 출판본 탐색
1. arXiv 페이지에서 "journal reference" 또는 "DOI" 필드 확인
2. WebSearch: "{논문 제목}" + "conference" OR "journal"
3. Semantic Scholar API (가능 시): `https://api.semanticscholar.org/graph/v1/paper/arXiv:{id}`

### 학회 수준 참고
- 탑티어: NeurIPS, ICML, ICLR, CVPR, ICCV, ECCV, ACL, EMNLP, NAACL, AAAI, IJCAI, CHI, WWW, KDD, SIGIR
- 주요: IEEE/ACM 계열 학회
- 소규모: 심사 엄격도 확인 필요

## 비학술 출처 식별
- 뉴스 기사, 블로그, 기업 보고서, 정부 문서 등
- `type: "non-academic"` + 학술 근거 적절성 코멘트

## 로그 템플릿
```
① WebSearch "{제목}" → ② {출판사} landing 확인 → ③ 저자/연도/저널 정합
① WebSearch 결과 없음 → ② Scholar 검색 → ③ {결과}
① arXiv 페이지 확인 → ② journal-ref 필드 존재 → ③ 정식 출판본: {DOI}
① CNKI 간접 검색 → ② 캡차 차단 → ③ UNVERIFIABLE
```

## 출력
`_workspace/03_intl_results.json`
