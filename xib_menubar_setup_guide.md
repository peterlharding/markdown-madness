# XIB-Based Menu Bar App Setup Guide

## Overview

XIB-based macOS apps have a different structure than Storyboard apps. Here's how to properly configure a XIB-based project for menu bar app development.

## Finding Info.plist in XIB Projects

### Method 1: Project Settings (Recommended)
1. Click your **project name** (blue icon) in Project Navigator
2. Select your **app target** under "TARGETS" 
3. Click the **"Info"** tab
4. Edit settings in the table format

### Method 2: Reveal in Finder
1. Right-click **project name** in Project Navigator
2. Choose **"Show in Finder"**
3. Find `Info.plist` in project folder
4. Drag file into Xcode Project Navigator

### Method 3: Check Supporting Files
Look for a **"Supporting Files"** folder in Project Navigator - XIB apps sometimes group Info.plist there.

## Required Info.plist Changes

### Remove These Keys:
| Key Name (UI) | XML Key | Action |
|---------------|---------|---------|
| "Main nib file base name" | `NSMainNibFile` | **DELETE** this row |
| "Main storyboard file base name" | `NSMainStoryboardFile` | **DELETE** if present |

### Add This Key:
| Key Name (UI) | XML Key | Value |
|---------------|---------|-------|
| "Application is agent (UIElement)" | `LSUIElement` | **YES** / `<true/>` |

## Step-by-Step Setup

### 1. Configure Info.plist
Using the **Info tab** method:
```
[Project] → [Target] → [Info Tab]

Current settings:
❌ Main nib file base name: MainMenu    ← DELETE THIS
❌ Bundle name: $(PRODUCT_NAME)

New settings:
✅ Application is agent: YES             ← ADD THIS
✅ Bundle name: $(PRODUCT_NAME)
```

### 2. Replace AppDelegate Code
Replace the entire contents of `AppDelegate.swift` with our menu bar app code.

### 3. Optional: Clean Up Files
You can delete these unused files:
- `MainMenu.xib` 
- Any default `ViewController.swift`

## Complete Info.plist (XML Format)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CFBundleExecutable</key>
    <string>$(EXECUTABLE_NAME)</string>
    <key>CFBundleIdentifier</key>
    <string>$(PRODUCT_BUNDLE_IDENTIFIER)</string>
    <key>CFBundleName</key>
    <string>$(PRODUCT_NAME)</string>
    <key>CFBundlePackageType</key>
    <string>APPL</string>
    <key>CFBundleShortVersionString</key>
    <string>1.0</string>
    <key>CFBundleVersion</key>
    <string>1</string>
    <key>LSMinimumSystemVersion</key>
    <string>$(MACOSX_DEPLOYMENT_TARGET)</string>
    
    <!-- REMOVE THESE LINES if present -->
    <!-- <key>NSMainNibFile</key> -->
    <!-- <string>MainMenu</string> -->
    
    <!-- ADD THIS LINE -->
    <key>LSUIElement</key>
    <true/>
    
</dict>
</plist>
```

## XIB vs Storyboard Differences

| Aspect | XIB-Based | Storyboard-Based |
|--------|-----------|------------------|
| **Main UI File** | `MainMenu.xib` | `Main.storyboard` |
| **Info.plist Key** | `NSMainNibFile` | `NSMainStoryboardFile` |
| **Default Structure** | Separate XIB files | Single storyboard file |
| **Menu Bar Setup** | Uses XIB for menu bar | Uses storyboard scenes |

## What These Changes Do

### Before Configuration:
- ❌ XIB file loads default menu and window
- ❌ App appears in dock with standard behavior
- ❌ Default macOS app experience

### After Configuration:
- ✅ No automatic UI loading
- ✅ App runs as background utility
- ✅ Only status bar icon visible
- ✅ Hidden from dock and App Switcher

## Testing the Setup

After making these changes:

1. **Build and run** your app
2. **Expected result**: Only status bar icon appears (no window)
3. **Click status bar icon**: Popover should appear
4. **Check dock**: App should be hidden
5. **Check Cmd+Tab**: App shouldn't appear in switcher

## Troubleshooting

### Still See Window on Launch?
- Double-check that `NSMainNibFile` key is completely removed
- Verify `LSUIElement` is set to `YES`/`<true/>`
- Clean build folder (Product → Clean Build Folder)

### Can't Find Info.plist?
- Try creating new Property List file named `Info.plist`
- Copy the XML content above
- Make sure it's added to your target

### Connection Errors?
- XIB apps may have outlet connections in MainMenu.xib
- Either disconnect them in Interface Builder or delete MainMenu.xib entirely

## Final Project Structure

```
MyMenuBarApp/
├── AppDelegate.swift          ← Our menu bar app code
├── Info.plist                ← Configured for background app
├── MainMenu.xib              ← Optional: can delete
├── Assets.xcassets           ← Add status bar icon here
└── MyMenuBarApp.entitlements ← Auto-generated
```

## Key Takeaway

XIB-based menu bar apps require removing the `NSMainNibFile` reference instead of `NSMainStoryboardFile`, but the overall approach is the same: disable automatic UI loading and enable background app behavior with `LSUIElement`.