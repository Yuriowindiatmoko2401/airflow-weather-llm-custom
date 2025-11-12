# Raspberry Pi Docker Fix

## Problem Fixed
The standalone Docker setup had two issues on Raspberry Pi:

1. **Permission Error**: `/opt/airflow/logs/scheduler` directory couldn't be created
2. **Deprecated Config**: `AIRFLOW__CORE__SQL_ALCHEMY_CONN` moved to `AIRFLOW__DATABASE__SQL_ALCHEMY_CONN`

## Solutions Applied

### 1. Fixed Configuration (`docker-compose-standalone.yml`)
```yaml
# OLD (deprecated):
AIRFLOW__CORE__SQL_ALCHEMY_CONN: sqlite:///opt/airflow/airflow.db

# NEW (correct):
AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: sqlite:///opt/airflow/airflow.db
```

### 2. Fixed Permissions
```yaml
# Use root user initially to create directories
user: "0:0"

# Pre-create log directories and set ownership
command: >
  bash -c "
    mkdir -p /opt/airflow/logs/scheduler /opt/airflow/logs/task /opt/airflow/logs/dag_processor &&
    chown -R airflow:airflow /opt/airflow &&
    airflow db init &&
    ...
  "
```

## Quick Fix Steps

```bash
# 1. Stop the broken container
docker-compose -f docker-compose-standalone.yml down

# 2. Clean up any existing volumes (optional)
docker volume rm airflow-weather-llm-custom_airflow-db

# 3. Start with fixed configuration
docker-compose -f docker-compose-standalone.yml up -d

# 4. Monitor the startup
docker logs -f airflow-weather-llm-custom-airflow-standalone-1
```

## Expected Output
You should see:
- `Creating log directories...`
- `Database initialization successful`
- `User created or already exists`
- `Starting scheduler and webserver`

## Access
- **URL**: http://localhost:8080
- **Username**: airflow
- **Password**: airflow

## Alternative: Use ARM-optimized Image
If you still have issues, try using the ARM-specific image:

```yaml
image: apache/airflow:2.10.3-python3.12-slim
```

## Resource Usage on Pi5
- **RAM**: ~300-500MB
- **CPU**: Minimal during idle
- **Storage**: ~500MB for image + data

This setup is optimized for Raspberry Pi 5 and should work without permission issues.