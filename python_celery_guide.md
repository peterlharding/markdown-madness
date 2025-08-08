# Python Celery Comprehensive Guide

## Overview

**Celery** is a distributed task queue system for Python that enables asynchronous processing of time-consuming tasks in the background, separate from your main web application. It allows you to offload heavy operations to background workers, providing better user experience and application scalability.

## What Celery Does

### Asynchronous Task Processing

Instead of making users wait for slow operations, Celery moves them to background workers:

```python
# Without Celery - User waits
@app.post("/send-email")
def send_email(email_data):
    send_email_to_user(email_data)  # Takes 5 seconds
    return {"message": "Email sent"}  # User waits 5 seconds

# With Celery - Instant response
@app.post("/send-email") 
def send_email(email_data):
    send_email_task.delay(email_data)  # Queued instantly
    return {"message": "Email queued"}  # User gets instant response

@celery_app.task
def send_email_task(email_data):
    send_email_to_user(email_data)  # Runs in background worker
```

## Common Use Cases

### 1. Email Sending

```python
@celery_app.task
def send_welcome_email(user_email, user_name):
    """Send welcome email without blocking user registration"""
    send_email(
        to=user_email,
        subject="Welcome!",
        template="welcome.html",
        context={"name": user_name}
    )

# Usage in your API
@app.post("/register")
def register_user(user_data):
    user = create_user(user_data)
    send_welcome_email.delay(user.email, user.name)  # Background
    return {"message": "User created"}
```

### 2. Data Processing

```python
@celery_app.task
def process_large_dataset(file_path):
    """Process large CSV files without timing out"""
    df = pd.read_csv(file_path)
    # Heavy processing that takes 10+ minutes
    processed_data = complex_analysis(df)
    save_results(processed_data)
    return "Processing complete"

# Usage
@app.post("/upload-data")
def upload_data(file):
    file_path = save_uploaded_file(file)
    task = process_large_dataset.delay(file_path)
    return {"task_id": task.id, "status": "processing"}
```

### 3. Report Generation

```python
@celery_app.task
def generate_monthly_report(user_id, month, year):
    """Generate heavy reports in background"""
    user_data = get_user_data(user_id)
    report_data = calculate_monthly_metrics(user_data, month, year)
    pdf_file = create_pdf_report(report_data)
    
    # Email the report
    send_email_with_attachment(user_data.email, pdf_file)
    return f"Report sent to {user_data.email}"
```

### 4. Scheduled Tasks (Cron-like)

```python
from celery.schedules import crontab

# Run every day at midnight
@celery_app.task
def daily_backup():
    """Backup database daily"""
    backup_database()
    cleanup_old_backups()

# Run every Monday at 9 AM
@celery_app.task  
def weekly_summary():
    """Send weekly summary emails"""
    users = get_all_active_users()
    for user in users:
        send_weekly_summary.delay(user.id)

# Configure in Celery beat schedule
celery_app.conf.beat_schedule = {
    'daily-backup': {
        'task': 'daily_backup',
        'schedule': crontab(hour=0, minute=0),
    },
    'weekly-summary': {
        'task': 'weekly_summary', 
        'schedule': crontab(hour=9, minute=0, day_of_week=1),
    },
}
```

### 5. Image/Video Processing

```python
@celery_app.task
def resize_image(image_path, sizes):
    """Resize images to multiple sizes"""
    from PIL import Image
    
    image = Image.open(image_path)
    resized_paths = []
    
    for size in sizes:
        resized = image.resize(size)
        path = f"{image_path}_{size[0]}x{size[1]}.jpg"
        resized.save(path)
        resized_paths.append(path)
    
    return resized_paths

@celery_app.task
def process_video(video_path):
    """Convert video to different formats"""
    import ffmpeg
    
    # Convert to MP4
    mp4_path = video_path.replace('.mov', '.mp4')
    ffmpeg.input(video_path).output(mp4_path).run()
    
    # Generate thumbnail
    thumbnail_path = video_path.replace('.mov', '_thumb.jpg')
    ffmpeg.input(video_path, ss=1).output(thumbnail_path, vframes=1).run()
    
    return {"mp4": mp4_path, "thumbnail": thumbnail_path}
```

