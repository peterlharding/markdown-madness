# Finding Your Apple Developer Team ID

## Method 1: Xcode Preferences (Most Reliable)

1. **Open Xcode**
2. **Xcode menu** → **Preferences** (or **Settings** in newer Xcode)
3. **Click "Accounts" tab**
4. **Select your Apple ID** on the left
5. **Look at the right panel** - you'll see teams listed
6. **Team ID appears** in parentheses next to team name

**Example display:**
```
John Doe (Personal Team) (ABC123DEF4)
                         ^^^^^^^^^^^
                         This is your Team ID
```

## Method 2: Apple Developer Portal

1. **Go to** [developer.apple.com](https://developer.apple.com)
2. **Sign in** with your Apple ID
3. **Click "Account"**
4. **Go to "Membership"** section
5. **Team ID** is displayed under your organization info

## Method 3: Command Line (Quick!)

Open **Terminal** and run:

```bash
# List all available teams
xcrun xcodebuild -showBuildSettings | grep DEVELOPMENT_TEAM

# Or more specific:
security find-identity -v -p codesigning
```

The second command shows your certificates and Team IDs.

## Method 4: Keychain Access

1. **Open Keychain Access** (Applications → Utilities)
2. **Select "login" keychain**
3. **Click "Certificates" category**
4. **Find your "Apple Development" certificate**
5. **Double-click the certificate**
6. **Look at "Organizational Unit (OU)"** - this is your Team ID

## Method 5: Existing Xcode Project

If you have any other Xcode project with signing configured:

1. **Select project** in Project Navigator
2. **Select target**
3. **Go to "Build Settings" tab**
4. **Search for "Development Team"**
5. **The value shown** is your Team ID

## Method 6: Export from Xcode

1. **Create a new iOS/macOS project** (temporary)
2. **Enable automatic signing**
3. **Select your team** from dropdown
4. **Go to Build Settings**
5. **Search "DEVELOPMENT_TEAM"**
6. **Copy the value**

## What Team ID Looks Like

**Format:** `XXXXXXXXXX` (10 alphanumeric characters)

**Examples:**
- `ABC123DEF4`
- `X9Y8Z7W6V5`
- `1234567890`

## Different Types of Teams

| Team Type | What You'll See |
|-----------|-----------------|
| **Personal Team** | `Your Name (Personal Team) (ABC123DEF4)` |
| **Organization** | `Company Name (ABC123DEF4)` |
| **Enterprise** | `Company Name (Enterprise) (ABC123DEF4)` |

## Command Line Verification

Once you have your Team ID, verify it works:

```bash
# Check if Team ID is valid
xcrun xcodebuild -showBuildSettings -project YourProject.xcodeproj | grep ABC123DEF4

# Or set it as environment variable to test
export DEVELOPMENT_TEAM="ABC123DEF4"
```

## Using Your Team ID

### In Xcode Project Settings:
```
Target → Signing & Capabilities → Team: [Select your team]
```

### In Build Settings:
```
DEVELOPMENT_TEAM = ABC123DEF4
```

### In .xcconfig files:
```bash
// Personal.xcconfig
DEVELOPMENT_TEAM = ABC123DEF4
```

### As Environment Variable:
```bash
# Add to ~/.zshrc or ~/.bash_profile
export DEVELOPMENT_TEAM="ABC123DEF4"
```

## Troubleshooting

### "No teams found" in Xcode?
1. **Check Apple ID** is signed in (Xcode → Preferences → Accounts)
2. **Refresh** by removing and re-adding Apple ID
3. **Verify** you have Apple Developer Program membership

### Team ID not working?
1. **Double-check** the 10-character format
2. **Ensure** your Apple ID has access to this team
3. **Try** signing out and back in to Xcode

### Multiple teams?
- **Personal Team**: For free Apple ID (limited features)
- **Paid Developer Program**: Full features ($99/year)
- **Enterprise**: For internal distribution only

## Quick Copy Commands

Run these in Terminal to get your Team ID quickly:

```bash
# Method A: From build settings
defaults read ~/Library/Preferences/com.apple.dt.Xcode.plist | grep -i team

# Method B: From keychain
security find-identity -v -p codesigning | grep "Apple Development" | head -1

# Method C: Using xcodebuild
xcrun xcodebuild -showBuildSettings 2>/dev/null | grep DEVELOPMENT_TEAM | head -1
```

## Best Practice

**Save your Team ID** in a secure note or password manager - you'll need it for:
- CI/CD configurations
- Build scripts
- Team collaboration setup
- Multiple project configurations

The Team ID never changes for your developer account, so once you find it, you can reuse it across all your projects.