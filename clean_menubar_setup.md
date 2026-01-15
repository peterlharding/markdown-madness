# Complete Menu Bar App Setup - Remove Unwanted Window

## Step 1: Fix Info.plist

**Remove storyboard reference to prevent automatic window:**

1. Open `Info.plist` in Xcode
2. **DELETE** these lines if they exist:
   ```xml
   <key>NSMainStoryboardFile</key>
   <string>Main</string>
   ```

3. **ADD** this line to make it a background app:
   ```xml
   <key>LSUIElement</key>
   <true/>
   ```

## Step 2: Your Info.plist Should Look Like This:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CFBundleDevelopmentRegion</key>
    <string>$(DEVELOPMENT_LANGUAGE)</string>
    <key>CFBundleExecutable</key>
    <string>$(EXECUTABLE_NAME)</string>
    <key>CFBundleIconFile</key>
    <string></string>
    <key>CFBundleIdentifier</key>
    <string>$(PRODUCT_BUNDLE_IDENTIFIER)</string>
    <key>CFBundleInfoDictionaryVersion</key>
    <string>6.0</string>
    <key>CFBundleName</key>
    <string>$(PRODUCT_NAME)</string>
    <key>CFBundlePackageType</key>
    <string>APPL</string>
    <key>CFBundleShortVersionString</key>
    <string>1.0</string>
    <key>CFBundleVersion</key>
    <string>1</string>
    
    <!-- ADD THIS - Makes it a background/menu bar app -->
    <key>LSUIElement</key>
    <true/>
    
    <!-- REMOVE THESE LINES if they exist -->
    <!-- <key>NSMainStoryboardFile</key> -->
    <!-- <string>Main</string> -->
    
    <key>LSMinimumSystemVersion</key>
    <string>$(MACOSX_DEPLOYMENT_TARGET)</string>
    <key>NSHumanReadableCopyright</key>
    <string>Copyright © 2025. All rights reserved.</string>
    <key>NSMainNibFile</key>
    <string>MainMenu</string>
    <key>NSPrincipalClass</key>
    <string>NSApplication</string>
</dict>
</plist>
```

## Step 3: Clean Up AppDelegate

Remove the dummy outlet since we won't use storyboards:

```swift
@main
class AppDelegate: NSObject, NSApplicationDelegate {
    
    // REMOVE this line:
    // @IBOutlet weak var window: NSWindow?
    
    var statusBarItem: NSStatusItem!
    var popover: NSPopover!
    var preferencesWindowController: PreferencesWindowController?
    
    // ... rest of code stays the same
}
```

## Step 4: Optional - Delete Unused Files

You can safely delete these files:
- `Main.storyboard`
- `ViewController.swift` (if it exists)

## What This Fixes:

| Before Fix | After Fix |
|------------|-----------|
| ❌ Window opens automatically | ✅ Only status bar icon appears |
| ❌ App shows in dock immediately | ✅ Hidden from dock |
| ❌ Storyboard connection errors | ✅ No connection errors |
| ❌ Unwanted default window | ✅ Clean menu bar app |

## Test the Fix:

1. Make the Info.plist changes
2. Remove the `@IBOutlet weak var window: NSWindow?` line
3. Run the app
4. **Result**: You should only see the status bar icon (⭐), no window

## What Was Happening:

```
App Starts
    ↓
Info.plist says: "Load Main.storyboard"
    ↓
Main.storyboard contains: Default Window
    ↓
Window appears automatically ← This is what you saw!

VS.

App Starts (after fix)
    ↓
Info.plist: No storyboard reference
    ↓
AppDelegate runs our code
    ↓
Only status bar icon appears ← This is what we want!
```