## Celery Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Web App       │    │   Message       │    │   Celery        │
│   (FastAPI)     │───▶│   Broker        │───▶│   Workers       │
│                 │    │   (Redis)       │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                 │
                                 ▼
                       ┌─────────────────┐
                       │   Result        │
                       │   Backend       │ 
                       │   (Redis/DB)    │
                       └─────────────────┘
```

### Components:

- **Web App**: Your FastAPI/Django application that queues tasks
- **Message Broker**: Redis or RabbitMQ that holds the task queue
- **Workers**: Separate processes that execute tasks
- **Result Backend**: Storage for task results and status

## Installation and Setup

### Installation

```bash
# With Redis broker (recommended)
pip install celery[redis]

# With RabbitMQ broker
pip install celery[rabbitmq]

# With additional dependencies
pip install celery[redis,auth,msgpack]
```

### Basic Celery Configuration

```python
# celery_app.py
from celery import Celery

# Create Celery instance
celery_app = Celery(
    "myapp",
    broker="redis://localhost:6379/0",      # Message queue
    backend="redis://localhost:6379/0",     # Store results
    include=["tasks"]  # Import task modules
)

# Configuration
celery_app.conf.update(
    task_serializer="json",
    accept_content=["json"],
    result_serializer="json",
    timezone="UTC",
    enable_utc=True,
    task_track_started=True,
    task_time_limit=30 * 60,  # 30 minutes
    task_soft_time_limit=25 * 60,  # 25 minutes
    worker_prefetch_multiplier=1,
    task_acks_late=True,
)
```

### Environment-based Configuration

```python
# config.py
import os

class CeleryConfig:
    broker_url = os.getenv('CELERY_BROKER_URL', 'redis://localhost:6379/0')
    result_backend = os.getenv('CELERY_RESULT_BACKEND', 'redis://localhost:6379/0')
    
    task_serializer = 'json'
    result_serializer = 'json'
    accept_content = ['json']
    
    timezone = 'UTC'
    enable_utc = True
    
    # Task routing
    task_routes = {
        'tasks.high_priority_task': {'queue': 'high'},
        'tasks.low_priority_task': {'queue': 'low'},
    }
    
    # Beat schedule for periodic tasks
    beat_schedule = {
        'cleanup-every-hour': {
            'task': 'tasks.cleanup_temp_files',
            'schedule': 3600.0,  # Every hour
        },
    }

# Apply configuration
celery_app.config_from_object(CeleryConfig)
```

## Creating Tasks

### Basic Tasks

```python
# tasks.py
from celery_app import celery_app
import time

@celery_app.task
def simple_task(x, y):
    """Simple background task"""
    time.sleep(10)  # Simulate work
    return x + y

@celery_app.task
def send_notification(user_id, message):
    """Send notification to user"""
    user = get_user(user_id)
    send_email(user.email, message)
    send_sms(user.phone, message)
    return f"Notification sent to {user.email}"
```

### Tasks with Progress Tracking

```python
@celery_app.task(bind=True)
def long_running_task(self, items):
    """Task with progress tracking"""
    total = len(items)
    
    for i, item in enumerate(items):
        # Do work
        process_item(item)
        
        # Update progress
        self.update_state(
            state="PROGRESS",
            meta={
                "current": i + 1, 
                "total": total,
                "status": f"Processing item {i + 1} of {total}"
            }
        )
    
    return {"status": "complete", "result": f"Processed {total} items"}
