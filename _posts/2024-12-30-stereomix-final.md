---
title: 스테레오믹스 개발 회고
categories:
tags:
toc: true
comments: true
mermaid: true
---
## 개요

학기 작에서도 멀티플레이 게임을 만들었는데, 어쩌다 보니 졸업작품도 멀티플레이 게임을 만들게 되었습니다. 하지만 멀티플레이 게임은 아무리 만들어봐도 어려운 건 마찬가지인 거 같습니다. 이번 게임은 노 베이스… 까지는 아니지만 거의 모든 게 새로워 아주 힘들었습니다.

그래도 최신의 새로운 개발 기법들을 배우는 건 재밌기 때문에… 나름대로 보람 있었던 개발이었네요.

스테레오믹스에서는 언리얼 데디케이티드 기반이기 때문에 게임 서버를 밑바닥부터 새로 만들진 않았고 아웃 게임(로그인, 로비 등등) 게임에 필요한 API를 웹서버로 제작했고 언리얼 데디케이티드 서버에서 필요한 서버 측 로직들을 작성했습니다.

클라이언트 UI 프로그래밍도 담당하였었는데요, 언리얼도 처음인데 UI는 어떻게 해야 하나 막막했는데 의외로 언리얼에서 제공하는 기능이 많아 힘들이지 않고 제작할 수 있었습니다. 더군다나 라이라 샘플 게임이 있었기 때문에 더 편했던 것 같습니다..

## 클라이언트

스테레오믹스는 언리얼 엔진 5로 개발되었고, 별도의 게임 서버가 존재하는 것이 아니라 언리얼 데디케이티드 서버를 사용하여 서버와 클라이언트가 같은 코드를 공유하는 방식입니다. 따라서 프로그래밍 팀원 모두 서버/클라이언트 구분 없이 제작에 참여했습니다.

### **Lyra 샘플 게임**

1학기에는 샘플 게임을 사용하지 않고 새로 제작했습니다. 하지만 개발이 진행될수록 필요한 기능들이 많았는데 이것들을 구현하기에는 너무 시간이 부족했습니다. 그러다 라이라 샘플을 알게 되었는데, 우리 게임과 매우 비슷한 멀티플레이 대전 방식이라는 점에서 매우 매력적으로 다가왔습니다. 라이라를 기반으로 다시 만들기에는 이미 짜인 코드 기반이 있어서 라이라의 코드를 전부 가져오진 못했고, 필요한 코드와 플러그인들을 가져와서 사용했습니다. 시간이 충분하다면 라이라를 기반으로 게임을 다시 만들고 싶을 정도로 라이라는 매우 유용한 샘플이었습니다.

언리얼 멀티플레이 게임을 만드는 후배들이 있다면 그냥 라이라 기반으로 만드는 게 시간 면에서도 좋고 최신 게임 제작 기법들을 배울 좋은 기회라고 알려주고 싶을 정도였습니다…

저희 게임에서 라이라를 어떤 식으로 활용했는지 설명해 드리겠습니다.

### GameplayMessageRouter

GameplayMessageRouter는 이벤트 버스 패턴을 구현한 플러그인입니다. 게임인스턴스 서브시스템으로 구현된 라우터를 통해 발행자 쪽에서 게임플레이 태그를 통해 특정 채널로 메시지를 발행하면 라우터에 등록한 리스너들이 메시지를 받는 형식입니다. 이를 통해 구독자들이 직접 발행자의 이벤트를 구독할 필요가 없어져 발행자-구독자 간의 의존성이 없어져 자주 생성되고 삭제되는 액터들에 대해서 일일이 이벤트를 등록할 필요가 없게 됩니다.

![](/assets/blobs/241230-GameplayMessageRouter.png)

GameplayMessageRouter의 특징으로는 게임플레이 태그를 사용하여 채널을 구분한다는 점인데요, 게임플레이 태그는 상위태그 개념이 존재하여 태그의 상위 태그에 대한 비교도 가능합니다. 예를 들어 A.B.C라는 태그가 있다면 이 태그의 상위 태그는 A, A.B가 되는 것이고 A.B.C == A, A.B.C == A.B 라는 비교도 가능하다는 것입니다. GameplayMessageRouter는 이를 활용하여 상위 채널의 메시지도 전부 구독할 수 있게 구현되어 있습니다.

가령 Message.Quest.A, Message.Quest.B 라는 태그가 있고, 발행자 측에서 A, B 퀘스트가 완료될 때 라우터를 통해 메시지를 발행하면, 리스너는 모든 태그에 대해 구독해두지 않고 Message.Quest 태그를 구독해 두면 라우터는 상위 태그를 인식하여 해당 리스너에게 전달하게 됩니다.

스테레오믹스에서는 GameplayMessageRouter를 UI와 게임플레이 액터 간의 의존성을 제거하기 위해 사용했습니다.

![](/assets/blobs/241230-IngameCursor.png)

게임에 적용된 대표적 사례 중 하나는 인게임 커서입니다. 인게임 커서는 캐릭터가 피격에 성공하면 색이 변하는 기능이 있습니다. GameplayMessageRouter를 사용하지 않는다면 로컬플레이어 컨트롤러의 PawnChangedEvent에 콜백을 등록하고 캐릭터가 변경될 때마다 관련 이벤트를 등록해 줘야 합니다. 하지만 GameplayMessageRouter를 사용하면 커서 위젯의 초기화 시점에 리스너 등록만 해두면 캐릭터가 바뀌는 것에 상관없이 라우터를 통해 들어오는 메시지만 받기만 하면 되기 때문에 로직이 매우 간단 해집니다. 

![](/assets/blobs/241230-IngameCursorBlueprint.png)

### **CommonGame**

CommonGame 플러그인은 게임의 전반적인 부분에서 편의성을 제공하는 플러그인입니다. 그중에서 UI와 관련된 기능들이 대표적인데, Common UI의 Activatable Widget을 활용한 UI 레이아웃 기능, 팝업과 같은 다이얼로그 표시 기능이 있습니다. 이 중 레이아웃에 대해서는 아래에서 설명할 예정이고 다이얼로그에 대해서 간단하게 사례를 보여드리겠습니다.

![](/assets/blobs/241230-CommonGame.png)

위 블루프린트는 로그인에 대한 로직입니다. Login for Online Play에서 로그인이 실패한 경우 팝업을 띄워야 하는데, 이 과정을 CommonGame에서 제공하는 `Show Confirmation~` 노드를 사용하거나 직접 커스텀하여 만든 노드를 사용하여 간단하게 팝업을 띄우고 그 뒤의 과정까지 하나의 노드로 처리가 가능합니다.

![](/assets/blobs/241230-CommonUser.png)

### **CommonUser**

