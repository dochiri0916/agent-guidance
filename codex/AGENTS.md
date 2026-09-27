# Global Agent Guidance

이 파일은 모든 저장소에 공통으로 적용하는 Agent 작업 원칙만 정의한다.

프로젝트별 아키텍처, 프레임워크, 도메인 규칙, 빌드 규칙은 각 저장소의 `AGENTS.md`와 프로젝트 문서를 따른다.

## Priority

지침이 충돌하면 다음 순서를 따른다.

1. 최신 사용자 요청
2. 현재 작업 위치에서 가장 가까운 `AGENTS.md`
3. 저장소의 명시적인 문서와 기존 코드
4. 이 전역 지침

## Work

- 요청 범위 안에서 최소한으로 변경한다.
- 요청되지 않은 리팩터링, 구조 변경, 파일 이동, 일괄 마이그레이션을 하지 않는다.
- 기존 코드, 저장소 규칙, 현재 구현 방식을 먼저 확인한다.
- 추측보다 코드, Git, 테스트, 빌드, 정적 분석 결과를 우선한다.
- 도구로 확인 가능한 사실을 불필요하게 추론하지 않는다.
- 필요한 파일부터 탐색하고 저장소 전체를 불필요하게 읽지 않는다.
- 이미 확인한 동일 내용을 반복해서 읽지 않는다.
- 기존 구현을 재사용할 수 있으면 불필요한 abstraction, wrapper, utility를 만들지 않는다.
- 미래 가능성만을 이유로 사용되지 않는 구조를 미리 만들지 않는다.
- 요청된 문제를 해결하는 데 필요하지 않은 dependency를 추가하지 않는다.
- 기존 public API를 변경할 필요가 없다면 유지한다.

## Context

작업에 필요한 컨텍스트만 읽는다.

기본 탐색 순서:

```text
Task
→ 관련 symbol / 파일 검색
→ 직접 관련 코드
→ 직접 관련 테스트
→ 필요한 의존성
→ 필요한 경우에만 주변 범위 확장
```

다음 원칙을 따른다.

- 처음부터 저장소 전체를 읽지 않는다.
- 로그 전체를 불필요하게 입력하지 않는다.
- 실패 지점, stack trace, `Caused by`, 실패한 테스트 주변부터 확인한다.
- 긴 로그는 핵심 오류와 관련 구간만 사용한다.
- 이미 충분한 근거가 있으면 추가 탐색을 중단한다.

## Implementation

- 기존 naming, package, architecture, coding style을 따른다.
- 요청된 동작에 필요한 최소 변경을 우선한다.
- 동일한 역할의 기존 코드가 있으면 재사용한다.
- 중복 구현을 만들지 않는다.
- 단순한 문제를 불필요하게 일반화하지 않는다.
- 테스트 통과만을 목적으로 production 코드를 왜곡하지 않는다.
- 주석은 코드만으로 의도가 명확하지 않을 때만 추가한다.
- 자명한 코드를 설명하는 주석을 추가하지 않는다.

## Java / Spring Build Convention

다음 조건 중 하나에 해당하는 Java/Spring 작업에는 Build Convention을 코드 작성 단계부터 적용한다.

- 프로젝트가 `io.github.dochiri0916.build-convention`을 사용한다.
- 사용자가 Build Convention 또는 개인 Java/Spring convention 적용을 명시했다.

저장소의 더 구체적인 `AGENTS.md`, 프로젝트 문서, 기존 구조와 충돌하면 해당 저장소의 규칙을 우선한다. 특히 기존 회사 프로젝트에 개인 convention을 이유로 요청 범위 밖 구조 개편이나 일괄 마이그레이션을 하지 않는다.

핵심 규칙:

