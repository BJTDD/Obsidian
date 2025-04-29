```cpp
// Fill out your copyright notice in the Description page of Project Settings.


#include "PlayerCharacterControll.h"
#include "Components/SkeletalMeshComponent.h"
#include "GameFramework/SpringArmComponent.h"
#include "Camera/CameraComponent.h"
#include "EnhancedInputComponent.h"
#include "EnhancedInputSubsystems.h"
#include "InputMappingContext.h"
#include "Components/StaticMeshComponent.h"
#include "PlayerAnimInstance.h"
#include "Kismet/GameplayStatics.h"
#include "Engine/DamageEvents.h"
#include "Blueprint/UserWidget.h"
#include "Components/Image.h"

// Sets default values
APlayerCharacterControll::APlayerCharacterControll()
{
 	// Set this character to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;


	ConstructorHelpers::FObjectFinder<USkeletalMesh>PlayerMesh(TEXT("/Script/Engine.SkeletalMesh'/Game/Asset/Player/Survival_Character/Meshes/SK_Survival_Character.SK_Survival_Character'"));
	if (PlayerMesh.Succeeded())
	{
		GetMesh()->SetSkeletalMesh(PlayerMesh.Object);
		GetMesh()->SetWorldLocationAndRotation(FVector(0, 0, -90), FRotator(0, -90, 0));
	}
	
	SpringArm = CreateDefaultSubobject<USpringArmComponent>(TEXT("SpringArm"));
	SpringArm->SetupAttachment(RootComponent);
	SpringArm->SetWorldLocation(FVector(0, 0, 65.0f));
	SpringArm->TargetArmLength = 150;
	SpringArm->SocketOffset = FVector(0, 40, 0);
	SpringArm->bUsePawnControlRotation = true;

	Camera = CreateDefaultSubobject<UCameraComponent>(TEXT("Camera"));
	Camera->SetupAttachment(SpringArm);
	
	static ConstructorHelpers::FClassFinder<UAnimInstance> AnimInstance(TEXT("/Game/Blueprints/ABP_Player.ABP_Player_C"));

	if (AnimInstance.Class)
	{
		GetMesh()->SetAnimInstanceClass(AnimInstance.Class);
	}
	InitializationFindWeaponMesh();
	InitializationIsWeaponMap();
	InitializationFindInput();
	
}

void APlayerCharacterControll::BeginPlay()
{
	Super::BeginPlay();

	InitializationFindWidget();


	if (APlayerController* PlayerController = Cast<APlayerController>(GetController()))
	{
		if (UEnhancedInputLocalPlayerSubsystem* SubSystem = ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(PlayerController->GetLocalPlayer()))
		{
			SubSystem->AddMappingContext(DefaultContext, 0);
		}
	}
	//SM_Weapon->bHiddenInGame = true;
	SM_RifleWeapon->SetVisibility(false);
	SM_PistolWeapon->SetVisibility(false);
	if (!HUDWidget && HUDClass)
	{
		HUDWidget = CreateWidget(GetWorld()->GetFirstPlayerController(), HUDClass);
	}
	if (HUDWidget)
	{
		HUDWidget->AddToViewport();

		CrossHairImage = Cast<UImage>(HUDWidget->GetWidgetFromName(TEXT("CrosshairImage")));
		WeaponIconImage = Cast<UImage>(HUDWidget->GetWidgetFromName(TEXT("WeaponIconImage")));
		BulletCountTextBlock = Cast<UTextBlock>(HUDWidget->GetWidgetFromName(TEXT("BulletText")));
		if (CrossHairImage)
		{
			CrossHairImage->SetVisibility(ESlateVisibility::Hidden);
		}
		if (WeaponIconImage)
		{
			WeaponIconImage->SetVisibility(ESlateVisibility::Hidden);
		}
		if (BulletCountTextBlock)
		{
			BulletCountTextBlock->SetVisibility(ESlateVisibility::Hidden);
		}
	}

}

void APlayerCharacterControll::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

void APlayerCharacterControll::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);
	UEnhancedInputComponent* EnhancedInputComponent = Cast<UEnhancedInputComponent>(PlayerInputComponent);
	EnhancedInputComponent->BindAction(MoveAction, ETriggerEvent::Triggered, this, &APlayerCharacterControll::Move);
	EnhancedInputComponent->BindAction(LookAction, ETriggerEvent::Triggered, this, &APlayerCharacterControll::Look);
	EnhancedInputComponent->BindAction(AimAction, ETriggerEvent::Started, this, &APlayerCharacterControll::OnAim);
	EnhancedInputComponent->BindAction(AimAction, ETriggerEvent::Completed, this, &APlayerCharacterControll::OffAim);
	EnhancedInputComponent->BindAction(FireAction, ETriggerEvent::Started, this, &APlayerCharacterControll::Fire);
	EnhancedInputComponent->BindAction(OperateAction, ETriggerEvent::Started, this, &APlayerCharacterControll::Operate);
	EnhancedInputComponent->BindAction(RifleChangeAction, ETriggerEvent::Started, this, &APlayerCharacterControll::RifleMode);
	EnhancedInputComponent->BindAction(PistolChangeAction, ETriggerEvent::Started, this, &APlayerCharacterControll::PistolMode);
}

void APlayerCharacterControll::Move(const FInputActionValue& value)
{
	const FVector2D MovementVector = value.Get<FVector2D>();
	const FRotator Rotation = Controller->GetControlRotation();
	const FRotator YawRotation(0, Rotation.Yaw, 0);

	const FVector ForwardDiretion = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X);
	const FVector RightDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y);

	AddMovementInput(ForwardDiretion, MovementVector.Y);
	AddMovementInput(RightDirection, MovementVector.X);
}

void APlayerCharacterControll::Look(const FInputActionValue& value)
{
	FVector2D LookAxisVector = value.Get<FVector2D>();

	AddControllerYawInput(LookAxisVector.X * GetWorld()->DeltaTimeSeconds * mouseSpeed);
	AddControllerPitchInput(LookAxisVector.Y * GetWorld()->DeltaTimeSeconds * -mouseSpeed);
}

void APlayerCharacterControll::OnAim()
{
	if(IsWeaponMap.Num() > 0)
	{
		if (CrossHairImage)
		{
			CrossHairImage->SetVisibility(ESlateVisibility::Visible);
		}
		if (WeaponIconImage)
		{
			WeaponIconImage->SetVisibility(ESlateVisibility::Visible);
		}
		if (BulletCountTextBlock)
		{
			BulletCountTextBlock->SetVisibility(ESlateVisibility::Visible);
		}

		
		if (IsWeaponMap.Contains(EWeaponType::Rifle) && currentWeapon == EWeaponType::Rifle)
		{
			OnAim_BP();
			UPlayerAnimInstance* animInstace = Cast<UPlayerAnimInstance>(GetMesh()->GetAnimInstance());
			isAim = true;
			animInstace->isAim = true;
			CurrentBulletCount = RifleBulletCount;
			//무기상태를 애니인스턴스한테 전달하기
		}
		else if (IsWeaponMap.Contains(EWeaponType::Pistol) && currentWeapon == EWeaponType::Pistol)
		{
			OnAim_BP();
			UPlayerAnimInstance* animInstace = Cast<UPlayerAnimInstance>(GetMesh()->GetAnimInstance());
			isAim = true;
			animInstace->isAim = true;
			CurrentBulletCount = PistolBulletCount;
			//무기상태를 애니인스턴스한테 전달하기
		}
		else if (IsWeaponMap.Contains(EWeaponType::SMG) && currentWeapon == EWeaponType::SMG)
		{
			OnAim_BP();
			UPlayerAnimInstance* animInstace = Cast<UPlayerAnimInstance>(GetMesh()->GetAnimInstance());
			isAim = true;
			animInstace->isAim = true;
			CurrentBulletCount = SMGBulletCount;
			//무기상태를 애니인스턴스한테 전달하기
		}
	}


}

void APlayerCharacterControll::OffAim()
{
	if (CrossHairImage)
	{
		CrossHairImage->SetVisibility(ESlateVisibility::Hidden);
	}
	if (WeaponIconImage)
	{
		WeaponIconImage->SetVisibility(ESlateVisibility::Hidden);
	}
	if (BulletCountTextBlock)
	{
		BulletCountTextBlock->SetVisibility(ESlateVisibility::Hidden);
	}
	OffAim_BP();
	UPlayerAnimInstance* animInstace = Cast<UPlayerAnimInstance>(GetMesh()->GetAnimInstance());
	animInstace->isAim = false;
	isAim = false;
}

void APlayerCharacterControll::Fire()
{
	if (isAim)
	{
		if (currentWeapon == EWeaponType::Rifle && RifleBulletCount > 0)
		{
			CurrentBulletCount--;
			RifleBulletCount = CurrentBulletCount;
			
			if (RifleFireCameraShakeClass)
			{
				GetWorld()->GetFirstPlayerController()->ClientStartCameraShake(RifleFireCameraShakeClass);
			}
		}
		else if (currentWeapon == EWeaponType::Pistol && PistolBulletCount > 0)
		{
			CurrentBulletCount--;
			PistolBulletCount = CurrentBulletCount;
		}
		else if (currentWeapon == EWeaponType::SMG && SMGBulletCount > 0)
		{
			CurrentBulletCount--;
			SMGBulletCount = CurrentBulletCount;
		}
		
		OnFire_BP();
		FHitResult Hit;
		FVector StartTrace = Camera->GetComponentLocation();
		FVector EndTrace = StartTrace + (Camera->GetForwardVector() * 1000);
		//DrawDebugLine(GetWorld(), StartTrace, EndTrace, FColor::Green, false, 5.0f);
		if (GetWorld()->LineTraceSingleByChannel(Hit, StartTrace, EndTrace, ECC_GameTraceChannel4))
		{
			Hit.GetActor()->Destroy();
		}

		UAnimInstance* AnimInstace = GetMesh()->GetAnimInstance();
		if (RifleFireMontage)
		{
			AnimInstace->Montage_Play(RifleFireMontage, 0.233333f);
		}
		FTimerHandle HitSoundDelayHandle;
		GetWorld()->GetTimerManager().SetTimer(HitSoundDelayHandle, this,
			&APlayerCharacterControll::WeaponFireSound,
			0.2f,
			false
		);
	}
	
}

void APlayerCharacterControll::Operate()
{
	if (!isItem)
	{
		return;
	}
	Operate_BP();
	
	FVector StartLocation = GetMesh()->GetSocketLocation("hand_r"); //손 위치 소켓 
	FVector EndLocation = StartLocation;
	float SphereRadius = 70.0f; //원 크기
	FHitResult HitResult;
	FCollisionQueryParams Params;
	Params.AddIgnoredActor(this); //자기 자신을 무시
	bool isHit = GetWorld()->
		SweepSingleByChannel(HitResult, StartLocation, EndLocation, FQuat::Identity, 
			ECC_GameTraceChannel8, FCollisionShape::MakeSphere(SphereRadius), Params);
	//DrawDebugSphere(GetWorld(), StartLocation, SphereRadius, 12, FColor::Red, false, 1.0f);
	if (isHit)
	{
		FString name = HitResult.GetActor()->GetName();
		name = name.LeftChop(name.Len() - name.Find(TEXT("_C_1"))); //10 - 6 = 4
		name = name.RightChop(3);
		//GEngine->AddOnScreenDebugMessage(-1, 3.0f, FColor::Blue, FString::Printf(TEXT("%s"), *name));


		if (name == TEXT("WeaponPistol"))
		{
			IsWeaponMap[EWeaponType::Pistol] = true;
		}
		else if (name == TEXT("WeaponSMG"))
		{
			IsWeaponMap[EWeaponType::SMG] = true;
		}
		else if (name == TEXT("WeaponRifle"))
		{
			IsWeaponMap[EWeaponType::Rifle] = true;
		}
		else if (name == TEXT("BP_RifleBullet"))
		{
			RifleBulletCount += 30;
			if (RifleBulletCount >= 200)
			{
				RifleBulletCount = 200;
			}
		}
		else if (name == TEXT("BP_PistolBullet"))
		{
			PistolBulletCount += 15;
			if (PistolBulletCount >= 120)
			{
				PistolBulletCount = 120;
			}
		}
		else if (name == TEXT("BP_SMGBullet"))
		{
			SMGBulletCount += 30;
			if (SMGBulletCount >= 150)
			{
				SMGBulletCount = 150;
			}
		}
		
		FTimerHandle HitDelayHandle;
		GetWorld()->GetTimerManager().SetTimer(HitDelayHandle, 
			FTimerDelegate::CreateLambda([HitResult]()
			{
				HitResult.GetActor()->Destroy();
			}),
			0.5f,
			false
		);
		
	}
}
void APlayerCharacterControll::RifleMode()
{
	if (IsWeaponMap.Num() > 0 && currentWeapon == EWeaponType::Pistol && IsWeaponMap.Contains(EWeaponType::Rifle))
	{
		if (WeaponChangeInMontage)
		{
			UAnimInstance* AnimInstace = GetMesh()->GetAnimInstance();
			float InMontageDuration = AnimInstace->Montage_Play(WeaponChangeInMontage);

			FTimerHandle HideHandle;
			GetWorld()->GetTimerManager().SetTimer(HideHandle,
				FTimerDelegate::CreateLambda([this]()
					{
						if (WeaponChangeSound)
						{
							UGameplayStatics::PlaySound2D(this, WeaponChangeSound);
							SM_PistolWeapon->SetVisibility(false);
						}
						if (WeaponChangeOutMontage)
						{
							UAnimInstance* AnimInstace = GetMesh()->GetAnimInstance();
							float OutMontageDuration = AnimInstace->Montage_Play(WeaponChangeOutMontage);

							FTimerHandle ShowHandle;
							GetWorld()->GetTimerManager().SetTimer(ShowHandle,
								FTimerDelegate::CreateLambda([this]()
									{
										if (WeaponChangeSound)
										{
											UGameplayStatics::PlaySound2D(this, WeaponChangeSound);
											SM_RifleWeapon->SetVisibility(true);
											UPlayerAnimInstance* playeranimInstance = Cast<UPlayerAnimInstance>(GetMesh()->GetAnimInstance());
											playeranimInstance->WeaponTypeCount = 1;
											currentWeapon = EWeaponType::Rifle;


										}
									}),
								OutMontageDuration,
								false
							);

						}
					}),
				InMontageDuration,
				false
			);
		}
		
		
	}

	else if (IsWeaponMap.Contains(EWeaponType::Rifle) && IsWeaponMap.Num() > 0 && IsWeaponMap[EWeaponType::Rifle] == true)
	{
		if (WeaponChangeOutMontage)
		{
			UAnimInstance* AnimInstace = GetMesh()->GetAnimInstance();
			AnimInstace->Montage_Play(WeaponChangeOutMontage);
		}
		FTimerHandle HitDelayHandle;
		GetWorld()->GetTimerManager().SetTimer(HitDelayHandle,
			FTimerDelegate::CreateLambda([this]()
				{
					if (WeaponChangeSound)
					{
						UGameplayStatics::PlaySound2D(this, WeaponChangeSound);
						SM_RifleWeapon->SetVisibility(true);
						UPlayerAnimInstance* playeranimInstance = Cast<UPlayerAnimInstance>(GetMesh()->GetAnimInstance());
						playeranimInstance->WeaponTypeCount = 1;
						currentWeapon = EWeaponType::Rifle;
					}
				}),
			0.5f,
			false
		);
	}
	else//총이 없는경우++
	{
		UGameplayStatics::PlaySound2D(this, WeaponChangeSound); 
	}
}

void APlayerCharacterControll::PistolMode()
{
	if (IsWeaponMap.Num() > 0 && currentWeapon == EWeaponType::Rifle&& IsWeaponMap.Contains(EWeaponType::Pistol))
	{
		if (WeaponChangeInMontage)
		{
			UAnimInstance* AnimInstace = GetMesh()->GetAnimInstance();
			float InMontageDuration = AnimInstace->Montage_Play(WeaponChangeInMontage);

			FTimerHandle HideHandle;
			GetWorld()->GetTimerManager().SetTimer(HideHandle,
				FTimerDelegate::CreateLambda([this]()
					{
						if (WeaponChangeSound)
						{
							UGameplayStatics::PlaySound2D(this, WeaponChangeSound);
							SM_RifleWeapon->SetVisibility(false);



						}
						if (WeaponChangeOutMontage)
						{
							UAnimInstance* AnimInstace = GetMesh()->GetAnimInstance();
							float OutMontageDuration = AnimInstace->Montage_Play(WeaponChangeOutMontage);

							FTimerHandle ShowHandle;
							GetWorld()->GetTimerManager().SetTimer(ShowHandle,
								FTimerDelegate::CreateLambda([this]()
									{
										if (WeaponChangeSound)
										{
											UGameplayStatics::PlaySound2D(this, WeaponChangeSound);
											SM_PistolWeapon->SetVisibility(true);
											UPlayerAnimInstance* playeranimInstance = Cast<UPlayerAnimInstance>(GetMesh()->GetAnimInstance());
											playeranimInstance->WeaponTypeCount = 2;
											currentWeapon = EWeaponType::Pistol;


										}
									}),
								OutMontageDuration,
								false
							);

						}
					}),
				InMontageDuration,
				false
			);
		}


	}

	else if (IsWeaponMap.Contains(EWeaponType::Pistol) && IsWeaponMap.Num() > 0 && IsWeaponMap[EWeaponType::Pistol] == true)
	{
		if (WeaponChangeOutMontage)
		{
			UAnimInstance* AnimInstace = GetMesh()->GetAnimInstance();
			AnimInstace->Montage_Play(WeaponChangeOutMontage);
		}
		FTimerHandle HitDelayHandle;
		GetWorld()->GetTimerManager().SetTimer(HitDelayHandle,
			FTimerDelegate::CreateLambda([this]()
				{
					if (WeaponChangeSound)
					{
						UGameplayStatics::PlaySound2D(this, WeaponChangeSound);
						SM_PistolWeapon->SetVisibility(true);
						UPlayerAnimInstance* playeranimInstance = Cast<UPlayerAnimInstance>(GetMesh()->GetAnimInstance());
						playeranimInstance->WeaponTypeCount = 2;
						currentWeapon = EWeaponType::Pistol;
					}
				}),
			0.5f,
			false
		);
	}
	else//총이 없는경우++
	{
		UGameplayStatics::PlaySound2D(this, WeaponChangeSound);
	}

}



void APlayerCharacterControll::WeaponFireSound()
{
	if (RifleSound)
	{
		UGameplayStatics::PlaySound2D(this, RifleSound);


	}
}

void APlayerCharacterControll::InitializationFindInput()
{

	static ConstructorHelpers::FObjectFinder<UInputMappingContext> InputContext(TEXT("/Script/EnhancedInput.InputMappingContext'/Game/Input/IMC_PlayerInput.IMC_PlayerInput'"));
	if (InputContext.Object != nullptr)
	{
		DefaultContext = InputContext.Object;
	}
	static ConstructorHelpers::FObjectFinder<UInputAction> InputMove(TEXT("/Script/EnhancedInput.InputAction'/Game/Input/IA_Move.IA_Move'"));
	if (InputMove.Object != nullptr)
	{
		MoveAction = InputMove.Object;
	}

	static ConstructorHelpers::FObjectFinder<UInputAction> InputLook(TEXT("/Script/EnhancedInput.InputAction'/Game/Input/IA_Look.IA_Look'"));
	if (InputLook.Object != nullptr)
	{
		LookAction = InputLook.Object;
	}

	static ConstructorHelpers::FObjectFinder<UInputAction> InputAim(TEXT("/Script/EnhancedInput.InputAction'/Game/Input/IA_Aim.IA_Aim'"));
	if (InputAim.Object != nullptr)
	{
		AimAction = InputAim.Object;
	}

	static ConstructorHelpers::FObjectFinder<UInputAction> InputFire(TEXT("/Script/EnhancedInput.InputAction'/Game/Input/IA_Fire.IA_Fire'"));
	if (InputFire.Object != nullptr)
	{
		FireAction = InputFire.Object;
	}

	static ConstructorHelpers::FObjectFinder<UInputAction> InputOperate(TEXT("/Script/EnhancedInput.InputAction'/Game/Input/IA_Operate.IA_Operate'"));
	if (InputOperate.Object != nullptr)
	{
		OperateAction = InputOperate.Object;
	}

	static ConstructorHelpers::FObjectFinder<UInputAction> InputRifleChange(TEXT("/Script/EnhancedInput.InputAction'/Game/Input/IA_RifleChange.IA_RifleChange'"));
	if (InputRifleChange.Object != nullptr)
	{
		RifleChangeAction = InputRifleChange.Object;
	}

	static ConstructorHelpers::FObjectFinder<UInputAction> InputPistolChange(TEXT("/Script/EnhancedInput.InputAction'/Game/Input/IA_PistolChange.IA_PistolChange'"));
	if (InputPistolChange.Object != nullptr)
	{
		PistolChangeAction = InputPistolChange.Object;
	}

}

void APlayerCharacterControll::InitializationFindWeaponMesh()
{

	SM_RifleWeapon = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("RifleWeaponMesh"));
	SM_RifleWeapon->SetupAttachment(GetMesh());
	static ConstructorHelpers::FObjectFinder<UStaticMesh> rifleWeaponMesh(TEXT("/Script/Engine.StaticMesh'/Game/Asset/Weapons/FPS_Weapon_Bundle/Weapons/Meshes/AR4/SM_AR4.SM_AR4'"));
	if (rifleWeaponMesh.Succeeded())
	{
		SM_RifleWeapon->SetStaticMesh(rifleWeaponMesh.Object);
	}

	SM_RifleWeapon->AttachToComponent(GetMesh(), FAttachmentTransformRules::KeepRelativeTransform, TEXT("RifleWeapon_Socket"));


	SM_PistolWeapon = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("PistolWeaponMesh"));
	SM_PistolWeapon->SetupAttachment(GetMesh());
	static ConstructorHelpers::FObjectFinder<UStaticMesh> pistolweaponMesh(TEXT("/Script/Engine.StaticMesh'/Game/Asset/PistolStaticMesh.PistolStaticMesh'"));
	if (pistolweaponMesh.Succeeded())
	{
		SM_PistolWeapon->SetStaticMesh(pistolweaponMesh.Object);
	}
	SM_PistolWeapon->AttachToComponent(GetMesh(), FAttachmentTransformRules::KeepRelativeTransform, TEXT("PistolWeapon_Socket"));

}

void APlayerCharacterControll::InitializationIsWeaponMap()
{
	IsWeaponMap.Add(EWeaponType::Pistol, false);
	IsWeaponMap.Add(EWeaponType::SMG, false);
	IsWeaponMap.Add(EWeaponType::Rifle, false);
}

void APlayerCharacterControll::InitializationFindWidget()
{

	if (HUDClass == nullptr)
	{
		HUDClass = LoadClass<UUserWidget>(nullptr, TEXT("/Game/UMG/WB_PlayerHUD.WB_PlayerHUD_C"));
	}

	/*static ConstructorHelpers::FClassFinder<UUserWidget>HUD(TEXT("WidgetBlueprint'/Game/UMG/WB_PlayerHUD.WB_PlayerHUD_C'"));

	if (HUD.Succeeded())
	{
		HUDClass = HUD.Class;
	}*/

}


```