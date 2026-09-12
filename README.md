# Brutal Takedown Squad

Unreal Engine 5의 Gameplay Ability System(GAS)을 기반으로 캐릭터 액션과 전투 시스템을 구현한 팀 포트폴리오 프로젝트입니다.

## 목차

- [프로젝트 개요](#프로젝트-개요)
- [프로젝트 요약](#프로젝트-요약)
- [사용 기술](#사용-기술)
- [클래스 구조 UML](#클래스-구조-uml)
- [담당 기능 상세](#담당-기능-상세)
- [빌드 및 실행](#빌드-및-실행)

## 프로젝트 개요

| 항목 | 내용 |
| --- | --- |
| 프로젝트 형태 | 팀 프로젝트 |
| 담당 개발자 | 유원석 (You Won Sock) |
| GitHub | [youwonsock](https://github.com/youwonsock) |
| 이메일 | qazwsx233434@gmail.com |
| 개발 기간 | 2024.02 ~ 2024.04 |
| 프로젝트 목적 | GAS 기반 TPS 캐릭터 액션·전투 시스템 설계 및 팀 단위 게임 제작 |
| 담당 영역 | 플레이어 이동·조준·파쿠르, 입력-어빌리티 연결, Ability System·Attribute·Gameplay Effect |
| 개발 언어 | C++, Blueprint |
| 게임 엔진 | Unreal Engine 5.3 |
| 대상 플랫폼 | Windows |
| 시연 영상 | [YouTube](https://www.youtube.com/watch?v=iBa4doPBSXI) |
| 실행 파일 | [Google Drive](https://drive.google.com/file/d/1tmAbgV77TfjLUCJOEMOyjTgP3fa2sdRQ/view?usp=drive_link) |

## 프로젝트 요약

Brutal Takedown Squad는 캐릭터 이동, 조준, 사격, 파쿠르와 상호작용을 결합한 3인칭 액션 게임입니다. 팀 프로젝트에서 플레이어와 GAS 영역을 담당해 개별 액션을 `GameplayAbility`로 분리하고, Gameplay Tag를 중심으로 입력과 능력을 연결했습니다.

담당 시스템의 주요 흐름은 다음과 같습니다.

1. `UBTS_InputConfig`가 `InputAction`과 Gameplay Tag의 대응 관계를 데이터로 보관합니다.
2. `UBTS_InputComponent`와 `ABTS_PlayerController`가 입력의 Started·Triggered·Completed 상태를 ASC에 전달합니다.
3. `UBTS_AbilitySystemComponent`가 입력 Tag와 일치하는 Ability를 찾아 활성화하거나 입력 상태를 갱신합니다.
4. Sprint, Crouch/Slide, ADS, Jump/Mantle 등의 Ability가 실행 조건·비용·종료 처리를 각각 관리합니다.
5. `UBTS_AttributeSet`이 체력·스태미나·방어력·탄약과 피해 계산을 처리하고 네트워크에 주요 Attribute를 복제합니다.
6. `ABTS_EffectActor`가 Instant·Duration·Infinite Gameplay Effect의 적용과 제거를 공통화합니다.

[![Brutal Takedown Squad 시연 영상](https://img.youtube.com/vi/iBa4doPBSXI/0.jpg)](https://www.youtube.com/watch?v=iBa4doPBSXI)

## 사용 기술

### Unreal Engine 5

Character, PlayerController, PlayerState와 컴포넌트 생명주기를 기반으로 플레이어를 구성했습니다. C++로 핵심 로직과 확장 지점을 작성하고, 카메라 전환·HUD 반응처럼 디자이너 조정이 필요한 부분은 Blueprint 이벤트로 연결했습니다.

### Gameplay Ability System

이동과 전투 행동을 독립적인 `GameplayAbility`로 분리하고 활성 조건, 비용 반영, 입력 해제, 취소와 종료 처리를 각 Ability 안에서 관리합니다. `AbilitySystemComponent`, `AttributeSet`, Gameplay Effect와 Gameplay Tag를 조합해 캐릭터 상태와 능력 실행을 연결했습니다.

### Enhanced Input / Gameplay Tag

입력 코드에 Ability 클래스를 직접 연결하지 않고 `InputAction → Gameplay Tag → GameplayAbility` 흐름을 구성했습니다. 같은 입력 처리 경로에서 Pressed·Held·Released 상태를 전달하므로 Ability를 추가하거나 입력을 변경할 때 결합 범위를 줄일 수 있습니다.

### Motion Warping / Animation Montage

Sphere Trace로 장애물 높이를 판정해 1m·2m Mantle을 구분하고, Curve Table로 계산한 목표 지점과 Motion Warping Target을 사용해 애니메이션과 실제 이동을 맞췄습니다. Slide와 Mantle의 완료·중단·취소 이벤트는 Ability 종료 처리로 연결했습니다.

### Network Replication

PlayerState가 플레이어의 ASC와 AttributeSet을 소유하고, 서버의 Possess 시점에 Actor Info와 기본 Attribute·Ability를 초기화합니다. Health, Stamina와 최대값은 RepNotify로 동기화하고 Gameplay Effect 처리 결과를 캐릭터 반응과 HUD에 전달합니다.

## 클래스 구조 UML

클래스 구조의 원본은 [`UML.plantuml`](UML.plantuml)에서 관리합니다.

> [UML 이미지 플레이스홀더 — `UML.plantuml`을 렌더링한 다이어그램 추가 예정]

### 핵심 관계

- `ABTS_Player`는 `ABTS_CharacterBase`를 상속하고 카메라·SpringArm·MotionWarping 컴포넌트를 구성합니다.
- `ABTS_PlayerState`가 `UBTS_AbilitySystemComponent`와 `UBTS_AttributeSet`을 소유합니다.
- `ABTS_PlayerController`는 `UBTS_InputComponent`에서 받은 Gameplay Tag 입력을 ASC에 전달합니다.
- `UBTS_AbilitySystemComponent`는 Tag와 일치하는 `UBTS_GameplayAbility`를 활성화하고 입력 상태를 관리합니다.
- `UBTS_AttributeSet`은 Gameplay Effect 적용 결과를 Attribute, 피격 반응과 사망 처리로 연결합니다.
- 캐릭터별 Ability는 `UBTS_CharacterGameplayAbility`를 기반으로 이동·조준·전투 행동을 독립적으로 구현합니다.

### 클래스별 역할

- `ABTS_Player`: 플레이어의 카메라, 회전 방식, Motion Warping과 Ability Actor Info 초기화를 담당합니다.
- `ABTS_PlayerController`: 이동·시점 입력을 처리하고 Ability 입력 Tag를 ASC로 전달합니다.
- `UBTS_InputConfig`: `InputAction`과 Gameplay Tag의 매핑 데이터를 관리합니다.
- `UBTS_InputComponent`: Enhanced Input 이벤트를 Pressed·Held·Released Ability 콜백에 바인딩합니다.
- `UBTS_AbilitySystemComponent`: Ability 부여, Tag 기반 탐색·활성화와 Effect Tag 알림을 담당합니다.
- `UBTS_AttributeSet`: 체력·스태미나·방어력·탄약 값을 관리하고 피해·복제 로직을 처리합니다.
- `ABTS_EffectActor`: 수명 유형별 Gameplay Effect 적용·제거와 SetByCaller 값을 공통 처리합니다.
- `UBTS_CharacterJumpAndMantle`: 장애물 높이 판정, Mantle Target 계산과 Montage 실행을 담당합니다.

## 담당 기능 상세

### Gameplay Tag 기반 입력·Ability 연결

**목적**

입력 처리 코드와 개별 Ability의 직접 의존을 줄이고 동일한 경로에서 입력 상태를 전달합니다.

**핵심 구현**

- `UBTS_InputConfig`의 배열에 `InputAction`과 Gameplay Tag를 데이터로 등록합니다.
- `UBTS_InputComponent`가 Started·Triggered·Completed 이벤트를 공통 템플릿 함수로 바인딩합니다.
- `ABTS_PlayerController`가 Pressed·Held·Released 입력을 `UBTS_AbilitySystemComponent`로 전달합니다.
- ASC가 `DynamicAbilityTags`에서 일치하는 Tag를 찾아 입력 상태 갱신과 `TryActivateAbility`를 수행합니다.

> [스크린샷 플레이스홀더 — InputAction·Gameplay Tag 설정]

### 플레이어 이동 및 상태 전환

**목적**

이동, Sprint, Crouch와 Slide를 Ability 단위로 분리하고 상태와 스태미나에 따라 일관되게 전환합니다.

**핵심 구현**

- 카메라 Yaw를 기준으로 전·후·좌·우 이동 방향을 계산합니다.
- Sprint 시작 시 이동 속도와 애니메이션 상태를 변경하고 조준을 제한합니다.
- 입력 해제 또는 스태미나 소진 시 Sprint Effect를 제거하고 기본 이동 상태로 복귀합니다.
- Crouch 입력 시 현재 속도에 따라 일반 Crouch와 Slide를 분기합니다.
- Slide Montage의 완료·중단·취소 이벤트에서 이동 속도와 조준 가능 상태를 복원합니다.

![걷기](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/a144333b-6b29-4b5b-84c0-d380fec51dee)

![달리기](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/dea5a989-ca5e-4193-9288-8a2218dcebbc)

![앉아서 이동](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/e5daa4d4-8d07-4e1b-8b6a-736f58d7917b)

![슬라이딩](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/618e5ddc-009d-4bf4-9009-d591783cfa51)

### 조준 및 전투 Ability

**목적**

조준, 사격, 재장전과 근접 공격의 실행 조건을 Ability로 분리해 다른 이동 상태와 충돌하지 않도록 합니다.

**핵심 구현**

- ADS Ability가 캐릭터의 조준 가능 상태를 확인한 뒤 Blueprint 카메라·UI 연출을 실행합니다.
- Shoot·Reload·Melee Attack을 별도 Ability로 분리해 입력과 실행 조건을 독립적으로 관리합니다.
- Sprint·Slide·Mantle 실행 중에는 조준 가능 상태를 변경해 동시에 실행되면 안 되는 행동을 제한합니다.
- 입력 해제와 Ability 취소 경로에서 조준·전투 상태를 정리합니다.

![조준](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/cc2e49ec-20d3-4e13-88b8-2d24399e2191)

> [스크린샷 플레이스홀더 — 사격·재장전 Ability 동작]

### Jump 및 Mantle

**목적**

일반 Jump와 장애물 Mantle을 하나의 입력 흐름에서 판정하고 장애물 높이에 맞는 동작을 재생합니다.

**핵심 구현**

- Sphere Trace와 캐릭터 소켓 높이를 비교해 일반 Jump, 1m Mantle, 2m Mantle을 구분합니다.
- Curve Table에서 방향·높이 보정값을 읽어 네 개의 Motion Warping Target을 계산합니다.
- Mantle 중 Collision과 Movement Mode를 전환하고 Montage 종료 시 원래 상태를 복원합니다.
- 장애물이 없으면 일반 Jump를 실행하고 짧은 대기 후 Ability를 종료합니다.

![점프](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/2a7d2df4-99dd-4259-aae4-e5e29fcc8ae6)

![Mantle](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/eefdfda5-3ca6-401d-83f3-1bc85b902d65)

### Ability System 및 Attribute

**목적**

캐릭터 Ability의 부여·입력 처리와 체력·스태미나·피해 데이터를 하나의 확장 가능한 시스템으로 관리합니다.

**핵심 구현**

- Ability를 부여할 때 `StartupInputTag`를 `DynamicAbilityTags`에 등록합니다.
- PlayerState를 Owner Actor, Player를 Avatar Actor로 초기화해 플레이어 정보와 물리적 표현을 분리합니다.
- Health, Stamina, MaxHealth와 MaxStamina를 RepNotify로 복제합니다.
- `PreAttributeChange`와 `PostGameplayEffectExecute`에서 값 범위, 방어력, 피해와 탄약을 처리합니다.
- 피해 적용 후 생존 여부에 따라 Hit React Ability 또는 사망 처리를 실행합니다.

![Ability System](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/348a5ab7-a7fe-40ee-ad3f-84e1ee59b798)

### Gameplay Effect 공통 처리

**목적**

아이템과 환경 오브젝트에서 여러 유형의 Gameplay Effect를 반복 코드 없이 적용하고 제거합니다.

**핵심 구현**

- Effect를 Instant·Duration·Infinite 수명 유형으로 구분해 에디터에서 설정합니다.
- 대상 Actor의 ASC를 찾아 Effect Context와 Spec을 생성하고 적용합니다.
- `SetByCaller` Tag와 Magnitude Map으로 런타임 수치를 Effect Spec에 전달합니다.
- Infinite Effect의 Handle을 보관하고 대상과 Effect 클래스가 일치하는 활성 Effect를 제거합니다.

![Gameplay Effect](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/b58af28a-b3c8-43c2-b6f0-513f01a6ef8d)

## 빌드 및 실행

### 요구 환경

- Unreal Engine 5.3
- Visual Studio 2022
- Game development with C++ 워크로드
- Windows 10/11 SDK

프로젝트는 `GameplayAbilities`, `GameplayTags`, `GameplayTasks`, `EnhancedInput`, `MotionWarping`, `Niagara`, `AIModule`을 사용합니다. `.uproject`와 모듈 설정에 필요한 의존성이 선언되어 있습니다.

### Unreal Editor

1. 저장소를 clone합니다.
2. [`BrutalTakedownSquad.uproject`](BrutalTakedownSquad.uproject)를 Unreal Engine 5.3으로 엽니다.
3. C++ 모듈 빌드가 필요하다는 안내가 표시되면 빌드를 진행합니다.
4. 에디터에서 프로젝트 맵을 연 뒤 Play로 실행합니다.

### Visual Studio

1. `BrutalTakedownSquad.uproject`를 우클릭해 Visual Studio 프로젝트 파일을 생성합니다.
2. 생성된 `BrutalTakedownSquad.sln`을 엽니다.
3. `Development Editor | Win64` 구성으로 빌드합니다.
4. 빌드가 완료되면 `.uproject`를 열어 실행합니다.

패키징된 실행 파일은 프로젝트 개요의 [Google Drive 링크](https://drive.google.com/file/d/1tmAbgV77TfjLUCJOEMOyjTgP3fa2sdRQ/view?usp=drive_link)에서 확인할 수 있습니다.
