# Changelog

`codeforge-requirements` plugin 릴리스 이력.

버전 체계: [Semantic Versioning 2.0.0](https://semver.org/lang/ko/). v1.0 이전은 minor bump도 breaking 가능.

## [0.6.1] - 2026-05-13

### CFP-264 — FeasibilityAgent · ContinuityAgent model Opus → Sonnet (PATCH)

ADR-057 Amendment 4 + ADR-042 Amendment 6 cross-ref atomic. **path B 정합** (Codex proactive check touchpoint #4 권장 + 사용자 CL-1 확정): ResearcherAgent 제외, 3종 (Feasibility / Continuity / ChangeImpact) Sonnet 다운. ChangeImpactAgent 는 CFP-448 wave 에서 이미 Sonnet (sibling 측 NO-OP) → 본 PR 실제 변경 = 2 agent file (Feasibility / Continuity).

#### Changed

- `agents/FeasibilityAgent.md` (UPDATE line 3) — `model: claude-opus-4-7` → `claude-sonnet-4-6`. mandate text 변경 0건 — ADR-042 §결정 1 Sonnet (a) single-mandate advocacy 정합 (구현 가능성 등급 + 경고 힌트, supervisor synthesis 영역 아님).
- `agents/ContinuityAgent.md` (UPDATE line 3) — `model: claude-opus-4-7` → `claude-sonnet-4-6`. mandate text 변경 0건 — ADR-042 §결정 1 Sonnet (a) single-mandate advocacy 정합 (충돌/중복/의존 분류, cross-Story pattern detection 영역 아님 — 본 lane 단일 Story 안 기존 ADR/Story cross-ref만 수행).
- `.claude-plugin/plugin.json` — version 0.6.0 → 0.6.1 PATCH (main advance rebase 후 재bump, agent file model field 변경 + mandate text 변경 0건, ADR-037 PATCH 정합). description CFP-264 entry append.

#### NO-OP (already-Sonnet evidence)

- `agents/ChangeImpactAgent.md` — main branch 이미 `model: claude-sonnet-4-6` (CFP-448 sibling commit c4084d8). path B 정합 결과 = sibling 측 변경 0건, SSOT 측 갱신만.

#### Path B rationale (Codex proactive check touchpoint #4)

- ADR-046 §결정 4 verbatim ("Sonnet 대수 불가 — deep concept reasoning 책임") + §결정 5 동일 + §결과 §긍정 ("Sonnet 대수 가능성 제거") — ResearcherAgent Sonnet 다운 자체가 ADR 본문 핵심 정책 reject 영역. 따라서 본 Story scope 에서 Researcher 제외.
- ADR-046 변경 0건 (path B 정합 invariant).

#### Why

CFP-448 (axis-A operational cost trade-off) 의 reverse direction 2차 evidence — selective rollback 패턴 확장 (CFP-448 = 3 agent Sonnet rollback, 본 CFP-264 = 2 agent 추가 Sonnet rollback). 사용자 §1 verbatim 의 "토큰이 너무 많이 쓰여서 opus를 조금 보수적으로 써야겠다" framing 직접 적용 + ADR-042 §결정 1 Sonnet (a) single-mandate advocacy criteria 회귀.

#### Compatibility

- **Wire**: 영향 0건 — agent name / permissions / description 변경 0건, model field 만 변경.
- **In-flight Story**: 본 PR merge 후 첫 spawn 부터 Sonnet 운영 — Sonnet → Opus rate-limit fallback (ADR-057 §결정 2) 자동 적용 보호.
- **wrapper sibling**: ADR-042 Amendment 6 + ADR-057 Amendment 4 + CLAUDE.md L130 mirror + `scripts/measure-rate-limit-fallback.sh` SONNET_AGENTS 배열 11종 (8 + 3 신규: Feasibility / Continuity / ChangeImpact) 동시 갱신 의무 (atomic cross-ref). wrapper version 5.28.0 → 5.29.0 MINOR (main advance rebase 후 재bump).
- **marketplace sync**: ADR-063 atomic invariant — wrapper + requirements bump 정합 marketplace.json mirrored field sync PR follow-up.

## [0.6.0] - 2026-05-13

### CFP-510 — RequirementsPLAgent divergence detection 4 영역 확장 (MINOR)

wrapper plugin-codeforge ADR-052 Amendment 3 sibling sync. CFP-411 (Amendment 1) 의 semantic 3 criteria 에 **4번째 영역 = fact-check** 추가. PL self-evaluation 의무 = synthesis fact claim 영역 marker 5종 (`[verified]` / `[hypothesis]` / `[fact-check-pending]` / `[user-input]` / `[verification-out-of-scope: <사유>]`). MINOR bump (mandate text 영역 확장 — agent definition signature 영역).

#### Changed

- `agents/RequirementsPLAgent.md` — "Divergence detection 3 criteria" → "Divergence detection 4 영역 (3 semantic + 1 factual)" 단락 확장 + "PL self-evaluation 의무 — fact claim marker 5종" 단락 신설 + Debate-protocol-v1 dispatch divergence_type 분류 (semantic / factual + polyfill) 명시.
- `agents/codex-proactive-check.md` — "dispatch_mode: auto_on_divergence" 단락 divergence 4 영역 명시 + "Fact-check 영역 (ADR-052 Amendment 3 / CFP-510)" 단락 신설 (4 sub-criteria 검증 방법 표 + PL marker 5종 인지 의무).
- `.claude-plugin/plugin.json` — version 0.5.1 → 0.6.0 MINOR. description CFP-510 entry append.

#### Why

axis-A (mandate 영역 확장 — fact-check 영역 명시): semantic 3 criteria implicit 영역 → 4 영역 explicit normative anchor. axis-B (PL synthesis quality — marker 5종 forcing function): 가설 vs verified 영역 구분 의무 부재 → false negative 차단. axis-C (Codex worker mandate 정합): codex-proactive-check.md 가 RequirementsPLAgent marker 5종 cross-verify 의무 명시.

#### Compatibility

- canonical: wrapper `plugin-codeforge` 5.24.0 (PR Phase 1 sibling sync — ADR-010 §단계 절차 wrapper-first 정합)
- Effective date: 본 PR merged + marketplace sync PR merged + consumer `/plugins install` 후 (ADR-053 structural change restart 적용 영역)

## [0.5.1] - 2026-05-12

### CFP-448 — ChangeImpactAgent Opus → Sonnet rollback (PATCH)

ADR-057 Amendment 3 (wrapper plugin-codeforge PR #488 merged, 2026-05-12) selective rollback 의 sibling sync. CFP-379 (Amendment 4) 의 6 agent Opus 상향 중 3 agent (ChangeImpactAgent / CodebaseMapperAgent / RefactorAgent) Sonnet 복귀 — 본 lane plugin 은 ChangeImpactAgent 만 해당.

#### Changed

- `agents/ChangeImpactAgent.md` `model:` field `claude-opus-4-7` → `claude-sonnet-4-6`. mandate text 변경 0건 — ChangeImpactAgent 는 ADR-042 §결정 2 invariant 자연 정합 (AS-IS → DELTA structured mapping = single-source map, advocacy/synthesis pattern 아님). exclusion criterion 정합 (CFP-448 §5.3 EC-5 universal mandate align).
- `.claude-plugin/plugin.json` — version 0.5.0 → 0.5.1 PATCH (model field 단순 변경, mandate / contract / agent definition signature 영역 변경 0건). description CFP-448 entry append.

#### Why

axis-A (operational cost) — ADR-042 original Sonnet 분류 정합 회복 (CFP-379 → CFP-448 1 주일 운영 evidence). axis-B (mandate 깊이) — single-mandate structured output (advocacy/synthesis pattern 아님) 으로 Sonnet sufficient cover. axis-C (SSOT alignment) — CLAUDE.md L127 8종 정합 회복 (CL-6 사용자 확정 Option (i)).

#### Compatibility

- **Wire**: codeforge wrapper >= 5.22.1 (Phase 2 PR pair atomic — wrapper 5.22.1 + design 0.7.0 + 본 0.5.1 + Story §8 internal-docs).
- **Codex re-review**: 면제 — mandate text 변경 0건 (Story §5.3 EC-2 정합). 단순 model tier rollback.
- **ADR-053 재구동**: agent definition 변경 = 구조적 변경. consumer 측 `/plugins install codeforge-requirements@mclayer` 의무.

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