CommonUser 플러그인은 OnlineSubsystem을 사용하여 유저 로그인 및 유저 정보를 관리하는 기능과 세션을 관리하는 기능들을 간편하게 다룰 수 있게 해줍니다. 로그인의 경우 위의 CommonGame에서 다이얼로그 표시 블루프린트에서 나와 있듯이 복잡한 로그인 절차를 블루프린트 비동기 노드 하나로 간단하게 처리할 수 있게 해줘 매우 많은 코드를 작성할 필요가 없어집니다.

세션도 OnlineSubsytem에서 제공하는 SessionInterface를 더 간단하게 사용할 수 있도록 서브시스템화 한 것인데, 블루프린트를 통해 매우 간단하게 세션 검색, 참가, 초대 등의 기능을 사용할 수 있습니다. 

## UI

### Common UI

스테레오믹스의 UI는 Common UI를 사용하여 제작되었습니다. Common UI는 레이어 단위로 UI를 구성하고 컨트롤러 지원 등 크로스플랫폼 UI를 편하게 개발할 수 있게 해주는 언리얼 기능 중 하나입니다. 언리얼로 제작되는 AAA급 게임들 대부분이 Common UI를 사용하고 있고 크로스플랫폼 게임을 만든다면 거의 필수로 사용해야 하는 유용한 플러그인입니다.

![](/assets/blobs/241230-LobbyUI-Controller.png)

![](/assets/blobs/241230-LobbyUI-Keyboard.png)

위 스크린샷은 팀 선택 UI입니다. 컨트롤러가 활성화됐을 때는 키 아이콘이 컨트롤러 아이콘을 보여주고, 키보드가 활성화됐을 때는 키보드 아이콘을 보여줍니다. 이 바뀌는 과정을 수동으로 변경해야 하는 것이 아니라 키보드/컨트롤러 간 전환되는 즉시 반영됩니다. 또한 중앙 하단의 UI에도 컨트롤러에만 키 아이콘이 표시되는 것을 볼 수 있는데 Common UI의 장점 중 하나가 특정 컨트롤러 또는 플랫폼(스팀, XBOX, PS)에 따라 UI를 숨기거나 보여줄 수 있다는 것입니다. 이를 사용하여 간단하게 플랫폼별 UI를 구현할 수 있습니다.

이 외에도 입력 라우팅 기능으로 마우스/컨트롤러 간 포커싱 단일화, InputConfig, Activatable Widget 등 매우 유용한 기능들이 많으므로 프로젝트에서 사용해 보길 권장합니다.

### 블루프린트 활용

스테레오믹스에서는 1학기에는 모든 위젯을 C++로 작성했지만 2학기에 UI를 갈아엎고 나서는 대부분의 UI를 블루프린트로 작성했습니다. 물론 복잡한 기능을 요구하거나 네이티브로만 접근해야 하는 요소들은 C++로 작성했지만 블루프린트로 작성하면 컴파일 시간이 없어 빠른 UI 제작 및 테스트가 가능했습니다. 또 다른 블루프린트의 강점으로는 비동기 코드를 쉽게 노드 연결만으로 작성할 수 있다는 것인데, 블루프린트에는 AsyncAction 노드를 지원하고 있어 C++로 작성했으면 무수히 많은 콜백 함수를 만들어야 할 것을 간단한 노드 연결만으로 구현할 수 있으며 그 흐름 또한 쉽게 파악이 가능합니다. 

![](/assets/blobs/241230-Blueprint.png)

### 레이아웃 기반 UI

스테레오믹스에서는 월드마다 하나의 프라이머리 레이아웃을 생성한 뒤 (정확히는 로컬플레이어가 생성되는 시점) C++ 코드 또는 블루프린트 노드를 사용하여 각 레이어에 ActivatableWidget을 추가하는 방식으로 위젯을 다룹니다.

![](/assets/blobs/241230-ActivatableWidgetTree.png)

![](/assets/blobs/241230-ActivatableWidgetBlueprint.png)

스크린샷처럼 레이어에 해당하는 ActivatableWidgetStack들을 모아놓은 PrimaryGameLayout 위젯을 만들고, 초기화 시점에 각 레이어를 레이어 태그와 함께 등록하면 아래 코드처럼 해당 레이어에 ActivatableWidget을 추가할 수 있습니다.

```cpp
void USMFrontendComponent::FlowStep_TryShowMainScreen(FControlFlowNodeRef SubFlow)
{
    if (UPrimaryGameLayout* RootLayout = UPrimaryGameLayout::GetPrimaryGameLayoutForPrimaryPlayer(this))
    {
        RootLayout->PushWidgetToLayerStackAsync<USMMenuLayout>(Tags::UI_Layer_Menu, true, MainScreenClass,
            [this, SubFlow](const EAsyncWidgetLayerState State, const USMMenuLayout* Screen) {
                switch (State)
                {
                    case EAsyncWidgetLayerState::AfterPush:
                        SubFlow->ContinueFlow();
                        break;
                    case EAsyncWidgetLayerState::Canceled:
                        break;
                    default:
                        break;
                }
            });
    }
}
```
### UI 머티리얼

스테레오믹스의 UI는 머티리얼을 활용하여 제작했습니다. 언리얼의 머티리얼은 블루프린트로 되어 있어 다루기가 쉽습니다. 쉐이더 자체를 이번 프로젝트에서 처음 만져봤던 저도 빠르게 배우고 적용했을 정도인데요, UI의 다양한 애니메이션과 효과를 머티리얼을 사용하여 제작할 수 있었습니다.

![](/assets/blobs/241230-UIMaterial.png)

위 머티리얼 하나로 UI의 다양한 보더 이미지, 효과들을 제작했습니다. 

![](/assets/blobs/241230-UIMaterialSample1.png)

![](/assets/blobs/241230-UIMaterialSample2.png)

![](/assets/blobs/241230-UIMaterialSample3.png)

머티리얼을 UI에 사용했을 때의 제일 큰 장점은 위젯 애니메이션과 함께 사용했을 때입니다. 위젯 애니메이션을 사용하여 머티리얼 인스턴스의 파라미터를 조정할 수 있는데, 이를 통해 각각의 위젯 요소들의 필드 값을 조절하는 대신 머티리얼 인스턴스 파라미터만 조작함으로써 위젯 애니메이션 측의 복잡함도 줄어듭니다. 또한 CPU 연산만 사용하는 위젯 애니메이션의 부담을 머티리얼을 통해 조작함으로써 GPU 연산으로 분담하여 성능상의 이점도 가져갈 수 있습니다. 리소스 면에서도 각각의 UI 텍스처를 각각 뽑을 필요가 없어 UI 텍스처 리소스 양도 현저히 줄어듭니다.

또 하나의 머티리얼 이점 중 하나는 UI의 계단 현상을 줄여줄 수 있다는 것입니다. 머티리얼에서 쉐이프를 그릴 때 아웃라인을 뭉갤 수 있는데, 이를 사용하여 게임 화면에서 별다른 옵션 없이 계단 현상을 줄이는 효과를 볼 수 있습니다.

