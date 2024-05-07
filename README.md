# Brutal Takedown Squad

## Developer Info
* 이름 : 유원석(You Won Sock)
* GitHub : https://github.com/youwonsock
* Mail : qazwsx233434@gmail.com

## Our Game
### Youtube

[![Brutal Takedown Squad](https://img.youtube.com/vi/iBa4doPBSXI/0.jpg)](https://www.youtube.com/watch?v=iBa4doPBSXI)

### Genres

TPS

<b><h2>Platforms</h2></b>

<p>
<img src="https://upload.wikimedia.org/wikipedia/commons/c/c7/Windows_logo_-_2012.png" height="30">
</p>

### Development kits

<p>
<img src="https://upload.wikimedia.org/wikipedia/commons/d/da/Unreal_Engine_Logo.svg" height="80">
</p>

<b><h2>Periods</h2></b>

* 2024-02 - 2024-04

## Contribution

### Player
* Movement
  ![walk](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/a144333b-6b29-4b5b-84c0-d380fec51dee)
  ![Run](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/dea5a989-ca5e-4193-9288-8a2218dcebbc)
  ![c_walk](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/e5daa4d4-8d07-4e1b-8b6a-736f58d7917b)
  ![jump](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/2a7d2df4-99dd-4259-aae4-e5e29fcc8ae6)
  ![turnaround](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/3ee35f60-d7cc-4857-99a3-f545848d2794)
  Player의 이동을 구현하였습니다.
  Sprint, Crouch, Jump를 따로 Ability로 분리하여 제작하였으며 태그를 지정해두면 매칭되는 태그가 들어올때 활성화됩니다.

* Input
  <details>
  <summary>InputConfig Code</summary>
  <div markdown="1">
    
  ```cpp
    #pragma once
  
    #include "CoreMinimal.h"
    #include "Engine/DataAsset.h"
    #include "GameplayTagContainer.h"
    
    #include "BTS_InputConfig.generated.h"
    
    class UInputAction;
    
    USTRUCT(BlueprintType)
    struct FBTS_InputAction
    {
    	GENERATED_BODY()
    
    	UPROPERTY(EditAnywhere, BlueprintReadWrite)
    	const UInputAction* InputAction = nullptr;
    
    	UPROPERTY(EditDefaultsOnly)
    	FGameplayTag InputTag = FGameplayTag();
    };
    
    // Ability input bindings with tags
    // 
    // Admim: YWS
    UCLASS()
    class BRUTALTAKEDOWNSQUAD_API UBTS_InputConfig : public UDataAsset
    {
    	GENERATED_BODY()
    
    public:
    	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    	TArray<FBTS_InputAction> AbilityInputActions;
    
    public:
    	const UInputAction* FindAbilityInputActionForTag(const FGameplayTag& InputTag, bool bLogNotFound = false) const;
    };

    const UInputAction* UBTS_InputConfig::FindAbilityInputActionForTag(const FGameplayTag& InputTag, bool bLogNotFound) const
    {
    	for (const FBTS_InputAction& Action : AbilityInputActions)
    	{
    		if (Action.InputAction && Action.InputTag == InputTag)
    		{
    			return Action.InputAction;
    		}
    	}
  
  	if (bLogNotFound)
  	{
  		UE_LOG(LogTemp, Error, TEXT("No input action found for tag %s"), *InputTag.ToString());
  	}
  
  	return nullptr;
    }

  ```
  
  </div>
  </details>
  </br>
  Ability와 Tag를 바인딩해주는 Class입니다.
  
  <details>
  <summary>InputComponent Code</summary>
  <div markdown="1">
    
  ```cpp
    
    #pragma once
    
    #include "CoreMinimal.h"
    #include "EnhancedInputComponent.h"
    
    #include "BTS_InputConfig.h"
    
    #include "BTS_InputComponent.generated.h"
    
    // InputComponent for ability bindings
    // Admim: YWS
    UCLASS()
    class BRUTALTAKEDOWNSQUAD_API UBTS_InputComponent : public UEnhancedInputComponent
    {
    	GENERATED_BODY()
    
    public:
    	template<class UserClass, typename PressedFuncType, typename ReleasedFuncType, typename HeldFuncType>
    	void BindAbilityActions(const UBTS_InputConfig* InputConfig, UserClass* Object, PressedFuncType PressedFunc, ReleasedFuncType ReleasedFunc, HeldFuncType HeldFunc);
    };
    
    template<class UserClass, typename PressedFuncType, typename ReleasedFuncType, typename HeldFuncType>
    inline void UBTS_InputComponent::BindAbilityActions(const UBTS_InputConfig* InputConfig, UserClass* Object, PressedFuncType PressedFunc, ReleasedFuncType ReleasedFunc, HeldFuncType HeldFunc)
    {
    	check(InputConfig);
    
    	for (const FBTS_InputAction& Action : InputConfig->AbilityInputActions)
    	{
    		if (Action.InputAction && Action.InputTag.IsValid())
    		{
    			if (PressedFunc)
    			{
    				BindAction(Action.InputAction, ETriggerEvent::Started, Object, PressedFunc, Action.InputTag);
    			}
    			if (ReleasedFunc)
    			{
    				BindAction(Action.InputAction, ETriggerEvent::Completed, Object, ReleasedFunc, Action.InputTag);
    			}
    			if (HeldFunc)
    			{
    				BindAction(Action.InputAction, ETriggerEvent::Triggered, Object, HeldFunc, Action.InputTag);
    			}
    		}
    	}
    }

  ```
  
  </div>
  </details>
  </br>
  EnhancedInput을 상속받아 실제 Input이 들어오는 Class
  
  <details>
  <summary>PlayerController Code</summary>
  <div markdown="1">
    
  ```cpp

    #pragma once
    
    #include "CoreMinimal.h"
    #include "GameFramework/PlayerController.h"
    #include "GameplayTagContainer.h"
    
    #include "BTS_PlayerController.generated.h"
    
    class UInputMappingContext;
    class UInputAction;
    class UBTS_InputConfig;
    class UBTS_AbilitySystemComponent;
    
    struct FInputActionValue;
    
    // Player Controller
    // 
    // Admin: YWS
    UCLASS()
    class BRUTALTAKEDOWNSQUAD_API ABTS_PlayerController : public APlayerController
    {
    	GENERATED_BODY()
    
    public:
    	ABTS_PlayerController();
    	virtual void PlayerTick(float DeltaTime) override;
    
    	float GetTurnRate() const { return bTurnRate; }
    
    protected:
    	virtual void BeginPlay() override;
    	virtual void SetupInputComponent() override;
    
    private:
    	//UPROPERTY(EditAnywhere, Category = "Input")
    	//TObjectPtr<UInputMappingContext> BTSContext;
    
    	UPROPERTY(EditAnywhere, Category = "Input")
    	TObjectPtr<UInputAction> MoveAction;
    
    	UPROPERTY(EditAnywhere, Category = "Input")
    	TObjectPtr<UInputAction> LookAction;
    
    	UPROPERTY(EditDefaultsOnly, Category = "Input")
    	TObjectPtr<UBTS_InputConfig> InputConfig;
    
    	UPROPERTY()
    	TObjectPtr<UBTS_AbilitySystemComponent> AbilitySystemComponent;
    
    	float bTurnRate = 0.0f;
    
    private:
    	void Move(const FInputActionValue& InputActionValue);
    
    	void Look(const FInputActionValue& InputActionValue);
    
    	void AbilityInputTagPressed(FGameplayTag InputTag);
    
    	void AbilityInputTagReleased(FGameplayTag InputTag);
    
    	void AbilityInputTagHold(FGameplayTag InputTag);
    
    	UBTS_AbilitySystemComponent* GetBTS_AbilitySystemComponent();
    };

  ```
  
  </div>
  </details>
  </br>

  PlayerController 클래스

* AIM
  ![aim](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/cc2e49ec-20d3-4e13-88b8-2d24399e2191)
  Player의 조준, 재장전, AIM Offset등을 구현하였습니다.

* Parkour
  ![parkur](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/eefdfda5-3ca6-401d-83f3-1bc85b902d65)
  ![slide](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/618e5ddc-009d-4bf4-9009-d591783cfa51)

   </div>
  </details>
  </br>
  EnhancedInput을 상속받아 실제 Input이 들어오는 Class
  
  <details>
  <summary>JumpAndMantle Code</summary>
  <div markdown="1">
    
  ```cpp

    #include "AbilitySystem/Ability/Character/BTS_CharacterJumpAndMantle.h"
    #include "Kismet/KismetSystemLibrary.h"
    #include "Character/Player/BTS_Player.h"
    #include "GameFramework/CharacterMovementComponent.h"
    #include "MotionWarpingComponent.h"
    #include "Abilities/Tasks/AbilityTask_PlayMontageAndWait.h"
    #include "Engine/CurveTable.h"
    #include "AbilitySystem/BTS_AbilitySystemComponent.h"
    
    #include "Abilities/Tasks/AbilityTask_WaitDelay.h"
    
    void UBTS_CharacterJumpAndMantle::OnGiveAbility(const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilitySpec& Spec)
    {
    	Super::OnGiveAbility(ActorInfo, Spec);
    
    	Character = Cast<ABTS_Player>(ActorInfo->AvatarActor.Get());
    }
    
    void UBTS_CharacterJumpAndMantle::OnRemoveAbility(const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilitySpec& Spec)
    {
    	Super::OnRemoveAbility(ActorInfo, Spec);
    
    	Character = nullptr;
    
    	UE_LOG(LogTemp, Warning, TEXT("UBTS_CharacterJumpAndMantle::OnRemoveAbility"));
    }
    
    void UBTS_CharacterJumpAndMantle::ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* OwnerInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData)
    {
    	UE_LOG(LogTemp, Warning, TEXT("UBTS_CharacterJumpAndMantle::ActivateAbility"));
    
    	//Super::ActivateAbility(Handle, OwnerInfo, ActivationInfo, TriggerEventData);
    
    	if (HasAuthorityOrPredictionKey(OwnerInfo, &ActivationInfo))
    	{
    		if (!CommitAbility(Handle, OwnerInfo, ActivationInfo))
    		{
    			return;
    		}
    
    		if (Character)
    		{
    			Character->Execute_SetIsAimable(Character, false);
    
    			MantleTrace();
    
    			// print MANTLE TYPE to UE_LOG
    			switch (MantleType)
    			{
    				case EMantleType::None:
    				UE_LOG(LogTemp, Warning, TEXT("MantleType : None"));
    				break;
    				case EMantleType::Mantle1M:
    					UE_LOG(LogTemp, Warning, TEXT("MantleType : Mantle1M"));
    					break;
    					case EMantleType::Mantle2M:
    						UE_LOG(LogTemp, Warning, TEXT("MantleType : Mantle2M"));
    						break;
    			}
    
    			if (MantleType != EMantleType::None)
    				Mantle(100);
    			else
    			{
    				Character->Jump();
    
    				FTimerHandle TimerHandle;
    				FTimerDelegate TimerDelegate;
    				TimerDelegate.BindLambda([this]()
    				{
    					EndAbility(CurrentSpecHandle, CurrentActorInfo, CurrentActivationInfo, true, false);
    				});
    
    				GetWorld()->GetTimerManager().SetTimer(TimerHandle, TimerDelegate, 0.5f, false);
    			}
    		}
    	}
    }
    
    bool UBTS_CharacterJumpAndMantle::CanActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayTagContainer* SourceTags, const FGameplayTagContainer* TargetTags, OUT FGameplayTagContainer* OptionalRelevantTags) const
    {
    	if (!Super::CanActivateAbility(Handle, ActorInfo, SourceTags, TargetTags, OptionalRelevantTags))
    	{
    		return false;
    	}
    
    	if (Character && Character->bIsCrouched)
    	{
    		Character->UnCrouch();
    
    		return false;
    	}
    
    	if (Character && Character->GetVelocity().Z != 0 && Character->GetCharacterMovement()->IsFalling())
    	{
    		return false;
    	}
    
    	return true;
    }
    
    void UBTS_CharacterJumpAndMantle::CancelAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, bool bReplicateCancelAbility)
    {
    	if (ScopeLockCount > 0)
    	{
    		WaitingToExecute.Add(FPostLockDelegate::CreateUObject(this, &UBTS_CharacterJumpAndMantle::CancelAbility, Handle, ActorInfo, ActivationInfo, bReplicateCancelAbility));
    		return;
    	}
    
    	Super::CancelAbility(Handle, ActorInfo, ActivationInfo, bReplicateCancelAbility);
    
    	if (Character)
    	{
    		Character->StopJumping();
    	}
    }
    
    void UBTS_CharacterJumpAndMantle::EndAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, bool bReplicateEndAbility, bool bWasCancelled)
    {
    	UE_LOG(LogTemp, Warning, TEXT("UBTS_CharacterJumpAndMantle::EndAbility"));
    
    	Super::EndAbility(Handle, ActorInfo, ActivationInfo, bReplicateEndAbility, bWasCancelled);
    
    	if (Character)
    	{
    		Character->Execute_SetIsAimable(Character, true);
    		
    		if (MantleType != EMantleType::None)
    		{
    			Character->SetActorEnableCollision(true);
    			Character->GetCharacterMovement()->SetMovementMode(EMovementMode::MOVE_Falling);
    			Character->bUseControllerRotationYaw = true;
    
    			MantleType = EMantleType::None;
    		}
    	}
    }
    
    bool UBTS_CharacterJumpAndMantle::CommitAbilityCost(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, OUT FGameplayTagContainer* OptionalRelevantTags)
    {
    	if (Super::CommitAbilityCost(Handle, ActorInfo, ActivationInfo, OptionalRelevantTags))
    		return true;
    
    	UE_LOG(LogTemp, Warning, TEXT("End by cost"));
    
    	EndAbility(Handle, ActorInfo, ActivationInfo, true, false);
    
    	return false;
    }
    
    void UBTS_CharacterJumpAndMantle::MantleTrace()
    {
    	FVector Start = Character->GetActorLocation(); Start.Z += 150;
    	Start += Character->GetActorForwardVector() * 50;
    	FVector End = Start; End.Z -= 200;
    
    	TArray<AActor*> IgnoredActors;
    	IgnoredActors.Add(Character);
    
    	// check if there is a wall
    	FHitResult CapsuleHitResult;
    	if (UKismetSystemLibrary::SphereTraceSingle(GetWorld(), Start, End, 10 // radius : 10
    		, UEngineTypes::ConvertToTraceType(ECC_Visibility), false, IgnoredActors
    		, EDrawDebugTrace::None, CapsuleHitResult, true))
    	{
    		FVector HeadPos = Character->GetMesh()->GetSocketLocation("head");
    		FVector calf = Character->GetMesh()->GetSocketLocation("calf_r");
    
    		if (Start.Z - CapsuleHitResult.Location.Z < 0.001)
    		{
    			MantleType = EMantleType::None;
    			return;
    		}
    
    		if (CapsuleHitResult.Location.Z > HeadPos.Z)
    		{
    			MantleType = EMantleType::Mantle2M;
    			CurveTable = MantleCurveTable2M;
    		}
    		else if (CapsuleHitResult.Location.Z > calf.Z)
    		{
    			MantleType = EMantleType::Mantle1M;
    			CurveTable = MantleCurveTable1M;
    		}
    		else
    		{
    			MantleType = EMantleType::None;
    			return;
    		}
    
    		FRealCurve* RichCurve = CurveTable->FindCurveUnchecked("DirOffset");
    
    		MantlePos1 = CapsuleHitResult.ImpactPoint + (Character->GetActorForwardVector() * RichCurve->Eval(1));
    		MantlePos2 = CapsuleHitResult.ImpactPoint + (Character->GetActorForwardVector() * RichCurve->Eval(2));
    		MantlePos3 = CapsuleHitResult.ImpactPoint + (Character->GetActorForwardVector() * RichCurve->Eval(3));
    		MantlePos4 = CapsuleHitResult.ImpactPoint + (Character->GetActorForwardVector() * RichCurve->Eval(4));
    	}
    	else
    		MantleType = EMantleType::None;
    }
    
    void UBTS_CharacterJumpAndMantle::Mantle(float ZOffsetLanding)
    {
    	Character->SetActorEnableCollision(false);
    	Character->GetCharacterMovement()->SetMovementMode(EMovementMode::MOVE_Flying);
    	Character->bUseControllerRotationYaw = false;
    
    	FRealCurve* RichCurve = CurveTable->FindCurveUnchecked("PosOffset");
    
    	// mantle
    	FMotionWarpingTarget Target1;
    	Target1.Name = "MantlePoint1";
    	Target1.Location = MantlePos1 + FVector(0, 0, RichCurve->Eval(1));
    	Target1.Rotation = Character->GetActorRotation();
    	Character->GetMotionWarpingComponent()->AddOrUpdateWarpTarget(Target1);
    
    	FMotionWarpingTarget Target2;
    	Target2.Name = "MantlePoint2";
    	Target2.Location = MantlePos2 + FVector(0, 0, RichCurve->Eval(2));
    	Target2.Rotation = Character->GetActorRotation();
    	Character->GetMotionWarpingComponent()->AddOrUpdateWarpTarget(Target2);
    
    	FMotionWarpingTarget Target3;
    	Target3.Name = "MantlePoint3";
    	Target3.Location = MantlePos3 + FVector(0, 0, RichCurve->Eval(3));
    	Target3.Rotation = Character->GetActorRotation();
    	Character->GetMotionWarpingComponent()->AddOrUpdateWarpTarget(Target3);
    
    	FMotionWarpingTarget Target4;
    	Target4.Name = "MantlePoint4";
    	Target4.Location = MantlePos4 + FVector(0, 0, RichCurve->Eval(4));
    	Target4.Rotation = Character->GetActorRotation();
    	Character->GetMotionWarpingComponent()->AddOrUpdateWarpTarget(Target4);
    
    	// montage
    	float PlayRate = 1.f; // The speed at which to play the montage
    	FName StartSectionName; // The section to start playing from
    
    	UAnimMontage* MantleMontage = MantleType == EMantleType::Mantle1M ? MantleMontage1M : MantleMontage2M;
    
    	// Create a 'PlayMontageAndWait' function
    	UAbilityTask_PlayMontageAndWait* PlayMontageTask = UAbilityTask_PlayMontageAndWait::CreatePlayMontageAndWaitProxy(this, EName::None, MantleMontage, PlayRate, StartSectionName);
    
    	PlayMontageTask->OnCompleted.AddDynamic(this, &UBTS_CharacterJumpAndMantle::EndJumpAndMantle);
    	PlayMontageTask->OnBlendOut.AddDynamic(this, &UBTS_CharacterJumpAndMantle::EndJumpAndMantle);
    	PlayMontageTask->OnInterrupted.AddDynamic(this, &UBTS_CharacterJumpAndMantle::EndJumpAndMantle);
    	PlayMontageTask->OnCancelled.AddDynamic(this, &UBTS_CharacterJumpAndMantle::EndJumpAndMantle);
    
    	PlayMontageTask->Activate();
    }
    
    void UBTS_CharacterJumpAndMantle::EndJumpAndMantle()
    {
    	EndAbility(CurrentSpecHandle, CurrentActorInfo, CurrentActivationInfo, true, false);
    }

  ```
  
  </div>
  </details>
  </br>
  Player의 Jump와 Mantle Ability을 구현한 Class입니다.
  AbilityTask_PlayMontageAndWait, MotionWarpingComponent을 사용하여 구현하였습니다.

### GAS
* AbilitySystem
  ![스크린샷 2024-05-08 004525](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/348a5ab7-a7fe-40ee-ad3f-84e1ee59b798)

  <details>
  <summary>AbilitySystemComponent Code</summary>
  <div markdown="1">
    
  ```cpp

    #pragma once
    
    #include "CoreMinimal.h"
    #include "AbilitySystemComponent.h"
    #include "BTS_AbilitySystemComponent.generated.h"
    
    DECLARE_MULTICAST_DELEGATE_OneParam(FEffectAssetTags, const FGameplayTagContainer&);
    
    // Feature expaneded AbilitySystemComponent
    // Admin: YWS
    UCLASS()
    class BRUTALTAKEDOWNSQUAD_API UBTS_AbilitySystemComponent : public UAbilitySystemComponent
    {
    	GENERATED_BODY()
    	
    public:
    	FEffectAssetTags EffectAssetTags;
    
    public:
    	void AbilityActorInfoSet();
    	// Rule: Always use this method to give ability to a character
    	void AddCharacterAbility(TSubclassOf<UGameplayAbility> Ability);
    
    	void AddCharacterAbilities(const TArray<TSubclassOf<UGameplayAbility>>& Abilities);
    
    	void AbilityInputTagPressed(const FGameplayTag& InputTag);
    
    	void AbilityInputTagHold(const FGameplayTag& InputTag);
    
    	void AbilityInputTagReleased(const FGameplayTag& InputTag);
    
    protected:
    
    	UFUNCTION(Client, Reliable)
    	void ClientEffectApplied(UAbilitySystemComponent* AbilitySystemComponent, const FGameplayEffectSpec& EffectSpec, FActiveGameplayEffectHandle ActiveEffectHandle);
    
    };


  ```
  
  </div>
  </details>
  </br>

  <details>
  <summary>AttributeSet Code</summary>
  <div markdown="1">
    
  ```cpp

    #pragma once
    
    #include "CoreMinimal.h"
    #include "AttributeSet.h"
    #include "AbilitySystemComponent.h"
    #include "BTS_AttributeSet.generated.h"
    
    #define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
    	GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
    	GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
    	GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
    	GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)
    
    USTRUCT()
    struct FEffectProperties
    {
    	GENERATED_BODY()
    
    	FEffectProperties() {}
    
    	FGameplayEffectContextHandle EffectContextHandle;
    
    	UPROPERTY();
    	UAbilitySystemComponent* SourceASC = nullptr;
    
    	UPROPERTY();
    	AActor* SourceAvatarActor = nullptr;
    
    	UPROPERTY();
    	AController* SourceController = nullptr;
    
    	UPROPERTY();
    	ACharacter* SourceCharacter = nullptr;
    
    	UPROPERTY();
    	UAbilitySystemComponent* TargetASC = nullptr;
    
    	UPROPERTY();
    	AActor* TargetAvatarActor = nullptr;
    
    	UPROPERTY();
    	AController* TargetController = nullptr;
    
    	UPROPERTY();
    	ACharacter* TargetCharacter = nullptr;
    };
    
    // Player AttributeSet
    // Admin: YWS
    UCLASS()
    class BRUTALTAKEDOWNSQUAD_API UBTS_AttributeSet : public UAttributeSet
    {
    	GENERATED_BODY()
    
    public:
    	UBTS_AttributeSet();
    
    	UPROPERTY(BlueprintReadOnly, ReplicatedUsing = OnRep_Health, Category = "Vital Attribures")
    	FGameplayAttributeData Health;
    	ATTRIBUTE_ACCESSORS(UBTS_AttributeSet, Health);
    
    	UPROPERTY(BlueprintReadOnly, ReplicatedUsing = OnRep_Stamina, Category = "Vital Attribures")
    	FGameplayAttributeData Stamina;
    	ATTRIBUTE_ACCESSORS(UBTS_AttributeSet, Stamina);
    
    	UPROPERTY(BlueprintReadOnly, ReplicatedUsing = OnRep_MaxHealth, Category = "Default Attribures")
    	FGameplayAttributeData MaxHealth;
    	ATTRIBUTE_ACCESSORS(UBTS_AttributeSet, MaxHealth);
    
    	UPROPERTY(BlueprintReadOnly, ReplicatedUsing = OnRep_MaxStamina, Category = "Default Attribures")
    	FGameplayAttributeData MaxStamina;
    	ATTRIBUTE_ACCESSORS(UBTS_AttributeSet, MaxStamina); 
    
    	// meta attribute
    	UPROPERTY(BlueprintReadOnly, Category = "Meta Attribures")
    	FGameplayAttributeData IncomingDamage;
    	ATTRIBUTE_ACCESSORS(UBTS_AttributeSet, IncomingDamage)
    
    	UPROPERTY(BlueprintReadOnly, Category = "Meta Attribures")
    	FGameplayAttributeData DefansePower;
    	ATTRIBUTE_ACCESSORS(UBTS_AttributeSet, DefansePower)
    
    	// ammo max value는 귀찮으니까 따로 만들지 않음
    	UPROPERTY(BlueprintReadOnly, Category = "Ammo Attribures")
    	FGameplayAttributeData Ammo_9mm;
    	ATTRIBUTE_ACCESSORS(UBTS_AttributeSet, Ammo_9mm)
    
    	UPROPERTY(BlueprintReadOnly, Category = "Ammo Attribures")
    	FGameplayAttributeData Ammo_5mm;
    	ATTRIBUTE_ACCESSORS(UBTS_AttributeSet, Ammo_5mm)
    
    	UPROPERTY(BlueprintReadOnly, Category = "Ammo Attribures")
    	FGameplayAttributeData Ammo_7mm;
    	ATTRIBUTE_ACCESSORS(UBTS_AttributeSet, Ammo_7mm)
    
    
    public:
    	UFUNCTION()
    	void OnRep_Health(const FGameplayAttributeData& OldHealth) const;
    
    	UFUNCTION()
    	void OnRep_Stamina(const FGameplayAttributeData& OldStamina) const;
    
    	UFUNCTION()
    	void OnRep_MaxHealth(const FGameplayAttributeData& OldMaxHealth) const;
    
    	UFUNCTION()
    	void OnRep_MaxStamina(const FGameplayAttributeData& OldMaxStamina) const;
    
    	virtual void GetLifetimeReplicatedProps(TArray< FLifetimeProperty >& OutLifetimeProps) const override;
    
    	virtual void PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue) override;
    
    	virtual void PostGameplayEffectExecute(const FGameplayEffectModCallbackData& Data) override;
    
    private:
    	void SetEffectProperties(FEffectProperties& Props, const FGameplayEffectModCallbackData& Data) const;
    
    	void HandleIncomingDamage(const FEffectProperties& Props);
    
    };
    
  ```
  
  </div>
  </details>
  </br>
  GAS의 핵심이 되는 두 Class입니다. AbilitySystem은 Ability와 Actor를 잇는 다리역할을 하며
  AttributeSet은 GAS에서 사용하는 float형 데이터 Attribute를 관리하는 클래스입니다.

  
* EffectActor
  ![스크린샷 2024-05-08 004711](https://github.com/youwonsock/Unreal_Team_Portfolio/assets/46276141/b58af28a-b3c8-43c2-b6f0-513f01a6ef8d)
 
  <details>
  <summary>AbilitySystemComponent Code</summary>
  <div markdown="1">
    
  ```cpp
    
    #pragma once
    
    #include "CoreMinimal.h"
    #include "GameFramework/Actor.h"
    #include "GameplayEffectTypes.h"
    #include "BTS_EffectActor.generated.h"
    
    class UGameplayEffect;
    
    UENUM(BlueprintType)
    enum class EEffectType : uint8
    {
    	None = 0 UMETA(DisplayName = "None"),
    	Instant = 1 << 1 UMETA(DisplayName = "Instant"),
    	Duration = 1 << 2 UMETA(DisplayName = "Duration"),
    	Infinite = 1 << 3 UMETA(DisplayName = "Infinite"),
    	ALL = Instant | Duration | Infinite UMETA(DisplayName = "ALL")
    };
    
    USTRUCT(BlueprintType)
    struct FCallerTagMagnitudesMap
    {
    	GENERATED_BODY()
    
    	UPROPERTY(EditAnywhere, BlueprintReadOnly)
    	TMap<FGameplayTag, float> TagAndMagnitudes;
    };
    
    // Easy to use applying and removing effects on the other actors
    // 
    // Admim: YWS
    UCLASS()
    class BRUTALTAKEDOWNSQUAD_API ABTS_EffectActor : public AActor
    {
    	GENERATED_BODY()
    	
    public:	
    	ABTS_EffectActor();
    
    	virtual void BeginPlay() override;
    
    	virtual void Tick(float DeltaTime) override;
    
    	//temp
    
    	UFUNCTION(BlueprintCallable)
    	void SetOwnerActor(AActor* NewOwner);
    
    	UFUNCTION(BlueprintCallable)
    	AActor* GetOwnerActor() { return OwnerActor; }
    
    protected:
    	UFUNCTION(BlueprintCallable)
    	void ApplyEffect(AActor* TargetActor);
    
    	UFUNCTION(BlueprintCallable)
    	void RemoveEffect(AActor* TargetActor);
    
    	UFUNCTION(BlueprintCallable)
    	void SetCallerMagnitude(FGameplayTag Tag, float Magnitude, const EEffectType EffectType);
    
    protected:
    	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Effects")
    	TMap<TSubclassOf<UGameplayEffect>, FCallerTagMagnitudesMap> InstantGameplayEffects;
    
    	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Effects")
    	TMap<TSubclassOf<UGameplayEffect>, FCallerTagMagnitudesMap> DurationGameplayEffects;
    
    	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Effects")
    	TMap<TSubclassOf<UGameplayEffect>, FCallerTagMagnitudesMap> InfiniteGameplayEffects;
    
    	TMap<FActiveGameplayEffectHandle, TSubclassOf<UGameplayEffect>> ActiveEffectHandles;
    
    	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Effects")
    	float ActorLevel = 1.0f;
    
    	// temp 
    	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Owner Actor")
    	TObjectPtr<AActor> OwnerActor;
    
    private:
    	UFUNCTION(BlueprintCallable)
    	void ApplyEffectToTarget(AActor* TargetActor, TSubclassOf<UGameplayEffect> GameplayEffectClass, FCallerTagMagnitudesMap TagMagnitudesMap);
    
    };

  ```
  
  </div>
  </details>
  </br>
  GameplayEffect를 편리하게 사용하기 위해 생성한 Class입니다.
  에디터에서 미리 값을 세팅해두거나 런타임에 함수로 SetByCaller 값을 변경하거나
  비트 연산을 통해 어떤 Type Effect를 적용시킬지 정할 수 있습니다.

### 
