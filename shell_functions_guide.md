# Shell Functions Guide

A comprehensive guide to defining and using shell functions in bash/zsh.

## Basic Syntax

### Method 1 (Preferred)
```bash
function_name() {
    # commands here
    echo "Hello from function"
}
```

### Method 2 (Alternative)
```bash
function function_name {
    # commands here
    echo "Hello from function"
}
```

## Simple Examples

### Basic Function
```bash
greet() {
    echo "Hello, World!"
}

# Call it
greet
```

### Function with Parameters
```bash
greet() {
    echo "Hello, $1!"
}

# Call with argument
greet "John"        # Output: Hello, John!
greet "$(whoami)"   # Output: Hello, username!
```

### Multiple Parameters
```bash
add_numbers() {
    local result=$(($1 + $2))
    echo "Result: $result"
}

add_numbers 5 3     # Output: Result: 8
```

## Parameter Handling

```bash
show_params() {
    echo "Function name: $0"
    echo "First parameter: $1"
    echo "Second parameter: $2"
    echo "All parameters: $@"
    echo "Number of parameters: $#"
}

show_params one two three
```

### Parameter Variables
- `$0` - Function name
- `$1`, `$2`, `$3`, ... - Individual parameters
- `$@` - All parameters as separate strings
- `$*` - All parameters as single string
- `$#` - Number of parameters

## Return Values

### Using Return Code
```bash
is_file() {
    if [ -f "$1" ]; then
        return 0    # success
    else
        return 1    # failure
    fi
}

# Use it
if is_file "myfile.txt"; then
    echo "File exists"
else
    echo "File not found"
fi
```

### Using Echo (Capture Output)
```bash
get_timestamp() {
    date +%Y%m%d_%H%M%S
}

# Capture the output
timestamp=$(get_timestamp)
echo "Backup name: backup_$timestamp.sql"
```

## Local Variables

```bash
calculate() {
    local num1=$1
    local num2=$2
    local result=$((num1 * num2))
    
    echo $result
}

result=$(calculate 4 5)
echo "4 x 5 = $result"
```

## Real-World Examples

### Database Backup Function
```bash
backup_db() {
    local db_name=$1
    local backup_dir=${2:-"/backups"}  # Default to /backups
    local timestamp=$(date +%Y%m%d_%H%M%S)
    
    echo "Backing up database: $db_name"
    pg_dump -d "$db_name" -f "$backup_dir/${db_name}_${timestamp}.sql"
    
    if [ $? -eq 0 ]; then
        echo "Backup successful: ${db_name}_${timestamp}.sql"
        return 0
    else
        echo "Backup failed!"
        return 1
    fi
}

# Usage
backup_db "myapp_db"
backup_db "myapp_db" "/custom/backup/path"
```

### Git Helpers
```bash
git_commit_push() {
    local message="$1"
    
    if [ -z "$message" ]; then
        echo "Error: Commit message required"
        return 1
    fi
    
    git add .
    git commit -m "$message"
    git push
}

# Usage
git_commit_push "Fix database connection issue"
```

### Docker Helpers
```bash
docker_logs() {
    local container_name=$1
    local lines=${2:-100}  # Default to 100 lines
    
    docker logs --tail "$lines" -f "$container_name"
}

# Usage
docker_logs "my_app"
docker_logs "my_app" 50
```

## Making Functions Available

### In Current Session
```bash
# Just define and use immediately
my_function() {
    echo "Hello"
}
my_function
```

### Permanent (Add to ~/.bashrc or ~/.zshrc)
```bash
# Edit your shell config file
nano ~/.bashrc

# Add your functions
backup_db() {
    # function definition here
}

# Reload your shell config
source ~/.bashrc
```

### In Separate File
```bash
# Create a file: ~/my_functions.sh
#!/bin/bash

my_function() {
    echo "Hello from external file"
}

# Source it in your shell
source ~/my_functions.sh

# Or add to ~/.bashrc:
# source ~/my_functions.sh
```