![](/assets/blobs/241230-Sharpness.png)

![](/assets/blobs/241230-Sharpness2.png)

### 그리드 뷰 기반 페이지 UI

![](/assets/blobs/241230-GridPageView.png)

커스텀 매치 목록을 검색하는 UI에는 페이지 기반으로 목록을 탐색하는 UI가 존재합니다. 하지만 언리얼에는 페이지 단위로 구성되는 리스트 뷰가 존재하지 않아 직접 구현해야 했습니다.

각 아이템을 구성하는 UI는 UniformGridPanel을 사용했고, EOS 세션 검색 결과의 특성상 페이지 단위로 검색하는 기능이 없고 한 화면에 보여주는 양도 많지 않아 다음 페이지로 넘어가면 목록을 새로 받아오는 것이 아니라 페이지 단위에 맞게 특정 범위의 아이템들을 보여주도록 구현했습니다.

```cpp
UCLASS(Abstract, Blueprintable, ClassGroup = UI, meta = (Category = "Common UI", DisableNativeTick))
class STEREOMIX_API USMGridPageView : public UCommonUserWidget
{
    GENERATED_BODY()

public:
    USMGridPageView();

    UFUNCTION(BlueprintPure, Category = Page)
    int GetPageColumn() const { return PageColumn; }

    UFUNCTION(BlueprintPure, Category = Page)
    int GetPageRow() const { return PageRow; }

    UFUNCTION(BlueprintPure, Category = Page)
    int GetMaxEntriesPerPage() const { return PageRow * PageColumn; }

    UFUNCTION(BlueprintPure, Category = Page)
    int GetActivePageIndex() const { return ActivePageIndex; }

    UFUNCTION(BlueprintPure, Category = Page)
    int GetPageCount() const { return PageCount; }

    UFUNCTION(BlueprintCallable, Category = Page)
    bool NextPage();

    UFUNCTION(BlueprintCallable, Category = Page)
    bool PreviousPage();

    UFUNCTION(BlueprintCallable, Category = Page)
    void Refresh(int PageIndex = 0);

    virtual void ReleaseSlateResources(bool bReleaseChildren) override;

    UPROPERTY(Transient, BlueprintReadWrite, Category = Page)
    TArray<TObjectPtr<USMPageEntryObject>> PageItems;
```

저는 UI 콘텐츠에 대한 부분은 블루프린트로 작성하는 것을 원칙으로 하므로 코어 부분만 C++로 작성하고 콘텐츠를 채워 넣는 부분은 블루프린트로 구현했습니다. 그래서 C++ 코드에는 범용적으로 사용할 수 있는 코드만 작성하고 블루프린트로 접근할 수 있도록 BlueprintCallable을 사용했습니다. 클래스의 public 멤버 목록을 통해 페이지의 기능들을 사용할 수 있습니다.

![](/assets/blobs/241230-GridPageViewBlueprint.png)

블루프린트에서는 목록을 새로 고침 할 때마다 `Page Items` 필드를 새로운 아이템으로 채워 넣은 후 Refresh()를 호출하여 UI를 새로운 목록으로 다시 보여줍니다.

```cpp
void USMGridPageView::Refresh(const int PageIndex)
{
    for (UWidget* Child : GridView->GetAllChildren())
    {
        UUserWidget* ChildWidget = Cast<UUserWidget>(Child);
        EntryWidgetPool.Release(ChildWidget, true);
    }
    GridView->ClearChildren();

    PageCount = FMath::CeilToInt32(static_cast<float>(PageItems.Num()) / (PageRow * PageColumn));
    ActivePageIndex = FMath::Clamp(PageIndex, 0, PageCount - 1);

    const int StartIndexInclude = ActivePageIndex * PageRow * PageColumn;
    const int EndIndexExclude = FMath::Min(StartIndexInclude + PageRow * PageColumn, PageItems.Num());

    for (int i = 0; i < GetMaxEntriesPerPage(); ++i)
    {
        UUserWidget* EntryWidget = nullptr;
        if (const int ItemIndex = StartIndexInclude + i; ItemIndex < EndIndexExclude)
        {
            if (EntryWidgetClass && EntryWidgetClass->ImplementsInterface(USMPageEntryInterface::StaticClass()))
            {
                EntryWidget = EntryWidgetPool.GetOrCreateInstance(EntryWidgetClass);
                ISMPageEntryInterface::SetItemObject(EntryWidget, PageItems[ItemIndex]);
            }
        }
        else
        {
            if (EmptyEntryWidgetClass)
            {
                EntryWidget = EntryWidgetPool.GetOrCreateInstance(EmptyEntryWidgetClass);
            }
        }

        if (EntryWidget)
        {
            GridView->AddChildToUniformGrid(EntryWidget, i / PageColumn, i % PageColumn);
        }
    }

    NativeOnRefreshPage();
}
```

`Refresh()`에서는 `PageItems`의 아이템 중에서 매개변수로 받은 페이지 인덱스의 범위에 맞는 아이템들을 그리드 뷰에 배치합니다. 그리드 뷰에 들어가는 위젯은 매번 새로 생성하는 것이 아니라 위젯 풀에서 가져오는 것을 볼 수 있는데, 언리얼에서 제공하는 `FUserWidgetPool`을 사용하여 잦은 새로 고침 상황에서 위젯을 매번 새로 생성해야 하는 부담을 줄였습니다. 또한 위젯 풀에서 가져온 위젯을 `ISMPageEntryInterface` 인터페이스를 통해 대상 엔트리 위젯에 아이템을 넘겨 초기화할 수 있도록 구현하였습니다. C++에서는 이렇게 위젯을 새로 넣는 등 네이티브에서만 가능한 로직들만 넣고, 나머지 페이지 넘김 버튼, 페이지 텍스트 등 비주얼 요소들은 `OnRefreshPage` 이벤트를 받은 블루프린트에서 처리하게 됩니다.

![](/assets/blobs/241230-GridPageView-OnRefreshPage.png)

### 어빌리티 슬롯

![](/assets/blobs/241230-AbilitySlot1.png)

![](/assets/blobs/241230-AbilitySlot2.png)

어빌리티 슬롯은 게임 HUD의 우측 하단에 위치하여 캐릭터의 공격, 스킬, 노이즈 브레이크 사용 상태를 보여주는 UI입니다. 그 이름에 맞게 캐릭터가 사용하는 GAS의 어빌리티에 따라 실시간으로 현재 상태를 보여줘야 합니다.

또 하나의 조건 중 하나는 하나의 슬롯에 하나의 어빌리티만 들어가는 것이 아니라 조건에 따라 보여주는 어빌리티가 달라진다는 것입니다. 위의 스크린샷에서 그 부분을 볼 수 있는데, 이를 위해 네이티브로 UI 구현이 필요했습니다.

