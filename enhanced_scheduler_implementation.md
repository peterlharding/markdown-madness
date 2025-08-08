# Enhanced EC2 Scheduler: IoT-Style Multiple Events

This design transforms your EC2 scheduler into a sophisticated scheduling system similar to Nest thermostats, supporting multiple discrete events per instance.

## 🎯 Key Improvements

### **Before (Simple)**
- One start time, one stop time per instance
- Limited flexibility
- Hard to manage complex schedules

### **After (IoT-Style)**  
- **Multiple named schedules** per instance
- **Multiple events** per schedule
- **Granular control** over individual events
- **Rich metadata** and descriptions
- **Easy enable/disable** of events or entire schedules

## 🏗️ New Data Architecture

### **Schedule Structure**
```json
{
  "schedule_id": "i-1234567890abcdef0#Weekday Work",
  "instance_id": "i-1234567890abcdef0", 
  "schedule_name": "Weekday Work",
  "description": "Standard business hours",
  "timezone": "America/New_York",
  "enabled": true,
  "events": [
    {
      "event_id": "evt_1234567890",
      "event_name": "Morning Startup", 
      "cron_expr": "0 9 * * 1-5",
      "action": "start",
      "enabled": true,
      "description": "Start work day at 9 AM"
    },
    {
      "event_id": "evt_1234567891",
      "event_name": "End of Day",
      "cron_expr": "0 17 * * 1-5", 
      "action": "stop",
      "enabled": true,
      "description": "End work day at 5 PM"
    }
  ]
}
```

## 🚀 Usage Examples

### **1. Create a Development Schedule**
```bash
curl -X POST https://your-api-gateway-url/trigger \
  -H "Content-Type: application/json" \
  -d '{
    "action": "create_schedule",
    "instance_id": "i-1234567890abcdef0",
    "schedule": {
      "schedule_name": "Development Hours",
      "description": "Standard dev work schedule",
      "timezone": "America/New_York", 
      "enabled": true,
      "events": [
        {
          "event_name": "Morning Start",
          "cron_expr": "0 9 * * 1-5",
          "action": "start",
          "enabled": true,
          "description": "Start at 9 AM EST"
        },
        {
          "event_name": "End of Day",
          "cron_expr": "0 17 * * 1-5",
          "action": "stop", 
          "enabled": true,
          "description": "Stop at 5 PM EST"
        }
      ]
    }
  }'
```

### **2. Add Weekend Maintenance Events**
```bash
curl -X POST https://your-api-gateway-url/trigger \
  -H "Content-Type: application/json" \
  -d '{
    "action": "create_schedule",
    "instance_id": "i-1234567890abcdef0",
    "schedule": {
      "schedule_name": "Weekend Maintenance",
      "description": "Automated weekend maintenance",
      "timezone": "UTC",
      "enabled": true,
      "events": [
        {
          "event_name": "Saturday Startup",
          "cron_expr": "0 2 * * 6",
          "action": "start",
          "enabled": true,
          "description": "Start for maintenance"
        },
        {
          "event_name": "Update Reboot", 
          "cron_expr": "0 3 * * 6",
          "action": "reboot",
          "enabled": true,
          "description": "Reboot for updates"
        },
        {
          "event_name": "Maintenance Complete",
          "cron_expr": "0 4 * * 6", 
          "action": "stop",
          "enabled": true,
          "description": "Shutdown after maintenance"
        }
      ]
    }
  }'
```

### **3. Add Single Event to Existing Schedule**
```bash
curl -X POST https://your-api-gateway-url/trigger \
  -H "Content-Type: application/json" \
  -d '{
    "action": "create_event",
    "schedule_id": "i-1234567890abcdef0#Development Hours",
    "event": {
      "event_name": "Lunch Break",
      "cron_expr": "0 12 * * 1-5",
      "action": "stop",
      "enabled": false,
      "description": "Optional lunch shutdown"
    }
  }'
```

### **4. Toggle Event On/Off**
```bash
curl -X POST https://your-api-gateway-url/trigger \
  -H "Content-Type: application/json" \
  -d '{
    "action": "update_event",
    "schedule_id": "i-1234567890abcdef0#Development Hours", 
    "event_id": "evt_1234567890",
    "event": {
      "enabled": false
    }
  }'
```

### **5. List All Schedules for Instance**
```bash
curl -X POST https://your-api-gateway-url/trigger \
  -H "Content-Type: application/json" \
  -d '{
    "action": "list_schedules",
    "instance_id": "i-1234567890abcdef0"
  }'
```

## 🎛️ Advanced Scheduling Scenarios