## Advanced Features

### Function with Error Handling
```bash
safe_copy() {
    local source=$1
    local dest=$2
    
    # Check if source exists
    if [ ! -f "$source" ]; then
        echo "Error: Source file '$source' not found" >&2
        return 1
    fi
    
    # Create destination directory if needed
    local dest_dir=$(dirname "$dest")
    mkdir -p "$dest_dir"
    
    # Copy file
    if cp "$source" "$dest"; then
        echo "Successfully copied $source to $dest"
        return 0
    else
        echo "Error: Failed to copy file" >&2
        return 1
    fi
}
```

### Function that Processes Multiple Arguments
```bash
process_files() {
    local action=$1
    shift  # Remove first argument, leaving just the files
    
    for file in "$@"; do
        echo "Processing: $file with action: $action"
        # Do something with each file
    done
}

# Usage
process_files "backup" file1.txt file2.txt file3.txt
```

### Default Parameter Values
```bash
create_backup() {
    local filename=$1
    local backup_dir=${2:-"./backups"}    # Default value
    local timestamp=${3:-$(date +%Y%m%d)}  # Dynamic default
    
    mkdir -p "$backup_dir"
    cp "$filename" "$backup_dir/${filename}.${timestamp}.bak"
}
```

## Development Helper Functions

### FastAPI Development Server
```bash
start_api() {
    local app=${1:-"main:app"}
    local port=${2:-8000}
    
    echo "Starting FastAPI on port $port"
    uvicorn "$app" --reload --host 0.0.0.0 --port "$port"
}

# Usage
start_api
start_api "app:application" 8080
```

### Database Reset Function
```bash
reset_db() {
    local db_name=$1
    
    if [ -z "$db_name" ]; then
        echo "Usage: reset_db <database_name>"
        return 1
    fi
    
    echo "Dropping database: $db_name"
    dropdb "$db_name"
    
    echo "Creating database: $db_name"
    createdb "$db_name"
    
    echo "Running migrations..."
    alembic upgrade head
}
```

### Project Directory Navigation
```bash
goto_project() {
    local project_name=$1
    local projects_dir="$HOME/projects"
    
    if [ -z "$project_name" ]; then
        echo "Available projects:"
        ls "$projects_dir"
        return 1
    fi
    
    if [ -d "$projects_dir/$project_name" ]; then
        cd "$projects_dir/$project_name"
        echo "Switched to: $(pwd)"
    else
        echo "Project '$project_name' not found"
        return 1
    fi
}
```

## Function Organization

### Grouping Related Functions
```bash
# Docker functions
docker_start() { docker start "$1"; }
docker_stop() { docker stop "$1"; }
docker_restart() { docker restart "$1"; }

# Git functions  
git_status() { git status --short; }
git_branch() { git branch -v; }
git_log() { git log --oneline -10; }

# System functions
disk_usage() { df -h; }
memory_usage() { free -h; }
process_list() { ps aux | head -20; }
```

### Function Libraries
Create themed function files:

**~/.functions/git.sh**
```bash
#!/bin/bash
# Git helper functions

git_commit_push() {
    # Implementation here
}

git_new_branch() {
    # Implementation here  
}
```

**~/.functions/docker.sh**
```bash
#!/bin/bash
# Docker helper functions

docker_cleanup() {
    # Implementation here
}

docker_logs() {
    # Implementation here
}
```

**Load in ~/.bashrc:**
```bash
# Load function libraries
for file in ~/.functions/*.sh; do
    [ -r "$file" ] && source "$file"
done
```

## Best Practices

### 1. Use Local Variables
```bash
# Good
my_function() {
    local temp_var="value"
    local result=$(some_command)
}

# Bad - creates global variables
my_function() {
    temp_var="value"
    result=$(some_command)
}
```