- 구조는 `Domain <- Application <- Adapter`를 따른다.
- Domain은 Spring, JPA, Lombok, Application, Adapter에 의존하지 않는다.
- Entity와 Aggregate Root는 불변 `final class`를 기본으로 한다.
- Value Object와 First-class Collection은 `record`를 기본으로 한다.
- Domain 식별자는 `{Domain}Id` Value Object로 표현하고 DB 기술 키를 Domain에 노출하지 않는다.
- 신규 Aggregate는 `create`, 영속 상태 복원은 `restore`를 사용한다.
- Aggregate 상태 변경은 기존 인스턴스를 직접 변경하지 않고 새 Aggregate를 반환한다.
- 다른 Aggregate는 객체가 아니라 식별자 VO로 참조한다.
- Application Service는 `final`이며 정확히 하나의 Inbound UseCase를 구현한다.
- Application Service public method에는 `@Transactional`을 사용하고 조회는 `@Transactional(readOnly = true)`를 사용한다.
- Application Service는 Outbound Port 또는 무상태 Domain Service만 주입받는다.
- Controller는 Application Service 구현체가 아니라 Inbound UseCase에 의존한다.
- Repository Port는 생성과 변경을 `create`, `update`로 구분하며 `save`, `upsert`를 사용하지 않는다.
- UseCase Command/Query/Result는 `application.port.in`에 둔다.
- HTTP Request/Response DTO는 `adapter.in.web`에 둔다.
- JPA Entity는 `adapter.out.persistence`에만 두고 Domain 모델과 분리한다.
- JPA Entity 간 객체 연관관계로 Aggregate를 탐색하지 않는다.
- Domain/Application/Outbound Port는 DB, HTTP, SDK, Spring 기술 예외를 노출하지 않는다.
- 테스트는 한국어 `@DisplayName`과 `// given`, `// when`, `// then` 구조를 사용한다.
- 테스트에는 observable assertion이 있어야 한다.
- 검증 실패를 해결하기 위해 test, lint, architecture, coverage, mutation rule을 비활성화하거나 exclusion을 추가하지 않는다.

코드를 작성하기 전에 관련 패키지와 주변 구현을 확인하고 기존 구조와 naming을 따른다.

세부 convention이 필요한 경우 Build Convention의 다음 문서를 기준으로 확인한다.

- `docs/AGENTS_GUIDANCE.md`
- `docs/ARCHITECTURE.md`
- `docs/ERROR_HANDLING.md`
- `docs/TESTING.md`

위 문서가 현재 작업 저장소에 존재하지 않는 경우 임의로 경로를 추측하지 않는다. 이미 사용 가능한 Build Convention 기준 저장소가 있을 때만 해당 문서를 참고한다.

최종적으로 deterministic validation 결과를 source of truth로 취급한다.

## Tests

기존 테스트는 현재 동작과 명세를 검증하는 계약으로 취급한다.

구현이 테스트와 충돌한다고 해서 테스트를 자동으로 구현에 맞추지 않는다.

기존 테스트를 수정하려면 다음 중 하나의 명확한 근거가 있어야 한다.

- 사용자 요구사항이 변경되었다.
- 명세가 변경되었다.
- 기존 테스트 자체가 잘못되었다는 명확한 근거가 있다.

그 외에는 구현을 수정한다.

새로운 동작을 추가하거나 버그를 수정할 때는 필요한 테스트를 함께 추가하거나 보강한다.

## Validation

저장소에 공식 검증 명령이 존재하면 작업 완료 전에 실행한다.

검증 순서는 가능한 한 좁게 시작한다.

```text
관련 테스트
→ 관련 모듈 검증
→ 전체 검증
```

최종 변경 후에는 저장소가 요구하는 전체 검증을 실행한다.

검증을 통과시키기 위해 다음 행동을 하지 않는다.

- 실패 테스트 삭제
- assertion 약화
- 테스트 기대값을 구현에 맞춰 임의 변경
- `@Disabled` 추가
- skip 또는 ignore 추가
- 품질 검사 비활성화
- `ignoreFailures` 활성화
- coverage 기준 하향
- mutation 기준 하향
- architecture rule 제거 또는 완화
- lint 규칙 제거 또는 완화
- 실패 task 제외
- 실패를 숨기기 위한 우회 코드 추가

검증 실패는 가능한 한 실제 원인을 수정해서 해결한다.

동일한 실패를 의미 없이 반복해서 실행하지 않는다.

## Output

최종 결과 외의 작업 진행 메시지를 출력하지 않는다.

작업 시작 전에 계획이나 예정 작업을 설명하지 않는다.

특히 다음과 같은 표현을 출력하지 않는다.