```

### Tasks with Retries

```python
@celery_app.task(bind=True, autoretry_for=(Exception,), retry_kwargs={'max_retries': 3, 'countdown': 60})
def unreliable_task(self, data):
    """Task with automatic retries"""
    try:
        result = call_external_api(data)
        return result
    except APIException as exc:
        # Custom retry logic
        if self.request.retries < 2:
            # Exponential backoff
            raise self.retry(countdown=60 * (2 ** self.request.retries), exc=exc)
        else:
            # Final attempt failed
            log_error(f"Task failed permanently: {exc}")
            return {"error": str(exc), "status": "failed"}
```

### Task Error Handling

```python
@celery_app.task(bind=True)
def robust_task(self, data):
    """Task with comprehensive error handling"""
    try:
        # Validate input
        if not validate_data(data):
            raise ValueError("Invalid input data")
        
        # Process data
        result = process_data(data)
        
        # Validate result
        if not validate_result(result):
            raise ValueError("Invalid result")
        
        return result
        
    except ValueError as exc:
        # Don't retry for validation errors
        self.update_state(state='FAILURE', meta={'error': str(exc)})
        raise
    except Exception as exc:
        # Retry for other errors
        if self.request.retries < 3:
            raise self.retry(countdown=60, exc=exc)
        else:
            self.update_state(state='FAILURE', meta={'error': str(exc)})
            raise
```

## Using Tasks in Web Applications

### FastAPI Integration

```python
# main.py
from fastapi import FastAPI, HTTPException
from tasks import simple_task, long_running_task
from celery_app import celery_app

app = FastAPI()

@app.post("/start-task")
def start_background_task(x: int, y: int):
    """Start a background task"""
    task = simple_task.delay(x, y)
    return {"task_id": task.id, "status": "started"}

@app.post("/process-data")
def process_data_endpoint(items: list):
    """Start data processing task"""
    task = long_running_task.delay(items)
    return {"task_id": task.id, "status": "processing"}

@app.get("/task-status/{task_id}")
def get_task_status(task_id: str):
    """Check task status"""
    task = celery_app.AsyncResult(task_id)
    
    if task.state == "PENDING":
        return {"state": "PENDING", "status": "Task is waiting..."}
    elif task.state == "PROGRESS":
        return {
            "state": "PROGRESS",
            "current": task.info.get("current", 0),
            "total": task.info.get("total", 1),
            "status": task.info.get("status", "Processing...")
        }
    elif task.state == "SUCCESS":
        return {
            "state": "SUCCESS",
            "result": task.result,
            "status": "Task completed successfully"
        }
    else:  # FAILURE
        return {
            "state": "FAILURE",
            "error": str(task.info),
            "status": "Task failed"
        }

@app.delete("/task/{task_id}")
def cancel_task(task_id: str):
    """Cancel a running task"""
    celery_app.control.revoke(task_id, terminate=True)
    return {"message": f"Task {task_id} cancelled"}
```

### Django Integration

```python
# Django views.py
from django.http import JsonResponse
from django.views import View
from .tasks import process_upload_task

class ProcessUploadView(View):
    def post(self, request):
        file_data = request.FILES['file']
        
        # Save file and start processing
        file_path = save_uploaded_file(file_data)
        task = process_upload_task.delay(file_path)
        
        return JsonResponse({
            'task_id': task.id,
            'status': 'processing'
        })
```

## Running Celery

### Starting Workers

```bash
# Basic worker
celery -A celery_app worker --loglevel=info

# Worker with specific concurrency
celery -A celery_app worker --loglevel=info --concurrency=4

# Worker for specific queues
celery -A celery_app worker -Q high_priority,low_priority --loglevel=info

# Worker with custom pool
celery -A celery_app worker --pool=gevent --concurrency=100

# Worker with autoscaling
celery -A celery_app worker --autoscale=10,2
```

### Starting Beat Scheduler

```bash
# Start scheduler for periodic tasks
celery -A celery_app beat --loglevel=info

# Store schedule in database (requires django-celery-beat)
celery -A celery_app beat --scheduler django_celery_beat.schedulers:DatabaseScheduler

