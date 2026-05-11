# Domain Knowledge 페이지 템플릿

DomainAgent가 `docs/domain-knowledge/<area>/<topic>.md` 직접 write 시 따르는 schema SSOT (CFP-26 Phase 0a 이후 owner direct write).

**사용 대상**: DomainAgent (생성·갱신 단독), DocsAgent (Story §3·§5 ADR·도메인 인용 처리 — 직접 write 안 함)

---

## 파일 위치

- **위치**: `docs/domain-knowledge/domain/<area>/<topic>.md`. `<area>`는 consumer overlay가 정의 (예: `policies/`, `accounting/`, `auth/`) — ADR-056 domain/ 서브디렉터리
- **계층**: 디렉토리 1-2단계 권장 (area / topic)
- **CODEOWNERS**: `docs/domain-knowledge/domain/**` → `@org/domain-experts` 자동 review (consumer overlay가 매핑)

---

## Frontmatter (필수)

```yaml
---
kind: domain_fact
title: <한 줄 제목>
area: <영역 — overlay에서 정의된 area 중 하나>
topic_slug: <kebab-case-slug>
status: draft | active | deprecated
sources:
  - "<원천 — 사용자 원문 / ADR / 코드 / 외부 표준 / 사내 위키 등>"
  - "<원천 2>"
related_adrs: [ADR-NNN, ADR-MMM]   # 도메인 결정과 연결되는 ADR
related_stories: [<KEY-N>, <KEY-M>] # 본 KB가 도출된 Story
updated: YYYY-MM-DD                 # 마지막 수정일 (DomainAgent가 갱신 시 업데이트)
---
```

---

## 본문 섹션 (고정 순서)

```markdown
# <Area> · <Topic>

## 정의
용어·개념의 한 줄 정의 (사전식). 비즈니스 맥락 반영.

## 컨텍스트
이 지식이 왜 필요한가, 어디서 쓰이는가 — Story·ADR·코드·운영 사례 인용.

## 핵심 규칙 / 불변식 (invariant)
- 규칙 1 (예: "한 사용자는 동시에 1개 active 세션만 보유")
- 규칙 2

## 경계 / 예외
규칙이 적용되지 않는 케이스 — 명시적 carve-out.

## 관련 ADR / Story / 코드
- [ADR-NNN](../../adr/ADR-NNN-<slug>.md) — 결정 인용
- [Story <KEY>](../../stories/<KEY>.md) — 도출 Story
- 코드 경로 (consumer 기준 relative)

## 변경 이력
- YYYY-MM-DD: 초기 작성 (Story <KEY>)
- YYYY-MM-DD: 규칙 1 추가 (Story <KEY>)
```

---

## DomainAgent 작성 절차

```
1. consumer overlay에서 area 목록 확인 (`.claude/_overlay/project.yaml` 또는 docs/domain-knowledge 기존 디렉토리)
2. 적절한 area 선택, topic-slug 결정 (kebab-case)
3. `Write(docs/domain-knowledge/domain/<area>/<topic>.md)` 호출 (ADR-056 — domain/ 서브디렉터리), frontmatter `kind: domain_fact` + 본문 작성
4. Story file §3 "관련 ADR" 또는 별도 §5 "도메인 지식" 섹션에 링크 추가 — Orchestrator 경유 DocsAgent에 의뢰 (Story file은 multi-writer)
5. 기존 page 갱신 시 frontmatter `updated` 필드 + "변경 이력" 섹션 append
```
