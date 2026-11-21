# FluxUI

A UI library for Roblox (Luau) inspired by the **macOS Settings** interface: rounded windows, a searchable sidebar, rows grouped into cards, subpages with breadcrumbs, sheet-style dialogs, stacked notifications, and config saving/loading.

> **Version:** 4.1 · **Language:** Luau · **File:** single file `FluxUI.lua`

---

## ⚠️ This is a "vibe coded" project

FluxUI was **vibe coded**: almost the entire codebase was written by **AI (Anthropic's Claude)** under the direction of the author (**vxr4**) through conversation. The author decided on the features and design, tested everything in Roblox, reported bugs, and requested fixes; the AI wrote and revised the code.

This means:

- The code **has not undergone a security audit** or in-depth review by other developers.
- The author **manually tested it in Roblox**: almost all features work as expected, but **it cannot be guaranteed to be completely bug-free**. The repository does not have an automated test suite yet.
- Bugs may remain, and the implementation might not follow "textbook" conventions. Please read the code before using it for anything critical.
- The software is provided **"as is", without warranty** (see [License](#license)).
- Pull requests and issues are always welcome. If you use AI to contribute, please **explicitly state so** in your PR description (see [Contributing](#contributing)).

## No Affiliation

FluxUI is **not affiliated with, sponsored by, or endorsed by** Apple, Roblox, or the authors of the projects mentioned in the [Credits](#credits) section. "macOS" and related names are trademarks of their respective owners; they are mentioned here solely as a source of design inspiration. This library does not use any Apple assets.

---

## Table of Contents

1. [Features](#features)
2. [Installation & Requirements](#installation--requirements)
3. [Quick Start](#quick-start)
4. [Core Concepts](#core-concepts)
5. [API: `FluxUI` Module](#api-fluxui-module)
6. [API: `CreateWindow` and Window](#api-createwindow-and-window)
7. [API: Tab, Section, Divider, Group](#api-tab-section-divider-group)
8. [API: Elements](#api-elements)
9. [Common API for All Elements](#common-api-for-all-elements)
10. [Page (Subpages)](#page-subpages)
11. [Dialog](#dialog)
12. [Profile Card](#profile-card)
13. [Notification](#notification)
14. [Configuration, Flags & Auto Save](#configuration-flags--auto-save)
15. [Theme](#theme)
16. [Callback Error Handling](#callback-error-handling)
17. [Window Architecture (Root / Window / Overlay)](#window-architecture-root--window--overlay)
18. [Full Example](#full-example)
19. [Troubleshooting](#troubleshooting)
20. [Known Limitations](#known-limitations)
21. [Contributing](#contributing)
22. [Credits](#credits)
23. [License](#license)

---

## Features

- macOS-style window: 3 traffic light buttons (close / minimize / maximize), draggable, **corner resizing**, bottom drag bar, and minimize chip.
- Sidebar: tabs, **tab sections (collapsible)**, **dividers**, a search box that automatically jumps to the tab/page with matching results, and a **profile card**.
- Elements: `Toggle`, `Slider`, `Button`, `Label`, `Dropdown` (single / multi-select), `Input`, `Keybind`.
- Adjacent rows are grouped into rounded **cards** with dividers; long descriptions **wrap automatically**.
- Infinitely nestable **Pages**, complete with a Back button and clickable breadcrumbs.
- Window-attached sheet **Dialogs** with a dialog queue.
- iOS-style stacked **Notifications**.
- Config saving/loading via **Flags**, Auto Load, **Auto Save & Load**, JSON Export / Import, and a built-in Config tab.
- Unified element API: `Destroy`, `Lock`, `SetVisible`, `SetFunction`, `CallFunction`, etc.
- **AMOLED Dark** and Light themes, switchable live at runtime.
- Your callbacks are wrapped in `xpcall`: errors are printed cleanly without breaking the library.

---

## Installation & Requirements

**Method 1: Load from GitHub (recommended to pin to a tag or commit):**

```lua
local FluxUI = loadstring(game:HttpGet("https://raw.githubusercontent.com/<user>/<repo>/<tag>/FluxUI.lua"))()
```

> Do not point to `main` in critical scripts: its contents can change at any time. Use a release tag (e.g., `v4.1.0`) or a commit hash.

**Method 2: ModuleScript** in Roblox Studio:

```lua
local FluxUI = require(path.to.FluxUI)
```

**Requirements**

| Component | Required? | Notes |
|---|---|---|
| Client execution environment (`LocalPlayer`) | Yes | The library creates GUIs on the client side |
| `gethui()` | No | Used as the GUI container if available; otherwise falls back to `PlayerGui` |
| `writefile` `readfile` `isfile` `isfolder` `makefolder` | No | Required for Save/Load config and Auto Save. If unavailable, these features will automatically report as unsupported |
| `listfiles` `delfile` | No | Listing / deleting configs |
| `setclipboard` | No | "Export to Clipboard" button |

**Icons:** The library uses a table of Lucide icons uploaded as Roblox assets in `FluxUI.Config.LucideIcons` (see [Icons](#icons)). If this table is empty, icons will not appear (text will still work normally).

---

## Quick Start

```lua
local FluxUI = loadstring(game:HttpGet("<raw url>"))()

local win = FluxUI:CreateWindow({
	Title = "My Hub",
	Folder = "MyHub",              -- configs are saved in FluxUI/MyHub/
	ToggleKey = Enum.KeyCode.RightShift,
})

win:AddTabSection("Main")
local tab = win:AddTab({ Title = "Combat", Icon = "swords" })

tab:AddSection("Aim")
tab:AddToggle({
	Title = "Auto aim",
	Description = "Aims at the closest target.",
	Icon = "crosshair",
	Default = false,
	Flag = "AutoAim",                         -- registers with the config save system
	Callback = function(enabled) print("auto aim:", enabled) end,
})

tab:AddSlider({ Title = "FOV", Min = 30, Max = 120, Default = 70, Flag = "FOV" })

win:AddConfigTab()   -- built-in config management tab
win:AutoLoad()       -- call at the VERY END, after all elements have been created
```

See [`examples/demo.lua`](examples/demo.lua) for a showcase of nearly every API.

---

## Core Concepts

```
Window
├─ Sidebar:  [Profile card] · Tab · TabSection · TabDivider · Search box
└─ Content:  Header (Back + breadcrumb)
              └─ Tab  ── Elements grouped into Cards
                  ├─ Section / Divider / Group (breaks card)
                  └─ Page (subpage) ── also contains Elements, Sections, and nested Pages
```

- **Card:** Consecutive elements are grouped into a single rounded card separated by thin divider lines. `AddSection`, `AddDivider`, and `AddGroup` break the current card; the next element starts a new card.
- **Flag:** A string identifier attached to an element so the library knows how to save and load its value (see [Configuration](#configuration-flags--auto-save)).
- **Search:** Type into the sidebar Search box to filter elements by `Title` / `Description`. The library automatically navigates to the first tab or page containing a match.
- **`:` vs `.`:** Methods are called with `:` (e.g., `toggle:SetState(true)`), while fields are accessed with `.` (e.g., `win.Flags.AutoAim`).

### Icons

Any `Icon` parameter accepts any of the following formats:

| Format | Example |
|---|---|
| Lucide icon name | `"settings"`, `"house"`, `"trash-2"` |
| `lucide:` prefix | `"lucide:user"` |
| Full asset ID | `"rbxassetid://123456"` |
| Number or numeric string | `123456`, `"123456"` |

Look up icon names at [lucide.dev/icons](https://lucide.dev/icons). Only icons present in `FluxUI.Config.LucideIcons` will render. You can add custom icons:

```lua
FluxUI.Config.LucideIcons["my-icon"] = "rbxassetid://123456"
```

(`"home"` is treated as an alias for `"house"`.)

---

## API: `FluxUI` Module

| Member | Description |
|---|---|
| `FluxUI:CreateWindow(options)` | Creates a window and returns a [Window](#api-createwindow-and-window) object |
| `FluxUI:Notify(options)` | Displays a [notification](#notification) and returns an object with `:Dismiss()` |
| `FluxUI:ClearNotifications()` | Dismisses all active notifications |
| `FluxUI.GetTheme()` | Returns the color table of the active theme |
| `FluxUI.Config` | Global configuration: `Assets`, `Shapes`, `Sheet`, `LucideIcons`, `Themes`, `CurrentTheme` |
| `FluxUI.Debug` | `boolean`. Set to `true` to include full tracebacks in callback error logs (default `false`) |
| `FluxUI.OnError` | Optional `function(context, message)` callback that receives all callback errors |
| `FluxUI.SafeCall(context, fn, ...)` | Calls `fn` wrapped in `xpcall`, returning `ok, result` |

---

## API: `CreateWindow` and Window

```lua
local win = FluxUI:CreateWindow(options)
```

### `CreateWindow` Options

| Option | Type | Default | Description |
|---|---|---|---|
| `Title` | string | `"FluxUI Window"` | Window title (topbar, minimize chip, and default config folder name) |
| `Size` | UDim2 | `620 × 400` offset | Initial window size |
| `MinimizeChip` | boolean | `true` | `false`: minimizing fades out and hides the window without showing a bottom chip |
| `ToggleKey` | KeyCode | `nil` | Keybind to toggle window visibility. Defaults to `RightShift` if `MinimizeChip = false` and unset |
| `ElementSpacing` | number | `6` | Vertical spacing between elements (in px) |
| `Folder` | string | window title | Config folder: `FluxUI/<Folder>/`. Subfolders are supported (`"MyHub/Configs"`). Invalid characters are stripped |
| `ConfigFolder` | string | `nil` | Overrides the full config path (ignores `Folder`) |
| `Profile` | table \| false | enabled | See [Profile Card](#profile-card). Set to `false` to disable |
| `Resizable` | boolean | `true` | Enables resizing from the bottom-right corner |
| `MinSize` | Vector2 | `480 × 320` | Minimum size when resizing |
| `MaxSize` | Vector2 | screen − 16 | Maximum size when resizing |

### Window Fields

| Field | Description |
|---|---|
| `win.Flags` | Lookup table mapping `flag name → element`, e.g., `win.Flags.AutoAim:GetState()` |
| `win.Root` | Unclipped root Frame (see [Window Architecture](#window-architecture-root--window--overlay)) |
| `win.Window` | CanvasGroup containing all window contents |
| `win.Overlay` | Transparent, unclipped overlay layer that tracks the window |

### Window State

| Method | Description |
|---|---|
| `win:Minimize()` / `win:Restore()` / `win:Toggle()` | Minimizes / restores / toggles window visibility |
| `win:IsMinimized()` | `boolean` |
| `win:ToggleMaximize()` | Centers and maximizes the window; call again to restore the previous size |
| `win:IsMaximized()` | `boolean` |
| `win:Close()` | Fades out and destroys the window |
| `win:Destroy()` | Destroys the window immediately |
| `win:SetTitle(text)` | Updates the window title |
| `win:SetTheme("Dark" \| "Light" \| custom theme name)` | Switches the theme immediately across the entire UI |
| `win:SetSize(UDim2 \| Vector2)` | Animates a resize (ignored while maximized) |
| `win:SetMinSize(Vector2)` | Sets the minimum resize dimensions |
| `win:SetResizable(boolean)` | Enables or disables resizing |
| `win:SetElementSpacing(px)` / `win:GetElementSpacing()` | Sets or gets vertical element spacing live |

### Topbar Buttons (3 Traffic Light Dots)

Button names: `"Close"`, `"Minimize"`, `"Maximize"` (case-insensitive; aliases include `exit`, `min`, `hide`, `max`, `zoom`, `fullscreen`).

| Method | Description |
|---|---|
| `win:ChangeTopbarButton(name, callback)` | Overrides button behavior. `callback(defaultFn)` receives the default handler; call `defaultFn()` to retain default behavior. Pass `nil` to restore |
| `win:ResetTopbarButton(name)` | Restores the original behavior |
| `win:DisableTopbarButton(nameOrList)` | Disables button(s) (turns grey). Accepts `"Minimize"`, `{ "Minimize", "Maximize" }`, or `"All"` |
| `win:EnableTopbarButton(nameOrList)` | Re-enables button(s) |

```lua
win:ChangeTopbarButton("Close", function(default)
	win:Confirm({ Title = "Close the hub?", OnConfirm = default })
end)
win:DisableTopbarButton("Maximize")
```

### Sidebar

| Method | Description |
|---|---|
| `win:AddTab({ Title, Icon })` | Creates a tab and returns a [Tab](#api-tab-section-divider-group) object |
| `win:AddTabSection(title \| options)` | Adds a tab group header. See below |
| `win:AddTabDivider()` | Adds a divider line between tab groups. Returns `{ SetVisible, IsVisible }` |
| `win:SelectTab(index)` | Selects a tab by creation order (1-indexed). Returns `true` if valid |
| `win:GetSelectedIndex()` | Returns the currently selected tab index |
| `win:AddConfigTab({ Title, Icon })` | Adds the built-in config management tab (default title `"Config"`, icon `"save"`) |

**`AddTabSection`**

```lua
local section = win:AddTabSection({ Title = "Advanced", Collapsible = true, Collapsed = false })
-- or shorthand: win:AddTabSection("Main")
```

| Option | Description |
|---|---|
| `Title` | Section header title |
| `Collapsible` | `true`: clicking the header collapses or expands the tabs in this section |
| `Collapsed` | Initial collapsed state (only applies when `Collapsible = true`) |

Tabs created **after** `AddTabSection` belong to that section until `AddTabDivider()` or another `AddTabSection()` is called. Selecting a tab inside a collapsed section automatically expands the section (including when jumped to via search).

Returned object methods: `SetCollapsed(bool)`, `Toggle()`, `IsCollapsed()`, `SetContent(text)`, `GetContent()`, `SetVisible(bool)`.

### Page Navigation

| Method | Description |
|---|---|
| `win:GoBack()` | Navigates back one page |
| `win:GoHome()` | Returns to the root page of the active tab |
| `win:GoToDepth(n)` | Navigates to breadcrumb depth `n` (`0` = root page of the tab) |
| `win:GetDepth()` | Returns the current page depth |

### Profile, Dialog, Config

See the [Profile Card](#profile-card), [Dialog](#dialog), and [Configuration](#configuration-flags--auto-save) sections.

---

## API: Tab, Section, Divider, Group

`Tab` (and [Page](#page-subpages)) expose the following element and layout methods:

| Method | Description |
|---|---|
| `tab:Select()` | Selects this tab (clicking an already-selected tab while inside a subpage returns to its root) |
| `tab:AddToggle / AddSlider / AddButton / AddLabel / AddDropdown / AddInput / AddKeybind(options)` | Creates an element and returns the element object |
| `tab:AddPage(options)` | Creates a subpage (see [Page](#page-subpages)) |
| `tab:AddSection(title \| { Title })` | Creates a section header in the content area and **breaks the current card**. Returns `{ SetContent, GetContent, SetState(visible), GetState }` |
| `tab:AddDivider()` | Breaks the current card and inserts a small vertical gap |
| `tab:AddGroup()` | Breaks the current card with no extra vertical gap |

---

## API: Elements

All elements accept `Title`, `Description` (wraps automatically if long), and `Icon` (vertically centered across the entire row). By default, callbacks fire **after the value has changed**. Any element can be assigned a `Flag`.

Every element provides a `Changed` signal with `:Connect(fn)`; `Keybind` also provides `Triggered`.

### Toggle

```lua
local toggle = tab:AddToggle({
	Title = "ESP", Description = "Draw boxes.", Icon = "eye",
	Default = false, Flag = "ESP",
	Callback = function(state) end,         -- state: boolean
})
```

| Method | Description |
|---|---|
| `SetState(boolean, animated?)` | Sets the toggle state (fires callback if the value changed) |
| `GetState()` | `boolean` |
| `SetContent(text)` / `GetContent()` | Sets or gets `Title` |

### Slider

```lua
local slider = tab:AddSlider({
	Title = "Speed", Description = "Walk speed.",
	Min = 0, Max = 100, Step = 1, Default = 16,
	LeftIcon = "volume-x", RightIcon = "volume-2",   -- optional icons on either end of the track
	Flag = "Speed",
	Callback = function(value) end,                  -- value: number
})
```

| Method | Description |
|---|---|
| `SetState(value, animated?)` | Sets the slider value (snapped to `Step` and clamped to `Min..Max`) |
| `GetState()` | `number` |
| `SetContent / GetContent` | Sets or gets `Title` |

Defaults: `Min = 0`, `Max = 100`, `Step = 1`, `Default = Min`.

### Button

```lua
local button = tab:AddButton({
	Title = "Reset", Description = "Back to defaults.", Icon = "rotate-ccw",
	Callback = function() end,
})
```

| Extra Option | Description |
|---|---|
| `Page` | Pass a Page object: turns the button into a page navigation row (with a `>` chevron) |

| Method | Description |
|---|---|
| `SetState(boolean)` | `false`: dims the button and disables clicking |
| `GetState()` | Whether the button is currently enabled |
| `SetContent / GetContent` | Sets or gets `Title` |

### Label

```lua
local label = tab:AddLabel({ Text = "Status", Description = "Everything is fine.", Icon = "info" })
```

| Method | Description |
|---|---|
| `SetContent(text)` / `GetContent()` | Sets or gets `Text` |
| `SetState(visible)` / `GetState()` | Equivalent to `SetVisible` / `IsVisible` (kept for compatibility) |

### Dropdown

```lua
local dropdown = tab:AddDropdown({
	Title = "Fruit", Description = "Pick one.", Icon = "apple",
	Options = {
		"Plain string",
		{ Title = "Banana", Value = "banana", Description = "yellow", Icon = "banana" },
	},
	Default = "banana",             -- Multi = true: Default = { "banana", ... }
	Multi = false,
	Placeholder = "Select",
	AllowNone = false,              -- (single-select) allows deselecting by clicking the active option again
	ShowSelectedIcon = false,       -- shows the selected option's icon to the left of the preview text
	ReserveIconSpace = false,       -- reserves an icon column on all rows if any option has an icon
	Width = 140, PopupWidth = 220, MaxHeight = 240,
	Flag = "Fruit",
	Callback = function(value) end, -- Multi: value is an array of selected Values
})
```

Each option can be a string or a table `{ Title | Text, Value, Description | Desc, Icon }`. `Value` defaults to `Title`.

| Method | Description |
|---|---|
| `SetState(value \| table)` | Sets the selection (pass a table of values when `Multi = true`) |
| `GetState()` | Selected value (or `nil`), or an array of values when `Multi = true` |
| `SetOptions(list, keepSelection?)` | Replaces the options list. Pass `keepSelection = false` to clear the current selection |
| `Open()` / `Close()` | Opens or closes the dropdown popup programmatically |
| `SetContent / GetContent` | Sets or gets `Title` |

The popup renders outside the window bounds so it is never clipped; click outside to dismiss. `Multi` mode keeps the popup open after each selection. When multiple items are selected and the preview string becomes too long, it displays `N selected`.

### Input

```lua
local input = tab:AddInput({
	Title = "Name", Description = "Press Enter.", Icon = "user",
	Placeholder = "Type here", Default = "",
	Numeric = false, Min = nil, Max = nil,
	MaxLength = nil, ClearOnFocus = false,
	Live = false,                                   -- true: callback runs on every keystroke
	Validate = function(text) return true end,      -- return false to reject input
	Width = 160, Flag = "Name",
	Callback = function(value, enterPressed) end,   -- Numeric: value is a number or nil
})
```

| Method | Description |
|---|---|
| `SetState(value)` | Sets the input value |
| `GetState()` | `string` (or `number \| nil` if `Numeric = true`) |
| `SetPlaceholder(text)` | Updates the placeholder text |
| `Focus()` | Focuses the text box |
| `SetContent / GetContent` | Sets or gets `Title` |

- By default, the callback fires when **pressing Enter or unfocusing** the box.
- `Numeric = true` restricts input to numbers (including `-` and `.`) and clamps to `Min` / `Max`.
- Values rejected by `Validate` revert to the previous value and flash a red border. If `Validate` itself throws an error, the library logs a warning and **accepts** the input.

### Keybind

```lua
local keybind = tab:AddKeybind({
	Title = "Magic key", Description = "Esc cancels, Backspace clears.", Icon = "keyboard",
	Default = Enum.KeyCode.E,       -- KeyCode, UserInputType.MouseButton2/3, or string like "E"
	Mode = "Toggle",                -- "Press" (default) | "Toggle" | "Hold"
	Width = 96, Flag = "MagicKey",
	Callback = function(state) end,       -- fires when the bound key is PRESSED
	OnChanged = function(key) end,        -- fires when the bound key is CHANGED
})
```

- Click the keybind button and press any key to bind it. Right-click (`MouseButton2`) and middle-click (`MouseButton3`) are also supported. Press `Esc` or click outside to cancel; press `Backspace` or `Delete` to clear the binding.
- Modes: `Press` calls `Callback()` on every press; `Toggle` calls `Callback(true / false)` alternating on each press; `Hold` calls `Callback(true)` on key down and `Callback(false)` on key up.
- Keybinds are ignored while typing in a text box, while waiting to bind a key, or while the element is locked.

| Method | Description |
|---|---|
| `SetState(key \| string)` | Sets the bound key (`nil` or `"None"` to clear) |
| `GetState()` | `EnumItem \| nil` |
| `GetKeyName()` | Display name, e.g., `"E"`, `"Mouse 2"`, `"None"` |
| `GetActive()` | Current active state (`Toggle` / `Hold`) |
| `SetMode(mode)` | Changes the trigger mode |
| `SetContent / GetContent` | Sets or gets `Title` |

Signals: `keybind.Changed:Connect(fn(key))`, `keybind.Triggered:Connect(fn(state))`.

---

## Common API for All Elements

Applies to Toggle, Slider, Button, Label, Dropdown, Input, and Keybind:

| Method | Description |
|---|---|
| `element:Destroy()` | Removes the row from the UI, unregisters it from search and Flags, and destroys any Dropdown popup. Empty cards are also removed |
| `element:IsDestroyed()` | Whether the element has been destroyed |
| `element:Lock(state?)` | Overlays a dim lock layer and blocks mouse/keyboard interaction. `Lock(false)` is equivalent to `Unlock()`. Programmatic `SetState` still works |
| `element:Unlock()` / `element:IsLocked()` | Unlocks / checks lock state |
| `element:SetVisible(bool)` / `element:IsVisible()` | Shows or hides the element (independent of search; surrounding cards resize automatically) |
| `element:SetFunction(fn)` | Replaces the primary callback (`nil` to clear) |
| `element:GetFunction()` | Returns the current primary callback |
| `element:CallFunction(...)` | Manually invokes the callback and returns its result (`nil` if the callback errors). If called with no arguments, passes the element's current state (Toggle → boolean, Slider → number, Dropdown → value, etc.) |
| `element.Flag` | The element's flag name, if assigned |

"Primary callback" refers to the element's `Callback`. For `Keybind`, this is the trigger callback (`Triggered`), whereas `OnChanged` is a separate callback for key rebinding.

```lua
local victim = tab:AddToggle({ Title = "Poor toggle", Default = true })
victim:Lock()                      -- lock it
victim:SetFunction(function(v) print("rewired:", v) end)
victim:CallFunction()              -- prints "rewired: true"
victim:Destroy()                   -- permanently remove
```

---

## Page (Subpages)

Pages are macOS Settings-style subpages: click a row to push a new page, complete with a Back button and clickable `Tab > Page > Subpage` breadcrumbs to **jump directly** to any ancestor level. Nesting depth is unlimited.

```lua
local page = tab:AddPage({
	Title = "Advanced",
	Description = "More settings.",
	Icon = "sliders-horizontal",
	Entry = true,                  -- default true: automatically creates a navigation row (with chevron) here
})

page:AddToggle({ Title = "Debug", Flag = "Debug" })
local deeper = page:AddPage({ Title = "Network", Icon = "wifi" })   -- nest further
```

| Member | Description |
|---|---|
| `page:Open()` | Opens the page programmatically (selects the parent tab and builds the breadcrumb trail) |
| `page:SetTitle(text)` | Renames the page (updates breadcrumbs) |
| `page.Entry` | The navigation button row (when `Entry ~= false`) |
| `page.Title` | Page title |
| All `AddToggle`, `AddSlider`, `AddSection`, `AddPage`... | Same as Tab |

- Set `Entry = false` if you want to open the page using a custom button (`AddButton({ Page = page })`) or `page:Open()`.
- Switching to another tab closes all open subpages. Clicking the currently active tab returns to its root page.
- **Search** traverses into subpages: a page's entry row stays visible if a nested element inside matches, and search automatically navigates into the page containing the match.

---

## Dialog

Dialogs are **window-attached sheets**: they dim only their parent window, follow the window when moved, hidden, or resized, and block interaction with the window (including the drag bar and resize handle) until dismissed.

```lua
local handle = win:Dialog({
	Title = "Save your changes?",
	Content = "You changed some stuff.",
	Icon = "save",                 -- optional
	IconColor = Color3.new(1, 0, 0),
	Accent = nil,                  -- Color3: tints the primary button with this color (default: high-contrast inverted button)
	Layout = nil,                  -- "Row" | "Stack" (default: chosen automatically based on button widths)
	Dismissible = false,           -- true: clicking outside triggers the Cancel button
	Buttons = {
		{ Title = "Cancel", Style = "Cancel" },
		{ Title = "Don't save", Style = "Destructive" },
		{ Title = "Save", Style = "Primary", Callback = function() end },
	},
})
handle:Close()
handle.Closed:Connect(function() end)
```

**Buttons** are listed in horizontal order from **left → right** (Cancel first, primary action last), right-aligned. When stacked vertically, the library automatically reverses the order so the primary action is at the top and Cancel is at the bottom.

| Button Field | Description |
|---|---|
| `Title` | Button label |
| `Style` | `"Default"` · `"Primary"` (high-contrast button, triggered by Enter) · `"Cancel"` (triggered by Esc) · `"Destructive"` (red text) |
| `Prominent` | `true`: solid fill (red fill if `Destructive`) and triggered by Enter |
| `Callback` | Runs on click (wrapped in `xpcall`) |
| `KeepOpen` | `true`: keeps the dialog open after clicking |

- Calling `win:Dialog` multiple times **queues** dialogs to display sequentially. Calling `handle:Close()` before a queued dialog is shown removes it from the queue.
- Default `Layout`: horizontal row if all buttons fit within the dialog width; otherwise stacked vertically.

### `win:Confirm(config)`

Shorthand confirmation dialog:

```lua
win:Confirm({
	Title = "Delete everything?", Content = "This cannot be undone.",
	ConfirmText = "Delete", CancelText = "Cancel",
	Destructive = true,            -- solid red confirm button
	OnConfirm = function() end,
	OnCancel = function() end,
})
```

---

## Profile Card

An avatar + name card displayed beneath the Search box in the sidebar.

```lua
Profile = {
	Enable = true,                 -- enable / disable
	DisplayName = "",              -- custom display name; nil or empty => uses player's DisplayName
	Username = "",                 -- custom username (muted line, "@" added automatically); nil or empty => uses player's Name
	Anonymous = false,             -- true => sets both to "Anonymous" and uses the Roblox account avatar
	Callback = function() end,     -- (optional) fires when the card is clicked
}
```

You can also pass `Profile = false` to quickly disable it. Update at runtime:

| Method | Description |
|---|---|
| `win:SetProfile(table)` | **Merges** new fields into the current profile config. Pass empty strings to revert to player data, or `Callback = false` to remove the callback |
| `win:SetAnonymous(bool)` / `win:IsAnonymous()` | Enables, disables, or checks anonymous mode |
| `win:SetProfileVisible(bool)` / `win:GetProfileVisible()` | Shows or hides the card (the tab list smoothly slides up to fill the space) |

---

## Notification

```lua
local note = FluxUI:Notify({
	Title = "Saved",
	Content = "Your config was saved.",
	Icon = "check",
	Duration = 3,                  -- seconds (default 3)
	Callback = function() end,     -- fires when the user clicks the notification
})
note:Dismiss()
```

- New notifications appear on top while older ones stack behind them (iOS style).
- **Hover** over the stack to expand it into a list; countdown timers pause while expanded. **Click** to dismiss (and trigger `Callback`).
- Up to 8 notifications can exist simultaneously; older ones are automatically pushed out when exceeded.
- Notifications are global and not tied to a specific window.

---

## Configuration, Flags & Auto Save

### Flags

Add `Flag = "Name"` to any element to include its value in the save system. Flag names should be unique (duplicate names will log a warning).

| Element | Saved Value |
|---|---|
| Toggle | `boolean` |
| Slider | `number` |
| Dropdown | selected value; array of values if `Multi`; `{ __none = true }` if nothing is selected |
| Input | `string` or `number` |
| Keybind | `"Enum.KeyCode.E"` / `"None"` |

Elements **without a Flag are not saved**. Values are serialized as JSON.

### Window Config API

| Method | Description |
|---|---|
| `win:GetFlag(name)` / `win:SetFlag(name, value)` | Reads or sets an element's value by flag name |
| `win:GetConfigData()` | Returns a `flag → value` table of current states |
| `win:ApplyConfigData(table)` | Applies a `flag → value` table; returns the number of flags loaded |
| `win:ExportConfig()` | Returns a JSON string of the current config (no filesystem required) |
| `win:ImportConfig(json)` | Loads from a JSON string; returns `true, count` or `false, error` |
| `win:SaveConfig(name)` | Saves to a file. Returns `true` or `false, error` |
| `win:LoadConfig(name)` | Loads from a file. Returns `true, count` or `false, error` |
| `win:DeleteConfig(name)` | Deletes a config file (and clears it from Auto Load if active) |
| `win:ListConfigs()` | Returns a list of saved config names |
| `win:GetConfigFolder()` | Returns the active config directory path |
| `win:SetAutoLoad(name \| nil)` | Sets the config to auto-load on startup (`nil` to clear) |
| `win:GetAutoLoad()` | Returns the current Auto Load config name |
| `win:ClearAutoLoad()` | Clears Auto Load |
| `win:AutoLoad()` | Runs auto-loading. **Call at the end of your script**, after all elements are created |
| `win:SetAutoSaveLoad(bool)` | Enables or disables [Auto Save & Load](#auto-save--load) |
| `win:GetAutoSaveLoad()` | Returns the saved state of Auto Save & Load |
| `win:IsAutoSaveActive()` | Whether Auto Save is currently running in this session |

Config names may only contain letters, numbers, `-`, `_`, and spaces. Names **starting with `_` are reserved** for internal system files.

### Generated Files

Inside `FluxUI/<Folder>/` (requires an executor with filesystem support):

| File | Contents |
|---|---|
| `<name>.json` | A user-saved configuration |
| `_autoload.txt` | Name of the auto-load config |
| `_autosave.json` | Auto-saved state (when Auto Save & Load is enabled) |
| `_settings.json` | Library settings (currently stores `AutoSaveAndLoad`) |

### Auto Save & Load

When enabled, **every change** to a flagged element is automatically saved to `_autosave.json` (debounced at 0.6 seconds), and subsequent runs of `win:AutoLoad()` will load this file instead of the standard Auto Load config.

- Enable via the toggle in the Config tab (under Advanced) or `win:SetAutoSaveLoad(true)`. This preference persists across sessions.
- Auto-saving **only begins after `win:AutoLoad()` runs** (or immediately when toggled on via UI/API), preventing initial default values during script startup from overwriting saved data before it can be loaded.
- While active, the Config tab's Save, Load, Delete, Refresh, Set as Auto Load, and Delete from Auto Load buttons—along with the name input and config list—are **locked**. Export / Import remain usable, and programmatic calls like `SaveConfig` still work.
- Destroying the window flushes any pending unsaved changes.

### Built-in Config Tab

`win:AddConfigTab()` creates a pre-built tab featuring: a config name input, saved configs list, Save / Load / Delete (with confirmation dialog) / Refresh buttons, Set as Auto Load / Delete from Auto Load buttons, Export to Clipboard / Import, and the Auto Save & Load toggle. If the environment does not support file writing, the tab automatically displays a notice and leaves only Export / Import available.

---

## Theme

```lua
win:SetTheme("Light")          -- "Dark" (AMOLED) or "Light"
FluxUI.Config.CurrentTheme = "Light"   -- set BEFORE creating a window to start with this theme
```

Creating custom themes: clone `FluxUI.Config.Themes.Dark`, modify the colors, and assign a new name. Make sure **all keys** listed below are present:

```lua
FluxUI.Config.Themes.Midnight = table.clone(FluxUI.Config.Themes.Dark)
FluxUI.Config.Themes.Midnight.On = Color3.fromRGB(10, 132, 255)
win:SetTheme("Midnight")
```

| Key | Used For |
|---|---|
| `WindowBackground` | Window background |
| `SidebarBackground` | Sidebar background |
| `WindowBorder` | Window / card / input borders |
| `TitleText` | Primary text |
| `BodyText` | Secondary text and descriptions |
| `On` | Active/accent color (toggles, sliders, highlights) |
| `Off` | Inactive color, scrollbars, subheaders |
| `Track` | Toggle switch track |
| `Thumb`, `ThumbGlass` | Toggle / slider knob |
| `ElementBackground` | Card background |
| `TabHover` | Hover background; dropdown / input / keybind value box background |
| `TabActive` | Selected tab; active/open value box |
| `Divider` | Divider lines |
| `DialogBackground` | Dialog sheet background |
| `DialogButton`, `DialogButtonHover` | Secondary dialog buttons |

---

## Callback Error Handling

All user callbacks are wrapped in `xpcall`: `Callback`, `OnChanged`, `Validate`, Dialog / Notification / Profile card / topbar button callbacks, `SetFunction`, and any listeners `:Connect`ed to element signals. Errors **will not break** other handlers or the library's execution flow; instead, they are logged to Output:

```
[FluxUI] Error in Toggle 'ESP' callback: Script:12: attempt to index nil with 'Position'
```

| Tool | Description |
|---|---|
| `FluxUI.Debug = true` | Includes full stack tracebacks in error messages |
| `FluxUI.OnError = function(context, message) end` | Global hook receiving all callback errors (for custom logging, alerts, etc.) |
| `FluxUI.SafeCall(context, fn, ...)` | Uses the same protection for your own functions; returns `ok, result` |

Exception for Input `Validate`: if `Validate` throws an error, the library logs a warning and **accepts** the value so a broken validator never blocks user input.

---

## Window Architecture (Root / Window / Overlay)

Because `CanvasGroup` clips everything outside its bounds by default, each window is structured into three layers:

```
screenGui
└─ WindowRoot   Standard Frame, UNCLIPPED    → holds Position / Size / Visible / UIScale
   ├─ Window    CanvasGroup, fills root      → clips contents to rounded corners, fades via GroupTransparency
   └─ Overlay   Transparent Frame over root  → holds anything extending OUTSIDE window bounds
```

The bottom drag bar and resize handle are parented to `Overlay`, so they automatically follow the window when moved, resized, or scaled without manual position updates. You can also parent drop shadows, badges, overflowing tooltips, or floating buttons to `win.Overlay`.

Conventions:

- To move, resize, or show/hide the window: manipulate `win.Root`.
- `GroupTransparency` and background color live on `win.Window`.
- `Overlay` covers the entire window without blocking input; only children with `Active = true` capture mouse events.

---

## Full Example

```lua
local FluxUI = loadstring(game:HttpGet("<raw url>"))()

local win = FluxUI:CreateWindow({
	Title = "Demo Hub", Folder = "DemoHub", ElementSpacing = 8,
	ToggleKey = Enum.KeyCode.RightShift,
	Profile = { Enable = true, Callback = function() print("profile clicked") end },
})

win:AddTabSection("Main")
local home = win:AddTab({ Title = "Home", Icon = "house" })

home:AddSection("Basics")
home:AddToggle({ Title = "Enabled", Flag = "Enabled", Default = true })
home:AddSlider({ Title = "Power", Min = 0, Max = 10, Default = 5, Flag = "Power" })
home:AddDropdown({
	Title = "Mode", Options = { "Easy", "Normal", "Hard" }, Default = "Normal", Flag = "Mode",
	Callback = function(mode) print("mode:", mode) end,
})
home:AddInput({ Title = "Name", Placeholder = "Your name", Flag = "Name" })
home:AddKeybind({ Title = "Panic key", Default = Enum.KeyCode.P, Flag = "Panic",
	Callback = function() win:Toggle() end })

local more = home:AddPage({ Title = "More options", Icon = "sliders-horizontal" })
more:AddToggle({ Title = "Verbose logs", Flag = "Verbose" })

win:AddTabDivider()
win:AddTabSection({ Title = "System", Collapsible = true })
win:AddConfigTab()

win:Dialog({
	Title = "Welcome",
	Content = "This UI was vibe coded. Be nice to it.",
	Buttons = { { Title = "OK", Style = "Primary" } },
})

win:AutoLoad()   -- always at the end
```

The full demo script is available at [`examples/demo.lua`](examples/demo.lua).

---

## Troubleshooting

| Symptom | Common Cause |
|---|---|
| Icons are missing | `FluxUI.Config.LucideIcons` is empty or missing that icon name. Unapproved, private, or moderated Roblox image assets also will not render |
| Custom `ImageLabel` does not show | The ID must be an **Image** asset ID, not a Decal ID; check Output for asset load errors |
| Config tab says "File saving is not available" | The execution environment lacks `writefile` / `readfile`... (e.g., Roblox Studio). Use Export / Import instead |
| `AutoLoad()` loads nothing | Called too early (before creating elements), no saved config/autosave exists yet, or elements are missing `Flag` |
| Values get overwritten with defaults after enabling Auto Save | `SetAutoSaveLoad(true)` was called **before** loading saved data. Let `win:AutoLoad()` run first |
| Element description text gets clipped | Handled automatically by measuring text height against actual width; if it still happens, open an issue with the element config |
| Callback appears "not to run" | Check Output: callback errors are printed as `[FluxUI] Error in ...`. Enable `FluxUI.Debug = true` for tracebacks |

---

## Known Limitations

- There is no automated test suite yet; testing is primarily manual (see the vibe-coded disclaimer at the top).
- No true background blur: the "glass" effect is achieved via semi-transparent panels.
- Description height calculation relies on `TextService:GetTextSize` (a legacy API that still works). If Roblox removes it, descriptions will fall back to a single line.
- Compatibility across different executors, screen DPI scales, and touch devices may vary; please open an issue if you encounter bugs.
- Notifications (`FluxUI:Notify`) are global rather than scoped to individual windows.
- Some names and layouts may change between versions; pin to a release tag in production.

---

## Contributing

- Open an **issue** before making major changes. Bug reports should include: execution environment, minimal reproduction snippet, and Output logs.
- Keep the library as a **single file** `FluxUI.lua` with no external dependencies.
- **If you use AI to write your contribution, please explicitly mention it in the PR description** (in the spirit of this project), and test your changes thoroughly before submitting.
- Do not commit assets or code that you do not have the rights to distribute.

---

## Credits

- [**WindUI**](https://github.com/Footagesus/WindUI) by **Footagesus** (MIT License): FluxUI took design inspiration and referenced the **Toggle and Slider "glass" effects** from this project.
- [**Footagesus/Icons**](https://github.com/Footagesus/Icons): Source of the Lucide icon list converted to Roblox assets, used for `FluxUI.Config.LucideIcons`.
- Original icon set: [**Lucide**](https://lucide.dev), ISC License.
- Design inspired by Apple's macOS Settings interface (no affiliation with Apple).
- Copyright notices and full license texts for third-party projects can be found in [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md).
- Written with AI assistance: [**Claude**](https://www.anthropic.com/claude) by Anthropic, directed by **vxr4**.

---

## License

MIT. See the `LICENSE` file. The software is provided "as is", without warranty of any kind; the author is not liable for any damages arising from its use.
