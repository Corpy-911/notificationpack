https://github.com/Corpy-911/notificationpack/releases

# NotificationPack — Default Roblox Notification Pack for Games

[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github)](https://github.com/Corpy-911/notificationpack/releases)

🚀 A simple, modular pack that exposes the Roblox default notification UI as a ready-to-use package. Use the familiar Roblox look and feel as a reusable module. This README covers installation, usage, API, customization, examples, testing, and development guidelines.

![Roblox notification mockup](https://images.unsplash.com/photo-1526378727093-6f5b5fe1e2b8?q=80&w=1200&auto=format&fit=crop&ixlib=rb-1.2.1&s=0b4b8a6f1a6b9a0d0f1f5b2a9e9a3c2b)
Image: Example mockup of an in-game notification shown in a test scene.

Table of contents
- What this pack does
- Key features
- Included assets
- Compatibility
- Install (Download and Execute)
- Quick start
- Full API reference
- Examples and patterns
- Customization guide
- Performance notes
- Testing tips
- Troubleshooting
- Migration and upgrade
- Contribution guide
- Changelog
- License
- Credits

What this pack does
- Wraps the default Roblox notification visual and behavior into a module.
- Exposes a consistent API to push notifications from server or client.
- Supports text, title, icon, duration, and basic actions.
- Ships as a self-contained release asset you can drop into a place or require from a module.

Key features
- Familiar Roblox UI style. The notifications follow the default Roblox look.
- Minimal API. Five main calls cover most use cases.
- Client and server friendly. Call from server via RemoteEvents or call on client directly.
- Stacking. Notifications stack and animate in order.
- Queueing. The pack queues notifications to avoid overlap.
- Custom styles. Override colors, fonts, sizes, and animation.
- Lightweight. Small memory footprint. No extra frameworks required.

Included assets
- Module: NotificationPack (Main module)
- Example scripts: client_example.lua, server_example.lua
- Test place: test_place.rbxlx (optional)
- Readme and license
- A release build file: notificationpack.rbxm (packaged module model)

Compatibility
- Roblox Studio, live places
- Recommended Roblox client versions from 2021 onward
- Works with filtering enabled when used via remote calls
- Works with both Lua and Luau in Roblox

Install (Download and Execute)
- The release for this pack bundles the module and example files. Download the release asset named notificationpack.rbxm from the Releases page and execute it.
- Steps:
  1. Open the Releases page linked above.
  2. Download the file named notificationpack.rbxm from the release assets.
  3. Open Roblox Studio.
  4. Insert the downloaded .rbxm file into your game (File > Insert from file).
  5. Place the NotificationPack model under ReplicatedStorage or as a ModuleScript in ServerScriptService or StarterPlayerScripts depending on use.
  6. Require the module and follow Quick start.

Quick start
- Single client notification
  1. Put NotificationPack as a ModuleScript under StarterPlayerScripts or in a common place like ReplicatedStorage and require it in a LocalScript.
  2. Call notify with options.

Example LocalScript (StarterPlayerScripts)
```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local NotificationPack = require(ReplicatedStorage:WaitForChild("NotificationPack"))

NotificationPack:notify({
  title = "Welcome",
  text = "You joined the game",
  duration = 4,    -- seconds
  icon = "rbxassetid://123456789",
})
```

- Server-driven notifications (recommended for events)
  1. Use a RemoteEvent.
  2. Server triggers the RemoteEvent with payload.
  3. Client receives and calls the pack.

Example server script
```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local SendNotification = Instance.new("RemoteEvent")
SendNotification.Name = "SendNotification"
SendNotification.Parent = ReplicatedStorage

-- When something happens, fire the event to a player
game.Players.PlayerAdded:Connect(function(player)
  SendNotification:FireClient(player, {
    title = "Hello",
    text = "Server says hi",
    duration = 3,
  })
end)
```

Example client receiver
```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local NotificationPack = require(ReplicatedStorage:WaitForChild("NotificationPack"))
local SendNotification = ReplicatedStorage:WaitForChild("SendNotification")

SendNotification.OnClientEvent:Connect(function(payload)
  NotificationPack:notify(payload)
end)
```

Full API reference
- Module style: The main module returns an object with functions and configuration.

Main functions
- notify(options)
  - Show a notification on the client.
  - options table keys:
    - title (string) - required
    - text (string) - required
    - duration (number) - seconds to show; defaults to 5
    - icon (string or number) - image id or nil
    - style (string) - optional style token like "success" or "warning"
    - onComplete (function) - callback when the notification fully closes
    - id (string) - optional id to dedupe or update an existing notification
    - keepOnScreen (boolean) - when true, the notification stays until dismissed manually
  - Returns a notification handle table with methods:
    - dismiss() - remove the notification early
    - update(newOptions) - update text, title, or duration

- configure(settings)
  - Change pack defaults.
  - settings keys:
    - maxVisible (number) - maximum notifications stacked; default 4
    - spacing (number) - pixels between stacked notifications
    - animationTime (number) - seconds for show/hide animation
    - anchor (string) - "topRight", "topLeft", "bottomRight", "bottomLeft" (default "topRight")

- clearAll()
  - Remove all active notifications and clear queue.

- on(eventName, callback)
  - Subscribe to internal events:
    - "shown" - payload: notification handle
    - "closed" - payload: notification handle

Examples of usage patterns

1) One-off updateable notification
```lua
local n = NotificationPack:notify({ title = "Loading", text = "Preparing assets...", duration = 30, id = "load" })
-- update progress
NotificationPack:update("load", { text = "50% complete" })
-- finish
NotificationPack:update("load", { title = "Ready", text = "All assets loaded", duration = 3 })
```

2) Persistent notification that must be closed
```lua
local handle = NotificationPack:notify({
  title = "Important",
  text = "Complete the tutorial",
  keepOnScreen = true
})
-- later
handle:dismiss()
```

