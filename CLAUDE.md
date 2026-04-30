# CLAUDE.md (codeforge-requirements)

codeforge ζ arc Requirements lane plugin. 4 agent 병렬 (PL + 3 sub) + 도메인 KB owner write.

## Plugin position

본 plugin 은 codeforge wrapper 의 dependency. 단독 동작 불가 — codeforge core (>= 2.0.0) 가 Orchestrator 보유.

설치 + dependency + architecture 는 [`README.md`](README.md) 참조.

## Inter-plugin contracts

- `requirements_output v1` — [`docs/inter-plugin-contracts/requirements-output-v1.md`](docs/inter-plugin-contracts/requirements-output-v1.md) (canonical SSOT)

## Self-write 책임 (CFP-37 ζ arc 패턴)

| Path | 책임 agent | Mechanism |
|---|---|---|
| `docs/stories/<KEY>.md §2` (도메인 분석) | RequirementsPLAgent (DomainAgent 결과 통합) | `Edit(docs/stories/**)` |
| `docs/stories/<KEY>.md §5` (요구사항 확장 해석) | RequirementsPLAgent (Analyst 결과 통합) | `Edit(docs/stories/**)` |
| `docs/stories/<KEY>.md §6` (외부 지식 배경) | RequirementsPLAgent (Researcher 결과 통합) | `Edit(docs/stories/**)` |
| `docs/domain-knowledge/<area>/<topic>.md` | DomainAgent direct (CFP-26 Phase 0a 보존) | `Edit(docs/domain-knowledge/**)` |
| GitHub comment `[요구사항]` prefix | RequirementsPLAgent | `mcp__github__add_issue_comment` |
| `phase:요구사항` → `phase:설계` transition | RequirementsPLAgent | `mcp__github__issue_write` |
| Discussions Q&A category routing | DomainAgent (선택) | `Bash(gh api repos/*/discussions*)` |

## Sub-agent 4-way 병렬 패턴

PL 이 한 메시지에 3 sub-agent 동시 dispatch (사용자 원문 verbatim Story §1 + ADR 목록 §3 + 코드 §4 공통 입력):
- Each sub-agent 가 독립 관점 (도메인 / 분석 / 외부) 도출
- "null 결과"도 valid (사유 명시)
- PL 이 dedup·상충 조정 통합

상세는 [codeforge wrapper CLAUDE.md](https://github.com/mclayer/plugin-codeforge/blob/main/CLAUDE.md) "스폰 시퀀스" 절 참조.

## Clarification 재스폰 패턴

서브에이전트 (DomainAgent · RequirementsAnalyst · Researcher) 는 one-shot 이라 PL ↔ 서브 continuous dialog 불가. PL 이 통합 중 추가 질의가 필요하면:

1. PL → Orchestrator 에 "<에이전트> 재스폰 요청 + clarification context + 이전 출력 pointer" 전달
2. Orchestrator 가 해당 에이전트를 신규 spawn (이전 출력 + 재질의 context 포함)
3. 재 spawn 결과를 PL 이 통합 재시도

**의미**: "각 책임 종료 전까지 보조" 메커니즘의 실제 구현 — 서브에이전트는 stateless 라 재 spawn 이 유일한 "clarification" 메커니즘.

**적용 lane**: 요구사항 (3 sub-agent) · 설계 (5 deputy) 양쪽 동일 패턴 — wrapper Orchestrator 가 routing.

## Domain Knowledge page schema

- **위치**: `docs/domain-knowledge/<area>/<topic>.md` (계층 구조). Consumer overlay 가 area 자유 정의
- **owner write**: DomainAgent 직접 (CFP-26 Phase 0a 후 wrapper write queue 미경유)
- **CODEOWNERS**: `docs/domain-knowledge/**` → `@org/domain-experts` 자동 review
- **template**: [`templates/domain-knowledge.md`](templates/domain-knowledge.md) (CFP-27 신설)

**Schema** (frontmatter + 본문 sections):

```yaml
---
title: <표시용 제목>
area: <area 이름>
topic_slug: <kebab-case topic>
status: draft | active | deprecated
sources:
  - <file path or URL>
related_adrs:
  - ADR-NNN
related_stories:
  - <KEY>-N
updated: YYYY-MM-DD
---
```

본문 섹션:
- `## 정의`
- `## 컨텍스트`
- `## 핵심 규칙`
- `## 경계`
- `## 관련 ADR`
- `## 변경 이력`

**검증**: wrapper repo (mclayer/plugin-codeforge) 의 `scripts/check-doc-frontmatter.sh` + `scripts/check-doc-section-schema.sh` (warning 모드 — CFP-28 strict 전환 후 fail).
