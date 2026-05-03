---
kind: contract
contract_version: "1.0"
status: Active
related_plugins:
  - codeforge (wrapper, consumer)
  - codeforge-requirements (lane plugin, producer + self-writer)
related_adrs:
  - ADR-008 (Inter-plugin Contract Versioning)
  - ADR-009 (Wrapper-only core + writer-distributed lane plugins)
authors:
  - CFP-37 ζ arc — Requirements lane extraction (2026-04-29)
---

# requirements_output v1 — Inter-plugin Contract

`codeforge-requirements` plugin → `codeforge` core (Orchestrator) 단방향 schema. 4 sub-agent (Domain · Analyst · Researcher) 병렬 스폰 후 RequirementsPLAgent 가 통합·자체 write 후 typed output 으로 결과 audit.

**상위 SSOT 위치**:
- `mclayer/plugin-codeforge-requirements/docs/inter-plugin-contracts/requirements-output-v1.md`: **canonical**
- `mclayer/plugin-codeforge/docs/inter-plugin-contracts/requirements-output-v1.md`: sibling reference

## 1. 흐름 개요

```
codeforge core (Orchestrator)
        │
        │ ① requirements_packet 작성 (Story §1 + §3 ADR list + §4 코드 경로 + Project Config)
        ▼
codeforge-requirements plugin
  └─ RequirementsPLAgent
        │
        │ ② Orchestrator가 한 메시지에 3 sub-agent 병렬 dispatch
        ▼
  ├─ DomainAgent       (도메인 지식 공백 분석)
  ├─ RequirementsAnalystAgent (Codex 호출 — ambiguity / 가정 / AC)
  └─ ResearcherAgent   (외부 기술·표준·선행사례)
        │
        │ ③ 3 결과 PL에 return
        ▼
  └─ RequirementsPLAgent dedup + 상충 조정 통합
        │
        │ ④ Self-write 단계:
        │    - Edit(docs/stories/<KEY>.md §2/§5/§6 동시 채움)
        │    - DomainAgent 의 KB 공백 해소 시: Edit(docs/domain-knowledge/**/*.md)
        │    - mcp__github__add_issue_comment ([요구사항] prefix)
        │    - phase 라벨 transition: phase:요구사항 → phase:설계
        ▼
        │ ⑤ requirements_output v1 typed output
        ▼
codeforge core (Orchestrator)
        │
        │ ⑥ Output 처리:
        │    - PASS → 다음 lane (설계) 진입
        │    - ESCALATE → 사용자 인터랙션 (Story §1 ambiguity 해결 필요)
```

## 2. requirements_packet (Orchestrator → RequirementsPLAgent)

```yaml
requirements_packet:
  contract_version: "1.0"           # 필수
  story_key: <STORY_KEY>             # 필수
  story_section_1: <markdown>        # Story §1 verbatim
  related_adr_paths:                 # 선택 — Story §3 fetch 결과
    - docs/adr/ADR-NNN-<slug>.md
  related_code_paths:                # 선택 — Story §4 fetch 결과
    - <path glob>
  project_config:                    # 필수 — overlay project.yaml slice
    domain: <domain identifier>
    discussions_kb_category: <gh discussions category id, optional>
```

## 3. requirements_output (PL → Orchestrator)

```yaml
requirements_output:
  contract_version: "1.0"
  story_key: <STORY_KEY>

  status: PASS | ESCALATE_USER_CLARIFICATION | ESCALATE_PACKET_INCOMPLETE  # 필수

  sub_agent_results:                # 필수 — 3 sub-agent 결과 audit
    domain:
      kb_gaps_filled: <int>          # 새로 KB 작성 페이지 수
      kb_paths: [<list>]
      null_result: <bool>            # 도메인 공백 없음
    analyst:
      ambiguities_found: <int>
      assumptions_listed: <int>
      acceptance_criteria_count: <int>
      null_result: <bool>            # 사용자 원문 완전 명확
    researcher:
      external_refs_count: <int>     # 인용 외부 자료 수
      libraries_evaluated: [<list>]
      null_result: <bool>            # 조사 불필요

  # PL self-write 결과 audit
  writes_completed:
    story_section_2: <bool>           # 도메인 분석 (§2)
    story_section_5: <bool>           # 요구사항 확장 해석 (§5)
    story_section_6: <bool>           # 외부 지식 배경 (§6)
    domain_kb_files: <int>            # 신규 또는 갱신된 docs/domain-knowledge/* 파일 수
    phase_comment: <bool>             # [요구사항] prefix GitHub comment
    phase_label_transitioned: <bool>  # phase:요구사항 → phase:설계

  # ESCALATE 시 추가 (사용자 clarification 요청 항목)
  user_clarification_needed:        # 선택 — null 또는 array
    - question: <text>
      context: <text>
      blocking: <bool>
```

## 3.1 Story §1 frontmatter schema (cross-repo Epic 지원, v1.1)

CFP-60 / [ADR-020](../adr/ADR-020-cross-repo-epic-pattern.md) 신설. Story `§1 메타` 의 YAML frontmatter 에 cross-repo Epic 정보 추가 (optional, backward compatible).

```yaml
---
key: <KEY>           # required (예: CFP-60)
title: <string>      # required
status: <phase:*>    # required
date: <ISO8601>      # required
type: story          # required
github_issue: <owner/repo#N>  # required

# Cross-repo Epic 지원 (v1.1, ADR-020 / CFP-60)
epic_owner_repo: <owner/repo> | null  # OPTIONAL — null if single-repo Story
epic_dependencies:                    # OPTIONAL — empty list if independent
  - type: hard_block | design_parallel | impl_parallel
    target: <KEY>
    repo: <owner/repo>
---
```

**Type 정의**:
- `hard_block`: blocking dependency — target merge 전 본 Story 작업 불가
- `design_parallel`: 설계 동시 진행 가능 (구현은 target 후)
- `impl_parallel`: 구현 동시 진행 가능 (target merge 와 무관)

**Backward compatibility**:
- Pre-v1.1 Story (CFP-1 ~ CFP-59) 는 `epic_dependencies` / `epic_owner_repo` field 없이 작성됨
- v1.1 consumer 가 default `[]` / `null` 로 처리
- 기존 Story 영향 X — 신규 Story 만 optional 사용

## 4. ESCALATE 처리

- **ESCALATE_USER_CLARIFICATION**: Analyst 가 user 원문 ambiguity 해결 불가. PL 이 specific 질문 list 반환 → Orchestrator 가 user 에게 전달
- **ESCALATE_PACKET_INCOMPLETE**: 필수 packet 필드 누락 (story_section_1 부재 등) — 즉시 Orchestrator 수동 복구

## 5. v1 → v2 변경 가능성

- 새 sub-agent 추가 (예: ComplianceAnalyst) — sub_agent_results schema 확장 minor (v1.1)
- 새 ESCALATE enum 추가 — minor
- writes_completed 필드 schema 확장 — minor

## 6. 본 contract 시점 동결 ATTRIBUTION

- 동결 일시: 2026-04-29 (CFP-37)
- 협업: Claude (codification) · CFP-31 parent spec §5.7
- Source: `mclayer/plugin-codeforge-requirements` 4 agent 책임 정의
