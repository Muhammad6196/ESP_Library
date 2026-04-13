# 👁️ ESP Library

A modular, feature-rich ESP (Extra Sensory Perception) library for Roblox. Supports player highlighting, item tracking, 2D/3D boxes, skeletons, chams, and fully customizable team-based coloring.

---

## Features

- **2D Box ESP** — Draws a box around players on screen
- **3D Box ESP** — Projects a full 3D bounding box around characters
- **Skeleton ESP** — Draws bone lines for R6 and R15 rigs
- **Name & Distance Labels** — Shows player name and distance in meters
- **Chams (Highlights)** — Always-on-top Roblox `Highlight` instances on players and items
- **Item ESP** — Tracks world objects of your choice
- **Outline Support** — Configurable outline color and thickness on all drawings
- **Unlimited Teams** — Add as many team color groups as you want
- **Max Distance Culling** — Hide ESP beyond a set range
- **Fully Destroyable** — Clean teardown of all drawings and connections

---

## Quick Start

```lua
-- 1. Load the script
-- 2. Define your logic (see below)
-- 3. Call ESP:Init()
```

---

## Settings Reference

All settings live in `ESP.Settings`.

| Setting | Type | Default | Description |
|---|---|---|---|
| `Enabled` | bool | `true` | Master toggle for all ESP |
| `Box2D` | bool | `true` | Show 2D screen-space box |
| `Box3D` | bool | `false` | Show 3D world-space box |
| `Name` | bool | `true` | Show player/item name label |
| `Distance` | bool | `true` | Show distance in meters |
| `Skeleton` | bool | `false` | Show skeleton bone lines |
| `MaxDistance` | number | `math.huge` | Max render distance (studs) |
| `ItemESP` | bool | `true` | Enable item tracking |
| `Chams` | bool | `true` | Enable player chams (Highlight) |
| `ItemChams` | bool | `true` | Enable item chams |
| `ChamFillTransparency` | number | `1` | Fill transparency (0 = opaque) |
| `ChamOutlineTransparency` | number | `0` | Outline transparency |
| `NameSize` | number | `24` | Font size for name labels |
| `DistanceSize` | number | `24` | Font size for distance labels |
| `Outline` | bool | `true` | Draw outlines on text/boxes |
| `OutlineColor` | Color3 | Black | Color of outlines |
| `OutlineThickness` | number | `4` | Thickness of box outlines |

---

## Colors

```lua
ESP.Settings.Colors = {
    Default = Color3.fromRGB(255, 255, 255),
    -- Add more keys as needed, matched by the string you return from ItemLogic
}
```

Colors are keyed by type string. Item ESP uses the string returned by `ESP.ItemLogic` to look up a color from this table, falling back to `Default` if no match is found.

---

## Teams

Teams define per-group colors. `ESP.Logic` returns a team key string and that team's color is applied to all drawings and chams for that player.

### Default Teams

```lua
ESP.Settings.Teams = {
    Team = {
        Color = Color3.fromRGB(0, 150, 255),
        State = "Team"
    },
    Enemy = {
        Color = Color3.fromRGB(255, 0, 0),
        State = "Enemy"
    }
}
```

### Adding Custom Teams

You can add unlimited teams at any time:

```lua
ESP.Settings.Teams.MyTeam = {
    Color = Color3.fromRGB(255, 215, 0),
    State = "MyTeam"
}
```

Then return the matching key from your `ESP.Logic` function and the library will automatically apply the correct color.

> If `ESP.Logic` returns `false` or `nil`, that player is fully hidden — no box, label, or cham.

---

## User-Defined Logic

These two callbacks are **entirely up to you** to implement based on your game. The library calls them automatically every frame.

### ESP.Logic

Controls whether a player is shown and which team color to use.

```lua
ESP.Logic = function(player)
    -- Return a team key string to show the player with that team's color
    -- Return false/nil to hide the player entirely
end
```

### ESP.ItemLogic

Controls whether an item is shown, what label to display, and what color to use.

```lua
ESP.ItemLogic = function(item)
    -- Return false            → skip this item
    -- Return "Label"          → show with Default color
    -- Return "Label", Color3  → show with a specific color
end
```

### ESP:WatchItems(func)

Pass a function that returns a table of instances to track. The library diffs the list every frame and automatically creates or removes item ESP entries.

```lua
ESP:WatchItems(function()
    -- Return a table of instances you want tracked
    return { ... }
end)
```

---

## API Reference

| Method | Description |
|---|---|
| `ESP:Init()` | Start player tracking (auto-handles join/leave) |
| `ESP:Create(player)` | Manually create ESP for a player |
| `ESP:Remove(player)` | Remove all drawings and chams for a player |
| `ESP:CreateCham(player, char)` | Attach a Highlight to a character |
| `ESP:AddItem(item)` | Register an item through `ItemLogic` |
| `ESP:RemoveItem(item)` | Remove drawings and cham for an item |
| `ESP:WatchItems(func)` | Provide a function returning items to track |
| `ESP:Destroy()` | Full cleanup of all connections, drawings, and chams |

---

## Notes

- Chams use Roblox's `Highlight` parented to `CoreGui`, rendering through walls via `AlwaysOnTop`.
- The library self-deduplicates via `_G.ESP_LOADED` — reloading the script destroys the previous instance automatically.
- Skeleton rendering supports both **R6** and **R15** rigs automatically.
- `ESP.Logic` and `ESP.ItemLogic` can be reassigned at any time and take effect on the next frame.
