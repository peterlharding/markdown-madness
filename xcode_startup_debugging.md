# Complete App Startup Debugging Guide

## Step 1: Verify App Actually Starts

### Check if Process Launches:
1. **Open Activity Monitor** (Applications → Utilities)
2. **Search for your app name**
3. **Run your app from Xcode**
4. **Watch Activity Monitor** - does the process appear and disappear quickly?

### Check Console for Crashes:
1. **Open Console.app** (Applications → Utilities)
2. **Click "Start streaming"**
3. **Filter by your app name**
4. **Run your app**
5. **Look for crash reports or error messages**

## Step 2: Create Absolute Minimal Test

Replace your **entire** AppDelegate.swift with this ultra-minimal version:

```swift
import Cocoa

@main
class AppDelegate: NSObject, NSApplicationDelegate {
    
    override init() {
        super.init()
        print("🎯 AppDelegate INIT called")
        NSLog("🎯 AppDelegate INIT called")
        fflush(stdout)
    }
    
    func applicationDidFinishLaunching(_ aNotification: Notification) {
        print("🚀 DID FINISH LAUNCHING!")
        NSLog("🚀 DID FINISH LAUNCHING!")
        fflush(stdout)
        
        // Show immediate proof app is running
        DispatchQueue.main.async {
            let alert = NSAlert()
            alert.messageText = "APP IS ALIVE!"
            alert.informativeText = "If you see this, the app started successfully"
            alert.runModal()
            
            print("✅ Alert shown successfully")
            NSLog("✅ Alert shown successfully")
        }
    }
    
    func applicationShouldTerminateAfterLastWindowClosed(_ sender: NSApplication) -> Bool {
        print("🚪 Asked if should terminate - saying NO")
        return false
    }
    
    func applicationWillTerminate(_ notification: Notification) {
        print("🛑 App terminating")
        NSLog("🛑 App terminating")
    }
}
```

## Step 3: Check Build Settings (Critical)

Go to **Target → Build Settings** and verify these are **exactly** as shown:

| Setting | Value | Status |
|---------|-------|--------|
| **Generate Info.plist File** | `YES` | ✅ Should be enabled |
| **Info.plist Key - LSUIElement** | `YES` | ✅ Should be YES |
| **Info.plist Key - Main Interface** | *(empty)* | ❌ Must be empty |
| **Info.plist Key - Main Nib** | *(empty)* | ❌ Must be empty |
| **Info.plist Key - NSPrincipalClass** | `NSApplication` | ✅ Critical |

### How to Check Each Setting:

1. **Target** → **Build Settings**
2. **Filter: "All" and "Combined"**
3. **Search for each setting name**:
   - `GENERATE_INFOPLIST_FILE`
   - `INFOPLIST_KEY_LSUIElement`
   - `INFOPLIST_KEY_UIMainStoryboardFile`
   - `INFOPLIST_KEY_NSMainNibFile`
   - `INFOPLIST_KEY_NSPrincipalClass`

## Step 4: Check for XIB Remnants

### Look for these files and connections:
- `MainMenu.xib` - **Delete or disconnect**
- Any `@IBOutlet` connections in AppDelegate
- Window outlets in Interface Builder

### Clean XIB References:
1. **If MainMenu.xib exists**, open it in Interface Builder
2. **Check "File's Owner"** (usually AppDelegate)
3. **Right-click File's Owner** → see if there are outlet connections
4. **Delete any outlet connections** (especially to windows)

## Step 5: Debugging Output Methods

### Terminal Launch (Most Reliable):
1. **Build your app** in Xcode (Cmd+B)
2. **Find the .app bundle**:
   ```bash
   # Usually located at:
   ~/Library/Developer/Xcode/DerivedData/YourApp-*/Build/Products/Debug/YourApp.app
   ```
3. **Run from Terminal**:
   ```bash
   /path/to/YourApp.app/Contents/MacOS/YourApp
   ```
4. **All output will show** in Terminal

### Xcode Console Check:
1. **View** → **Debug Area** → **Activate Console**
2. **Run app** (Cmd+R)
3. **Look at bottom panel** for output

## Step 6: Code Signing Check

App might fail to start due to signing issues:

### Check Signing:
1. **Target** → **Signing & Capabilities**
2. **Verify**: "Automatically manage signing" is checked
3. **Check**: Team is selected
4. **Look for**: Any red error messages

### Manual Verification:
```bash
# Check if app is signed
codesign -dv /path/to/YourApp.app

# Check if app can run
spctl -a -v /path/to/YourApp.app
```

## Step 7: Create Fresh Test Project

To isolate the issue, create a **brand new test project**:

### New Project Settings:
1. **macOS** → **App**
2. **Interface**: "XIB"
3. **Language**: "Swift"
4. **Use Core Data**: NO

### Immediate Test:
1. **Run default project** - does it work?
2. **Add one print statement** to applicationDidFinishLaunching
3. **Run again** - do you see output?

## Step 8: Emergency XIB Restore Test

To prove the issue is XIB-related:

### Temporarily Restore XIB:
1. **Build Settings** → **Info.plist Key - Main Nib** → Set to `MainMenu`
2. **Run app** - does it start with window?
3. **If YES**, the issue is in our menu bar configuration
4. **If NO**, there's a deeper project issue

## Step 9: Common Modern Xcode Issues

### Issue: Hardened Runtime
Some security settings prevent background apps:
- **Signing & Capabilities** → **Hardened Runtime**
- **Disable** temporarily to test

### Issue: App Sandbox
Sandbox might interfere:
- **Signing & Capabilities** → **App Sandbox**
- **Disable** temporarily to test

### Issue: macOS Version
Some APIs require newer macOS:
- **Deployment Target** → Set to macOS 11.0 or newer

## Diagnostic Commands

Run these in Terminal to check your app:

```bash
# Find your built app
find ~/Library/Developer/Xcode/DerivedData -name "*.app" -path "*/Debug/*" | grep YourAppName

# Check app info
/usr/libexec/PlistBuddy -c "Print" /path/to/YourApp.app/Contents/Info.plist

# Test launch with verbose output
/path/to/YourApp.app/Contents/MacOS/YourApp &
echo "App PID: $!"
sleep 2
kill $! 2>/dev/null || echo "App already terminated"
```

## Expected Results

After fixing the configuration:

✅ **Terminal output**: Should show print/NSLog statements  
✅ **Activity Monitor**: App process should stay running  
✅ **No windows**: Should only see status bar icon  
✅ **Console.app**: Should show NSLog messages  

## Next Steps

1. **Try the minimal AppDelegate** first
2. **Use Terminal launch** to see all output
3. **Check Activity Monitor** to see if process starts
4. **Report back** what you find in Console.app and Activity Monitor

The generated Info.plist approach is actually better than separate files, but menu bar apps need very specific configuration to work properly.