![](/assets/blobs/241230-AbilitySlot-Types.png)

저는 이러한 형태의 구현을 위해 각 슬롯 위젯이 어빌리티와 1:1로 대응하는 엔트리 위젯을 1:N 형태로 가지도록 구현했습니다.

![](/assets/blobs/241230-AbilitySlot-Graph.png)

슬롯 정보는 데이터 에셋을 사용하여 적용합니다.

![](/assets/blobs/241230-AbilitySlot-Editor1.png)

![](/assets/blobs/241230-AbilitySlot-Editor2.png)

각 슬롯은 상태 머신 패턴을 사용하여 틱마다 각 엔트리의 순서대로 상태를 업데이트하도록 구현했습니다.

```cpp
void USMActiveAbilityDisplaySlotWidget::NativeTick(const FGeometry& MyGeometry, float InDeltaTime)
{
    Super::NativeTick(MyGeometry, InDeltaTime);
    UpdateState();
}

void USMActiveAbilityDisplaySlotWidget::UpdateState()
{
    if (!bIsInitialized || !AbilitySystemComponent)
    {
        return;
    }

    bool bVisible = false;
    for (auto& EntryWidget : EntryWidgets)
    {
        if (!bVisible)
        {
            ESMAbilityDisplayState OldState, NewState;
            if (EntryWidget->TryUpdateState(OldState, NewState))
            {
                if (NewState != ESMAbilityDisplayState::Hidden)
                {
                    if (EntrySlot->GetActiveWidget() != EntryWidget)
                    {
                        EntrySlot->SetActiveWidget(EntryWidget);
                    }
                }
            }
            if (EntryWidget->GetCurrentState() != ESMAbilityDisplayState::Hidden)
            {
                bVisible = true;
            }
        }
        else
        {
            EntryWidget->SetState(ESMAbilityDisplayState::Hidden);
        }
    }
}
```

슬롯에서는 매 틱마다 데이터 에셋에서 정한 엔트리의 순서대로 상태를 업데이트합니다. 만약 업데이트한 엔트리의 상태가 Hidden이 아니라면 이후의 엔트리는 무조건 Hidden 상태가 되어 결과적으로는 슬롯에 하나의 엔트리만 보이게 됩니다.

```cpp
bool USMActiveAbilityDisplayEntryWidget::TryUpdateState(ESMAbilityDisplayState& OutOldState, ESMAbilityDisplayState& OutNewState)
{
    USMAbilitySystemComponent* ASC = EntrySlot->GetAbilitySystemComponent();
    if (!ASC)
    {
        return false;
    }

    ESMAbilityDisplayState NextState;
    if (const USMDisplayableGameplayAbility* GA = RefreshAbilityInstance())
    {
        const FGameplayAbilitySpecHandle Handle = GA->GetCurrentAbilitySpecHandle();
        const FGameplayAbilityActorInfo& ActorInfo = GA->GetActorInfo();

        float CurrentGauge, MaxGauge;
        bool bHasGauge = TryGetGauge(CurrentGauge, MaxGauge);

        if (GA->IsActive())
        {
            NextState = ESMAbilityDisplayState::Activated;
        }
        else if (!GA->CanActivateAbility(Handle, &ActorInfo))
        {
            if (bHasGauge && CurrentGauge != MaxGauge)
            {
                NextState = ESMAbilityDisplayState::Cooldown;
                BP_OnUpdateGauge(CurrentGauge, MaxGauge);
            }
            else
            {
                if (EntryData.bHideOnDeactivate)
                {
                    NextState = ESMAbilityDisplayState::Hidden;
                }
                else
                {
                    NextState = ESMAbilityDisplayState::Deactivated;
                }
            }
        }
        else
        {
            NextState = ESMAbilityDisplayState::Idle;
        }
    }
    else
    {
        NextState = ESMAbilityDisplayState::Hidden;
    }

    if (CurrentState == NextState)
    {
        return false;
    }

    OutOldState = CurrentState;
    OutNewState = NextState;
    SetState(NextState);
    return true;
}
```

엔트리에서는 엔트리와 대응하는 어빌리티 인스턴스를 가져와서 어빌리티의 상태에 따라 엔트리 상태를 변경합니다. 이전 상태와 바뀐 상태를 비교하여 상태가 같으면 상태가 변경됨을 블루프린트에서 받을 수 있게 이벤트를 호출하고, true를 반환하여 슬롯에 엔트리의 상태가 바뀌었음을 알립니다.

이렇게 상태머신을 사용한 덕분에 2중으로 된 복잡한 어빌리티 슬롯의 상태를 직관적이고 유연하게 구현할 수 있었습니다.

## 서버

제목이 서버라 게임플레이 로직을 담당하는 게임 서버를 생각하시겠지만 언리얼 게임 서버가 이미 잘 구현되어 있어 게임 로직만 잘 만들면 돌아가는 게 언리얼 게임 서버이기 때문에… 이 글에서는 게임 외적인 부분, 백엔드에 대해 주로 설명할 예정입니다. 그래도 마지막에는 언리얼 데디케이티드 게임 서버를 어떤 식으로 사용했는지에 대한 부분을 짧게(?) 넣어봤습니다.

1학기와 2학기의 백엔드 서버 구현이 다르기 때문에 첫 부분에는 1, 2학기 공통으로 들어간 내용들을 넣었고, 이후 1학기와 2학기 각각 구현했던 부분을 적었습니다.

### ASP.NET Core

스테레오믹스의 백엔드는 ASP.NET Core를 사용하여 구현했습니다. ASP.NET Core를 사용한 이유는 무엇보다 제가 C#에 제일 익숙했기 때문이었습니다.

현재의 ASP.NET Core는 옛날의 윈도우 플랫폼 전용, 느린 속도라는 단점을 많이 벗어났습니다. .NET Core 2.0이 발표되면서 리눅스 플랫폼을 지원하고, .NET 7부터 Native AOT를 사용하여 속도가 향상됐습니다. 로직 작성에서도 Minimal API를 사용하면 적은 코드로 빠르게 API 구현이 가능합니다.

스테레오믹스 백엔드 서버는 이러한 ASP.NET Core의 최신 기능들을 적극 채용했습니다. 졸업작품이라 가능하기도 했고 덕분에 최신 트렌드를 많이 경험해 보는 경험이 됐습니다.

이 글에서는 저희 백엔드 코드의 전체적인 구조를 설명해 드리겠습니다.

![](/assets/blobs/241230-ApiService.png)

저희 프로젝트는 코드를 프로젝트의 설정과 관련된 Configuration, API 엔드 포인트 구현체인 Endpoints, 데이터베이스의 Document 객체를 추상화하고 .NET 객체와 대응하는 Entities, 데이터베이스 레이어를 추상화한 Repositories, ASP.NET의 Authentication, Authorization 및 JWT와 같은 보안 기능을 제공하는 Security로 크게 구분했습니다.