3) Alerts with action callback
```lua
local handle = NotificationPack:notify({
  title = "Friend Request",
  text = "PlayerX sent a request",
  duration = 10,
  onComplete = function()
    print("Player saw the request")
  end,
  id = "friend_req_42",
})
```

Customization guide
- Visual overrides
  - The pack exposes a theme table for color and font overrides.
  - Example:
    ```lua
    NotificationPack:configure({
      theme = {
        background = Color3.fromRGB(20, 20, 20),
        textColor = Color3.fromRGB(235, 235, 235),
        titleColor = Color3.fromRGB(255, 255, 255),
        accent = Color3.fromRGB(0, 150, 255),
      },
      font = Enum.Font.GothamBold,
      size = Vector2.new(350, 64),
    })
    ```
  - Fonts and sizes accept Roblox datatypes. Use Enum.Font for font selection.

- Layout
  - Anchor options: topRight, topLeft, bottomRight, bottomLeft.
  - Position can accept offsets via configure:
    ```lua
    NotificationPack:configure({ anchor = "bottomLeft", offset = UDim2.new(0, 12, 1, -12) })
    ```

- Behavior
  - You can set how many notifications show at once with maxVisible.
  - animationTime controls fade and slide duration.

Styling example sets
- Minimal (compact)
  - Smaller size, short duration, less spacing.
- Focused (large)
  - Larger fonts, more visible icon, longer duration.

Accessibility
- Use high contrast themes for readability.
- Use text-to-speech integrations by hooking into the onComplete or shown events.
- Keep duration long enough for users with low reading speed. The default is 5 seconds.

Performance notes
- The module pools UI frames. It reuses frames instead of creating new ones for each notification.
- The animation uses tweening via TweenService. Set animationTime to 0 to disable animations.
- Queue size remains small; do not flood players with hundreds of notifications per second.
- Keep icons small and use compressed images to reduce memory.

Testing tips
- Use the test_place.rbxlx included in the release file to see examples.
- Use the example LocalScript and ServerScript to simulate common flows.
- Test with filtering enabled. When sending messages from server, send only IDs and trust the client to render UI.
- Test on low-end devices via emulator in Studio.

Troubleshooting
- If notifications do not show:
  - Confirm NotificationPack module is required correctly.
  - Confirm the code runs on the client when calling notify.
  - If you call from server, confirm a RemoteEvent forwards the payload to client and filtering is enabled.
- If icons do not load:
  - Confirm the icon uses a valid asset id string or rbxassetid URL.
  - Some asset ids block loading on certain platforms.
