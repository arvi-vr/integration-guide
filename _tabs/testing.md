---
title: Testing
icon: fas fa-bug
order: 8
---

You can debug and check the correctness of the integration at an early stage -- even without installing the Client and the Admin panel on your computer, without creating a location and without uploading the game build to our website. To do this, download and run the [API Tester]({{ '/tabs/downloads-dev-tools/' | relative_url }}) utility.

> **Note**: If the Client is already installed on your computer, you should close it. API Tester cannot be launched simultaneously with the Client.

# Preparing for testing
To test the game using API Tester, during testing you need to use the test game key specified in API Tester. It should be used as the **AppKey** of the game.

<a href="{{ '/assets/img/api-tester-appkey.png' | relative_url }}" data-toggle="lightbox"><img src="{{ '/assets/img/api-tester-appkey.png' | relative_url }}" class="img-fluid" style="max-width:500px;width:100%;" /></a>

# Testing game-to-platform requests
The first thing to do is to check if the game is verified correctly. To do this, start API Tester and then start the game. As you remember, the `IsApplicationEntitled()` method in [Unity]({{ '/tabs/integrations-unity/#integration-isapplicationentitled' | relative_url }}) and [Unreal Engine]({{ '/tabs/integrations-unreal/#integration-isapplicationentitled' | relative_url }}) integrations must be called early in the game.  
If everything is done correctly, in API Tester you will see a verification request from the game.

<a href="{{ '/assets/img/api-tester-isapplicationentitled.png' | relative_url }}" data-toggle="lightbox"><img src="{{ '/assets/img/api-tester-isapplicationentitled.png' | relative_url }}" class="img-fluid" style="max-width:500px;width:100%;" /></a>

Similarly, you can check the rest of the commands sent from the game. Thus, if you see messages with commands in the API Tester, it means that everything has been done correctly, and the Client will also receive these commands. In addition, you can make sure that the parameters and data transmitted by the command are correct.

<a href="{{ '/assets/img/api-tester-commands-from-game.png' | relative_url }}" data-toggle="lightbox"><img src="{{ '/assets/img/api-tester-commands-from-game.png' | relative_url }}" class="img-fluid" style="max-width:500px;width:100%;" /></a>

## Testing Session Data
If you set or get some session data from the game, you will see corresponding notifications in the API Tester with the data names and values.

<a href="{{ '/assets/img/api-tester-session-data.png' | relative_url }}" data-toggle="lightbox"><img src="{{ '/assets/img/api-tester-session-data.png' | relative_url }}" class="img-fluid" style="max-width:500px;width:100%;" /></a>

# Testing platform-to-game requests
## Testing Time Left request
Using API Tester, you can check the handling of **Time Left** request in [Unity]({{ '/tabs/integrations-unity/#integration-time-left-request' | relative_url }}) and [Unreal Engine]({{ '/tabs/integrations-unreal/#integration-time-left-request' | relative_url }}) integrations. To do this, at the bottom of the window enter the command `/timeleft` and press **SEND**. If your game correctly handled the **Time Left** request and returned the result, you will see the response in the API Tester window.

<a href="{{ '/assets/img/api-tester-time-left.png' | relative_url }}" data-toggle="lightbox"><img src="{{ '/assets/img/api-tester-time-left.png' | relative_url }}" class="img-fluid" style="max-width:500px;width:100%;" /></a>

## Testing Player Position request
Similarly, you can check the correctness of the **Player Position** request handling. We remind you -- with the help of it in [Unity]({{ '/tabs/integrations-unity/#integration-player-position-request' | relative_url }}) and [Unreal Engine]({{ '/tabs/integrations-unreal/#integration-player-position-request' | relative_url }}) integrations you return the data about in-game player's position. So, to test this request, enter the command `/position` and hit **SEND**. If you have correctly returned the player's position, then in the API Tester window you will see JSON data with coordinates.

<a href="{{ '/assets/img/api-tester-player-position.png' | relative_url }}" data-toggle="lightbox"><img src="{{ '/assets/img/api-tester-player-position.png' | relative_url }}" class="img-fluid" style="max-width:500px;width:100%;" /></a>

## Testing in-game commands
API Tester can also emulate sending in-game commands to the game. If at the stage of [creating the manifest]({{ '/tabs/getting-started-adding-the-game/#creating-manifest' | relative_url }}) you added support for [in-game commands]({{ '/tabs/getting-started-adding-the-game/#manifest-ingame-commands' | relative_url }}) for your game, then remember that when creating commands you set them the **`Command Url`** field. Copy this field from the command you want to test, paste it into the command field in API Tester and click **SEND**. The command will be sent to the game as if the operator from the Admin panel had done it.

<a href="{{ '/assets/img/api-tester-in-game-command.png' | relative_url }}" data-toggle="lightbox"><img src="{{ '/assets/img/api-tester-in-game-command.png' | relative_url }}" class="img-fluid" style="max-width:500px;width:100%;" /></a>

## Testing Player Name and Player Dominant Hand
You can test the setting of player's name and dominant hand by specifying their values in the corresponding fields and clicking **SET**.

<a href="{{ '/assets/img/api-tester-set-name-hand.png' | relative_url }}" data-toggle="lightbox"><img src="{{ '/assets/img/api-tester-set-name-hand.png' | relative_url }}" class="img-fluid" style="max-width:500px;width:100%;" /></a>

You can test setting the player's name and dominant hand from the game using the `PlayerName` and `PlayerDominantHand` properties in Unity or `SetPlayerName()` and `SetPlayerDominantHand()` methods in Unreal Engine. When you set the name and dominant hand in the game, you will see a notifications about their changes. The values of the corresponding fields in the API Tester will also be changed.

<a href="{{ '/assets/img/api-tester-name-hand-changed.png' | relative_url }}" data-toggle="lightbox"><img src="{{ '/assets/img/api-tester-name-hand-changed.png' | relative_url }}" class="img-fluid" style="max-width:500px;width:100%;" /></a>