### Endpoints

ASP.NET의 Minimal API는 적은 코드로 API를 구현할 수 있지만 API가 커질수록 MVC 패턴에 비해 API 관리가 힘들어질 것을 느꼈습니다. 그래서 프로젝트 자체적으로 규칙을 만들었는데, 아래 코드와 같이 Endpoint에 대한 static class를 만들고 Extension의 형태로 Configure…Endpoint, Add…Endpoint 확장 메소드를 만들어 API를 등록하게 하는 것입니다.

```csharp
public static class UserEndpoint
{
    public static IServiceCollection ConfigureUserEndpoint(this IServiceCollection services)
    {
        ArgumentNullException.ThrowIfNull(services);
        return services.AddScoped<UserService>();
    }

    public static IEndpointRouteBuilder AddUserEndpoint(this IEndpointRouteBuilder route)
    {
        ArgumentNullException.ThrowIfNull(route);
        var group = route.MapGroup("/user");

        group.MapGet("/", static (UserService service) => service.GetRoot());

        group.MapPost("/", static (HttpRequest request, UserDTO? userDTO, UserService service) => service.CreateUser(request, userDTO))
            .RequireAuthorization(StereoMixPolicy.UserPolicy);

        group.MapPut("/", static (HttpRequest request, UserDTO? userDTO, UserService service) => service.UpdateUser(request, userDTO))
            .RequireAuthorization(StereoMixPolicy.UserPolicy);

        group.MapDelete("/", static (HttpRequest request, UserService service) => service.DeleteUser(request))
            .RequireAuthorization(StereoMixPolicy.UserPolicy);

        group.MapGet("/{id}", static (long id, UserService service) => service.GetByUserId(id));

        return route;
    }
}
```

모든 Endpoint들은 위 코드처럼 2개의 확장 메소드를 가지고 있어야 합니다. Configure…Endpoint 메소드는 엔드포인트의 로직을 가지고 있는 Scoped 범위의 서비스 등록하는 코드를, Add…Endpoint 메소드는 실제로 API에 사용할 엔드포인트를 등록하는 코드를 넣습니다. 각 엔드포인트의 로직은 Configure…Endpoint에서 등록한 Scoped 범위의 서비스에 구현합니다.

이렇게 구현된 Endpoint는 Program.cs에서 Configure…Endpoint를 호출하여 서비스를 등록하고 Add…Endpoint를 호출하여 엔드포인트를 등록합니다.

### Repositories / Entities

Repositories와 Entities는 데이터베이스와 관련된 코드입니다. Repositories라는 이름 그대로 저희는 레포지토리 패턴을 사용하여 데이터베이스를 다룹니다. 레포지토리 패턴을 사용하면 비즈니스 레이어와 데이터베이스 레이어 간의 의존성을 떨어트릴 수 있다는 장점이 있습니다. 실제로 저희 코드의 Endpoint 로직에서는 Dependency Injection을 통한 레포지토리 인터페이스에만 접근하여 데이터를 수정합니다.

```csharp
public interface IUserRepository
{
    Task<User?> GetByIdAsync(long id);

    Task<User?> GetByAccountAsync(UserAccount? account);

    Task<User?> CreateAsync(User user);

    Task<bool> UpdateAsync(User user);

    Task<bool> DeleteAsync(User user);
}

public class UserRepository(ILogger<UserRepository> logger, DatastoreDb db) : IUserRepository
{
    private const string Kind = "User";
    private readonly KeyFactory _keyFactory = db.CreateKeyFactory(Kind);

    public async Task<User?> GetByIdAsync(long id)
    {
        try
        {
            var entity = await db.LookupAsync(_keyFactory.CreateKey(id)).ConfigureAwait(false);
            return entity is null ? null : UserMapper.ToUser(entity);
        }
        catch (Exception e)
        {
            logger.LogError(e, "Failed to retrieve from DatastoreDB.");
            return null;
        }
    }
    // ...
}
```

Entities는 데이터베이스에서 사용하는 모델을 정의하고 데이터베이스와 C# 객체를 매핑하는 메소드들이 모여 있습니다.

초기에 백엔드 서버를 설계할 때 EF Core를 사용할까 했었는데 아직 EF Core가 NativeAOT 지원을 하지 않아 저희가 일일이 매핑 메소드를 작성하여 구현하는 형태로 진행했습니다.

각 Entity는 2가지로 구현됩니다. 하나는 record 타입의 C# 타입, 또 하나는 데이터베이스 객체와 record 타입의 C#객체를 매핑하는 메소드를 구현한 매핑 메소드 전용 정적 클래스입니다. C# 객체 타입으로 record를 사용한 이유는 한번 매핑 메소드를 통해 생성된 객체는 수정할 필요가 없기 때문이기도 했고 record의 편리한 기능 중 하나인 ToString() 호출 시 record 내의 값들을 읽기 좋게 출력해 줘 디버깅에 용이했기 때문이었습니다.

```csharp
public record User(long Id, UserData Data, UserAccount Account);
public record UserData(string Name);
public record UserAccount(string? EpicId, string? SteamId);

internal static class UserKeyMapping
{
    public const string NameKey = "name";
    public const string EpicIdKey = "epic_id";
    public const string SteamIdKey = "steam_id";
}

public static class UserMapper
{
    public static bool Validate(this User user)
    {
        return !string.IsNullOrWhiteSpace(user.Data.Name);
    }

    public static Entity CreateEntity(this User user, KeyFactory keyFactory)
    {
        return new Entity
        {
            Key = keyFactory.CreateIncompleteKey(),
            [UserKeyMapping.NameKey] = user.Data.Name,
            [UserKeyMapping.EpicIdKey] = user.Account.EpicId ?? string.Empty,
            [UserKeyMapping.SteamIdKey] = user.Account.SteamId ?? string.Empty
        };
    }

    public static Entity ToEntity(this User user, KeyFactory keyFactory)
    {
        return new Entity
        {
            Key = keyFactory.CreateKey(user.Id),
            [UserKeyMapping.NameKey] = user.Data.Name,
            [UserKeyMapping.EpicIdKey] = user.Account.EpicId ?? string.Empty,
            [UserKeyMapping.SteamIdKey] = user.Account.SteamId ?? string.Empty
        };
    }

    public static User ToUser(Entity entity)
    {
        return new User(
            entity.Key.Path.Last().Id,
            new UserData(
                entity[UserKeyMapping.NameKey].StringValue),
            new UserAccount(
                entity[UserKeyMapping.EpicIdKey].StringValue,
                entity[UserKeyMapping.SteamIdKey].StringValue)
        );
    }

    public static UserAccount ToUserAccount(string? type, string? id)
    {
        return type switch
        {
            UserKeyMapping.EpicIdKey => new UserAccount(EpicId: id, SteamId: null),
            UserKeyMapping.SteamIdKey => new UserAccount(EpicId: null, SteamId: id),
            _ => new UserAccount(EpicId: null, SteamId: null)
        };
    }
}
```