# Combined worker and beat (development only)
celery -A celery_app worker --beat --loglevel=info
```

### Monitoring

```bash
# Real-time monitoring with Flower
pip install flower
celery -A celery_app flower

# Access web interface at http://localhost:5555

# Command-line monitoring
celery -A celery_app status
celery -A celery_app inspect active
celery -A celery_app inspect stats
```

## Advanced Features

### Task Routing and Queues

```python
# Define multiple queues
@celery_app.task(queue="high_priority")
def urgent_task():
    return "This runs immediately"

@celery_app.task(queue="low_priority") 
def bulk_task():
    return "This can wait"

@celery_app.task(queue="email_queue")
def send_email_task():
    return "Email specific queue"

# Configure routing
celery_app.conf.task_routes = {
    'tasks.urgent_task': {'queue': 'high_priority'},
    'tasks.bulk_task': {'queue': 'low_priority'},
    'tasks.send_*': {'queue': 'email_queue'},
}

# Start workers for specific queues
# celery -A celery_app worker -Q high_priority
# celery -A celery_app worker -Q low_priority,email_queue
```

### Task Chaining and Workflows

```python
from celery import chain, group, chord

# Chain tasks (run in sequence)
@celery_app.task
def step_one(data):
    return process_step_one(data)

@celery_app.task
def step_two(result):
    return process_step_two(result)

@celery_app.task
def step_three(result):
    return process_step_three(result)

# Execute chain
workflow = chain(step_one.s("input_data"), step_two.s(), step_three.s())
result = workflow.apply_async()

# Group tasks (run in parallel)
parallel_jobs = group(
    process_file.s(file1),
    process_file.s(file2),
    process_file.s(file3)
)
result = parallel_jobs.apply_async()

# Chord (parallel tasks + callback)
callback_task = process_results.s()
chord_result = chord(parallel_jobs)(callback_task)
```

### Custom Task Classes

```python
from celery import Task

class DatabaseTask(Task):
    """Custom task base class with database connection"""
    _db = None
    
    @property
    def db(self):
        if self._db is None:
            self._db = create_database_connection()
        return self._db

@celery_app.task(base=DatabaseTask, bind=True)
def database_intensive_task(self, query):
    """Task that needs database connection"""
    result = self.db.execute(query)
    return result.fetchall()
```

### Rate Limiting

```python
# Limit to 10 tasks per minute
@celery_app.task(rate_limit='10/m')
def api_call_task(url):
    """Rate-limited API calls"""
    response = requests.get(url)
    return response.json()

# Limit to 100 tasks per hour
@celery_app.task(rate_limit='100/h')
def heavy_computation():
    """CPU-intensive task"""
    return perform_calculation()
```

## Real-World Examples

### Analytics Application (Superset-like)

```python
# Analytics tasks for dashboard application
@celery_app.task(bind=True)
def execute_sql_query(self, query_id, sql, user_id):
    """Execute long-running SQL queries asynchronously"""
    try:
        # Update status
        self.update_state(
            state="RUNNING", 
            meta={"status": "Executing query...", "query_id": query_id}
        )
        
        # Execute query (could take minutes)
        connection = get_database_connection()
        result = connection.execute(sql)
        
        # Process and cache results
        processed_result = process_query_result(result)
        cache_query_result(query_id, processed_result)
        
        # Notify user
        notify_user_query_complete(user_id, query_id)
        
        return {
            "status": "success", 
            "rows": len(processed_result), 
            "query_id": query_id,
            "execution_time": time.time() - start_time
        }
        
    except Exception as exc:
        log_error(f"Query {query_id} failed: {exc}")
        notify_user_query_failed(user_id, query_id, str(exc))
        raise

