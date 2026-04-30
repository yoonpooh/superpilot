<p align="center">
  <h1 align="center">Superpilot</h1>
  <p align="center">
    설계부터 검증까지, 외부 스킬 팩 없이 코딩 에이전트를 이끄는 독립형 워크플로우 스킬.
  </p>
  <p align="center">
    <a href="README.md">English</a> | <a href="README.ko.md">한국어</a>
  </p>
</p>

---

## Superpilot이란?

Superpilot은 기존 저장소에서 작업하는 AI 코딩 에이전트를 위한 독립형 워크플로우 스킬입니다. 협업 설계 → 자율 실행 → 강한 셀프 리뷰까지, 단일 스킬 안에서 체계적인 End-to-End 프로세스를 정의합니다.

대부분의 에이전트 워크플로우 시스템은 여러 외부 스킬을 조합해 동작합니다. Superpilot은 다릅니다. 핵심 프로세스 전체가 이 저장소 하나에 담겨 있고, 외부 의존성이 없습니다.

설계 목표는 `중간 개입 없는 완주`입니다. 외부 사실이 중요하면 먼저 조사하고, 요청된 범위를 끝까지 보존하고, 실행 토폴로지를 스스로 고르고, 실제 검증이 끝날 때까지 멈추지 않는 방향으로 설계됩니다.

## 워크플로우

```
 탐색 ──▶ 명확화 ──▶ 설계 ──▶ Spec ──▶ Plan ──▶ 실행 ──▶ 리뷰 ──▶ 검증 ──▶ 완료
```

| 단계 | 설명 |
|---|---|
| **탐색** | 저장소 맥락, 로컬 규칙(`AGENTS.md`, `CLAUDE.md`), 최근 변경사항, 현재 baseline, 격리 실행 필요 여부 파악 |
| **명확화** | 코드 기반 가정을 먼저 제시하고, 사용자가 틀린 부분만 교정하는 방식으로 질문 최소화 |
| **설계** | 필요 시 먼저 조사하고, framing과 숨은 범위를 압박 점검한 뒤, trade-off가 있는 접근법을 제안하고 하나를 추천하며 사용자와 협업 |
| **Spec** | 목표, requirement ledger, 설계, 실행 전략, 엣지 케이스, 테스트 전략을 담은 구체적인 spec 작성 |
| **Plan** | 파일 대상, ownership, workspace 전략, requirement coverage, 검증 단계를 포함한 실행 가능한 작업으로 분해 |
| **디버깅** | 버그 관련 작업인 경우: 수정 제안 전에 증거 기반으로 근본 원인 조사 |
| **실행** | TDD 원칙으로 구현 — 실패하는 테스트 먼저, 최소한의 수정, 그린 확인. Pre-edit investigation gate로 편집 전 파일 읽기 강제. 테스트 실패 시 in-branch/pre-existing/flaky로 분류 후 조사. 자율성과 diff 선명도를 위해 필요하면 workspace 격리와 fresh subagent를 사용. GREEN은 완료가 아닌 리뷰로의 mandatory transition을 트리거 |
| **리뷰** | diff 기반의 강한 리뷰 루프 + stall detection (3회 연속 findings 미감소 시 에스컬레이션). requirement 보존, schema/contract drift, threat model, UX, DevEx도 finding이 될 수 있음. 대규모 diff에는 병렬 전문가 리뷰 (Security, Performance, API Contract, Data Consistency, Test Coverage, UX/Design, DevEx) 옵션. 패치마다 fresh re-review 필수 |
| **검증** | 4단계 검증 (Exists → Substantive → Wired → Functional) + stub 감지 패턴. 필요 시 user-flow, onboarding/DX, migration/codegen, abuse-path 검증까지 포함. "아마 될 거예요"는 허용되지 않음 |

명시적 구현 요청에서는 Spec을 올바르게 작성할 수 있을 만큼 충분히 명확해지면, 원래 사용자 요청 자체를 실행 승인으로 간주합니다. 에이전트가 중간에 멈추는 경우는 실제 safety gate뿐입니다.

## 핵심 원칙

- **자기 완결** — 외부 워크플로우 스킬 불필요; 런타임 도구와 subagent를 직접 사용
- **기본값은 무중단 자율 실행** — routine approval 없이 계속 진행하고, 실제 safety gate나 중대한 모호성에서만 멈춤
- **추측보다 조사 우선** — 외부 사실이나 라이브러리 동작이 중요하면 1차 소스를 먼저 확인
- **Spec과 Plan 우선** — 비사소한 작업은 반드시 written spec과 구현 plan을 먼저 작성
- **Requirement ledger 규율** — 사용자 요청이 verification까지 추적되어 범위가 조용히 줄어들지 않음
- **TDD 기본** — 테스트 가능한 표면이 있으면 실패하는 테스트 없이 프로덕션 코드 작성 금지
- **강한 리뷰** — 코드베이스가 아닌 diff를 리뷰하고, 구현 작업이면 actionable findings를 루프 안에서 직접 패치해 0이 될 때까지 반복
- **증거 > 확신** — 검증은 관찰 가능한 증거를 만들어야 하며, 주장만으로는 불충분
- **리뷰 우선 완료** — GREEN 테스트는 완료가 아닌 리뷰를 트리거하며, 패치할 때마다 fresh re-review pass가 필요
- **실행 토폴로지 선택** — 작업 위험도와 컨텍스트 예산에 따라 in-place, isolated worktree, fresh subagent를 스스로 선택
- **상태 추적 규율** — 보이거나 저장되는 상태를 바꾸는 변경은 divergence, correction, boundary behavior를 리뷰에서 명시적으로 추적해야 함
- **실제 흐름 검증** — 사용자 플로우, onboarding, DX 변경은 실제 walkthrough나 그에 준하는 강한 증거 없이는 완료 아님
- **drift 감지** — schema, contract, generated artifact, fixture, docs drift를 “나중에 정리”가 아닌 실제 결함으로 취급
- **가정 기반 질문** — 코드베이스를 먼저 분석하고, 신뢰도 레벨과 함께 가정을 제시하여 사용자가 틀린 부분만 교정
- **범위 통제** — 추측성 리팩터링이나 관련 없는 정리 금지
- **Pre-edit investigation** — 모든 파일은 편집 전에 읽고 caller를 파악해야 함; 가정 기반 편집 금지
- **단계 전환 마커** — 각 단계에 exit marker가 정의되어 있으며, 조건 충족 전 다음 단계 진행 불가
- **컨텍스트 예산 관리** — PEAK/GOOD/DEGRADING/POOR 티어별로 행동을 자동 조절하여 출력 품질 유지
- **컨텍스트 부패 저항성** — fresh subagent, stage 요약, bounded research로 장기 작업 품질 저하를 줄임
- **에이전트 자기 복구** — 에이전트 루프, 범위 이탈, 컨텍스트 저하에 대한 구조화된 감지 및 복구 절차
- **테스트 실패 분류** — 실패를 in-branch/pre-existing/flaky로 분류한 후에 디버깅 시간 투자