### **Scenario 1: Development Team with Flexible Hours**
```json
{
  "schedule_name": "Flexible Dev Hours",
  "events": [
    {
      "event_name": "Early Bird Option",
      "cron_expr": "0 7 * * 1-5",
      "action": "start",
      "enabled": false,
      "description": "Optional early start"
    },
    {
      "event_name": "Standard Start", 
      "cron_expr": "0 9 * * 1-5",
      "action": "start",
      "enabled": true,
      "description": "Standard 9 AM start"
    },
    {
      "event_name": "Lunch Break",
      "cron_expr": "0 12 * * 1-5",
      "action": "stop",
      "enabled": false,
      "description": "Optional lunch shutdown"
    },
    {
      "event_name": "Afternoon Resume",
      "cron_expr": "0 13 * * 1-5", 
      "action": "start",
      "enabled": false,
      "description": "Resume after lunch"
    },
    {
      "event_name": "Standard End",
      "cron_expr": "0 17 * * 1-5",
      "action": "stop",
      "enabled": true,
      "description": "Standard 5 PM end"
    },
    {
      "event_name": "Late Work Option",
      "cron_expr": "0 20 * * 1-5",
      "action": "stop", 
      "enabled": false,
      "description": "Optional late work cutoff"
    }
  ]
}
```

### **Scenario 2: Multi-Environment Instance**
```json
{
  "schedule_name": "Multi-Environment",
  "events": [
    {
      "event_name": "Dev Hours Start",
      "cron_expr": "0 8 * * 1-5",
      "action": "start",
      "enabled": true
    },
    {
      "event_name": "Testing Window",
      "cron_expr": "0 14 * * 1-5", 
      "action": "reboot",
      "enabled": true,
      "description": "Daily testing restart"
    },
    {
      "event_name": "Dev Hours End",
      "cron_expr": "0 18 * * 1-5",
      "action": "stop",
      "enabled": true
    },
    {
      "event_name": "Weekend Staging",
      "cron_expr": "0 10 * * 6",
      "action": "start", 
      "enabled": false,
      "description": "Weekend staging environment"
    }
  ]
}
```

## 🔄 Migration from Current System

### **Step 1: Export Current Schedules**
```bash
# Export existing schedules
aws dynamodb scan \
  --table-name ec2-schedules \
  --region ap-southeast-2 \
  --query 'Items[*]' > current_schedules.json
```

### **Step 2: Convert to New Format**
Convert your existing schedules to the new multi-event format:

```bash
# For each existing schedule, create equivalent new schedule
curl -X POST https://your-api-gateway-url/trigger \
  -H "Content-Type: application/json" \
  -d '{
    "action": "create_schedule",
    "instance_id": "i-030fc8717a89a545a",
    "schedule": {
      "schedule_name": "Migrated Schedule",
      "description": "Converted from simple start/stop",
      "timezone": "UTC",
      "enabled": true,
      "events": [
        {
          "event_name": "Daily Start",
          "cron_expr": "0 8 * * 1-5",
          "action": "start",
          "enabled": true,
          "description": "Migrated start time"
        },
        {
          "event_name": "Daily Stop", 
          "cron_expr": "0 18 * * 1-5",
          "action": "stop",
          "enabled": true,
          "description": "Migrated stop time"
        }
      ]
    }
  }'
```

## 🆕 New API Actions

| Action | Description | Example |
|--------|-------------|---------|
| `create_schedule` | Create named schedule with multiple events | Development Hours |
| `update_schedule` | Update schedule metadata | Change timezone |
| `list_schedules` | List all schedules for instance | See all schedules |
| `delete_schedule` | Remove entire schedule | Remove old schedule |
| `create_event` | Add event to existing schedule | Add lunch break |
| `update_event` | Modify specific event | Enable/disable event |
| `delete_event` | Remove specific event | Remove weekend work |
| `list_events` | List events in schedule | See all events |

## 🎯 Benefits of New Design

### **Flexibility**
- **Mix and match**: Combine different types of events
- **Easy experimentation**: Enable/disable events without deletion
- **Seasonal adjustments**: Different schedules for different times of year

### **Organization**
- **Named schedules**: "Development", "Maintenance", "Testing"
- **Rich descriptions**: Clear documentation of purpose
- **Grouped events**: Related events stay together

### **Granular Control**
- **Individual event control**: Enable/disable specific events
- **Event metadata**: Track when events were created/modified  
- **Action variety**: Start, stop, reboot actions

### **User Experience**
- **IoT-like interface**: Similar to smart home scheduling
- **Visual organization**: Events grouped by schedule
- **Easy modifications**: Change one event without affecting others

## 🚀 Implementation Plan

1. **Phase 1**: Implement new data structures and basic CRUD operations
2. **Phase 2**: Add enhanced scheduling logic with timezone support
3. **Phase 3**: Build migration tools from current system
4. **Phase 4**: Add advanced features (templates, bulk operations)
5. **Phase 5**: Create web UI for visual schedule management

This enhanced design provides the flexibility and control of modern IoT scheduling systems while maintaining the robust automation capabilities of your current EC2 scheduler!