@celery_app.task
def generate_dashboard_export(dashboard_id, user_email, format="pdf"):
    """Generate and email dashboard exports"""
    try:
        # Get dashboard data
        dashboard = get_dashboard_data(dashboard_id)
        
        # Generate export
        if format == "pdf":
            file_path = create_dashboard_pdf(dashboard)
        elif format == "excel":
            file_path = create_dashboard_excel(dashboard)
        else:
            raise ValueError(f"Unsupported format: {format}")
        
        # Email the export
        send_email_with_attachment(
            user_email, 
            file_path, 
            f"Dashboard Export - {dashboard['title']}"
        )
        
        # Cleanup temporary file
        os.remove(file_path)
        
        return f"Dashboard export sent to {user_email}"
        
    except Exception as exc:
        log_error(f"Dashboard export failed: {exc}")
        send_error_notification(user_email, f"Export failed: {exc}")
        raise

@celery_app.task
def refresh_dataset_cache(dataset_id):
    """Refresh cached dataset in background"""
    dataset = get_dataset(dataset_id)
    
    # Re-query data source
    fresh_data = query_data_source(dataset.connection, dataset.query)
    
    # Update cache
    update_dataset_cache(dataset_id, fresh_data)
    
    # Update metadata
    update_dataset_metadata(dataset_id, {
        'last_refresh': datetime.utcnow(),
        'row_count': len(fresh_data)
    })
    
    return f"Dataset {dataset_id} refreshed with {len(fresh_data)} rows"
```

### E-commerce Application

```python
@celery_app.task
def process_order(order_id):
    """Process order workflow"""
    order = get_order(order_id)
    
    # Validate inventory
    if not check_inventory(order.items):
        raise ValueError("Insufficient inventory")
    
    # Process payment
    payment_result = process_payment(order.payment_info)
    if not payment_result.success:
        raise ValueError("Payment failed")
    
    # Reserve inventory
    reserve_inventory(order.items)
    
    # Send confirmation email
    send_order_confirmation.delay(order_id)
    
    # Schedule fulfillment
    schedule_fulfillment.delay(order_id)
    
    return f"Order {order_id} processed successfully"

@celery_app.task
def send_order_confirmation(order_id):
    """Send order confirmation email"""
    order = get_order(order_id)
    send_email(
        to=order.customer_email,
        subject="Order Confirmation",
        template="order_confirmation.html",
        context={"order": order}
    )

@celery_app.task
def generate_daily_sales_report():
    """Generate daily sales report"""
    today = datetime.now().date()
    sales_data = get_sales_data(today)
    
    report = create_sales_report(sales_data)
    
    # Email to management
    management_emails = get_management_emails()
    for email in management_emails:
        send_email_with_attachment(email, report, "Daily Sales Report")