### 2. Quote Variables
```bash
# Good - handles spaces and special characters
process_file() {
    local filename="$1"
    if [ -f "$filename" ]; then
        echo "Processing: $filename"
    fi
}

# Bad - breaks with spaces
process_file() {
    local filename=$1
    if [ -f $filename ]; then
        echo "Processing: $filename"
    fi
}
```

### 3. Validate Input Parameters
```bash
create_user() {
    local username=$1
    local email=$2
    
    # Validate required parameters
    if [ -z "$username" ] || [ -z "$email" ]; then
        echo "Usage: create_user <username> <email>" >&2
        return 1
    fi
    
    # Validate email format (simple check)
    if [[ ! "$email" =~ ^[^@]+@[^@]+\.[^@]+$ ]]; then
        echo "Error: Invalid email format" >&2
        return 1
    fi
    
    # Proceed with user creation
    echo "Creating user: $username ($email)"
}
```

### 4. Use Meaningful Return Codes
```bash
backup_file() {
    local source=$1
    local dest=$2
    
    # Different return codes for different errors
    [ -z "$source" ] && return 1    # Missing parameter
    [ ! -f "$source" ] && return 2  # Source not found
    [ -f "$dest" ] && return 3      # Destination exists
    
    # Perform backup
    if cp "$source" "$dest"; then
        return 0  # Success
    else
        return 4  # Copy failed
    fi
}

# Handle different return codes
if ! backup_file "data.txt" "backup.txt"; then
    case $? in
        1) echo "Error: Missing parameters" ;;
        2) echo "Error: Source file not found" ;;
        3) echo "Error: Destination already exists" ;;
        4) echo "Error: Copy operation failed" ;;
    esac
fi
```

### 5. Document Your Functions
```bash
# Backup a PostgreSQL database
# Usage: backup_postgres_db <database_name> [backup_directory]
# Returns: 0 on success, 1 on failure
backup_postgres_db() {
    local db_name=$1
    local backup_dir=${2:-"/backups"}
    local timestamp=$(date +%Y%m%d_%H%M%S)
    local backup_file="$backup_dir/${db_name}_${timestamp}.sql"
    
    # Validate parameters
    if [ -z "$db_name" ]; then
        echo "Error: Database name required" >&2
        return 1
    fi
    
    # Create backup directory if it doesn't exist
    mkdir -p "$backup_dir"
    
    # Perform backup
    if pg_dump -d "$db_name" -f "$backup_file"; then
        echo "Backup successful: $backup_file"
        return 0
    else
        echo "Backup failed!" >&2
        return 1
    fi
}
```

## Common Patterns

### Menu Function
```bash
show_menu() {
    echo "=== Project Manager ==="
    echo "1. Start development server"
    echo "2. Run tests"
    echo "3. Deploy to staging"
    echo "4. Backup database"
    echo "5. Exit"
    echo -n "Choose an option: "
}

project_manager() {
    while true; do
        show_menu
        read choice
        
        case $choice in
            1) start_dev_server ;;
            2) run_tests ;;
            3) deploy_staging ;;
            4) backup_database ;;
            5) echo "Goodbye!"; break ;;
            *) echo "Invalid option. Please try again." ;;
        esac
    done
}
```

### Logging Function
```bash
log() {
    local level=$1
    shift
    local message="$@"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    
    echo "[$timestamp] [$level] $message"
}

# Usage
log "INFO" "Starting application"
log "ERROR" "Database connection failed"
log "DEBUG" "User ID: $user_id"
```

## Key Takeaways

- **Use `local`** for variables inside functions to avoid global scope issues
- **Quote variables** (`"$1"`) to handle spaces in arguments  
- **Check parameters** exist before using them
- **Use meaningful return codes** (0 = success, non-zero = error)
- **Add functions to ~/.bashrc** for permanent availability
- **Use `$@` for all parameters**, `$#` for parameter count
- **Functions can call other functions**
- **Document complex functions** with usage examples
- **Group related functions** in separate files for better organization

Functions make your shell work much more efficient by automating repetitive tasks and creating reusable utilities for your development workflow!