- If animations stutter:
  - Reduce animationTime or switch to no-animation mode.
  - Check heavy loops on the client that may block render frames.

Advanced integration patterns
- Notification center UI
  - Build a persistent panel that shows history. Use the handle returned by notify to feed the center.
  - Save recent notifications for each player in a client-side list. Allow replay.

- Grouped notifications
  - Batch similar notifications into one item with a counter.
  - Example:
    ```lua
    local function batchAdd(player, key, newText)
      -- store in table on client, update existing id with count
      NotificationPack:update(key, { text = newText, title = "Messages (3)" })
    end
    ```

- Server-side logging
  - Emit events on server when you call SendNotification:FireClient to keep logs.
  - Store important notification events to game analytics or Datastore for audit.

Security and data flow
- The pack runs UI code on clients. Do not send secret server data in notification payloads.
- Use server to send short, non-sensitive payloads. If you require sensitive data, use secure server-side handling and only send tokens or sanitized messages.
- When implementing RemoteEvents, validate payload on the client before using it in UI code if any dynamic logic depends on it.

Testing checklist
- Test on PC, mobile, and console where possible.
- Test network latency scenarios by using NetworkEmulation in Studio.
- Test with multiple stacked notifications to ensure layout holds.
- Test with long titles and long body text; implement truncation or line wrap as needed.

Event hooks and lifecycle
- shown - fires when a notification completes its show animation.
- closed - fires when a notification completes closing.
- actionPressed - if the pack adds action buttons, this fires with action id.

Sample code for event hook
```lua
NotificationPack:on("shown", function(handle)
  print("Notification shown:", handle.id)
end)

NotificationPack:on("closed", function(handle)
  print("Closed:", handle.id)
end)
```

Extending the module
- Create custom styles by copying the Notification template and placing it under a Themes folder in the module model. The module loads Themes by name.
- Inject your own Button actions by adding action callbacks to the module's plugin script.

Common patterns
- Use id to ensure a single notification shows for a given topic.
  - Example: show only one "quest" notification at a time. Use id = "quest-123".
- Use prioritize by updating an existing notification instead of creating a new one.
- Keep messages short. Provide a link or panel for longer details.

Quality checklist for PRs
- Keep functions small and focused.
- Add tests to the example place that show the new behavior.
- Document public API changes in the Changelog.
- Tag performance impacts in PR description.

Contribution guide
- Fork the repository.
- Make a branch named feature/your-feature or fix/issue-number.
- Run tests in the included test place.
- Keep code style consistent with module. Use short functions and comments.
- Open a pull request with a clear description of behavior change.
- If you add a public function, add API docs and update changelog.

Development notes
- The module uses TweenService for animation and a small internal queue.
- UI is built with Frame, TextLabel, ImageLabel, and UIListLayout.
- Notifications position relative to a container anchored to screen corners.

Changelog (high level)
- v1.0.0 - Initial release. Module, examples, test place, and basic API.
- v1.1.0 - Added id-based updates and maximum visible config.
- v1.2.0 - Added theme overrides and accessibility tweaks.
- v1.3.0 - Performance improvements and pooling.

How updates land in your game
- Download the new release asset from the Releases page.
- Replace the NotificationPack module in your place with the new .rbxm content.
- Run the test place to confirm behavior.

Releases and downloads
- The releases page contains the packaged file notificationpack.rbxm. Download that file and execute it according to the Install section at the top of this README.
- The release page contains version notes, example builds, and the asset for insertion into Studio.

FAQ
Q: Can I use this on mobile?
A: Yes. The UI uses scaled units and anchors. Test on mobile to confirm spacing and touch interaction.

Q: Can players dismiss a notification manually?
A: Yes. The pack supports actions such as a dismiss button and callbacks. Set keepOnScreen to true to force a manual dismissal.

Q: How do I localize text?
A: Provide localized strings on the client and send keys or tokens from the server. The pack does not include localization, but it supports dynamic titles and text.

Q: Are there built-in action buttons?
A: The core includes a dismiss action. You can add extra action buttons by extending the template in the module model.

Q: Can I change the sound?
A: The pack does not ship a sound by default. Add a sound instance to the template or play a sound when the shown event fires.

