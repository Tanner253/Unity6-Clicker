# Unity6-Clicker
Deployed game 


deployed to google store too

- 4/22
enhanced leaderboards, feedback on production rate now includes manual clicks, prestige costs scales, lchat ui fixed for mobile 

- 4/23
fixed username logic, moved username logic to settings panel, age verificationd oes not reset username now, feedback renders decimals again, prestie costs scales exponentially

- 4/24 setup refactor
- 4/25 refactor complete - fruit upgrades now scalable and use OOP for duplication and reusabiltiy. prestige cost scaling balance. lemon spawn balance. formatting fix on count, username saving logic retouched.
- 4/28 :
Initial Problem: I observed that after successfully identifying a player and receiving a JWT access token via a POST to /api/players/identify, subsequent authorized GET requests (like /api/players/{id}/gameState and /api/players/{id}/muted) were failing with a 403 Forbidden error in my Unity client. The API logs confirmed this, indicating the user subject (sub claim) was being read as NULL within the controller action, causing my authorization check to fail.
Investigation & Diagnostics:
I first checked my API's appsettings.json and corrected the Jwt:Issuer and Jwt:Audience settings to match my local development URL (http://localhost:5238) instead of the production Azure URL.
When the issue persisted, I enabled more detailed Debug logging in the ASP.NET Core API.
I used curl to send requests directly to the API. Initial attempts failed due to connection errors or using incorrect placeholder tokens (resulting in 401 Unauthorized). However, sending a request with the correct token via curl resulted in a 403 Forbidden, matching the Unity client's behavior but suggesting the server could read the Authorization header if sent correctly.
I increased API logging further to Trace and added detailed logging to the OnMessageReceived event handler in the JWT configuration.
As a diagnostic step, I modified the Unity client to send the token in an additional custom header (X-Custom-AuthToken) and updated the API controller to log this header's value. The API logs confirmed the custom header arrived, but the standard Authorization header was still not resulting in a valid user principal (Token Sub='NULL') within the controller for the Unity requests.
I attempted a workaround by modifying the OnMessageReceived handler to read the token from the custom header if the standard one was missing. This still resulted in Token Sub='NULL' in the controller.
Root Cause Identified: Based on the logs showing successful authentication attempts (token validation succeeding) but the controller failing to get the user ID, I realized the problem wasn't with token validation by the middleware, but with how the claim was being accessed within the controller action. I was using User.FindFirstValue(JwtRegisteredClaimNames.Sub).
The Fix: I changed the claim retrieval logic within the controller actions (GetGameState, GetMutedPlayers, UpdateGameState, etc.) to use the standard ASP.NET Core identifier claim type: User.FindFirstValue(ClaimTypes.NameIdentifier). The JWT middleware maps the incoming sub claim to ClaimTypes.NameIdentifier by default.
Resolution & Cleanup: After implementing this change, the API logs correctly showed Token Sub='3', the authorization checks passed (AuthCheck PASSED), and both the GET and subsequent PUT requests succeeded with 200 OK / 204 No Content statuses. I then removed the diagnostic custom header code (X-Custom-AuthToken) from the Unity client and the OnMessageReceived handler from the API's JWT configuration, as they were no longer necessary.
The JWT authentication and authorization flow is now working as intended, ensuring users can only access their own data.


- 4/29:

I investigated why the fruit timer debug values weren't updating correctly.
I added debug logs to the FruitManager.FruitSpawnLoop coroutine to confirm the nextSpawnTimestamp was being set appropriately.
I added logs to the FruitManager.Update method to verify the secondsRemaining calculation.
I identified that the RestartAllActiveSpawnLoops method was incorrectly resetting the debug timer values (nextSpawnTimestamp and secondsRemaining) to -1 immediately after starting the coroutine.
I removed the lines that reset the debug timer values in RestartAllActiveSpawnLoops, allowing the coroutine itself to set the correct initial timestamp.
I reviewed the logic for how purchasing spawn rate upgrades affects the fruit timers and confirmed it correctly reduces the time between spawns by modifying the currentMinSpawnTime and currentMaxSpawnTime based on the PrestigeManager bonus.
I diagnosed an issue where the prestige shop scroll view snapped back to the top after scrolling.
I found that an unnecessary Content Size Fitter component was added to the Viewport GameObject instead of the Content GameObject.
I removed the Content Size Fitter from the Viewport GameObject, resolving the scrolling issue.
I identified a discrepancy where the upgrade descriptions for Lemon Value and Kiwi Value were showing the same "Current" and "Next level" stats, despite having different underlying data.
I determined that the PrestigeUpgradeButtonUI.UpdateStatsDisplay method needed fruit-specific data.
I added fruit-specific accessor methods (GetCurrentLifespan, GetCurrentRewardMinutes, GetCurrentSpawnTimes) to FruitManager.
I added a _fruitUnlockId field to PrestigeUpgradeButtonUI to associate buttons with specific fruits.
I modified the SetupFromExplicitSubData method in PrestigeUpgradeButtonUI to correctly store the _fruitUnlockId and handle both the legacy Lemon... and generic Fruit... PrestigeEffectType enums using fall-through logic in the switch statement, allowing the UI to fetch the correct data.
I resolved compiler errors in PrestigeUpgradeButtonUI that occurred after a faulty automated edit by restoring the necessary methods and variables and correctly reapplying the SetupFromExplicitSubData changes.
I investigated why the Lemon Value upgrade showed "5 min CPS" when its base value was 2 and it was only level 1.
I added debug logs to FruitManager.ManageSingleFruitLoop to trace the currentRewardMinutes calculation.
I corrected the debug logs after introducing a bug related to fetching the upgrade level.
I confirmed via the logs that the calculation BaseMins (2) + BonusMins (3) = Final RewardMins (5) was correct based on the valueEffectPerLevel of 3 configured for the Lemon Value upgrade.
I diagnosed an issue where offline time was not accruing per click, even though it was a previously working feature.
I traced the RegisterClick call from GameManager.ProcessClick to OfflineProgressionService.
I determined that the _offlineServiceInitialized flag in GameManager was likely false when clicks occurred because the initialization depended on API data loading.
I found that the InitializeOfflineProgressionServiceIfNeeded method wasn't being called because the flow was exiting early in ProcessOfflineCheckUsingApiData.
I diagnosed a subsequent issue where the offline progress alert panel wasn't appearing upon returning to the game, even though time accrual per click was now working.
I identified that GameManager.FindOfflineProgressionUI likely wasn't finding the panel because it might be inactive on scene load.
I modified FindOfflineProgressionUI to use FindFirstObjectByType<OfflineProgressionUI>(FindObjectsInactive.Include), restoring the connection to the UI panel and fixing the offline alert popup.
