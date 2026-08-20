# Snowy Studios UI

A UI library for Roblox executor scripts. Built for real hubs: many tabs, hundreds
of controls, mobile and desktop, without costing frames.

Two files — `SnowyStudios.luau` (the library) and `example.luau` (a full working
reference you can run as-is).

```lua
local Snowy = loadstring(readfile("SnowyStudios.luau"))()
```

Hosting it remotely works the same way with `loadstring(game:HttpGet(url))()`.

**Contents** — [Quick start](#quick-start) · [Window](#window) ·
[Tabs and sections](#tabs-and-sections) · [Controls](#controls) ·
[Values and flags](#values-and-flags) · [Notifications](#notifications) ·
[Theming](#theming) · [Recipes](#recipes) · [How it behaves](#how-it-behaves) ·
[Gotchas](#gotchas)

---

## Quick start

A complete, runnable script:

```lua
local Snowy = loadstring(readfile("SnowyStudios.luau"))()

local win = Snowy:Window({
    Subtitle = "Murder Mystery 2",
    Logo = "rbxassetid://123802801726537",
})

local tab = win:Tab({
    Name = "Farming",
    Icon = "zap",
    Description = "Automatic collection and rejoin",
})

local box = tab:Section({ Title = "Collection", Column = 1 })

box:Toggle({
    Text = "Auto collect",
    Info = "Picks up every dropped coin in range",
    Default = true,
    Flag = "auto_collect",
    Callback = function(on)
        print("auto collect:", on)
    end,
})

box:Slider({
    Text = "Range",
    Min = 10, Max = 500, Default = 120,
    Suffix = " studs",
    Flag = "collect_range",
})

win:Notify({ Type = "success", Title = "Loaded", Text = "Press RightShift to hide." })
```

---

## Window

```lua
local win = Snowy:Window({
    Subtitle     = "Murder Mystery 2",       -- game name, shown as a chip
    Title        = "Snowy Studios",          -- only used when no Logo is set
    Logo         = "rbxassetid://123802801726537",
    IconPack     = "phosphor",               -- "phosphor" | "lucide" | "tabler"
    Keybind      = Enum.KeyCode.RightShift,
    Width        = 940,
    Height       = 648,
    MinHeight    = 382,
    MaxHeight    = 800,
    MobileButton = true,                     -- top-centre MENU pill
    NotifySound  = "rbxasset://sounds/electronicpingshort.wav",
    NotifyVolume = 0.45,
})
```

| Method | Purpose |
| --- | --- |
| `win:Tab(opts)` | Add a tab, returns the tab |
| `win:Select(index)` | Switch to a tab |
| `win:Notify(opts)` | Toast in the bottom-right, returns a handle |
| `win:Toggle(force)` | Show/hide; omit `force` to flip |
| `win:Search(query)` | Filter every control by text |
| `win:SetKeybind(key)` | Change the show/hide key at runtime |
| `win:Destroy()` | Unload and disconnect everything |

### The MENU pill

A rounded pill sits top-centre on every device showing your logo, live **ping**
and live **FPS**. Clicking it shows or hides the window, so there is always a
click target alongside the keybind.

Both readouts colour-code themselves — ping green ≤90ms, amber ≤160ms, red above;
FPS green ≥50, amber ≥30, red below. Frame deltas are averaged over each half
second rather than sampled per frame, so the number does not flicker.

Set `MobileButton = false` to hide it. A device with no keyboard keeps it
regardless — otherwise closing the menu would strand the user with no way back.

### Logos

Logos are usually artwork floating inside a larger square canvas. The library
measures the real bounding box and crops to it, so the mark fills its slot instead
of being letterboxed down to mush.

```lua
Logo = "rbxassetid://123802801726537"          -- measured once, then cached
```

Skip the measurement entirely if you already know the numbers:

```lua
Logo           = "rbxassetid://123802801726537",
LogoRectOffset = Vector2.new(40, 256),
LogoRectSize   = Vector2.new(945, 457),
LogoHeight     = 42,   -- width derives from the measured aspect ratio
```

---

## Tabs and sections

```lua
local tab = win:Tab({
    Name = "Aim Assist",
    Icon = "focus",
    Description = "Targeting, smoothing and hit filters",
})

local left  = tab:Section({ Title = "Aim",     Column = 1 })
local right = tab:Section({ Title = "Filters", Column = 2 })
```

`Description` is worth filling in. Each page draws a header with the tab's own
icon, its name, and that line underneath, so a user landing on a tab can tell what
it controls without reading every row. The header animates on switch — icon
springs in, name and description fade up — so changing tabs reads as a change of
context rather than a word quietly swapping.

`Column = 1` is the left stack, `Column = 2` the right. Narrow screens collapse
both into one column automatically. Sections stack in creation order; pass `Order`
to override.

### Icon names

These are mapped across all three packs, so switching `IconPack` keeps working:

`focus` `binoculars` `globe` `settings` `code-xml` `zap` `shield` `sword`
`palette` `user` `bell` `info` `search` `hexagon` `layout-grid` `check`
`check-circle` `warning` `wifi` `gauge` `ellipsis` `chevron-down` `minus` `x`

Any other name is passed straight through to the active pack, so
`Icon = "skull-bold"` works on Phosphor — but it will not resolve if you later
switch packs.

---

## Controls

Every control accepts `Text`, plus optional `Info` (tooltip on the ⓘ badge),
`Flag` (writes into `Snowy.Flags`), and `Callback`.

### Toggle

```lua
box:Toggle({
    Text = "Enable aimbot",
    Info = "Master switch",
    Default = true,
    Disabled = false,      -- greys it out and blocks input
    Flag = "aim_enabled",
    Callback = function(on) end,
})
```

### Slider

```lua
box:Slider({
    Text = "Field of view",
    Min = 0, Max = 180, Default = 165,
    Decimals = 0,          -- 1 or 2 for fractional values
    Suffix = "°",
    Flag = "aim_fov",
    Callback = function(value) end,
})
```

### Dropdown

```lua
box:Dropdown({
    Text = "Sort mode",
    Options = { "Closest", "Lowest health", "Highest threat" },
    Default = "Closest",
    Flag = "sort_mode",
    Callback = function(choice) end,
})
```

Set `Multi = true` for multi-select. `Default` becomes a list, the callback
receives a list, and the menu stays open so several picks are one gesture:

```lua
box:Dropdown({
    Text = "Hitbox parts",
    Multi = true,
    Options = { "Head", "Neck", "UpperTorso", "LeftArm", "RightArm" },
    Default = { "Head", "Neck" },
    Placeholder = "No parts",     -- shown when nothing is selected
    Flag = "hitbox_parts",
    Callback = function(parts)
        for _, part in ipairs(parts) do print(part) end
    end,
})
```

The button summarises as `Head, Neck +2`, or `All` when everything is picked.

### Keybind

**Left click** rebinds (Backspace clears). **Right click** opens the mode picker.

```lua
box:Keybind({
    Text = "Aim key",
    Default = Enum.KeyCode.C,
    Mode = "Hold",         -- "Toggle" | "Hold" | "Always"
    Flag = "aim_key",
    Callback = function(state, mode, key) end,
    OnChanged = function(key, mode) end,
})
```

| Mode | Behaviour |
| --- | --- |
| `Toggle` | Pressing the key flips state on/off |
| `Hold` | State is true only while the key is held |
| `Always` | State stays true, key ignored |

`Callback` fires when the bound key is **used**. `OnChanged` fires when the
binding or mode is **changed** — that is the one you want for rebinding the menu:

```lua
box:Keybind({
    Text = "Menu keybind",
    Default = Enum.KeyCode.RightShift,
    OnChanged = function(key) win:SetKeybind(key) end,
})
```

### Button, Input, Label, Divider

```lua
box:Button({
    Text = "Rejoin server",
    ButtonText = "Run",
    Callback = function() end,
})

box:Input({
    Text = "Config name",
    Placeholder = "default",
    Default = "default",
    Flag = "config_name",
    Callback = function(str) end,   -- fires on Enter only
})

box:Label("Local only. Never replicates to the server.")
box:Divider()
```

### Curve

Draggable cubic bezier for easing and smoothing settings. Four handles: two
anchors and two control points.

```lua
box:Curve({
    Text = "Smoothing curve",
    Default = { {0.05, 0.86}, {0.42, 0.86}, {0.58, 0.14}, {0.95, 0.14} },
    Callback = function(points)
        -- points.P0, points.C1, points.C2, points.P3 are Vector2
    end,
})
```

---

## Values and flags

Every control returns a handle:

```lua
local toggle = box:Toggle({ Text = "Enabled", Default = true })

toggle:Get()        -- current value
toggle:Set(false)   -- update it
toggle.Instance     -- the row itself
```

| Control | `Get()` returns | `Set()` fires `Callback`? |
| --- | --- | --- |
| Toggle | boolean | yes |
| Slider | number | yes |
| Dropdown | choice, or a list when `Multi` | yes |
| Keybind | `key, mode, state` | no |
| Input | string | no |
| Curve | the raw node table | — |
| Button / Label / Divider | — | — |

Or read everything flat through flags:

```lua
box:Slider({ Text = "Speed", Flag = "walk_speed", Min = 16, Max = 100, Default = 16 })

print(Snowy.Flags.walk_speed)      --> 16
```

| Control | Flag value |
| --- | --- |
| Toggle | `true` / `false` |
| Slider | number |
| Dropdown | the choice, or an array when `Multi` |
| Keybind | `{ Key = ..., Mode = "Hold", State = false }` |
| Input | string |

---

## Notifications

```lua
win:Notify({
    Type = "success",              -- info | success | warning | error
    Title = "Configuration saved",
    Text = "Stored default with all current values.",
    Duration = 4,
})
```

`Type` picks icon, colour and sound pitch together:

| Type | Icon | Colour | Pitch |
| --- | --- | --- | --- |
| `info` | info | accent blue | 1.00 |
| `success` | check circle | green | 1.16 |
| `warning` | warning triangle | amber | 0.92 |
| `error` | cross | red | 0.80 |

Optional overrides:

```lua
win:Notify({
    Type = "info",
    Title = "Custom",
    Text = "...",
    Icon = "zap",                  -- any icon name
    Color = Snowy.Theme.Accent,    -- overrides the type colour
    Sound = false,                 -- mute just this one
    Duration = 6,
})
```

Hovering pauses the countdown so a long message can actually be read. The × or
the returned handle dismisses early:

```lua
local toast = win:Notify({ Title = "Working...", Duration = 30 })
toast:Close()
```

Sound uses `rbxasset://sounds/electronicpingshort.wav`, a file that ships with the
Roblox client — unlike a catalog asset it cannot be moderated, deleted, or fail to
download. Override per window with `NotifySound` and `NotifyVolume`.

---

## Theming

```lua
Snowy.Theme.Accent = Color3.fromRGB(255, 90, 140)   -- before creating the window

Snowy.Theme   -- Accent AccentDim AccentGlow Good Warn Bad
              -- Text Sub Faint Line Shell Rail Surface Elevated Hover
Snowy.Fonts   -- Regular Medium SemiBold Bold (BuilderSans FontFaces)
```

Mutate `Snowy.Theme` **before** `Snowy:Window()`. Colours are read at build time,
so changing them afterwards will not repaint existing controls.

---

## Recipes

**Save and load a config.** Flags are a plain table, so serialise them directly:

```lua
local HttpService = game:GetService("HttpService")

local function save(name)
    writefile("snowy-" .. name .. ".json", HttpService:JSONEncode(Snowy.Flags))
end

local function load(name)
    local path = "snowy-" .. name .. ".json"
    if not isfile(path) then return end
    for key, value in pairs(HttpService:JSONDecode(readfile(path))) do
        Snowy.Flags[key] = value
    end
end
```

Flags hold raw values, not controls — call `:Set()` on the handles you kept if you
want the UI to visually reflect a loaded config.

**Drive a loop from a toggle.** Read the flag inside the loop rather than starting
and stopping threads:

```lua
task.spawn(function()
    while task.wait(0.1) do
        if Snowy.Flags.auto_collect then
            -- collect
        end
    end
end)
```

**Hold-to-aim.** `Hold` mode gives you press and release for free:

```lua
box:Keybind({
    Text = "Aim key",
    Default = Enum.KeyCode.C,
    Mode = "Hold",
    Callback = function(active)
        aiming = active
    end,
})
```

**Clean unload.** Wire a button to `win:Destroy()`; it disconnects every input
connection and removes the ScreenGui.

---

## How it behaves

**Responsive.** The window fits itself to the viewport, drops to a single column
below 820px wide, and scales down (to a 0.55 floor) on small screens.

**Height.** The window measures the tallest column of the active tab and animates
to fit, clamped between `MinHeight` and `MaxHeight`. Sparse tabs leave no void;
dense tabs scroll.

**Icons** are fetched from Iconify at 128px, cached to disk, and resolved in the
background — window creation never blocks on HTTP. Anything already on screen
swaps from its glyph fallback the moment the real icon lands.

**Reload safety.** Re-running the script disposes the previous instance through
`getgenv().SnowyStudiosUnload`, so connections never stack across executions.

**Performance.** Measured at 14 tabs / 448 controls / 5,560 instances:

| | |
| --- | --- |
| Build time | 0.05s |
| FPS under load | 179 (idle: 179) |
| Frame time | 5.58ms (idle: 5.56ms) |
| Full-text search | 0.8ms |

Sections are plain frames rather than CanvasGroups — each CanvasGroup is its own
render target, and a large hub would pay for dozens every frame. Only pages and
the window root use them. Search and height-fitting are debounced, and every
draggable shares one input connection instead of registering its own.

---

## Gotchas

**Flags are global to the library, not per window.** Two windows in the same
session share `Snowy.Flags`. Prefix your flag names if that matters.

**`Input:Set()` and `Keybind:Set()` do not fire callbacks**, by design — they are
for restoring state. Toggle, Slider and Dropdown do fire.

**Theme changes after `Snowy:Window()` will not apply** to controls already built.

**`Callback` on a Keybind is not a rebind hook.** It fires when the key is
pressed. Use `OnChanged` to react to the binding itself changing.

**Icon names outside the mapped list are pack-specific.** They resolve against
whichever `IconPack` is active and will break if you switch packs.