Project structure (release asset contents)
- NotificationPack (Model or ModuleScript)
  - ModuleScript (Main)
  - Templates
    - NotificationTemplate (Frame)
  - Themes
    - default.theme
  - Examples
    - client_example.lua
    - server_example.lua
  - test_place.rbxlx (external, included in release)

Best practices when using NotificationPack
- Keep the title short. Use the body for details.
- Avoid sending too many notifications per minute.
- Use id to prevent duplicates.
- Use RemoteEvents to send only necessary data from server to client.
- Test in context with other UI to avoid overlap.

Sample in-depth use case: Quest system
- Server decides when a new quest starts.
- Server sends quest id and short description via RemoteEvent to client.
- Client calls NotificationPack:notify with id = "quest-" .. questId.
- Client shows a "View quest" action that opens the quest panel.

Quest example
Server side
```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local QuestNotify = Instance.new("RemoteEvent")
QuestNotify.Name = "QuestNotify"
QuestNotify.Parent = ReplicatedStorage

local function newQuest(player, questId, title, description)
  QuestNotify:FireClient(player, {
    id = "quest-" .. questId,
    title = title,
    text = description,
    duration = 8,
    icon = "rbxassetid://987654321"
  })
end
```

Client side
```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local NotificationPack = require(ReplicatedStorage:WaitForChild("NotificationPack"))
local QuestNotify = ReplicatedStorage:WaitForChild("QuestNotify")

QuestNotify.OnClientEvent:Connect(function(payload)
  NotificationPack:notify(payload)
end)
```

Design notes
- The UI uses safe area insets for mobile notches.
- The container calculates stacking based on maxVisible.
- The pack uses a simple state machine: queued -> showing -> closing -> free

Testing scenarios
- High frequency: simulate 50 notifications in 10 seconds and confirm queueing behavior.
- Long text: check wrap and clipping.
- Multiple players: check server-based broadcast to all players and per-player notifications.

Localization and internationalization
- The pack accepts any Unicode text for title and text.
- For languages with long words, test sizing and line breaks.
- For RTL languages, you may need to flip layout via configure options or theme.

Accessibility tips
- Keep font sizes large enough for readability.
- Add a spoken version by integrating with a TTS service on the client if allowed.
- Respect user preferences for animations: add a reduceMotion option in configure.

Roadmap
- Add built-in action buttons with labels and callbacks.
- Add more themes in the Themes folder.
- Add a lightweight owner-only debug console for designers.
- Add a built-in history panel showing recent notifications.

Contributing code style
- Use clear function names.
- Keep comments short and on point.
- Avoid big nested blocks; break into smaller helper functions.
- Add tests in test_place.

Where to get the release
- Download the release asset notificationpack.rbxm from the Releases page above and execute it in Roblox Studio.

Legal and licensing
- The pack includes a permissive license in the release. See LICENSE file in the release for details.

Credits
- Core design and module by Corpy-911 (pack author)
- Contributors listed in the repository contributors section
- Icons and images in examples come from open sources or Roblox assets by authors cited in the release package

Contact and support
- Open issues in the repository issues tab for bugs or feature requests.
- For help, provide a minimal reproduction place and steps to reproduce.

Developer notes and internals
- Main.lua handles queue and pooling.
- Templates use UIListLayout and AutomaticSize for adaptive layouts.
- The module exposes a simple event emitter for hooking into lifecycle events.

Examples summary (what you get in the release)
- Simple LocalScript example to show single notifications.
- Server example showing how to push notifications to players with RemoteEvent.
- Test place containing many scenarios: stacking, updates, and action callbacks.

Maintenance checklist
- Keep API stable across minor releases. Use semver for breaking changes.
- Document breaking changes in Changelog.
- Add tests to examples for every new feature.

Reference patterns
- Use unique ids for long-lived notifications.
- Use update instead of creating many notifications for the same topic.
- Keep icons to 64x64 or smaller for best performance.

End of file: For the packaged release asset, download notificationpack.rbxm from the Releases page and execute it inside Roblox Studio to install the module and see examples.

License
- See the LICENSE file in the release asset for full terms.

More releases and downloads
- Visit the Releases page above to get the packaged file, test place, and version notes.

Acknowledgements
- Roblox UI paradigms and default notification concept inspired the visual style.
- Contributors and testers who provided bug reports and feedback.