```

## Production Deployment

### Docker Configuration

```dockerfile
# Dockerfile for Celery worker
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Default command for worker
CMD ["celery", "-A", "celery_app", "worker", "--loglevel=info"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  web:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - redis
    environment:
      - CELERY_BROKER_URL=redis://redis:6379/0
      - CELERY_RESULT_BACKEND=redis://redis:6379/0

  worker:
    build: .
    command: celery -A celery_app worker --loglevel=info --concurrency=4
    depends_on:
      - redis
    environment:
      - CELERY_BROKER_URL=redis://redis:6379/0
      - CELERY_RESULT_BACKEND=redis://redis:6379/0

  beat:
    build: .
    command: celery -A celery_app beat --loglevel=info
    depends_on:
      - redis
    environment:
      - CELERY_BROKER_URL=redis://redis:6379/0
      - CELERY_RESULT_BACKEND=redis://redis:6379/0

  flower:
    build: .
    command: celery -A celery_app flower
    ports:
      - "5555:5555"
    depends_on:
      - redis
    environment:
      - CELERY_BROKER_URL=redis://redis:6379/0
      - CELERY_RESULT_BACKEND=redis://redis:6379/0
```

### Systemd Service Configuration

```ini
# /etc/systemd/system/celery-worker.service
[Unit]
Description=Celery Worker
After=network.target

[Service]
Type=forking
User=celery
Group=celery
WorkingDirectory=/opt/myapp
Environment=CELERY_BROKER_URL=redis://localhost:6379/0
ExecStart=/opt/myapp/venv/bin/celery -A celery_app worker --loglevel=info --pidfile=/var/run/celery/worker.pid --logfile=/var/log/celery/worker.log
ExecReload=/bin/kill -HUP $MAINPID
KillMode=mixed
Restart=always

[Install]
WantedBy=multi-user.target
```

### Monitoring and Logging

```python
# Enhanced logging configuration
import logging
from celery.signals import task_prerun, task_postrun, task_failure

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

@task_prerun.connect
def task_prerun_handler(sender=None, task_id=None, task=None, args=None, kwargs=None, **kwds):
    logging.info(f'Task {task_id} ({task.name}) started with args: {args}, kwargs: {kwargs}')

@task_postrun.connect
def task_postrun_handler(sender=None, task_id=None, task=None, args=None, kwargs=None, retval=None, state=None, **kwds):
    logging.info(f'Task {task_id} ({task.name}) completed with state: {state}')

@task_failure.connect
def task_failure_handler(sender=None, task_id=None, exception=None, traceback=None, einfo=None, **kwds):
    logging.error(f'Task {task_id} failed: {exception}', exc_info=einfo)
```

## Best Practices

### Task Design
- Keep tasks idempotent (safe to run multiple times)
- Use small, focused tasks rather than large monolithic ones
- Implement proper error handling and retries
- Set appropriate timeouts for tasks
- Use progress tracking for long-running tasks

### Performance
- Use appropriate serialization (JSON is recommended)
- Implement result expiration to save memory
- Use task routing for different priorities
- Monitor queue lengths and worker utilization
- Scale workers based on workload

### Security
- Validate all task inputs
- Use secure broker connections (Redis AUTH, SSL)
- Limit task execution time
- Implement proper access controls
- Sanitize task results

### Monitoring
- Use Flower for real-time monitoring
- Implement custom metrics collection
- Set up alerting for failed tasks
- Monitor resource usage
- Log task execution times

## When to Use Celery

### ✅ Use Celery when:
- Tasks take more than a few seconds to complete
- You need to send emails, generate reports, or process files
- Building a web application that needs background processing
- You need scheduled/periodic tasks
- You want to scale task processing independently
- You need task retry and failure handling
- Processing large datasets or calling external APIs

### ❌ Don't use Celery when:
- All operations are fast (< 1 second)
- Building simple scripts or CLI tools
- You need immediate results from every operation
- Adding Redis/RabbitMQ complexity isn't justified
- You have very simple, single-user applications
- Memory and resource constraints are severe

## Troubleshooting Common Issues

### Tasks Not Executing
```bash
# Check if workers are running
celery -A celery_app inspect active

# Check worker registration
celery -A celery_app inspect registered

# Check queue status
celery -A celery_app inspect reserved
```

### Memory Issues
```python
# Configure worker memory limits
celery_app.conf.worker_max_memory_per_child = 200000  # 200MB

# Restart workers after N tasks
celery_app.conf.worker_max_tasks_per_child = 1000
```

### Task Debugging
```python
# Enable task result tracking
celery_app.conf.task_track_started = True

# Add detailed logging
import logging
logging.getLogger('celery').setLevel(logging.DEBUG)
```

## Resources and Learning

- **Official Documentation**: [docs.celeryproject.org](https://docs.celeryproject.org/)
- **Flower Monitoring**: [flower.readthedocs.io](https://flower.readthedocs.io/)
- **Redis Configuration**: [redis.io/documentation](https://redis.io/documentation)
- **Production Deployment**: [celery production guide](https://docs.celeryproject.org/en/stable/userguide/daemonizing.html)

Celery is an essential tool for building scalable Python web applications that need to handle time-consuming operations without blocking user interactions. It provides the foundation for robust background processing, scheduled tasks, and distributed computing in production systems.