# Securing Xcode Projects for Git - Credentials & Certificates

## Overview

Keep sensitive signing information (certificates, provisioning profiles, team IDs) out of your Git repository while maintaining a buildable project for collaborators.

## Step 1: Configure Signing Settings

### Option A: Automatic Signing (Recommended for Development)

1. **Select your target** in Project Navigator
2. **Go to Signing & Capabilities tab**
3. **Check "Automatically manage signing"**
4. **Set Team to your development team**
5. **Bundle Identifier**: Use a generic placeholder like `com.yourcompany.${PRODUCT_NAME}`

### Option B: Manual Signing (Advanced)

1. **Uncheck "Automatically manage signing"**
2. **Leave Provisioning Profile as "None" in project**
3. **Use build configuration files** (see below)

## Step 2: Essential .gitignore Configuration

Create/update `.gitignore` in your project root:

```gitignore
# Xcode
#
# gitignore contributors: remember to update Global/Xcode.gitignore, Objective-C.gitignore & Swift.gitignore

## User settings
xcuserdata/

## Xcode 8 and earlier
*.xcscmblueprint
*.xccheckout

## Xcode 4 - 8
*.xcuserstate
*.xcuserdatad

## Build generated
build/
DerivedData/

## Various settings
*.pbxuser
!default.pbxuser
*.mode1v3
!default.mode1v3
*.mode2v3
!default.mode2v3
*.perspectivev3
!default.perspectivev3

## Other
*.moved-aside
*.xccheckout
*.xcscmblueprint

## Obj-C/Swift specific
*.hmap

## App packaging
*.ipa
*.dSYM.zip
*.dSYM

## Playgrounds
timeline.xctimeline
playground.xcworkspace

# Swift Package Manager
.build/

## CocoaPods
Pods/

## Fastlane
fastlane/report.xml
fastlane/Preview.html
fastlane/screenshots/**/*.png
fastlane/test_output

# CRITICAL: Signing & Certificates
*.mobileprovision
*.p12
*.cer
*.p8
AuthKey_*.p8
*.certSigningRequest

# Development Team Settings (add these)
*.xcconfig
Config/*.xcconfig
!Config/Base.xcconfig

# Sensitive Info
secrets.plist
Config.plist
APIKeys.plist
```

## Step 3: Use Configuration Files

### Create Base Configuration (Committed)

Create `Config/Base.xcconfig`:
```bash
// Base.xcconfig - Safe for Git
PRODUCT_NAME = MyMenuBarApp
PRODUCT_BUNDLE_IDENTIFIER = com.placeholder.$(PRODUCT_NAME)
MACOSX_DEPLOYMENT_TARGET = 11.0
SWIFT_VERSION = 5.0

// Placeholder values - override in personal config
DEVELOPMENT_TEAM = TEAM_ID_PLACEHOLDER
CODE_SIGN_IDENTITY = Apple Development
```

### Create Personal Configuration (Not Committed)

Create `Config/Personal.xcconfig`:
```bash
// Personal.xcconfig - DO NOT COMMIT
#include "Base.xcconfig"

// Your actual values
DEVELOPMENT_TEAM = ABC123XYZ9
PRODUCT_BUNDLE_IDENTIFIER = com.yourname.$(PRODUCT_NAME)
CODE_SIGN_IDENTITY = Apple Development: Your Name (ABC123XYZ9)
```

### Apply Configuration Files

1. **Select your project** (blue icon)
2. **Go to Info tab**
3. **Under Configurations**, set:
   - Debug: `Personal` (for your local builds)
   - Release: `Personal` (for your releases)

## Step 4: Environment Variables (Alternative)

### Using Xcode Build Settings

1. **Select target → Build Settings**
2. **Search for "Development Team"**
3. **Set value to**: `$(DEVELOPMENT_TEAM)`
4. **In your shell profile** (`~/.zshrc` or `~/.bash_profile`):

```bash
# Xcode Development Settings
export DEVELOPMENT_TEAM="ABC123XYZ9"
export CODE_SIGN_IDENTITY="Apple Development"
```

## Step 5: Team Collaboration Setup

### For Team Members

Create `README.md` with setup instructions:

```markdown
# Development Setup

## Prerequisites
1. Join the Apple Developer team
2. Install Xcode
3. Configure signing

## Setup Steps
1. Clone repository
2. Copy `Config/Base.xcconfig` to `Config/Personal.xcconfig`
3. Edit `Personal.xcconfig` with your team ID:
   ```
   DEVELOPMENT_TEAM = YOUR_TEAM_ID
   PRODUCT_BUNDLE_IDENTIFIER = com.yourname.MyMenuBarApp
   ```
4. Build project

## Finding Your Team ID
- Open Xcode
- Preferences → Accounts
- Select your Apple ID
- Your Team ID is shown in the team list
```

## Step 6: Keychain Access Management

### Store Certificates Securely

Certificates are automatically stored in **macOS Keychain**, not in your project:

1. **Download certificates** from Apple Developer portal
2. **Double-click to install** in Keychain
3. **Never commit** `.p12`, `.cer`, or `.mobileprovision` files

### Team Certificate Sharing (If Needed)

For team distribution certificates:
1. **Export certificate** from Keychain (with password)
2. **Share securely** (encrypted email, secure file share)
3. **Never commit to Git**

## Step 7: Verification Checklist

### Before Committing, Ensure:
- [ ] No `.mobileprovision` files in repo
- [ ] No certificates (`.p12`, `.cer`) in repo  
- [ ] No hardcoded Team IDs in project files
- [ ] `.gitignore` includes signing files
- [ ] Personal config files are ignored
- [ ] Project builds with placeholder values

### Test Your Setup:
1. **Fresh clone** of your repo
2. **Try to build** without personal config
3. **Should fail gracefully** with clear error messages
4. **Add personal config** → should build successfully

## Step 8: Advanced Security

### For Production Apps

1. **Use separate Apple ID** for development vs distribution
2. **Rotate certificates** regularly
3. **Use CI/CD systems** with secure environment variables
4. **Never commit API keys** or secrets

### Environment-Specific Configs

```
Config/
├── Base.xcconfig          ← Committed (safe defaults)
├── Development.xcconfig   ← Committed (dev settings)
├── Production.xcconfig    ← Committed (prod settings) 
├── Personal.xcconfig      ← NOT COMMITTED (your creds)
└── Team.xcconfig          ← NOT COMMITTED (team creds)
```

## Common Pitfalls to Avoid

| ❌ Don't Do | ✅ Do Instead |
|-------------|---------------|
| Hardcode Team ID in project | Use variables/config files |
| Commit `.mobileprovision` files | Add to `.gitignore` |
| Share certificates in Slack | Use secure, encrypted sharing |
| Same config for all developers | Personal config files |
| Ignore keychain security | Regular certificate rotation |

## Quick Setup Summary

1. **Enable automatic signing** in Xcode
2. **Add comprehensive `.gitignore`**
3. **Use config files** for sensitive values
4. **Test with fresh clone**
5. **Document setup** for team members

This approach keeps your credentials secure while maintaining a collaborative development environment!