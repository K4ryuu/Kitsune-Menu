# KitsuneMenu 2.0

> [!WARNING] > **Experimental Project Notice**
>
> This was an experimental project created for learning and exploration purposes. Developers are free to use this code in their own projects or extract useful components from it. However, I do not plan to actively develop or maintain this project further.

A modern, fluent menu system for Counter-Strike 2 built on CounterStrikeSharp.

## Features

- **Fluent Builder Pattern** - Create menus with clean, chainable syntax
- **Typed Menu Items** - Strongly typed buttons, toggles, sliders, choices, and more
- **Event System** - Comprehensive lifecycle events for menu interactions
- **Async Support** - Built-in support for async operations with loading states
- **Conditional Items** - Show/hide and enable/disable items based on conditions
- **Menu Templates** - Pre-built templates for common patterns (confirmations, color pickers, etc.)
- **Reactive Updates** - Dynamic content that updates automatically
- **State Management** - Session-based state persistence across menu navigation

## Quick Start

```csharp
// Initialize in your plugin
public override void Load(bool hotReload)
{
    KitsuneMenu.Init();
}

// Don't forget to cleanup
public override void Unload(bool hotReload)
{
    KitsuneMenu.Cleanup();
}

// Create a simple menu
var menu = KitsuneMenu.Create("Main Menu")
    .AddButton("Teleport", player => {
        // teleport logic
    })
    .AddToggle("God Mode", false, (player, enabled) => {
        player.GodMode = enabled;
    })
    .AddSlider("Speed", 1, 10, 5, (player, value) => {
        player.Speed = value;
    })
    .MaxVisibleItems(7)  // Limit visible items
    .Build();

menu.Show(player);
```

## Menu Items

### Button

```csharp
.AddButton("Click Me", player => {
    player.PrintToChat("Button clicked!");
})
.CloseOnSelect() // Close menu after button press

// With button overrides
.AddButton("Select Option", player => { /* logic */ })
.OverrideSelectButton("Jump") // Use Jump key instead of default
```

### Toggle

```csharp
.AddToggle("Enable Feature", defaultValue: false, (player, enabled) => {
    // Handle toggle change
})
```

### Slider

```csharp
.AddSlider("Volume", min: 0, max: 100, defaultValue: 50, (player, value) => {
    // Handle value change
})
```

### Choice

```csharp
.AddChoice("Team", new[] { "CT", "T", "Spectator" }, (player, choice) => {
    player.ChangeTeam(choice);
})
```

### Submenu

```csharp
.AddSubmenu("Settings", settingsMenu)
// Or with lazy loading
.AddSubmenu("Settings", () => BuildSettingsMenu())
```

### Text & Separators

```csharp
.AddText("Information", TextAlign.Center)
.AddText("Small text", TextAlign.Left, MenuTextSize.Small)
.AddSeparator()
.AddDynamicText(() => $"Players: {GetPlayerCount()}", TimeSpan.FromSeconds(1))
```

## Advanced Features

### Conditional Items

```csharp
.AddButton("Admin Panel", ShowAdminPanel)
    .VisibleWhen(player => player.IsAdmin)
    .EnabledWhen(player => !player.IsBanned)
```

### Async Operations

```csharp
.AddAsyncButton("Load Stats", async player => {
    var stats = await DatabaseService.GetPlayerStats(player.SteamId);
    ShowStatsMenu(player, stats);
})
```

### Events

```csharp
menu.OnOpen += player => Console.WriteLine($"{player.PlayerName} opened menu");
menu.OnItemSelected += (player, item) => LogSelection(player, item);
menu.OnClose += player => SavePlayerState(player);
```

### Menu Templates

```csharp
// Confirmation dialog
var confirm = MenuTemplates.Confirm(
    "Delete Account",
    "Are you sure?",
    onConfirm: player => DeleteAccount(player),
    onCancel: player => ShowMainMenu(player)
);

// Color picker
var colorPicker = MenuTemplates.ColorPicker(
    "Choose Color",
    (player, color) => player.SetColor(color)
);

// Number picker
var numberPicker = MenuTemplates.NumberPicker(
    "Set Health",
    min: 1, max: 100, defaultValue: 100,
    (player, value) => player.Health = value
);

// Number picker with custom steps
var customPicker = MenuTemplates.NumberPicker(
    "Set Money",
    min: 0, max: 16000, defaultValue: 800,
    (player, value) => player.SetMoney(value),
    steps: new[] { 100, 500, 1000, 5000 }
);
```

### Progress Bars

```csharp
.AddProgressBar("Download Progress", () => downloadProgress)
```

### Session State

```csharp
// Store data between menu navigations
menu.AddButton("Set Spawn", (player, session) => {
    session.Set("spawn_position", player.Position);
});

menu.AddButton("Teleport to Spawn", (player, session) => {
    if (session.TryGet<Vector3>("spawn_position", out var pos)) {
        player.Teleport(pos);
    }
});
```

## Configuration

The menu system uses a JSON configuration file (`kitsune_menu_config.jsonc`) for button mappings:

```json
{
  "Select": "Jump",
  "Back": "Speed",
  "Up": "Forward",
  "Down": "Back",
  "Left": "Moveleft",
  "Right": "Moveright",
  "Exit": "Scoreboard"
}
```

## Menu Management

### Closing Menus

```csharp
// Close menu for specific player
KitsuneMenu.CloseMenu(player);

// Close all menus with specific title
KitsuneMenu.CloseMenusByTitle("My Menu", exactMatch: true);

// Close all menus with title containing text
KitsuneMenu.CloseMenusByTitle("Settings", exactMatch: false);
```

### Menu Configuration

```csharp
.MaxVisibleItems(7)  // Limit visible menu items
.OverrideSelectButton("Jump")  // Override select button
```

## Real-World Example

```csharp
public void ShowBombWireMenu(CCSPlayerController player, bool isPlanting)
{
    var colors = new[] { "Red", "Blue", "Yellow", "Green" };

    var menu = KitsuneMenu.Create("Bomb Wires")
        .AddText("Choose wire to cut:", TextAlign.Left, MenuTextSize.Small)
        .MaxVisibleItems(6)
        .OverrideSelectButton("Jump");

    foreach (var color in colors.OrderBy(_ => Random.Shared.Next()))
    {
        menu.AddButton($"<font color=\"#{GetColorHex(color)}\">{color} Wire</font>", player =>
        {
            HandleWireSelection(player, color, isPlanting);
        })
        .CloseOnSelect();
    }

    var builtMenu = menu.Build();
    builtMenu.OnClose += p => HandleMenuClose(p, isPlanting);
    builtMenu.Show(player);
}
```
