---
title: codeforge-requirements lane 구조 (요구사항 레인 — 사용자 요구 접수 → 통합 요구사항 명세)
last_captured: 2026-05-18
kind: architecture_doc
family_ref: ../../../plugin-codeforge/docs/architecture/codeforge-family.md#모듈
---

> **목표 invariant (ADR-078 §결정 1 verbatim)**: 코드 직접 read 없이 architecture_doc 1개 read 로 전체 구조 (모듈 + 경계 + 인터페이스 + 데이터 흐름) 파악.

<!-- 본 file = lane plugin self-owned architecture_doc (Sub-Epic CFP-949 Wave 1, Story #968).
     family-level 구조는 wrapper `codeforge-family.md` 참조 (frontmatter `family_ref`). 본 doc 은 lane 내부 구조만.
     누적 현재 상태 SSOT (Story key 독립, 영속). 델타는 Change Plan SSOT (disjoint). -->

## 모듈

codeforge-requirements lane = **사용자 요구 접수 → 통합 요구사항 명세** 책임. **7 agent 병렬 모듈** (PL + 3 reshape sub + 3 code-context sub). `[verified: lane plugin CLAUDE.md @ 3d31b758]`:

| 모듈 (agent) | 책임 1줄 | reshape dimension |
|---|---|---|
| **RequirementsPLAgent** | lane Lead — 6 sub-agent fan-out + dedup·상충 조정·통합 synthesis (§2/§5/§6 owner) | synthesis (PL monopoly) |
| **DomainAgent** | known knowns — 사내 도메인 KB + ADR + 코드베이스 기반 해석 (§2 owner) | known knowns |
| **RequirementsAnalystAgent** | ambiguity-driven — 원문 edge case + implicit constraint + assumption 확장 (§5 owner) | ambiguity-driven reshape |
| **ResearcherAgent** | concept-driven — 외부 unknown unknowns + 개념 명시화 + requirement 재편 (§6 owner) | concept-driven reshape |
| **ChangeImpactAgent** | code-delta — `src/**` 전체 → AS-IS/DELTA 매핑 (§4.1 owner) | code-delta |
| **FeasibilityAgent** | feasibility — `src/**` + ADR 기반 구현 가능성 등급 + 경고 힌트 (§4.2 owner) | feasibility |
| **ContinuityAgent** | continuity — `docs/stories` · `docs/change-plans` · `docs/adr` 기반 충돌/중복/의존 분류 (§4.3 owner) | continuity |

> 각 agent 의 prompt / lifecycle 상세 = 해당 `agents/<Name>.md` SSOT. 본 표 = 모듈 단위 책임 enumeration (라인 수준 0건).

## 경계

**self-write boundary** (lane 내부 모듈 ↔ Story file 섹션 매핑 — `[verified: lane plugin CLAUDE.md @ 3d31b758 "Self-write 책임" 표]`):

| 모듈 | self-write 영역 | Mechanism |
|---|---|---|
| RequirementsPLAgent | Story §2 (도메인 분석) · §5 (확장 해석) · §6 (개념·외부 지식) · `[요구사항]` GitHub comment · `phase:요구사항 → phase:설계` label 전환 | `Edit(docs/stories/**)` + `mcp__github__add_issue_comment` + `mcp__github__issue_write` |
| ChangeImpactAgent · FeasibilityAgent · ContinuityAgent | Story §4.1 / §4.2 / §4.3 (code-context 3-row) | `.claude-work/doc-queue/` (write queue drain — PL 통합 시점) |
| DomainAgent | `docs/domain-knowledge/domain/<area>/<topic>.md` (CFP-26 Phase 0a — owner direct write) | `Edit(docs/domain-knowledge/domain/**)` |
| ResearcherAgent | `docs/domain-knowledge/concept/<slug>.md` (ADR-056) | `Edit(docs/domain-knowledge/concept/**)` |

**scope partition** (lane 책임 경계):

- **PL synthesis monopoly** — RequirementsPLAgent 만 §2/§5/§6 통합 write. 6 sub-agent 는 독립 관점만 반환 (PL 에 input). sub ↔ sub 직접 통신 0 (one-shot stateless, ADR-039 default subagent context).
- **parallel 6 agent fan-out** — PL 이 한 메시지에 6 sub-agent 동시 dispatch (사용자 원문 verbatim Story §1 + ADR 목록 §3 + 코드 §4 공통 입력). dimension 분리 invariant (각 sub 가 서로 다른 input 축 reshape).
- **dogfood scope partition** — runtime SSOT (`agents/**` · `CLAUDE.md` · `templates/**` · `docs/inter-plugin-contracts/**`) 만 lane plugin repo. dogfood artifacts (specs/plans/retros/stories/change-plans) = `mclayer/codeforge-internal-docs/codeforge-requirements/**` monorepo SSOT (ADR-013).
- **Orchestrator monopoly disjoint** — Story §9 (final verdict) · §10 (FIX Ledger, fix-event-v1 contract) · §14 (Lane Evidence) · phase 전환 label 은 Orchestrator self-write (lane plugin agent write 영역 외).

**clarification 재스폰 경계** — sub-agent stateless 라 PL ↔ sub continuous dialog 불가. PL 이 통합 중 추가 질의 필요 시 Orchestrator 에 "<agent> 재스폰 + clarification context + 이전 출력 pointer" 요청. Orchestrator 가 신규 spawn (cross-lane 동일 패턴, lane plugin 단독 처리 불가 — wrapper Orchestrator routing).

## 인터페이스 계약

lane 의 외부 surface — kind:contract producer / kind:registry consumer / governance ADR anchor 3 영역. 계약 schema field-level / version literal 미박제 (`MANIFEST.yaml` SSOT 가 권위, drift 회피).

**producer (1)**:

| contract | role | SSOT |
|---|---|---|
| `requirements_output` | RequirementsPLAgent → 설계 lane (ArchitectPLAgent) 핸드오프 packet (Story §1-§6 synthesis 통합 요구사항 명세) | `docs/inter-plugin-contracts/requirements-output-v1.md` (canonical, lane plugin repo) + wrapper sibling sync mirror (ADR-010) |

**consumer (1)**:

| contract | role | SSOT |
|---|---|---|
| `debate-protocol-v1` | RequirementsPLAgent multi-round adversarial debate (Codex Proactive Check Touchpoint #4 divergence 감지 시 auto-trigger) consumer | `docs/inter-plugin-contracts/debate-protocol-v1.md` (wrapper canonical, kind:registry — sibling sync 면제 ADR-010 §결정 2) |

**governance ADR anchor**:

- **ADR-077** (clarification 강제 재조사 — recheck_counter cap=5) — RequirementsPLAgent 가 lane 진입 후 clarification 답변이 §2/§5/§6 의미 변경 도달 시 재spawn 의무 (§9.0 Clarification 재스폰 이력 §10 FIX Ledger 와 disjoint 제3 채널, fix:* 미부착).
- **ADR-052 Amendment 1 Touchpoint #4** (mandatory) — RequirementsPLAgent §1-§6 완료 직후 Codex Proactive Check 자동 dispatch + divergence 감지 시 `debate-protocol-v1` 발동. divergence 영역 4 criteria (AC 의미 / Edge Case 누락 / Why 해석 mismatch / fact-check drift). PL synthesis fact claim 영역 marker 4종 (`[verified]` / `[hypothesis]` / `[fact-check-pending]` / `[user-input]`) + reverse-explicit `[verification-out-of-scope: <사유>]` 의무.
- **ADR-046** (ResearcherAgent role redefinition) — Opus tier rationale + 3 mandate (Concept formulation + Deep exploration + Requirement reshape) anchor.
- **ADR-056** (concept-knowledge schema) — `docs/domain-knowledge/concept/**` ResearcherAgent direct write + `kind: concept_definition` schema.

> 본 섹션 = surface enumeration (계약 이름 + SSOT pointer). 계약 schema field-level 상세 = 해당 contract file + `MANIFEST.yaml` SSOT.

## 데이터 흐름

lane spawn / event / artifact propagation 수준 (함수 호출 trace / 변수 전달 라인 0건):

```
사용자 요구 접수 (Story §1 user_request 원문 verbatim)
  │
  ▼
Orchestrator → RequirementsPLAgent spawn (lane Lead, one-shot per attempt)
  │
  ▼
PL 가 한 메시지에 6 sub-agent 병렬 dispatch (parallel fan-out):
  ├─ DomainAgent          (input: §1 + 사내 KB + ADR + src/**)     → known knowns 관점
  ├─ RequirementsAnalyst  (input: §1 + ADR + src/**)              → ambiguity-driven 관점
  ├─ ResearcherAgent      (input: §1 + 외부 source)               → concept-driven 관점
  ├─ ChangeImpactAgent    (input: §1 + src/** 전체)               → AS-IS/DELTA 매핑 → §4.1
  ├─ FeasibilityAgent     (input: §1 + src/** + ADR)              → 가능성 등급 + 경고 → §4.2
  └─ ContinuityAgent      (input: §1 + docs/stories + change-plans + adr) → 충돌/중복/의존 → §4.3
  │
  ▼
6 sub-agent return (one-shot, stateless) — 독립 관점 산출물 PL 에 input
  │
  ▼
PL synthesis (dedup + 상충 조정 + 통합):
  ├─ §2 (도메인 분석)         ← DomainAgent 통합
  ├─ §5 (확장 해석)            ← RequirementsAnalyst 통합
  └─ §6 (개념·외부 지식·재편)  ← ResearcherAgent 통합
  │
  ▼
Codex Proactive Check Touchpoint #4 (ADR-052 Amd 1 mandatory)
  │
  ├─ divergence 0 → 다음 단계
  │
  └─ divergence detected (4 criteria)
       │
       ▼
     debate-protocol-v1 multi-round adversarial debate (auto-trigger, min 3 / max 5 round)
       │
       ├─ consensus → PL synthesis 재 write
       │
       └─ max round 도달 미합의 → AskUserQuestion escalation
  │
  ▼
ADR-077 clarification 강제 재조사 check (recheck_counter cap=5)
  │
  ├─ §2/§5/§6 의미 변경 도달 시 → PL self redo (재spawn, §9.0 ledger row append)
  │
  └─ no change → 다음 단계
  │
  ▼
requirements_output packet emit (Story §1-§6 synthesis 통합 요구사항 명세)
  │
  ▼
Orchestrator → phase:요구사항 → phase:설계 label 전환 → 설계 lane (ArchitectPLAgent) 핸드오프
```

**artifact propagation**:

- **Story file** (`internal-docs/codeforge-requirements/stories/<KEY>.md` §2/§5/§6) — lane 산출물 SSOT, PL self-write.
- **domain knowledge** (`docs/domain-knowledge/domain/**` / `docs/domain-knowledge/concept/**`) — DomainAgent / ResearcherAgent direct write (CFP-26 Phase 0a, lane plugin repo).
- **requirements_output packet** — Story file pointer (lane plugin 단독 emit, 설계 lane 가 consume).
- **§9.0 Clarification 재스폰 이력** (ADR-077, §10 FIX Ledger 와 disjoint) — Orchestrator monopoly write, lane plugin agent read-only.

---

### anti-scope guard (ADR-078 §결정 1 verbatim — 작성자 필독)

본 doc 은 **구조 수준 only**. closed-enum 4 영역 외 다음 4종 패턴은 **금지** (라인 수준 허용 시 갱신 즉시 stale + "코드에 한 단계 더한 것" 전락 — Epic §위험신호 §1):

1. **클래스 / 함수 / 변수 라인 단위 열거** — 클래스 list, 변수 enumeration 금지.
2. **의존성 import graph 라인-level** — import 관계 라인 단위 그래프 금지.
3. **함수 signature / parameter list / return type** — API 의 line-level 시그니처 금지.
4. **코드 mirror** — `agents/` · `templates/` 구조를 1:1 복사한 디렉터리 트리 dump 금지.

→ 위 4종이 필요하면 그것은 코드 / Change Plan / ADR 영역. architecture_doc 은 "코드 read 없이 구조 파악" 목표만 만족하면 된다.

---

### ADR-076 declarative reconciliation 3-layer 답습

본 doc 은 ADR-076 (declarative reconciliation upgrade flow) 의 3-layer pattern 을 설계 lane 도메인으로 답습한 것 (도메인 disjoint, 패턴 동형 — ADR-078 §결정 2 정합):

- **desired state** = 본 doc (`docs/architecture/codeforge-requirements.md`) — lane 의 영속 구조 SSOT.
- **current state** = lane plugin agent file (`agents/RequirementsPLAgent.md` · `agents/DomainAgent.md` · `agents/RequirementsAnalystAgent.md` · `agents/ResearcherAgent.md` · `agents/ChangeImpactAgent.md` · `agents/FeasibilityAgent.md` · `agents/ContinuityAgent.md` · `agents/codex-proactive-check.md`) + `CLAUDE.md` (runtime 실제 동작).
- **converge** = ArchitectAgent self-write 확장 (Sub-Epic CFP-949 S3 carrier `mclayer/plugin-codeforge-design` ADR-082 §결정 1 mechanism) + design lane verdict gate (drift lint CFP-923 detection class d, Wave 4 carrier).

> 본 cross-ref = 패턴 답습 (pattern). 도메인 (upgrade flow ↔ 설계 lane) 은 disjoint. wording SSOT = ADR-076 본문 + ADR-078 §결정 2.
