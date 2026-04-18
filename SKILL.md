---
name: advisory-strategy
description: "Anthropic 공식 Advisory Strategy 패턴. 사용자가 'advisory-strategy로 작업하자', '최고 품질로', '꼼꼼하게 검토하면서' 등을 요청하면 이 스킬을 활성화. Opus 4.7이 사전 검토(advisor)→계획(opusplan)→Sonnet/Haiku 실행→Opus 4.7 사후 자기비판/수정(advisor) 4단계 루프를 수행."
---

# ADVISORY STRATEGY v2.0
## Anthropic 공식 패턴 — 4단계 표준 작업 루프

---

## 스킬 활성화 조건

이 스킬은 다음 키워드/상황에서 자동으로 활성화된다:

**명시적 트리거:**
- "advisory-strategy로 작업하자"
- "advisor-strategy 가동"
- "최고 품질로 작업해줘"
- "꼼꼼하게 검토하면서 작업해줘"

**암묵적 트리거 (작업 복잡도 기반):**
- 새로운 기능의 아키텍처 설계
- 여러 파일에 걸친 대규모 리팩토링
- 외부 API 통합 또는 인프라 변경
- 되돌리기 어려운 작업 (git 이력 수정, DB 마이그레이션 등)

---

## 4단계 실행 워크플로우

### ━━━ PHASE 0: ADVISOR 사전 검토 (Pre-flight Review) ━━━
> **목적:** 잘못된 방향으로 삽질하지 않도록 Opus 4.7이 먼저 전체 방향을 점검

**실행 순서:**
1. `/advisor` 실행 → 현재 세션을 Opus 4.7로 전환
2. 코드베이스 탐색 (필요한 파일 Read/Glob/Grep 수행)
3. `advisor()` 도구 호출 → 다음을 전달:
   - 사용자 요청의 핵심 목표
   - 현재 상태 (관련 파일/코드 요약)
   - 검토 요청 포인트 (무엇을 점검해야 하는지)
4. Advisor 피드백을 수용하여 Phase 1 계획에 반영

**Advisor 호출 시 포함할 내용:**
```
- 요청 목표: [사용자가 원하는 것]
- 현재 상태: [코드베이스 탐색 결과]
- 내 계획 초안: [구현하려는 방향]
- 검토 요청: [어떤 부분이 불확실한지]
```

---

### ━━━ PHASE 1: PLAN — Opus 4.7 계획 수립 ━━━
> **목적:** 실행 전 명확한 로드맵을 수립하여 방향 없이 코딩하는 것을 방지

**모델:** `opusplan` (settings.json의 `"model": "opusplan"` → Opus 4.7 Thinking Mode)

**필수 산출물:**
```markdown
## 작업 목표 및 범위
- 핵심 목표: ...
- 범위 내: ...
- 범위 외: ...

## 단계별 WBS
1. [단계 1]: [내용] → 담당: Sonnet/Haiku
2. [단계 2]: [내용] → 담당: Sonnet/Haiku
...

## 위험 요소 및 엣지 케이스
- ...

## 되돌리기 어려운 작업 (사용자 확인 필요)
- ...

## 완료 기준 (Definition of Done)
- ...
```

**규칙:**
- 계획을 사용자에게 공유하고 **반드시 승인을 받은 후** 실행 단계로 진입
- 되돌리기 어려운 작업은 계획에 명시적으로 표기

---

### ━━━ PHASE 2: EXECUTE — Sonnet 4.6 / Haiku 4.5 실행 ━━━
> **목적:** 승인된 계획을 효율적으로 구현

**모델 선택 기준:**

| 태스크 유형 | 모델 | 선택 이유 |
|------------|------|----------|
| 코드 작성, 복잡한 편집, API 연동 | `claude-sonnet-4-6` | 품질·속도 균형 |
| 파일 탐색, 검색, 단순 텍스트 수정 | `claude-haiku-4-5-20251001` | 최경량·최고속 |
| 아키텍처 결정 필요 구현 중간 | `advisor()` 중간 호출 | 복잡도 대응 |

**실행 원칙:**
1. WBS 순서를 따라 단계별로 진행
2. 각 단계 완료 시 간결하게 보고 ("Step 1 완료: [내용]")
3. 예상치 못한 문제 발생 → 즉시 중단 후 사용자에게 보고
4. 장시간 작업 → 주요 단계마다 체크포인트 보고
5. 되돌리기 어려운 작업 전 → 반드시 사용자 확인

---

