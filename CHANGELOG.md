# Changelog

`codeforge-requirements` plugin 릴리스 이력.

버전 체계: [Semantic Versioning 2.0.0](https://semver.org/lang/ko/). v1.0 이전은 minor bump도 breaking 가능.

## [0.5.0] - 2026-05-11

### CFP-411 — Requirements lane Codex proactive check + semantic divergence debate (MINOR)

ADR-052 touchpoint #4 (RequirementsPLAgent §1~§6 완료 직후 Codex proactive check) 를 single-shot 검토에서 multi-round adversarial debate 로 격상. Story 1 (CFP-391) 의 `debate-protocol-v1` + ADR-059 + ADR-044 Amendment 1 `auto_on_divergence` 를 Requirements lane 에 transplant.

#### Added

- `agents/codex-proactive-check.md` — Codex worker entry 신설. `dispatch_mode: auto_on_divergence`. ADR-052 D2 touchpoint #4 spawn timing + debate-protocol-v1 dispatch 흐름 정의.

#### Changed

- `agents/RequirementsPLAgent.md` — 새 section "Codex Proactive Check + 의미적 divergence debate (touchpoint #4)" 추가:
  - **Semantic divergence detection 3 criteria**: AC 의미 차이 / Edge Case 누락 / Why 해석 mismatch (1개 이상 hit = divergence = true)
  - **Debate-protocol-v1 dispatch**: trigger.lane=requirements, divergence_type=semantic, min 3 / max 5 / soft default 4
  - **dispatch_mode auto_on_divergence** 우선순위 룰 정합
- `.claude-plugin/plugin.json` — version 0.4.0 → 0.5.0, description 갱신 (codex-proactive-check 추가, CFP-411 entry).

#### Why

본 lane 의 Codex proactive check 가 PL synthesis 와 의미 차이를 보일 때 single-shot 결과로 단순 PROCEED/ADDRESS_FIRST 분기만 수행했음. AC 분기·Edge Case 누락·why 해석 mismatch 같은 의미 차이는 multi-round 대화로 해소되어야 함 — Story 1 의 lane-agnostic debate-protocol-v1 을 transplant.

#### Compatibility

- **Wire**: codeforge >= 5.13.0 (wrapper ADR-052 Amendment 1 + ADR-059 + ADR-044 Amendment 1 정합)
- **Backward compat**: divergence 미검출 시 기존 ADR-052 single-shot 흐름 유지 — 새 동작은 superset
- **Sibling**: marketplace.json codeforge-requirements version 0.4.0 → 0.5.0 sync 의무 (ADR-016)

#### Related

- Story: [CFP-411](https://github.com/mclayer/plugin-codeforge/issues/392) — doc-only fast-path applied
- Wrapper PR: [#411](https://github.com/mclayer/plugin-codeforge/pull/411) merged 2026-05-11
- ADR: [ADR-052 Amendment 1](https://github.com/mclayer/plugin-codeforge/blob/main/docs/adr/ADR-052-codex-proactive-check-touchpoints.md), [ADR-059](https://github.com/mclayer/plugin-codeforge/blob/main/docs/adr/ADR-059-debate-protocol-v1.md), [ADR-044 Amendment 1](https://github.com/mclayer/plugin-codeforge/blob/main/docs/adr/ADR-044-phase-scoped-sequential-team.md)

## [0.1.0] - 2026-04-29

### CFP-37 (codeforge ζ arc) — Initial extraction (NEW)

codeforge ζ arc 세 번째 lane plugin 추출 (parent spec mclayer/plugin-codeforge CFP-31 §5.7). 4 sub-agent + 도메인 KB owner write 이전.

### Added

- `agents/RequirementsPLAgent.md` — synthesizer, 4 sub-agent 통합 + Story §2/§5/§6 self-write
- `agents/DomainAgent.md` — 도메인 KB direct write, WebFetch/WebSearch
- `agents/RequirementsAnalystAgent.md` — Bash(codex exec *) wrapper
- `agents/ResearcherAgent.md` — 외부 지식 리서치, WebFetch/WebSearch
- `templates/domain-knowledge.md` — 도메인 KB 페이지 schema (CFP-27 신설본)
- `docs/inter-plugin-contracts/requirements-output-v1.md` — canonical contract
- `overlay/hooks/{regen-agents,session-start-deps-check}.sh`
- README + CLAUDE.md

### Why

CFP-31 §5.7: Requirements lane 4 agents 가 PL 산하 병렬 패턴 + DomainAgent 의 KB owner write 가 코드 이동 첫 case (PMO 보다 큰 표면). codeforge-review v1.0 + codeforge-pmo v0.1 검증 후 진입.

### Compatibility

- **Wire**: codeforge >= 2.0.0 (wrapper 측 4 agent 삭제 + plugin install 의무 BREAKING)
- **Migration**: codeforge wrapper 와 본 plugin 동시 install 의무
- **Marketplace sync**: 본 plugin 신규 entry 등록 + codeforge wrapper version sync