### Security (Authentication, Authorization, JWT)

ASP.NET은 Authentication(인증)과 Authorization(인가)을 통해 미들웨어 단에서 API 접근에 대한 액세스를 제어할 수 있습니다.

Authentication에서는 유저의 인증 방법을 정의합니다. 웹에서 인증을 하는 방법은 여러 가지가 있지만 스테레오믹스에서는 JWT를 사용하여 유저를 인증합니다. 왜냐하면 뒤에서 설명하겠지만 첫 번째로 저희 백엔드 서버가 서버리스 기반으로 돌아가기 때문에 세션과 같은 방식이 불가능한 점, 게임 클라이언트에서 인증을 처리할 때 쉬운 방법이 JWT이기 때문입니다. (무엇보다 EOS에서는 유저 인증 토큰을 JWT로 줍니다)

저희 백엔드는 JWT 인증 방법을 2가지를 구현했습니다. 하나는 저희 서버에서 발급하는 JWT 토큰을 사용하여 인증하는 기본 JWT 인증 방법, 또 하나는 EOS를 통해 발급받은 토큰을 사용하여 인증하는 Epic 토큰 인증 방법입니다.

첫 번째 인증 방법은 게임 서버에서 백엔드에 인증하기 위해 사용됩니다. 백엔드에서 새로운 게임 서버를 생성할 때 토큰을 생성하여 게임 서버로 함께 넘겨주는데, 게임 서버에서는 이 토큰을 사용하여 백엔드에 접근해 유저 정보를 가져오는 등 API를 사용할 수 있습니다.

두 번째 인증 방법은 게임 클라이언트에서 백엔드에 인증하기 위해 사용됩니다. 유저는 EOS를 사용하여 게임 클라이언트에서 인증을 하게 되는데, 이 때 EOS는 인증된 유저 정보를 가지고 있는 JWT 토큰을 줍니다. 인증된 유저는 이 토큰을 백엔드에 API 요청을 할 때 함께 전달하고 백엔드에서는 토큰을 검증하여 인증된 유저만 API에 접근할 수 있게 합니다.

기본 JWT 인증 방법은 위 코드에서 IssuerSigningKey를 통해 서버에서 자체적으로 가지고 있는 비밀키로 전달받은 토큰이 올바른 토큰인지 검증합니다.

```csharp
public static AuthenticationBuilder AddDefaultJwtBearer(this AuthenticationBuilder builder, IConfiguration configuration)
{
    var secret = configuration[JwtTokenProvider.JwtSecretKeyName] ?? throw new KeyNotFoundException($"{JwtTokenProvider.JwtSecretKeyName} is not set in Environment Variables");

    return builder.AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.ASCII.GetBytes(secret))
        };
    });
}
```

Epic JWT 인증 방법은 EOS 공식 문서 사이트에서 나온 방법대로 토큰을 검증합니다.

```csharp
public static AuthenticationBuilder AddEpicJwtBearer(this AuthenticationBuilder builder, IConfiguration configuration)
{
    return builder.AddJwtBearer("Epic", options =>
    {
        options.Authority = configuration["EOS_ISSUER"];
        Log.Information(configuration["EOS_ISSUER"]);
        Log.Information(configuration["EOS_AUDIENCE"]);
        options.TokenValidationParameters = new TokenValidationParameters
        {
            // Audience 체크는 Authorization의 Policy에서 확인함

            ValidateAudience = true,
            ValidateIssuer = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,

            IssuerValidator = (issuer, token, parameters) =>
            {
                if (issuer.StartsWith(configuration["EOS_ISSUER"] ?? string.Empty, StringComparison.OrdinalIgnoreCase))
                {
                    return issuer;
                }

                throw new SecurityTokenInvalidIssuerException($"Invalid issuer: {issuer}");
            },

            ValidAudience = configuration["EOS_AUDIENCE"],

            IssuerSigningKeyResolver = (token, securityToken, kid, validationParameters) =>
            {
                var signingKey = EpicJwtTokenProvider.GetSigningKey(kid);

                if (signingKey is null)
                {
                    throw new SecurityTokenInvalidSigningKeyException($"Signing key with kid '{kid}' not found.");
                }

                return [signingKey];
            }
        };

        options.Events = new JwtBearerEvents
        {
            OnAuthenticationFailed = context =>
            {
                var logger = context.HttpContext.RequestServices.GetRequiredService<ILogger<EpicJwtTokenProvider>>();
                logger.LogError(context.Exception, "Authentication failed");
                context.Response.StatusCode = 401;
                return Task.CompletedTask;
            }
        };
    });
}
```

이렇게 추가한 Authentication은 Authorization에서 설정한 각 엔드포인트의 Policy에 맞게 인증하는 절차를 진행합니다.

```csharp
public static class AuthorizationExtensions
{
    public static AuthorizationBuilder AddUserPolicy(this AuthorizationBuilder builder, IConfiguration configuration)
    {
        // 클라이언트의 EOS Client ID가 API서버의 Client ID와 일치하는지 확인함
        var audience = configuration["EOS_AUDIENCE"] ?? throw new KeyNotFoundException("EOS_AUDIENCE is not set in Environment Variables");
        return builder
            .AddPolicy(StereoMixPolicy.UserPolicy, policy => policy
                .AddAuthenticationSchemes("Epic")
                .RequireAuthenticatedUser()
                .RequireClaim("aud", audience));
    }

    public static AuthorizationBuilder AddGameServerPolicy(this AuthorizationBuilder builder)
    {
        return builder
            .AddPolicy(StereoMixPolicy.GameServerPolicy, policy => policy
                .AddAuthenticationSchemes(JwtBearerDefaults.AuthenticationScheme)
                .RequireAuthenticatedUser()
                .RequireRole(StereoMixRole.GameServer));
    }
}
```

 추가한 Policy는 엔드포인트를 추가할 때 함께 등록함으로써 유저가 해당 엔드포인트에 접근할 때 Policy에 맞는 인증 절차를 진행할 수 있게 합니다.

