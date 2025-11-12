# Docker Compose Options for Airflow Weather LLM

## Current Setup (Heavy)
**File:** `docker-compose.yml`
**Services:** 8 services (postgres, redis, airflow-webserver, airflow-scheduler, airflow-worker, airflow-triggerer, airflow-init, airflow-cli) + optional flower
**Pros:**
- Full Celery executor for distributed processing
- Production-ready architecture
- Separate services for better monitoring
- Flower UI for task monitoring

**Cons:**
- High resource usage (4GB+ RAM recommended)
- Complex setup
- Overkill for simple projects

---

## Option 1: Lightweight PostgreSQL + Sequential Executor
**File:** `docker-compose-lightweight.yml`
**Services:** 2 services (postgres, airflow)
**Pros:**
- Much lighter resource usage (~1-2GB RAM)
- Simple setup
- Still uses PostgreSQL for persistence
- Single container runs both scheduler and webserver

**Cons:**
- Sequential executor (one task at a time)
- Not suitable for heavy workloads
- No distributed processing

**Usage:**
```bash
# Copy env file
cp .env-lightweight .env

# Run lightweight version
docker-compose -f docker-compose-lightweight.yml up -d

# Access: http://localhost:8080
# Username: airflow
# Password: airflow
```

---

## Option 2: Standalone with SQLite
**File:** `docker-compose-standalone.yml`
**Services:** 1 service (airflow-standalone)
**Pros:**
- Minimal resource usage (~500MB RAM)
- Simplest setup
- No external database required
- Fast startup

**Cons:**
- SQLite database (not production-ready)
- LocalExecutor (limited parallelism)
- Single point of failure

**Usage:**
```bash
# Run standalone version
docker-compose -f docker-compose-standalone.yml up -d

# Access: http://localhost:8080
# Username: airflow
# Password: airflow
```

---

## Recommendation for Your Project

For the **Airflow Weather LLM** project, I recommend **Option 1 (Lightweight PostgreSQL)** because:

1. ✅ **Perfect fit**: Your project runs simple sequential tasks anyway
2. ✅ **Resource efficient**: Reduces RAM from 4GB+ to ~1-2GB
3. ✅ **Data persistence**: PostgreSQL keeps your DAG run history
4. ✅ **Easy migration**: Can upgrade to full setup later if needed
5. ✅ **Faster development**: Quick startup and simple management

The sequential execution is actually ideal for your weather → LLM → notification pipeline since each step depends on the previous one anyway.

---

## Quick Switch Guide

```bash
# Stop current heavy setup
docker-compose down

# Start lightweight version
docker-compose -f docker-compose-lightweight.yml up -d

# Or use standalone (even lighter)
docker-compose -f docker-compose-standalone.yml up -d
```

All your DAGs, plugins, and configurations will work the same way - just with much lower resource requirements!