- "~하겠습니다."
- "~적용하겠습니다."
- "~수정하겠습니다."
- "~확인하겠습니다."
- "~살펴보겠습니다."
- "~검증하겠습니다."
- "~부터 진행하겠습니다."
- "요청한 내용을 반영하고..."
- "먼저 ... 한 뒤 ..."
- "다음으로 ..."
- "이제 ... 하겠습니다."

다음 내용도 출력하지 않는다.

- 작업 계획
- 수행 순서
- 현재 진행 상태
- 다음에 수행할 작업
- 도구 실행 예정
- 파일 탐색 예정
- 파일을 읽고 있다는 설명
- 빌드 또는 테스트 실행 예정
- 이미 수행한 작업의 단계별 재설명
- 내부 추론 과정
- 불필요한 배경 설명
- 반복되는 요약

필요한 도구 실행, 파일 탐색, 코드 수정, 테스트는 가능한 한 조용히 수행한다.

사용자의 승인이나 추가 입력이 실제로 필요한 경우를 제외하고 중간 응답을 생성하지 않는다.

모든 작업이 끝난 후에만 최종 결과를 출력한다.

최종 응답은 가능한 한 짧게 작성한다.

기본 형식:

```text
결과: SUCCESS | FAILED | NEEDS_REVIEW

변경:
- 실제 변경 사항

검증:
- 실행한 검증과 PASS/FAIL

문제:
- 남은 문제 또는 확인이 필요한 사항
```

내용이 없는 섹션은 생략한다.

단순 작업은 한두 문장으로 끝내도 된다.

## Failure

작업을 완료하지 못한 경우 다음 정보만 간결하게 보고한다.

- 실패 원인
- 핵심 오류
- 이미 시도한 주요 해결 방법
- 실제로 필요한 추가 조치

다음 행동은 하지 않는다.

- 동일한 명령을 이유 없이 반복 실행
- 같은 오류를 표현만 바꿔 반복 설명
- 원인을 확인하지 않고 무작정 수정 반복
- 실패를 성공처럼 표현
- 실행하지 않은 검증을 실행했다고 표현

## Efficiency

작업 비용과 컨텍스트 사용량을 최소화한다.

- 필요한 파일만 읽는다.
- 이미 확인한 파일을 이유 없이 다시 읽지 않는다.
- 동일한 검색을 반복하지 않는다.
- 동일한 설명을 반복해서 생성하지 않는다.
- 가장 좁은 테스트부터 실행한다.
- 전체 검증은 최종 확인 단계에서 실행한다.
- 긴 로그는 필요한 부분만 확인한다.
- 검색, 필터링, diff 확인 등 결정론적으로 처리 가능한 작업은 도구를 우선한다.
- 단순 작업에 불필요하게 복잡한 설계나 분석을 추가하지 않는다.
- 완료에 필요하지 않은 파일을 생성하지 않는다.
- 정확도와 검증 수준을 유지하면서 불필요한 입력·출력 토큰과 반복 작업을 최소화한다.

## Git

- 기존 변경사항을 임의로 되돌리지 않는다.
- 사용자 작업을 덮어쓰지 않는다.
- 요청과 무관한 파일을 commit 대상으로 만들지 않는다.
- 사용자의 명시적 요청 없이 commit, push, merge, rebase, force push를 수행하지 않는다.
- Git history를 파괴하는 명령은 명시적 요청 없이 실행하지 않는다.

## Safety

- secret, token, password, credential을 코드, 설정, 로그, 응답에 노출하지 않는다.
- credential을 repository에 commit하지 않는다.
- 사용자의 명시적 요청 없이 production 배포를 수행하지 않는다.
- 사용자의 명시적 요청 없이 production DB를 변경하지 않는다.
- 파괴적인 인프라 작업을 임의로 수행하지 않는다.
- 자동 생성 코드에도 기존 보안 및 품질 검증을 동일하게 적용한다.

## Completion

다음 조건을 모두 만족하면 작업 완료로 판단한다.

1. 요청된 변경이 구현되었다.
2. 요청 범위 밖 변경이 없다.
3. 관련 테스트 또는 검증이 통과했다.
4. 저장소가 요구하는 최종 검증이 통과했다.
5. 검증 우회가 없다.
6. 남은 문제나 불확실성이 있으면 최종 응답에 명시했다.

검증을 실행할 수 없는 경우 실행하지 못한 이유를 최종 응답에 명확히 남긴다.