```csharp
public static IEndpointRouteBuilder AddMatchmakingEndpoint(this IEndpointRouteBuilder route)
{
    ArgumentNullException.ThrowIfNull(route);

    var rootGroup = route.MapGroup("/match");

    // 방 만들어서 진행하는 커스텀 매치
    var customMatchGroup = rootGroup.MapGroup("/custom");

    // 새로운 커스텀 매치 생성
    customMatchGroup
        .MapPost("/", static (HttpRequest request, CustomMatchRequestDTO matchDTO, CustomMatchService service) => service.Create(request, matchDTO))
        .RequireAuthorization(StereoMixPolicy.UserPolicy);

    // 커스텀 매치 생성 정보 조회
    customMatchGroup
        .MapGet("/{ticketId}", static (HttpRequest request, string ticketId, CustomMatchService service) => service.GetTicketInfo(request, ticketId))
        .RequireAuthorization(StereoMixPolicy.UserPolicy);

    // 커스텀 매치 삭제
    customMatchGroup
        .MapDelete("/{ticketId}", static (HttpRequest request, string ticketId, CustomMatchService service) => service.Delete(request, ticketId))
        .RequireAuthorization(StereoMixPolicy.UserPolicy);

    return route;
}
```

### Epic Online Services

스테레오믹스는 유저 인증, 세션 관리 기능에 Epic Online Services를 사용했습니다. EOS는 에픽게임즈에서 제공하는 서비스로 다양한 게임 서비스를 무료로 제공합니다.

유저 인증에 대해서는 EOS의 커넥트 인터페이스를 사용했습니다. 커넥트 인터페이스는 스팀과 같은 외부 계정 ID를 사용하여 EOS 서비스를 사용할 수 있게 해주는데, 커넥트 인터페이스를 통해 로그인하면 EOS는 PUID라는 고유한 ID를 반환합니다. 백엔드 서버에서는 아직 구현되지는 않았지만 추후 이 PUID를 사용하여 유저 정보를 데이터베이스에 저장할 때 고유한 키로써 사용할 예정입니다.

세션 인터페이스는 유저가 생성한 커스텀 매치 로비를 관리하는 데 사용됩니다. 현재 저희 게임은 매치 메이킹 방식이 아닌 유저가 로비를 만들어 다른 유저를 모은 뒤 플레이하는 방식인데, 이 때 이 로비 생성, 조회, 참가 기능을 EOS를 사용하여 관리합니다.

참고로 EOS에는 로비 인터페이스가 존재하지만, 저희 게임에서는 로비 인터페이스를 사용하지 않고 세션 인터페이스만을 사용하여 제작했는데, 저희 게임이 데디케이티드 방식이라 로비 인터페이스를 통해 로비를 제작하면 세션 인터페이스만을 사용하여 제작하는 것보다 더 어렵기 때문이었습니다. 데디케이티드 서버에서 세션 인터페이스를 사용한 방법은 뒷 내용인 ‘언리얼 게임 서버 아키텍처’ 부분에서 설명할 예정입니다.

언리얼 엔진에서는 EOS를 사용하기 위한 방법으로 OnlineSubsystemEOS와 OnlineServicesEOS 두 가지 방법을 제공하는데, OnlineServicesEOS의 경우 아직 베타 버전인 데다가 스팀과 같은 다른 플랫폼과 함께 쓰기에는 아직 불안정해 현시점에서는 OnlineSubsystemEOS를 사용하여 제작했습니다.

### Google Cloud

스테레오믹스의 백엔드 서버는 Google Cloud의 Cloud Run 플랫폼 위에서 돌아갑니다. Cloud Run은 서버리스 형태로 컨테이너로 서버를 배포하여 0부터 미리 정해 놓은 최대 인스턴스 수까지 트래픽에 따라 자동으로 인스턴스를 늘리거나 줄일 수 있습니다. 이 특징 때문에 트래픽이 없다면 인스턴스가 생성되지 않아 비용이 청구되지 않으므로 비용 부담이 덜합니다. 특히 졸업작품 정도면 유저가 수시로 플레이하지는 않을 것이기 때문에 유휴 비용이라는 것이 존재하면 매우 부담되는데, Cloud Run은 이러한 부담을 매우 줄여줍니다.

![](/assets/blobs/241230-CloudRun.png)

 만약 웹서버를 배포해야 할 때 Docker를 이용한 컨테이너를 배포하는데 익숙지 않거나 클라우드 사용이 힘드시다면 AWS의 EC2 프리 티어 인스턴스를 사용하거나 오라클 클라우드의 무료 인스턴로 VM을 직접 사용하는 것도 방법이 될 수 있습니다. (심지어 오라클은 1코어에 한해서 기간이 없는 무료 인스턴스입니다!)

### Edgegap

