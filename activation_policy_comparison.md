# macOS App Activation Policy Comparison

## .regular Policy (Normal App)

```
Top Menu Bar: [🍎] [MyApp] [File] [Edit] [Window] [Help]    [🔋][📶][🔊][⏰]
                    ^^^^^^ App name shows here

Dock: [Finder][Safari][MyApp][TextEdit][Trash]
                      ^^^^^^ App shows in dock

Cmd+Tab Switcher: [Finder] [Safari] [MyApp] [TextEdit]
                                     ^^^^^^ Shows in app switcher
```

**Behavior:**
- App can become "active" (frontmost)
- All windows belong to this app
- Full menu bar with File, Edit, etc.
- Can be quit with Cmd+Q

## .accessory Policy (Background App)

```
Top Menu Bar: [🍎] [Finder] [File] [Edit] [Window]          [⭐][🔋][📶][🔊][⏰]
                                                             ^^ Only status icon

Dock: [Finder][Safari][TextEdit][Trash]
      ^^^^^^ App is completely hidden from dock

Cmd+Tab Switcher: [Finder] [Safari] [TextEdit] 
                  ^^^^^^ App is hidden from switcher
```

**Behavior:**
- App runs invisibly in background
- Only status bar icon visible
- Windows are temporary/floating
- Usually quit from status bar menu

## Dynamic Switching Example

Many menu bar apps switch policies dynamically:

```swift
// Start as background app
NSApp.setActivationPolicy(.accessory)

// When user opens preferences window
func openPreferences() {
    NSApp.setActivationPolicy(.regular)      // Show in dock temporarily
    preferencesWindow.makeKeyAndOrderFront(nil)
    NSApp.activate(ignoringOtherApps: true)  // Bring to front
}

// When preferences window closes
func windowDidClose() {
    NSApp.setActivationPolicy(.accessory)    // Hide from dock again
}
```

## Common Patterns

### Pattern 1: Pure Background App
```swift
// Apps like system monitors, clipboard managers
NSApp.setActivationPolicy(.accessory)  // Never show in dock
```

### Pattern 2: Hybrid App  
```swift
// Start hidden, show when needed
NSApp.setActivationPolicy(.accessory)

// Show in dock when main window opens
func showMainWindow() {
    NSApp.setActivationPolicy(.regular)
    mainWindow.makeKeyAndOrderFront(nil)
}

// Hide when main window closes
func windowDidClose() {
    NSApp.setActivationPolicy(.accessory)
}
```

### Pattern 3: Normal App with Status Bar
```swift
// Regular app that ALSO has a status bar icon
NSApp.setActivationPolicy(.regular)  // Normal app + status bar icon
```

## Third Option: .prohibited

There's also `.prohibited` (rarely used):
- App cannot activate at all
- Cannot become key or main
- Very specialized use cases

## User Experience Impact

| Aspect | .regular | .accessory |
|--------|----------|------------|
| **User Awareness** | High - visible everywhere | Low - only status bar |
| **Distraction Level** | Higher | Minimal |
| **Professional Feel** | Standard app | Utility/tool |
| **Discoverability** | Easy to find | Must know it exists |
| **Memory in User's Mind** | Remembered as "app" | Remembered as "feature" |

## Best Practices

**Use `.accessory` for:**
- Menu bar utilities
- Background monitors  
- System tools
- Apps that supplement other work

**Use `.regular` for:**
- Document-based apps
- Main productivity tools
- Apps with complex interfaces
- Apps users work in extensively

**Use dynamic switching for:**
- Menu bar apps with preferences
- Apps that can work both ways
- Complex utilities with setup windows