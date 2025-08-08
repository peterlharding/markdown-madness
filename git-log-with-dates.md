# Git Log with Dates - Quick Reference

## 📅 Basic Date Formats

### Most Common Usage
```bash
# Short and clean format (YYYY-MM-DD)
git log --pretty=format:"%h %ad %s" --date=short
# Output: a1b2c3d 2025-01-15 Your commit message
```

### Different Date Format Options
```bash
# ISO format (YYYY-MM-DD HH:MM:SS)
git log --pretty=format:"%h %ad %s" --date=iso

# Relative dates (2 days ago, 1 week ago)
git log --pretty=format:"%h %ar %s" --date=relative

# RFC format (Mon, 15 Jan 2025 14:30:00 +1000)
git log --pretty=format:"%h %ad %s" --date=rfc

# Human-readable format
git log --pretty=format:"%h %ad %s" --date=human
```

## 🎨 Enhanced Formats

### With Color
```bash
git log --pretty=format:"%C(yellow)%h%C(reset) %C(green)%ad%C(reset) %s" --date=short
```

### With Author Name
```bash
git log --pretty=format:"%h %ad %an: %s" --date=short
```

### With Branch/Tag Info
```bash
git log --pretty=format:"%h %ad %d | %s" --date=short --decorate
```

### Both Author and Commit Dates
```bash
git log --pretty=format:"%h %ad | %cd %s" --date=short
```

## 📋 Format Placeholders Reference

| Placeholder | Description |
|-------------|-------------|
| `%h` | Short commit hash |
| `%H` | Full commit hash |
| `%ad` | Author date |
| `%cd` | Commit date |
| `%ar` | Author date (relative) |
| `%cr` | Commit date (relative) |
| `%s` | Subject (commit message) |
| `%an` | Author name |
| `%ae` | Author email |
| `%d` | Ref names (branches, tags) |

## 🔧 Create Git Aliases

Add to your `.gitconfig` for easy reuse:

```bash
# Short log with dates
git config --global alias.logd "log --pretty=format:'%h %ad %s' --date=short"

# Colorized log with dates
git config --global alias.logdc "log --pretty=format:'%C(yellow)%h%C(reset) %C(green)%ad%C(reset) %s' --date=short"

# Log with author and date
git config --global alias.logda "log --pretty=format:'%h %ad %an: %s' --date=short"
```

Then use:
```bash
git logd    # Short log with dates
git logdc   # Colorized log with dates
git logda   # Log with author and dates
```

## 🎯 Recommended Formats

### For Daily Use
```bash
git log --pretty=format:"%h %ad %s" --date=short
```

### For Code Reviews
```bash
git log --pretty=format:"%h %ad %an: %s" --date=short --decorate
```

### For Debugging
```bash
git log --pretty=format:"%h %ad | %cd %s" --date=iso
```

## 📅 Date Format Options

| Format | Example | Description |
|--------|---------|-------------|
| `short` | 2025-01-15 | YYYY-MM-DD |
| `iso` | 2025-01-15 14:30:00 +1000 | ISO 8601 format |
| `rfc` | Mon, 15 Jan 2025 14:30:00 +1000 | RFC 2822 format |
| `relative` | 2 days ago | Human relative time |
| `human` | yesterday | Human-friendly format |
| `raw` | 1705123456 +1000 | Unix timestamp |

---

**Quick Tip**: The `--date=short` format is usually the most readable for one-line logs!