## 저장소 구조

```
superpilot/
├── SKILL.md                        # 스킬 계약서 — 워크플로우, 규칙, safety gate
├── LICENSE
├── agents/
│   └── openai.yaml                 # OpenAI Codex 에이전트 인터페이스 정의
└── references/
    ├── spec.md                     # 협업 설계 및 spec 작성
    ├── plan.md                     # 구현 계획 및 subagent 분리 규칙
    ├── debugging.md                # 근본 원인 조사 절차
    ├── implementation.md           # TDD, 실행 규율, 블로커 처리
    ├── review.md                   # 강한 diff 리뷰 절차 및 체크리스트
    ├── verification.md             # 증거 기반 검증 및 완료 규칙
    └── agent-recovery.md           # 에이전트 루프, 이탈, 저하 복구
```

### 파일 역할

- **`SKILL.md`** — 메인 계약서. Superpilot 사용 시점, 전체 워크플로우, 하드 룰, 사소한 예외, 저장 경로, safety gate를 정의합니다.
- **`references/`** — 단계별 가이드. 각 파일은 워크플로우의 한 단계를 실행하는 방법을 상세히 다룹니다. 에이전트 수준의 실패 처리를 위한 자기 복구 가이드 포함.
- **`agents/openai.yaml`** — OpenAI Codex 통합을 위한 에이전트 인터페이스.

## 설치

### 1. 공용 스킬 소스

여러 에이전트 런타임에서 참조할 수 있는 공용 위치에 복사합니다:

```bash
mkdir -p ~/.agents/skills/superpilot
rsync -a ./ ~/.agents/skills/superpilot/
```

### 2. Claude Code

공용 소스에서 심링크:

```bash
mkdir -p ~/.claude/skills
ln -s ~/.agents/skills/superpilot ~/.claude/skills/superpilot
```

### 3. Codex

Codex는 `~/.codex/skills` 아래의 실제 디렉터리만 인덱싱할 수 있으므로, 심링크 대신 복사를 권장합니다:

```bash
mkdir -p ~/.codex/skills
cp -R ~/.agents/skills/superpilot ~/.codex/skills/superpilot
```

## 권장 에이전트 설정

에이전트 규칙 파일에 아래를 추가하면 Superpilot을 기본 워크플로우로 사용할 수 있습니다:

```md
# AGENTS.md 또는 CLAUDE.md

- 기존 저장소에서 비사소한 작업은 `superpilot` 스킬을 먼저 사용한다.
- research, spec, plan, TDD, 리뷰, 검증 흐름은 `superpilot`을 단일 기준으로 삼는다.
- requirement coverage 보존과 worktree/subagent 선택도 `superpilot`이 맡긴다.
- 명시적 구현 요청에서는 올바른 spec을 쓸 수 있을 만큼 명확해지면, 원래 사용자 요청을 실행 승인으로 간주한다.
```

## 적용 범위

### 주요 대상

- 버그 수정
- 기능 개발
- 실제 요구사항과 연결된 리팩터링
- 설정, CI, 워크플로우 변경
- 테스트 추가 또는 테스트 수리
- 기존 diff, 브랜치, 커밋, PR에 대한 review-only 점검

### 기본 대상 아님

- 완전 신규 프로젝트 생성
- 배포 오케스트레이션 (기본값 기준)
- 커밋 또는 PR 워크플로우 (기본값 기준)

## Spec과 Plan 저장 위치

Superpilot은 spec과 plan을 로컬 디렉터리에 저장합니다:

```
~/.superpilot/docs/<repo-name>/
├── specs/    # YYYY-MM-DD-<short-slug>.md
└── plans/    # YYYY-MM-DD-<short-slug>.md
```

`<repo-name>`은 git root의 basename입니다. Slug는 브랜치가 아닌 작업 자체를 설명합니다.

## Safety Gate

에이전트가 멈추고 사용자에게 확인하는 경우:

- 파괴적이거나 되돌릴 수 없는 작업
- 자격 증명, 시크릿, 접근 권한 부족
- 요구사항이 모순되거나 새로 안전하지 않은 경우

일상적인 실행, 계획, 구현 선택에는 사용자 승인이 필요하지 않습니다.

## 라이선스

[MIT](LICENSE)
