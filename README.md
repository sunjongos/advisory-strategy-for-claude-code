# Advisory Strategy v2.0
## Anthropic 공식 패턴 기반 표준 작업 프로토콜

---

## 핵심 개념

**"강한 모델이 계획·검토, 빠른 모델이 실행"** — Anthropic이 공식 발표한 Claude Code Advisory Strategy를 그대로 구현한 표준 작업 프로토콜.

모든 중요 작업은 반드시 이 4단계 루프를 따른다:

```
ADVISOR 사전 검토 → PLAN (Opus) → EXECUTE (Sonnet/Haiku) → ADVISOR 사후 검토
```

---

## 4단계 워크플로우

### Phase 0: Advisor 사전 검토 (Pre-flight)
- `/advisor` → Opus 4.7로 전환
- `advisor()` 도구 호출 → 작업 방향 점검
- Advisor 피드백 반영 후 계획 단계 진입

### Phase 1: Plan — Opus 4.7
- `opusplan` 모드로 전체 아키텍처 및 WBS 수립
- 위험 요소·엣지 케이스 사전 식별
- 계획을 사용자에게 공유 → 승인 후 실행

### Phase 2: Execute — Sonnet 4.6 / Haiku 4.5
- 복잡한 코드 작성·API 연동 → **Sonnet 4.6**
- 파일 탐색·단순 수정·검색 → **Haiku 4.5**
- WBS 순서 준수, 중간 체크포인트 보고

### Phase 3: Advisor 사후 검토 (Self-Critique & Self-Correction)
- `advisor()` 재호출 → 결과물 전체 검토
- 문제 발견 시 즉시 자기 수정 (Self-Correction)
- 최종 이상 없을 때만 결과 보고

---

## 트리거 키워드

| 사용자 발화 | 동작 |
|------------|------|
| "advisory-strategy로 작업하자" | 4단계 루프 전체 활성화 |
| "최고 품질로 작업해줘" | 4단계 루프 전체 활성화 |
| "꼼꼼하게 검토하면서" | 4단계 루프 전체 활성화 |
| 복잡한 아키텍처/인프라 변경 요청 | 자동 활성화 |

---

## 모델 역할 분담

| 역할 | 모델 | 단계 |
|------|------|------|
| Advisor / Planner | `claude-opus-4-7` | Phase 0, 1, 3 |
| Executor (주력) | `claude-sonnet-4-6` | Phase 2 |
| Executor (경량) | `claude-haiku-4-5-20251001` | Phase 2 단순 작업 |

> 모델 ID는 `~/CLAUDE.md` Model Version Registry에서 중앙 관리.

---

## 핵심 원칙 5가지

1. **Advisor First, Always** — 중요 작업은 반드시 Opus 사전 검토
2. **Plan Before Execute** — 승인된 계획 없이 실행 없음
3. **Right Model for Right Job** — 역할에 맞는 모델 사용
4. **Self-Correct Before Report** — 보고 전 반드시 Advisor 재검토
5. **Human in the Loop** — 되돌리기 어려운 작업은 사용자 확인

---

## 선택적 확장: 로컬 LLM 팀

기본 Advisory Loop 완수 후, 추가 도메인 자문이 필요할 때만 활용:

| 모델 | 역할 |
|------|------|
| `gpt-oss:20b` | 수석 자문 — 고급 아키텍처·보안 분석 |
| `gemma4:e4b` | 초경량 인턴 — 컨텍스트 압축·빠른 초안 |
| `gemma4:e2b` | 멀티모달 — 이미지/UI 분석, 대용량 문서 |
| `gemma4:ndb` | 도메인 전문 — 남양주백병원 특화 |
| `moondream:latest` | 컴팩트 비전 — 스크린샷 판독 |

자세한 내용은 `SKILL.md` 참조.