### ━━━ PHASE 3: ADVISOR 사후 검토 — Self-Critique & Self-Correction ━━━
> **목적:** 결과물을 사용자에게 보고하기 전 Opus 4.7이 스스로 검토하고 수정

**실행 순서:**
1. 실행 결과물을 모두 저장 (Write/Edit 완료)
2. `advisor()` 도구 호출 → 전체 결과물 검토 요청

**Self-Critique 체크리스트:**
```
□ 사용자 요청의 핵심 목표가 완전히 달성되었는가?
□ 코드/결과물에 보안 취약점(OWASP Top 10 등)이 없는가?
□ 엣지 케이스가 적절히 처리되었는가?
□ 불필요한 복잡성이나 over-engineering이 없는가?
□ 기존 코드 컨벤션과 일관성이 유지되는가?
□ 되돌리기 어려운 변경이 의도치 않게 발생하지 않았는가?
□ 테스트/검증이 충분히 수행되었는가?
□ Phase 0 Advisor의 피드백이 반영되었는가?
```

**Self-Correction 행동:**
- Advisor가 문제 발견 → 즉시 수정 (사용자에게 묻지 않고)
- 수정 후 재검토 (필요 시 `advisor()` 재호출)
- 모든 체크리스트 통과 후에만 최종 보고

**최종 보고 형식:**
```markdown
## 작업 완료 보고

### 수행한 작업
- ...

### 검증 결과
- Advisor 사전 검토: [주요 피드백 및 반영 내용]
- Advisor 사후 검토: [이상 없음 / 수정한 내용]

### 추가 권장 사항 (있을 경우)
- ...
```

---

## 모델 티어 레지스트리

| 역할 | 모델 ID | Phase |
|------|---------|-------|
| Advisor / Planner | `claude-opus-4-7` | 0, 1, 3 |
| Executor (주력) | `claude-sonnet-4-6` | 2 |
| Executor (경량) | `claude-haiku-4-5-20251001` | 2 단순 작업 |

> 모델 ID는 `~/CLAUDE.md` Model Version Registry에서 중앙 관리.
> 새 모델 출시 시 CLAUDE.md만 수정하면 이 스킬에 자동 반영됨.

---

## 빠른 참조 (Quick Reference)

```
사용자: "advisory-strategy로 작업하자"

1. /advisor          → Opus 4.7 전환
2. advisor()         → 방향 사전 점검
3. opusplan          → WBS 계획 수립 → 사용자 승인
4. Sonnet/Haiku      → 계획 실행 (Auto 모드)
5. advisor()         → Self-Critique → Self-Correct
6. 최종 보고         → 검증 결과 포함
```

---

## 핵심 원칙 5가지 (절대 위반 금지)

1. **Advisor First, Always** — 중요 작업은 반드시 Opus 사전 검토
2. **Plan Before Execute** — 승인된 계획 없이 실행 없음
3. **Right Model for Right Job** — 계획/검토는 Opus, 실행은 Sonnet/Haiku
4. **Self-Correct Before Report** — 보고 전 반드시 Advisor 재검토
5. **Human in the Loop** — 되돌리기 어려운 작업은 사용자 확인 필수

---

## 선택적 확장: 로컬 LLM 팀

> 기본 4단계 루프 완수 후, 추가 도메인 자문이 필요할 때만 활용.
> Anthropic 공식 패턴의 대체재가 아닌 보완재.

| 모델 | 역할 | `mcp_ollama_local_ask_ollama` model 파라미터 |
|------|------|---------------------------------------------|
| `gpt-oss:20b` | 수석 자문 — 고급 아키텍처·보안 분석 | `gpt-oss:20b` |
| `gemma4:e4b` | 초경량 인턴 — 컨텍스트 압축, 빠른 초안 | `gemma4:e4b` |
| `gemma4:e2b` | 멀티모달 — 이미지/UI 분석, 128k 문서 | `gemma4:e2b` |
| `gemma4:ndb` | 도메인 전문 — 남양주백병원 특화 | `gemma4:ndb` |
| `moondream:latest` | 컴팩트 비전 — 스크린샷 판독 | `moondream:latest` |

**로컬 LLM 호출 필수 규칙:**
1. 컨텍스트가 길면 `gemma4:e4b`로 먼저 압축 선행
2. 프롬프트 끝에 `Respond ONLY in valid JSON format. No conversational text.` 강제
3. 로컬 LLM 결과는 보조 자료 — 최종 판단은 항상 Claude(Opus/Sonnet)