스테레오믹스는 데디케이티드 서버 기반이기 때문에 게임 서버를 호스팅하지 않는 이상 게임플레이가 불가능합니다. 이 게임 서버를 자동으로 배포하고 관리하는 것이 데디케이티드 서버를 사용하는 게임의 매우 중요한 사항 중 하나입니다. 제가 게임업계를 다녀본 것이 아니기 때문에 인터넷에서 조사한 자료들에 의하면 보통 매치의 형태로 이뤄지는 게임 서버들은 싱글 게임 서버를 사용할 경우 특정 게임에서 크래시가 발생하면 다른 게임까지 영향이 가기 때문에 독립적으로 각각의 게임 하나 당 하나의 게임 서버를 배치하여 진행한다고 합니다. 이 때문에 언리얼 엔진과 유니티 엔진의 데디케이티드 서버도 하나의 프로세스 당 하나의 게임을 진행하는 방식으로 설계되어 있습니다. 이러한 게임 서버들을 관리하는 플랫폼을 오케스트레이션 플랫폼이라고 하는데 큰 회사에서는 구글의 Agones나 쿠버네티스 등으로 여러 대의 게임 서버를 관리한다고 합니다. (Agones 사이트: [https://agones.dev/site/](https://agones.dev/site/) )

하지만 이러한 시스템은 기본적으로 24시간 돌아가는 클러스터가 1개 이상 있어야 하며 클라우드에서 돌린다면 매우 높은 비용이 들어갑니다.

이러한 오케스트레이션 플랫폼을 서비스로써 제공하는 제품들이 있습니다. AWS의 GameLift가 제일 대표적이고 이외에 Hathora, Edegap 등이 있습니다.

3월달부터 GameLift부터 시작해서 Hathora, Edegap, Unity Gaming Service까지 전부 시도해 보고(이 R&D기간이 매우 길어 일정이 매우 빠듯했습니다…) 결국에는 저희 게임에 Edegap을 사용하여 게임 서버를 관리하게 됐습니다.

AWS GameLift의 경우 제가 경험한 플랫폼 중 가장 프리 티어 제한이 심했고 비용도 비쌌습니다. GameLift는 게임 서버를 호스팅하기 위해 Fleet이라는 것을 만들어야 하는데 빌드를 업로드하고 Fleet를 생성하기까지의 시간이 30분 이상 소요됐습니다. 프리 티어에서는 Fleet을 1개만 생성할 수 있므로 테스트를 위해 Fleet을 삭제하고 다시 생성하는 데에 매우 긴 시간이 걸렸습니다…

Hathora는 Edgegap 다음으로 괜찮았던 플랫폼이었는데 제일 중요한 한국 리전이 없었습니다.

UGS는 유니티에서 출시한 플랫폼인데 UGS의 제품 중 멀티플레이어라는 게임 서버 호스팅 서비스로 기능을 제공합니다. 다만 언리얼 SDK을 제공하긴 해도 기본적으로 UGS가 유니티에 특화되어 있기도 하고 언리얼 SDK에 대한 정보가 별로 없어 사용하지는 않았습니다. 만약 유니티 프로젝트라면 UGS를 사용해 보는 것도 나쁘지 않을 것 같습니다.

Edgegap은 REST API가 잘 되어 있고 쉽게 사용할 수 있어 적응이 매우 빨랐습니다. 특히 Edgegap은 IP를 기반으로 유저 위치와 제일 가까운 리전에 게임 서버를 생성해 줘서 이 부분이 마음에 들었습니다. 최근에는 Gen2 Matchmaker라는 이름으로 매치메이커도 리뉴얼 되었습니다. (비용은 비쌉니다)

여기서 설명한 플랫폼들은 기본적으로 서버가 실행한 시간만큼만 비용을 청구하기 때문에 부담이 적습니다. 만약 데디케이티드 서버 기반의 멀티플레이 게임을 만드실 예정이라면 여기서 설명한 플랫폼들을 한번 살펴보시는 것을 추천해 드립니다.

### 언리얼 게임 서버

지금까지 설명한 것들의 관계를 그림으로 표현하면 다음과 같습니다.

![](/assets/blobs/241230-GameServer-Diagram.png)

게임 클라이언트는 새로운 커스텀 매치 로비를 생성하기 위해 백엔드 서버로 커스텀 매치 생성 요청을 합니다. 그러면 백엔드에서는 Edgegap으로 REST API를 호출해 새로운 게임 서버(Edgegap에서는 Deployment라고 합니다)를 배포합니다. 배포하는 시간은 배포가 얼마나 자주 이뤄지냐에 따라 컨테이너의 캐싱 여부가 달라져 짧게 걸릴 수도 있고 오래 걸릴 수도 있기 때문에 백엔드는 배포가 완료되기까지 기다리지 않고 배포에 대한 티켓을 클라이언트로 전달합니다. 게임 클라이언트는 초 단위로 전달받은 티켓을 다시 백엔드 서버로 전달해 현재 배포 상태를 수시로 확인합니다. 배포가 완료되면 백엔드 서버는 게임 클라이언트에 게임 서버의 주소와 함께 게임 클라이언트로 전달하고 게임 클라이언트는 전달받은 게임 서버의 주소로 접속합니다. (일반적인 매치 메이킹의 티켓 방식을 사용했습니다.)

Edgegap에 의해 배포된 게임 서버는 시작 시 Entry라는 레벨이 열립니다. 이 Entry 레벨은 아무것도 없는 빈 레벨입니다. 게임모드에서는 현재 레벨이 Entry 레벨일 때 서버가 새로 시작한 것으로 간주하고 서버 초기화를 시작합니다.

초기화 과정에서는 커맨드라인으로 전달받은 값을 확인하여 이 서버가 커스텀 매치 인지, 매치 메이킹 인지와 어떤 게임을 플레이할 것 인지에 따라 다르게 초기화합니다.

```cpp
void ASMGameMode::InitializeServerMatch() const
{
    FString DefaultMap = UGameMapsSettings::GetGameDefaultMap();
    UWorld* World = GetWorld();
    UGameInstance* GameInstance = World->GetGameInstance();
    if (!GameInstance || World->GetNetMode() != NM_DedicatedServer || World->URL.Map != DefaultMap)
    {
        return;
    }

    USMMatchSubsystem* MatchSubsystem = USMMatchSubsystem::Get(this);
    if (!MatchSubsystem)
    {
        UE_LOG(LogStereoMix, Error, TEXT("Match subsystem not found."));
        return;
    }

    const USMMatchDefinition* MatchDefinition = MatchSubsystem->GetMatchDefinition();
    if (!MatchDefinition)
    {
        UE_LOG(LogStereoMix, Error, TEXT("Match definition not found."));
        return;
    }

    if (MatchSubsystem->GetServerType() == ESMMatchServerType::OnlineMatch)
    {
        StartOnlineMatch(MatchDefinition);
    }
    else
    {
        StartCustomMatch(MatchDefinition);
    }
}

```

 현재 저희 서버는 매치 메이킹에 대해서는 구현되어 있지 않기 때문에 로비를 생성하는 커스텀 매치에 관해서 설명하겠습니다.

 저희 게임에서 커스텀 매치라는 것은 유저가 로비를 열고 다른 유저들이 그 로비로 들어오면 해당 로비의 오너가 게임을 시작하고, 게임이 끝나면 다시 로비로 이동하는 순환 방식의 게임을 말합니다.

 이를 세션의 라이프사이클과 함께 그림으로 표현해 봤습니다.

![](/assets/blobs/GameServer-LifeCycle.png)

위 그림에서는 커스텀 매치의 흐름과 EOS 세션의 생성, 시작, 종료 시점을 표현하고 있습니다. EOS 세션은 CreateSession 시점부터 다른 유저의 세션 목록에 표시되고, StartSession부터 EndSession까지는 입장이 불가합니다. 따라서 캐릭터 선택 구간부터는 StartSession을 호출하여 유저가 더 이상 입장하지 못하도록 했습니다.

게임서버는 유저가 나갈 때마다 게임서버에 남은 인원을 확인하여 아무도 없을 시 자동으로 삭제됩니다.

원래는 로비 구간에 EOS 로비를 사용하여 웹 소켓으로 로비를 유지하다가 시작하는 시점에 로비 오너가 게임 서버를 생성하여 해당 게임 서버에서 플레이하는 방식으로 설계했으나 구현 중에 OnlineSubsystemEOS의 로비 기능이 완전하지 못하다는 점 (멤버 어트리뷰트 등에서 제대로 작동하지 않는 것 같았습니다) 등으로 인해 현재는 커스텀 매치의 모든 구간을 하나의 게임 서버에서 진행하는 형태입니다. 물론 이 방식은 커스텀 매치의 모든 플레이어가 나가기 전까지 게임 서버가 계속 돌아가야 하므로 유저가 로비에서 잠수를 타는 등 (…) 로비를 오래 유지할 경우 그만큼 비용이 지속적으로 나간다는 치명적인 단점이 있습니다.

## 추가 내용

서버 쪽 내용에 1학기 때 사용했던 gRPC와 Firestore, 그리고 2학기에 사용한 Datastore에 대한 부분도 넣고 싶었지만, 글이 너무 길어져 생략했습니다…만 gRPC 구현에 대해서는 GitHub에 공개되어 있으므로 다음 링크를 통해 확인할 수 있습니다.

[https://github.com/CK24-Surround/stereomix-grpc](https://github.com/CK24-Surround/stereomix-grpc)
