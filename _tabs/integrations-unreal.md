---
title: Unreal Engine
order: 7
menu: child
---

# General requirements to the game
To avoid unwanted firewall initiation, ARVI platform creates firewall rules for the following port range during its installation: **1770, 7777-7780**. We advise developers to choose a network port for this game from this specific range.  
Please also note that ARVI platform uses a number of ports for its own purposes, and the game should not be using them. The following ports should not be used: **3580, 3581, 3585, 3587, 5586, 11000-11020, 11022, 22022**.  

# SDK installation and setup
This Integration SDK requires **Unreal Engine 4.22** or newer. It comes in the form of a plugin and available in few options:
* Source code plugin
* Compiled plugin for Unreal Engine 4.22
* Compiled plugin for Unreal Engine 4.25

All these options are available in [DOWNLOADS]({{ '/tabs/downloads-unreal/' | relative_url }}) section.

## Installation from sources
If you decide to install the plugin from sources, then follow next steps:
1. Download sources from GitHub
    * Go to [https://github.com/arvi-vr/unreal-integration](https://github.com/arvi-vr/unreal-integration)
    * Click the "Code" button and select "Download Zip"

2. In your project's root directory, create a new folder if it doesn't exist: **Plugins**.

3. Unzip the downloaded file directly to the **Plugins** folder. There should now be a folder called unreal-integration-master; you can rename this if you'd like.

4. Finally, double-click the *.uproject file at the root of your project to compile and open the project. If you are prompted to rebuild any files select yes.

## Compiled plugin installation
If you decide to install the compiled plugin, then follow the next steps:
1. Download compiled plugin of appropriate version from [DOWNLOADS]({{ '/tabs/downloads-unreal/' | relative_url }}) section.

2. In your project's root directory, create a new folder if it doesn't exist: **Plugins**.

3. Unzip the downloaded file directly to the **Plugins** folder.

Go to **Edits 🡒 Plugins** to check that the plugin is installed and set up correctly.

<img src="{{ '/assets/img/unreal-integration-plugins-menu.png' | relative_url }}" class="img-fluid" />
<img src="{{ '/assets/img/unreal-integration-plugins-window.png' | relative_url }}" height="150px" class="img-fluid"/>

# Platform integration
In source code, integration comes in the form of a game instance sub-system. The object is accessible via the `UGameInstance` class object.
```c++
#include "ARVIIntegrationSubSystem.h"
...
void AExampleActor::BeginPlay()
{
  Super::BeginPlay();
  if (UGameInstance* GameInstance = GetGameInstance())
    if (UARVIIntegrationSubSystem* ArviIntegration = GameInstance->GetSubsystem<UARVIIntegrationSubSystem>())
    {
      UE_LOG(LogIntegrationExample, Log, TEXT("ARVI SDK %s"), *ArviIntegration->GetSDKVersion());
      if (ArviIntegration->GetMessagesWasInitialize())
      {
        UE_LOG(LogIntegrationExample, Log, TEXT("Initialized successfully"));
      }
      else
      {
        UE_LOG(LogIntegrationExample, Warning, TEXT("Initialization failed. Error message: %s"), *ArviIntegration->GetMessagesError());
      }
    }
}
```
The class is described in `ARVIIntegrationSubSystem.h`. Class methods duplicate the names of communication methods. All methods receive the required parameters and two asynchronous operations completion handlers, and the returned value indicates whether the operation was queued for handling.  
You may also use Blueprint:

<img src="{{ '/assets/img/bp-initialization.png' | relative_url }}" class="img-fluid" />

<a name="integration-isapplicationentitled"></a>
## Verification
To make sure that the game is launched on the computer with the running ARVI platform and is authorized for launch at the current game location, the verification process should be executed as soon as possible. The check does not require an Internet connection.
Run the following code to execute the check:
```c++
  void AExampleActor::BeginPlay()
  {
    Super::BeginPlay();
    if (UGameInstance* GameInstance = GetGameInstance())
      if (UARVIIntegrationSubSystem* ArviIntegration = GameInstance->GetSubsystem<UARVIIntegrationSubSystem>())
        if (!ArviIntegration->IsApplicationEntitled(AppKey,
              FOnARVIIntegrationRequesCompleted::CreateUObject(this, &ThisClass::HandlerApplicationEntitledCompleted),
              FOnARVIIntegrationRequesFailed::CreateUObject(this, &ThisClass::HandlerApplicationEntitledFailed)))
          HandlerApplicationEntitledFailed(-1, TEXT("Request not created"));
  }

  void AExampleActor::HandlerApplicationEntitledCompleted()
  {
    UE_LOG(LogIntegrationExample, Log, TEXT("Entitlement check passed"));
  }

  void AExampleActor::HandlerApplicationEntitledFailed(int ErrorCode, FString ErrorMessage)
  {
    UE_LOG(LogIntegrationExample, Error, TEXT("Entitlement check failed. Error code: %d. Error message: %s"), ErrorCode, *ErrorMessage);
    UKismetSystemLibrary::QuitGame(this, nullptr, EQuitPreference::Type::Quit, false);
  }
```
Or use Blueprint:

<img src="{{ '/assets/img/bp-entitlement.png' | relative_url }}" class="img-fluid" />

**AppKey** is a [Game Key]({{ '/tabs/getting-started-adding-the-game/#game-key' | relative_url }}), you can get this key in the game info section of your developer account on our website.

If the check fails, **game must shut down**.

## Platform request handling
From time to time, the platform may be sending requests for certain information. There are several types of requests that may be sent by the platform:
* **Time Left Request**. If the game has any time limitations, it will respond with the number of seconds left until the game is over. This information will appear in the platform control panel.
* **Player Position Request**. Our platform features 3D audio chat, and accurate voice positioning requires physical location of the player in the game world. The game will respond to this request by sending the player's Transform.

> **Note**: The frequency of the Player Position Request is approximately 10 times per second.

<a name="integration-time-left-request"></a>
Handling example **Time Left Request**:
```c++
void AExampleActor::BeginPlay()
{
  Super::BeginPlay();
  if (UGameInstance* GameInstance = GetGameInstance())
    if (UARVIIntegrationSubSystem* ArviIntegration = GameInstance->GetSubsystem<UARVIIntegrationSubSystem>())
      ArviIntegration->TimeLefRequestHandler.BindDynamic(this, &ThisClass::HandlerTimeLeft);
}

bool AExampleActor::HandlerTimeLeft(int& TimeLeftInSeconds)
{
  TimeLeftInSeconds = FMath::TruncToInt(FMath::Max(GameDuration - (GetGameTimeSinceCreation() - StartTime), 0.0f));
  return true;
}
```

<img src="{{ '/assets/img/bp-time-left-1.png' | relative_url }}" class="img-fluid" />

<img src="{{ '/assets/img/bp-time-left-2.png' | relative_url }}" class="img-fluid" />

<a name="integration-player-position-request"></a>
Handling example **Player Position Request**:
```c++
bool AExampleActor::HandlerPlayerPosition(FTransform& Transform)
{
  if (UGameInstance* GameInstance = GetGameInstance())
    if (APlayerController * PlayerController = GameInstance->GetFirstLocalPlayerController())
      if (APawn* Pawn = PlayerController->GetPawn())
        if (UCameraComponent* Camera = Pawn->FindComponentByClass<UCameraComponent>())
        {  
          Transform = Camera->GetComponentTransform();
          return true;
        }
  return false;
}

void AExampleActor::BeginPlay()
{
  Super::BeginPlay();
  if (UGameInstance* GameInstance = GetGameInstance())
    if (UARVIIntegrationSubSystem* ArviIntegration = GameInstance->GetSubsystem<UARVIIntegrationSubSystem>())
      ArviIntegration->PlayerPositionRequestHandler.BindDynamic(this, &ThisClass::HandlerPlayerPosition);
}
```

<img src="{{ '/assets/img/bp-player-position-1.png' | relative_url }}" class="img-fluid" />

<img src="{{ '/assets/img/bp-player-position-2.png' | relative_url }}" class="img-fluid" />

## Handling of game commands
In addition to the above-mentioned standard requests, the platform may send [in-game commands]({{ '/tabs/getting-started-adding-the-game/#in-game-commands' | relative_url }}) that you can create in the developer's account on our website. They will be sent into the game from our control panel by the operator.
For example, on the website, you created a SkipTask command that skips tasks in the game.  To make sure that your game handles this command, you need to subscribe to messages from the platform and specify the handler that will execute SkipTask check.  
```c++
#include "ARVIPlatformMessage.h"
#include "ARVIIntegrationLibrary.h"
...
void AExampleActor::BeginPlay()
{
  Super::BeginPlay();
  if (UGameInstance* GameInstance = GetGameInstance())
    if (UARVIIntegrationSubSystem* ArviIntegration = GameInstance->GetSubsystem<UARVIIntegrationSubSystem>())
      ArviIntegration->PlatformMessageRequestHandler.BindDynamic(this, &ThisClass::HandlerPlatformMessages);
}

bool AExampleActor::HandlerPlatformMessages(const FARVIPlatformMessage& Message, FString& ContentType, TArray<uint8>& Data)
{
  if (Message.Name == FString(TEXT("SkipTask"))) {
    SkipTask();
    return true;
  }
  else if (Message.Name == FString(TEXT("OpenDoor"))) {
    OpenDoor(Message.Params[0].Name);
    return true;
  }
  else if (Message.Name == FString(TEXT("GetCurrentTask"))) {
    Data = UARVIIntegrationLibrary::ConvertPlatformMessageDataFromString(GetCurrentTaskName());
    ContentType = TEXT("text/plain");
    return true;
  }
  return false;
}
```

<img src="{{ '/assets/img/bp-platform-messages-1.png' | relative_url }}" class="img-fluid" />

<img src="{{ '/assets/img/bp-platform-messages-2.png' | relative_url }}" class="img-fluid" />

## Sending messages to the platform
### Game server start message
To ensure correct operation of the platform, the computer acting as the game server must report that the game server has started. Use the `ServerStarted()` method to send this message. A single-player game **must** send this message immediately upon startup.
```c++
void AExampleActor::BeginPlay()
{
  Super::BeginPlay();
  if (UGameInstance* GameInstance = GetGameInstance())
    if (UARVIIntegrationSubSystem* ArviIntegration = GameInstance->GetSubsystem<UARVIIntegrationSubSystem>())
      if (!ArviIntegration->ServerStarted(
          FOnARVIIntegrationRequesCompleted::CreateUObject(this, &ThisClass::HandlerServerStartedSuccessed),
          FOnARVIIntegrationRequesFailed::CreateUObject(this, &ThisClass::HandlerServerStartedFailed)))
        HandlerServerStartedFailed(-1, TEXT("Request not created"));
}

void AExampleActor::HandlerServerStartedSuccessed()
{
  UE_LOG(LogIntegrationExample, Log, TEXT("ServerStarted completed."));
}

void AExampleActor::HandlerServerStartedFailed(int ErrorCode, FString ErrorMessage)
{
  UE_LOG(LogIntegrationExample, Error, TEXT("ServerStarted failed. Error code: %d. Error message: %s"), ErrorCode, *ErrorMessage);
}
```

<img src="{{ '/assets/img/bp-server-started.png' | relative_url }}" class="img-fluid" />

### Game completed notification
The game must also notify the platform of its completion. Use the `GameCompleted()` method for this. We recommend call it just before closing the game, and close the game in its `HandlerGameCompletedSuccessed` handler.
```c++
void AExampleActor::BeginPlay()
{
  Super::BeginPlay();
  if (UGameInstance* GameInstance = GetGameInstance())
    if (UARVIIntegrationSubSystem* ArviIntegration = GameInstance->GetSubsystem<UARVIIntegrationSubSystem>())
      if (!ArviIntegration->GameCompleted(
        FOnARVIIntegrationRequesCompleted::CreateUObject(this, &ThisClass::HandlerGameCompletedSuccessed),
        FOnARVIIntegrationRequesFailed::CreateUObject(this, &ThisClass::HandlerGameCompletedFailed)))
        HandlerGameCompletedFailed(-1, TEXT("Request not created"));
}


void AExampleActor::HandlerGameCompletedSuccessed()
{
  UE_LOG(LogIntegrationExample, Log, TEXT("GameCompleted completed."));
}

void AExampleActor::HandlerGameCompletedFailed(int ErrorCode, FString ErrorMessage)
{
  UE_LOG(LogIntegrationExample, Error, TEXT("GameCompleted failed. Error code: %d. Error message: %s"), ErrorCode, *ErrorMessage);
}
```

<img src="{{ '/assets/img/bp-game-completed.png' | relative_url }}" class="img-fluid" />

### Operator call
Sometimes a player may require the operator's assistance. You can call the `CallOperator()` method, which will trigger a message on the operator's control panel indicating that the player needs help.
```c++
void AExampleActor::BeginPlay()
{
  Super::BeginPlay();
  if (UGameInstance* GameInstance = GetGameInstance())
    if (UARVIIntegrationSubSystem* ArviIntegration = GameInstance->GetSubsystem<UARVIIntegrationSubSystem>())
      if (!ArviIntegration->CallOperator(
        FOnARVIIntegrationRequesCompleted::CreateUObject(this, &ThisClass::HandlerCallOperatorSuccessed),
        FOnARVIIntegrationRequesFailed::CreateUObject(this, &ThisClass::HandlerCallOperatorFailed)))
        HandlerCallOperatorFailed(-1, TEXT("Request not created"));
}

void AExampleActor::HandlerCallOperatorSuccessed()
{
  UE_LOG(LogIntegrationExample, Log, TEXT("CallOperator completed."));
}

void AExampleActor::HandlerCallOperatorFailed(int ErrorCode, FString ErrorMessage)
{
  UE_LOG(LogIntegrationExample, Error, TEXT("CallOperator failed. Error code: %d. Error message: %s"), ErrorCode, *ErrorMessage);
}
```

<img src="{{ '/assets/img/bp-call-operator.png' | relative_url }}" class="img-fluid" />

### Audio chat channel selection
By default, all players in the game are on a common audio chat and can hear one another. However, multi-player games that support multiple teams might require team-based separation of audio streams. For example, players in the same team have to hear one another only, and not any other teams. For this purpose, the game may ask the platform to assign a relevant audio chat channel to the user that will correspond to the player's team number. This can be done using the `SetAudioChatChannel(channel)` method, where **channel** is the channel number. Channels 1...10 and Public are supported.
```c++
void AExampleActor::BeginPlay()
{
  Super::BeginPlay();
  if (UGameInstance* GameInstance = GetGameInstance())
    if (UARVIIntegrationSubSystem* ArviIntegration = GameInstance->GetSubsystem<UARVIIntegrationSubSystem>())
      if (!ArviIntegration->SetAudioChatChannel(EAudioChatChannel::ACC_Team01,
        FOnARVIIntegrationRequesCompleted::CreateUObject(this, &ThisClass::HandlerSetAudioChatChannelSuccessed),
        FOnARVIIntegrationRequesFailed::CreateUObject(this, &ThisClass::HandlerSetAudioChatChannelFailed)))
        HandlerSetAudioChatChannelFailed(-1, TEXT("Request not created"));
}

void AExampleActor::HandlerSetAudioChatChannelSuccessed()
{
  UE_LOG(LogIntegrationExample, Log, TEXT("SetAudioChatChannel completed."));
}

void AExampleActor::HandlerSetAudioChatChannelFailed(int ErrorCode, FString ErrorMessage)
{
  UE_LOG(LogIntegrationExample, Error, TEXT("SetAudioChatChannel failed. Error code: %d. Error message: %s"), ErrorCode, *ErrorMessage);
}
```

<img src="{{ '/assets/img/bp-audiochat-channel.png' | relative_url }}" class="img-fluid" />

### Activating and deactivating in-game commands
You can control the display of in-game commands in the admin panel at a specific point in time. For example, the game provides an in-game command to restart the level, but you want to display it in the operator panel only after the level has started. Or you want to hide the puzzle solving command when it has already been solved.  
Use `ActivateInGameCommand(activationMessage)` and `DeactivateInGameCommand(deactivationMessage)` methods for these purposes. **activationMessage** and **deactivationMessage** are messages for activating and deactivating a specific command or several commands. For more information on activating and deactivating commands, see the Developer's Guide.
```c++
void AExampleActor::HandlerActivateInGameCommandSuccessed()
{
  UE_LOG(LogIntegrationExample, Log, TEXT("ActivateInGameCommand completed."));
}

void AExampleActor::HandlerActivateInGameCommandFailed(int ErrorCode, FString ErrorMessage)
{
  UE_LOG(LogIntegrationExample, Error, TEXT("ActivateInGameCommand failed. Error code: %d. Error message: %s"), ErrorCode, *ErrorMessage);
}

void AExampleActor::BeginPlay()
{
  Super::BeginPlay();
  if (UGameInstance* GameInstance = GetGameInstance())
    if (UARVIIntegrationSubSystem* ArviIntegration = GameInstance->GetSubsystem<UARVIIntegrationSubSystem>())
      if (!ArviIntegration->ActivateInGameCommand(TEXT("EnterInRoom"),
        FOnARVIIntegrationRequesCompleted::CreateUObject(this, &ThisClass::HandlerActivateInGameCommandSuccessed),
        FOnARVIIntegrationRequesFailed::CreateUObject(this, &ThisClass::HandlerActivateInGameCommandFailed)))
        HandlerSendLogMessageFailed(-1, TEXT("Request not created"));
}
```

<img src="{{ '/assets/img/bp-activate-command.png' | relative_url }}" class="img-fluid" />

```c++
void AExampleActor::HandlerDeactivateInGameCommandSuccessed()
{
  UE_LOG(LogIntegrationExample, Log, TEXT("DeactivateInGameCommand completed."));
}

void AExampleActor::HandlerDeactivateInGameCommandFailed(int ErrorCode, FString ErrorMessage)
{
  UE_LOG(LogIntegrationExample, Error, TEXT("DeactivateInGameCommand failed. Error code: %d. Error message: %s"), ErrorCode, *ErrorMessage);
}

void AExampleActor::BeginPlay()
{
  Super::BeginPlay();
  if (UGameInstance* GameInstance = GetGameInstance())
    if (UARVIIntegrationSubSystem* ArviIntegration = GameInstance->GetSubsystem<UARVIIntegrationSubSystem>())
      if (!ArviIntegration->DeactivateInGameCommand(TEXT("LeaveRoom"),
        FOnARVIIntegrationRequesCompleted::CreateUObject(this, &ThisClass::HandlerDeactivateInGameCommandSuccessed),
        FOnARVIIntegrationRequesFailed::CreateUObject(this, &ThisClass::HandlerDeactivateInGameCommandFailed)))
        HandlerSendLogMessageFailed(-1, TEXT("Request not created"));
}
```

<img src="{{ '/assets/img/bp-deactivate-command.png' | relative_url }}" class="img-fluid" />

### Sending messages
If you need to report any event in the game (such as a puzzle solved), you can send a text message to the platform. It will be saved in game logs and will be available for display in the game session information on the website. Use the `SendGameMessage(message, messageGroup)` method to forward your message. As an optional second parameter, you can specify the group (designated using any word of your choosing), and all messages referencing this group will be grouped when displayed. Examples:
```c++
void AExampleActor::BeginPlay()
{
  Super::BeginPlay();
  if (UGameInstance* GameInstance = GetGameInstance())
    if (UARVIIntegrationSubSystem* ArviIntegration = GameInstance->GetSubsystem<UARVIIntegrationSubSystem>())
      if (!ArviIntegration->SendGameMessage(TEXT("SomeText"), TEXT("SomeGroup"),
        FOnARVIIntegrationRequesCompleted::CreateUObject(this, &ThisClass::HandlerSendGameMessageSuccessed),
        FOnARVIIntegrationRequesFailed::CreateUObject(this, &ThisClass::HandlerSendGameMessageFailed)))
        HandlerSendGameMessageFailed(-1, TEXT("Request not created"));
}

void AExampleActor::HandlerSendGameMessageSuccessed()
{
  UE_LOG(LogIntegrationExample, Log, TEXT("SendGameMessage completed."));
}

void AExampleActor::HandlerSendGameMessageFailed(int ErrorCode, FString ErrorMessage)
{
  UE_LOG(LogIntegrationExample, Error, TEXT("SendGameMessage failed. Error code: %d. Error message: %s"), ErrorCode, *ErrorMessage);
}
```

<img src="{{ '/assets/img/bp-game-message.png' | relative_url }}" class="img-fluid" />


You can also send a message that will set the timescale to a specific color when it is displayed on it. For example, you want to see (as a color on the timescale) how long it took the player to complete a certain action. Call the `SendTrackingMessage(message)` method and specify the relevant marker as the message.
```c++
void AExampleActor::BeginPlay()
{
  Super::BeginPlay();
  if (UGameInstance* GameInstance = GetGameInstance())
    if (UARVIIntegrationSubSystem* ArviIntegration = GameInstance->GetSubsystem<UARVIIntegrationSubSystem>())
      if (!ArviIntegration->SendTrackingMessage(TEXT("SomeText"),
        FOnARVIIntegrationRequesCompleted::CreateUObject(this, &ThisClass::HandlerSendTrackingMessageSuccessed),
        FOnARVIIntegrationRequesFailed::CreateUObject(this, &ThisClass::HandlerSendTrackingMessageFailed)))
        HandlerSendTrackingMessageFailed(-1, TEXT("Request not created"));
}

void AExampleActor::HandlerSendTrackingMessageSuccessed()
{
  UE_LOG(LogIntegrationExample, Log, TEXT("SendTrackingMessage completed."));
}

void AExampleActor::HandlerSendTrackingMessageFailed(int ErrorCode, FString ErrorMessage)
{
  UE_LOG(LogIntegrationExample, Error, TEXT("SendTrackingMessage failed. Error code: %d. Error message: %s"), ErrorCode, *ErrorMessage);
}
```

<img src="{{ '/assets/img/bp-tracking-message.png' | relative_url }}" class="img-fluid" />

Sometimes you might need to send a technical message without cluttering the game session log. For example, you want to log an error message, or send the error code, module and method that initiated it, etc. For these purposes, we recommend using the `SendLogMessage(message)` method. It will save your message in a separate "technical" log only without affecting the game log. Example:
```c++
void AExampleActor::BeginPlay()
{
  Super::BeginPlay();
  if (UGameInstance* GameInstance = GetGameInstance())
    if (UARVIIntegrationSubSystem* ArviIntegration = GameInstance->GetSubsystem<UARVIIntegrationSubSystem>())
      if (!ArviIntegration->SendLogMessage(TEXT("SomeText"),
        FOnARVIIntegrationRequesCompleted::CreateUObject(this, &ThisClass::HandlerSendLogMessageSuccessed),
        FOnARVIIntegrationRequesFailed::CreateUObject(this, &ThisClass::HandlerSendLogMessageFailed)))
        HandlerSendLogMessageFailed(-1, TEXT("Request not created"));
}

void AExampleActor::HandlerSendLogMessageSuccessed()
{
  UE_LOG(LogIntegrationExample, Log, TEXT("SendLogMessage completed."));
}

void AExampleActor::HandlerSendLogMessageFailed(int ErrorCode, FString ErrorMessage)
{
  UE_LOG(LogIntegrationExample, Error, TEXT("SendLogMessage failed. Error code: %d. Error message: %s"), ErrorCode, *ErrorMessage);
}
```

<img src="{{ '/assets/img/bp-log-message.png' | relative_url }}" class="img-fluid" />

## Advanced settings
### Cord twist tracking
We recommend that developers implement cord twist tracking for VR headset in their games. This avoids damage of the VR headset and prolongs its service life. However, in some cases (for example, for wireless headsets) such tracking should not be used.  
Our platform sends information about whether it is necessary to activate cord twist tracking or not. Use the `GetShouldApplicationTrackCordTwisting()` function at any time to determine if the game should activate cord twist tracking.
```c++
void AExampleActor::BeginPlay()
{
  Super::BeginPlay();
  if (UGameInstance* GameInstance = GetGameInstance())
    if (UARVIIntegrationSubSystem* ArviIntegration = GameInstance->GetSubsystem<UARVIIntegrationSubSystem>())
    {
      if (ArviIntegration->GetShouldApplicationTrackCordTwisting())
      {

      }
      else
      {

      }
    }
}
```

<img src="{{ '/assets/img/bp-cord-twisting.png' | relative_url }}" class="img-fluid" />

### Trial Mode
When starting the game from the control panel, the operator in some cases may indicate that the game should be launched in Trial Mode. Typically, this launch mode is used for testing or demonstration purposes and is not intended for commercial use.
You can use the `GetIsApplicationInTrialMode()` function to understand that the game is running in trial mode and display a corresponding message in the game about it. The message can also indicate that commercial use is prohibited.
```c++
void AExampleActor::BeginPlay()
{
  Super::BeginPlay();
  if (UGameInstance* GameInstance = GetGameInstance())
    if (UARVIIntegrationSubSystem* ArviIntegration = GameInstance->GetSubsystem<UARVIIntegrationSubSystem>())
    {
      if (ArviIntegration->GetIsApplicationInTrialMode())
      {

      }
      else
      {

      }
    }
}
```

<img src="{{ '/assets/img/bp-is-trial-mode.png' | relative_url }}" class="img-fluid" />

<script src="https://cdn.jsdelivr.net/npm/@widgetbot/crate@3" async defer>
  new Crate({
    server: '789059057814208523',
    channel: '789133359174778922'
  })
</script>
