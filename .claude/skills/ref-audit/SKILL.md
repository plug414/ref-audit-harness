---
name: ref-audit
description: "논문 참고문헌(레퍼런스)의 진위여부를 종합 검증하고 HTML 보고서를 생성하는 오케스트레이터. 논문 레퍼런스 검증, 참고문헌 진위 확인, 레퍼런스 검사, 레퍼런스 체크, reference audit, 참고문헌 검증 보고서 요청 시 사용. 후속 작업: 검증 결과 수정, 부분 재검증, 보고서 업데이트, 레퍼런스 재확인, 다시 검증, 이전 결과 개선, 특정 항목만 재검증 요청 시에도 반드시 이 스킬을 사용."
---

# Reference Audit Orchestrator

논문 참고문헌의 진위여부를 5단계 에이전트 파이프라인으로 검증하고 HTML 보고서를 생성하는 통합 오케스트레이터.

## 실행 모드: 서브 에이전트 (파이프라인 + 팬아웃)

검증 에이전트들은 할당된 레퍼런스를 독립적으로 검증하며 상호 통신이 불필요하다. 오케스트레이터가 DOI 검증 실패 항목을 언어별로 분배하고 결과를 수집한다.

## 에이전트 구성

| 에이전트 | subagent_type | 역할 | 스킬 | 출력 |
|---------|--------------|------|------|------|
| ref-extractor | general-purpose | 참고문헌 추출·구조화 | ref-extraction | `_workspace/01_references.json` |
| doi-verifier | general-purpose | DOI/Crossref 1차 검증 | doi-verification | `_workspace/02_doi_results.json` |
| domestic-verifier | general-purpose | 국내문헌 RISS 검증 | domestic-verification | `_workspace/03_domestic_results.json` |
| intl-verifier | general-purpose | 영문/중문 검증 | intl-verification | `_workspace/03_intl_results.json` |
| ref-report-writer | general-purpose | HTML 보고서 생성 | ref-report | `{날짜}_테스트리뷰{N}_레퍼런스검증.html` |

## 워크플로우

### Phase 0: 컨텍스트 확인

기존 산출물 존재 여부로 실행 모드를 결정한다:

1. `_workspace/` 디렉토리 존재 여부 확인
2. 실행 모드 결정:
   - **`_workspace/` 미존재** → 초기 실행. Phase 1로 진행
   - **`_workspace/` 존재 + 부분 수정 요청** → 부분 재실행. 해당 Phase의 에이전트만 재호출
   - **`_workspace/` 존재 + 새 논문 제공** → 새 실행. 기존 `_workspace/`를 `_workspace_{YYYYMMDD_HHMMSS}/`로 이동

### Phase 1: 입력 분석 및 레퍼런스 추출

1. 사용자 입력 분석:
   - 논문 파일 경로 확인 (PDF/HWP/DOCX)
   - 테스트리뷰 번호 확인 (파일명 또는 사용자 지정)
   - 직접 텍스트 입력 여부 확인

2. `_workspace/` 디렉토리 생성

3. **ref-extractor 서브 에이전트 호출:**
   ```
   Task(
     name: "ref-extractor",
     subagent_type: "general-purpose",
     model: "opus",
     prompt: "
       .claude/agents/ref-extractor.md를 읽고 역할에 따라 행동하라.
       .claude/skills/ref-extraction/SKILL.md를 읽고 절차를 따르라.

       대상 파일: {파일경로}
       출력: _workspace/01_references.json

       참고문헌 목록을 추출하고 각 항목의 DOI·언어·유형을 분류하라.
     "
   )
   ```

4. 결과 확인: `_workspace/01_references.json` 읽기

### Phase 2: DOI 1차 검증

**doi-verifier 서브 에이전트 호출:**
```
Task(
  name: "doi-verifier",
  subagent_type: "general-purpose",
  model: "opus",
  prompt: "
    .claude/agents/doi-verifier.md를 읽고 역할에 따라 행동하라.
    .claude/skills/doi-verification/SKILL.md를 읽고 절차를 따르라.

    입력: _workspace/01_references.json
    출력: _workspace/02_doi_results.json

    모든 참고문헌에 대해 DOI-first 검증을 수행하라.
    DOI가 없거나 검증 불가한 항목은 needs_further_check에 추가하라.
  "
)
```

결과 확인: `needs_further_check` 목록에서 언어별 분류

### Phase 3: 언어별 심층 검증 (팬아웃)

`needs_further_check` 항목을 언어별로 분배하여 **병렬 실행:**

