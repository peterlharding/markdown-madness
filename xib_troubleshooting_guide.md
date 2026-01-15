# Troubleshooting XIB App That Won't Start

## Step 1: Check Required Info.plist Keys

Your `Info.plist` MUST contain these keys for the app to start:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- REQUIRED: App executable -->
    <key>CFBundleExecutable</key>
    <string>$(EXECUTABLE_NAME)</string>
    
    <!-- REQUIRED: Bundle identifier -->
    <key>CFBundleIdentifier</key>
    <string>$(PRODUCT_BUNDLE_IDENTIFIER)</string>
    
    <!-- REQUIRED: App name -->
    <key>CFBundleName</key>
    <string>$(PRODUCT_NAME)</string>
    
    <!-- REQUIRED: Bundle version -->
    <key>CFBundleVersion</key>
    <string>1</string>
    <key>CFBundleShortVersionString</key>
    <string>1.0</string>
    
    <!-- CRITICAL: Principal class (makes app work) -->
    <key>NSPrincipalClass</key>
    <string>NSApplication</string>
    
    <!-- Menu bar app setting -->
    <key>LSUIElement</key>
    <true/>
    
    <!-- REMOVE these if present: -->
    <!-- <key>NSMainNibFile</key> -->
    <!-- <string>MainMenu</string> -->
</dict>
</plist>
```

## Step 2: Add Debug Logging to AppDelegate

Update your AppDelegate with more extensive logging:

```swift
import Cocoa
import os.log

@main
class AppDelegate: NSObject, NSApplicationDelegate {
    
    let logger = Logger(subsystem: Bundle.main.bundleIdentifier ?? "MenuBarApp", category: "AppDelegate")
    
    var statusBarItem: NSStatusItem!
    var popover: NSPopover!
    
    func applicationDidFinishLaunching(_ aNotification: Notification) {
        // Multiple logging methods to catch the message
        print("🚀 App starting - print version")
        NSLog("🚀 App starting - NSLog version")
        logger.info("🚀 App starting - Logger version")
        
        // Force flush immediately
        fflush(stdout)
        
        setupApp()
    }
    
    func setupApp() {
        print("📱 Setting up menu bar app...")
        NSLog("📱 Setting up menu bar app...")
        
        // Hide from dock - this makes it a true background app
        NSApp.setActivationPolicy(.accessory)
        print("✅ Set activation policy to accessory")
        
        setupStatusBar()
        print("✅ Status bar setup complete")
        
        print("🎉 Menu bar app started successfully!")
        NSLog("🎉 Menu bar app started successfully!")
    }
    
    func setupStatusBar() {
        // Create status bar item
        statusBarItem = NSStatusBar.system.statusItem(withLength: NSStatusItem.squareLength)
        
        guard let button = statusBarItem.button else {
            print("❌ Failed to create status bar button")
            return
        }
        
        // Use SF Symbol
        button.image = NSImage(systemSymbolName: "bolt.circle", accessibilityDescription: "Menu Bar App")
        button.action = #selector(statusBarClicked)
        button.target = self
        
        print("✅ Status bar button configured")
    }
    
    @objc func statusBarClicked() {
        print("🖱️ Status bar clicked!")
        NSLog("🖱️ Status bar clicked!")
        
        let alert = NSAlert()
        alert.messageText = "Menu Bar App Working!"
        alert.informativeText = "Your app is running correctly."
        alert.runModal()
    }
    
    // Add termination logging
    func applicationWillTerminate(_ notification: Notification) {
        print("🛑 App will terminate")
        NSLog("🛑 App will terminate")
    }
}
```

## Step 3: Check Build Settings

1. **Select your target** in Xcode
2. **Go to Build Settings**
3. **Search for "Principal Class"**
4. **Verify it's set to**: `NSApplication`

## Step 4: Debugging Steps

### A. Run from Xcode with Console Open
1. **Open Xcode console** (View → Debug Area → Activate Console)
2. **Run your app**
3. **Look for any print statements or errors**

### B. Check Activity Monitor
1. **Open Activity Monitor**
2. **Search for your app name**
3. **See if the process starts and immediately quits**

### C. Run from Terminal
1. **Build your app** in Xcode
2. **Find the .app bundle** in Products folder
3. **Run from Terminal**:
```bash
cd /path/to/your/app
./YourApp.app/Contents/MacOS/YourApp
```
4. **This shows all output** and error messages

## Step 5: Common Issues and Fixes

### Issue: NSPrincipalClass Missing
**Symptom**: App doesn't start at all
**Fix**: Add `NSPrincipalClass = NSApplication` to Info.plist

### Issue: AppDelegate Not Connected
**Symptom**: App starts but delegate methods don't run
**Fix**: Ensure your AppDelegate class has `@main` attribute

### Issue: Immediate Termination
**Symptom**: App quits right after starting
**Fix**: App might think it has no windows and quits. Add this:

```swift
func applicationShouldTerminateAfterLastWindowClosed(_ sender: NSApplication) -> Bool {
    return false  // Don't quit when windows close
}
```

### Issue: Silent Crash
**Symptom**: No output, app just doesn't appear
**Fix**: Check Console.app for crash reports

## Step 6: Minimal Working Info.plist Test

Try this absolutely minimal Info.plist to test:

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
    <key>CFBundleVersion</key>
    <string>1</string>
    <key>NSPrincipalClass</key>
    <string>NSApplication</string>
</dict>
</plist>
```

Test if app starts with this basic configuration first.

## Step 7: Emergency Restore

If nothing works, temporarily restore the XIB:

1. **Add back to Info.plist**:
```xml
<key>NSMainNibFile</key>
<string>MainMenu</string>
```

2. **Test that app starts**
3. **Then gradually remove XIB dependency**

## Quick Diagnostic Commands

Run these in Terminal to check your app:

```bash
# Check Info.plist contents
plutil -p /path/to/your/app/Info.plist

# Check if executable exists
ls -la YourApp.app/Contents/MacOS/

# Check app launch
open YourApp.app
```