# Superpilot

`superpilot`은 기존 코드베이스 작업을 위한 독립형 워크플로우 스킬입니다.

이 스킬은 코딩 에이전트가 다음 흐름으로 움직이도록 설계되어 있습니다.

- 꼭 필요한 모호성만 질문
- 비사소한 작업에는 `spec`과 `plan` 작성
- 실제 테스트 표면이 있으면 TDD 적용
- 작업 독립성이 분명할 때만 subagent 사용
- diff 기준의 강한 리뷰 루프 수행
- 완료 주장 전 새로운 검증 증거 확보

여러 외부 워크플로우 스킬 조합에 의존하는 방식과 달리, `superpilot`은 핵심 절차를 이 저장소 안에 직접 담는 것을 목표로 합니다. 런타임 도구나 subagent는 사용할 수 있지만, 핵심 프로세스 자체는 이 레포가 기준입니다.

## 철학

`superpilot`의 기본 흐름은 아래와 같습니다.

1. 저장소 맥락 탐색
2. 필요한 범위만 명확화
3. spec 작성
4. plan 작성
5. 자율 실행
6. diff 기반 강한 리뷰
7. 증거 기반 검증
8. 완료 보고

요청이 올바른 spec으로 정리될 정도로 충분히 명확해지면, 원래 사용자 요청 자체를 실행 승인으로 간주합니다. 중간에 멈추는 경우는 파괴적 작업, 자격 증명 부족, 모순된 요구사항 같은 실제 safety gate뿐입니다.

## 저장소 구조

```text
superpilot/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── debugging.md
    ├── implementation.md
    ├── plan.md
    ├── review.md
    ├── spec.md
    └── verification.md
```

## 설치

### 공용 스킬 소스

```bash
mkdir -p ~/.agents/skills
cp -R ./superpilot ~/.agents/skills/superpilot
```

### Claude Code

```bash
mkdir -p ~/.claude/skills
ln -s ~/.agents/skills/superpilot ~/.claude/skills/superpilot
```

### Codex

Codex는 `~/.codex/skills` 아래의 실제 디렉터리를 기준으로 인덱싱하는 경우가 있으니, 여기서는 심링크보다 복사를 권장합니다.

```bash
mkdir -p ~/.codex/skills
cp -R ~/.agents/skills/superpilot ~/.codex/skills/superpilot
```

## 권장 규칙

기존 저장소에서 비사소한 작업을 할 때는 `superpilot`을 기본 워크플로우 기준으로 두는 것이 좋습니다.

예시:

```md
# AGENTS.md 또는 CLAUDE.md

- 기존 저장소에서 비사소한 작업은 `superpilot` 스킬을 먼저 사용한다.
- spec, plan, TDD, 리뷰, 검증 흐름은 `superpilot`을 단일 기준으로 삼는다.
```

## 적용 범위

`superpilot`이 주로 다루는 작업:

- 버그 수정
- 기능 개발
- 실제 요구사항과 연결된 리팩터링
- 설정 변경
- 기존 저장소 안의 워크플로우/스킬 변경

`superpilot`이 기본 대상으로 삼지 않는 작업:

- 완전 신규 프로젝트 생성
- 기본값으로서의 배포 오케스트레이션
- 기본값으로서의 commit 또는 PR 워크플로우

## 라이선스

MIT