```
# 국내문헌이 있으면:
Task(
  name: "domestic-verifier",
  subagent_type: "general-purpose",
  model: "opus",
  run_in_background: true,
  prompt: "
    .claude/agents/domestic-verifier.md를 읽고 역할에 따라 행동하라.
    .claude/skills/domestic-verification/SKILL.md를 읽고 절차를 따르라.

    입력: _workspace/02_doi_results.json의 needs_further_check 중 language='ko' 항목
    원본 참조: _workspace/01_references.json
    출력: _workspace/03_domestic_results.json

    RISS 5단계 방법론으로 각 항목을 검증하라.
  "
)

# 영문/중문이 있으면:
Task(
  name: "intl-verifier",
  subagent_type: "general-purpose",
  model: "opus",
  run_in_background: true,
  prompt: "
    .claude/agents/intl-verifier.md를 읽고 역할에 따라 행동하라.
    .claude/skills/intl-verification/SKILL.md를 읽고 절차를 따르라.

    입력: _workspace/02_doi_results.json의 needs_further_check 중 language='en' 또는 'zh' 항목
    원본 참조: _workspace/01_references.json
    출력: _workspace/03_intl_results.json

    영문/중문 검증 절차에 따라 각 항목을 검증하라.
  "
)
```

모든 서브 에이전트 완료 대기 후 결과 수집.

### Phase 4: 보고서 생성

**ref-report-writer 서브 에이전트 호출:**
```
Task(
  name: "ref-report-writer",
  subagent_type: "general-purpose",
  model: "opus",
  prompt: "
    .claude/agents/ref-report-writer.md를 읽고 역할에 따라 행동하라.
    .claude/skills/ref-report/SKILL.md를 읽고 절차를 따르라.

    입력 파일들:
    - _workspace/01_references.json (원본)
    - _workspace/02_doi_results.json (DOI 검증)
    - _workspace/03_domestic_results.json (국내문헌, 있으면)
    - _workspace/03_intl_results.json (국제문헌, 있으면)

    출력: {YYYY-MM-DD}_테스트리뷰{N}_레퍼런스검증.html
    저장 경로: (현재 작업 디렉토리)

    모든 검증 결과를 통합하여 표준 HTML 보고서를 생성하라.
    생성 후 품질 체크(카드 숫자 대조)를 반드시 수행하라.
  "
)
```

### Phase 5: 정리 및 보고

1. `_workspace/` 보존 (감사 추적용)
2. 사용자에게 결과 요약:
   - 전체 N건 중: CONFIRMED X건, CITATION_ISSUE Y건, FABRICATED Z건, MANUAL_REVIEW W건
   - 보고서 파일 경로
   - 주요 발견 사항 (있으면)

## 데이터 흐름

```
[사용자] → 논문 파일
    ↓
[오케스트레이터] → Phase 1
    ↓
[ref-extractor] → 01_references.json
    ↓
[doi-verifier] → 02_doi_results.json
    ↓                    ↓
    ├── ko 항목 ──→ [domestic-verifier] → 03_domestic_results.json
    └── en/zh 항목 → [intl-verifier] → 03_intl_results.json
                          ↓
              [ref-report-writer] → 최종 HTML
                          ↓
              [사용자] ← 결과 요약
```

## 에러 핸들링

| 상황 | 전략 |
|------|------|
| 파일 읽기 실패 (HWP) | 사용자에게 텍스트 복사 요청 |
| DOI verifier 타임아웃 | 1회 재시도. 실패 시 전항목 needs_further_check |
| 국내/국제 verifier 실패 | 해당 영역 MANUAL_REVIEW 처리, 보고서에 "검증 미완료" 명시 |
| 보고서 생성 실패 | 오케스트레이터가 직접 간이 보고서 생성 |
| 모든 verifier 실패 | 사용자에게 알리고 수동 검증 가이드 제공 |
| needs_further_check가 0건 | Phase 3 건너뜀 → Phase 4로 직행 |

## 부분 재실행 가이드

사용자가 특정 항목이나 영역만 재검증을 요청하면:

1. 기존 `_workspace/` 결과 확인
2. 재검증 대상 식별:
   - "3번 항목만 다시" → DOI verifier에 해당 항목만 재전달
   - "국내문헌만 다시" → domestic-verifier만 재호출
   - "보고서만 다시" → ref-report-writer만 재호출
3. 변경된 결과만 기존 JSON에 병합
4. 보고서 재생성

## 테스트 시나리오

### 정상 흐름
1. 사용자가 PDF 논문 파일 제공 (레퍼런스 25건)
2. Phase 1: 25건 추출 (DOI 15건, 국문 5건, 영문 5건)
3. Phase 2: DOI 15건 중 12건 CONFIRMED, 3건 needs_further_check
4. Phase 3: 국문 5+1건 RISS 검증 + 영문 5+2건 국제 검증 (병렬)
5. Phase 4: 전체 25건 통합 HTML 보고서 생성
6. 결과: CONFIRMED 20, CITATION_ISSUE 3, MANUAL_REVIEW 2

### 에러 흐름
1. HWP 파일 읽기 실패
2. 사용자에게 참고문헌 텍스트 복사 요청
3. 텍스트 입력으로 Phase 1 재시작
4. domestic-verifier가 RISS 접속 차단으로 실패
5. 국내문헌 전체 MANUAL_REVIEW 처리
6. 보고서에 "국내문헌 영역: RISS 접속 차단으로 자동 검증 미완료. 수동 확인 필요" 명시
7. 인간 위임 가이드 포함
