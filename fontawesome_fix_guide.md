# FontAwesome Vite Import Fix Guide

## Problem
```
Uncaught SyntaxError: The requested module does not provide an export named 'IconProp'
```

## Package Dependencies
Ensure you have the correct versions in your `package.json`:

```json
{
  "dependencies": {
    "@fortawesome/fontawesome-svg-core": "^6.5.0",
    "@fortawesome/free-solid-svg-icons": "^6.5.0",
    "@fortawesome/react-fontawesome": "^0.2.0"
  }
}
```

## Solution Options

### Option 1: Remove Type Assertion (Recommended)
```typescript
// ❌ Problematic
import type { IconProp } from '@fortawesome/fontawesome-svg-core';
<FontAwesomeIcon icon={faUser as IconProp} />

// ✅ Simple fix - no type assertion needed
import { FontAwesomeIcon } from '@fortawesome/react-fontawesome';
import { faUser, faCog, faToriiGate, faYinYang } from '@fortawesome/free-solid-svg-icons';

<FontAwesomeIcon icon={faUser} />
<FontAwesomeIcon icon={faCog} />
```

### Option 2: Use IconDefinition
```typescript
import { FontAwesomeIcon } from '@fortawesome/react-fontawesome';
import type { IconDefinition } from '@fortawesome/fontawesome-svg-core';
import { faUser, faCog } from '@fortawesome/free-solid-svg-icons';

<FontAwesomeIcon icon={faUser as IconDefinition} />
```

### Option 3: Import IconProp from react-fontawesome
```typescript
import { FontAwesomeIcon } from '@fortawesome/react-fontawesome';
import type { IconProp } from '@fortawesome/react-fontawesome';
import { faUser, faCog } from '@fortawesome/free-solid-svg-icons';

<FontAwesome icon={faUser as IconProp} />
```

## Updated Component Example

### Before (Problematic)
```typescript
import { FontAwesomeIcon } from '@fortawesome/react-fontawesome';
import type { IconProp } from '@fortawesome/fontawesome-svg-core';
import { faUser, faCog, faToriiGate, faYinYang } from '@fortawesome/free-solid-svg-icons';

// This causes the error
<FontAwesomeIcon icon={faYinYang as IconProp} />
```

### After (Fixed)
```typescript
import { FontAwesomeIcon } from '@fortawesome/react-fontawesome';
import { faUser, faCog, faToriiGate, faYinYang } from '@fortawesome/free-solid-svg-icons';

// Clean and simple - TypeScript infers the type correctly
<FontAwesomeIcon icon={faYinYang} />
<FontAwesomeIcon icon={faToriiGate} />
<FontAwesomeIcon icon={faUser} />
<FontAwesomeIcon icon={faCog} />
```

## Vite Configuration
If you're still having issues, add this to your `vite.config.ts`:

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  optimizeDeps: {
    include: [
      '@fortawesome/fontawesome-svg-core',
      '@fortawesome/free-solid-svg-icons',
      '@fortawesome/react-fontawesome'
    ]
  }
});
```

## Complete Working Example
```typescript
import React from 'react';
import { FontAwesomeIcon } from '@fortawesome/react-fontawesome';
import { 
  faUser, 
  faCog, 
  faToriiGate, 
  faYinYang,
  faHome,
  faSearch 
} from '@fortawesome/free-solid-svg-icons';

const MyComponent = () => {
  return (
    <div>
      <FontAwesomeIcon icon={faUser} /> User
      <FontAwesomeIcon icon={faCog} /> Settings
      <FontAwesomeIcon icon={faToriiGate} /> Groups
      <FontAwesomeIcon icon={faYinYang} /> Roles
      <FontAwesomeIcon icon={faHome} /> Home
      <FontAwesomeIcon icon={faSearch} /> Search
    </div>
  );
};

export default MyComponent;
```

## Troubleshooting Steps

1. **Clear Vite cache:**
   ```bash
   rm -rf node_modules/.vite
   npm run dev
   ```

2. **Reinstall FontAwesome packages:**
   ```bash
   npm uninstall @fortawesome/fontawesome-svg-core @fortawesome/free-solid-svg-icons @fortawesome/react-fontawesome
   npm install @fortawesome/fontawesome-svg-core @fortawesome/free-solid-svg-icons @fortawesome/react-fontawesome
   ```

3. **Check package versions match:**
   ```bash
   npm list @fortawesome/fontawesome-svg-core
   npm list @fortawesome/react-fontawesome
   ```

4. **Restart dev server:**
   ```bash
   npm run dev
   ```

## Why This Happens

- Vite's dependency pre-bundling can sometimes cause export mismatches
- TypeScript strict mode may not recognize the export
- Package version mismatches between FontAwesome core and React packages
- The `IconProp` type location changed in newer versions

## Best Practice

**Simply remove the type assertion** - TypeScript will correctly infer the icon type from the imported icon definitions. This is the cleanest and most reliable approach.