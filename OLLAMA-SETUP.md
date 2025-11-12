# Ollama Setup Guide - Localhost Connection

## Option 1: Current Setup (Recommended)
**File:** `docker-compose-standalone.yml`
```yaml
OLLAMA_URL: "http://host.docker.internal:11434"
extra_hosts:
  - "host.docker.internal:host-gateway"
```

### How it works:
- `host.docker.internal` resolves to your host machine's IP
- `host-gateway` enables Docker container to reach localhost services
- Container connects to Ollama running on your Raspberry Pi (port 11434)

---

## Option 2: Direct Host IP
If `host.docker.internal` doesn't work on your system:

```yaml
OLLAMA_URL: "http://192.168.1.100:11434"  # Replace with your Pi's IP
```

### Find your IP:
```bash
ip route get 1.1.1.1 | awk '{print $7}'
# or
hostname -I
```

---

## Option 3: Host Networking (Linux/Raspberry Pi)
```yaml
OLLAMA_URL: "http://localhost:11434"
network_mode: "host"
```

### Modify docker-compose-standalone.yml:
```yaml
services:
  airflow-standalone:
    # ... other config
    network_mode: "host"
    ports: []  # Remove ports section, uses host network directly
```

---

## Setup Instructions

### 1. Install Ollama on Raspberry Pi (if not already):
```bash
# Install Ollama
curl -fsSL https://ollama.ai/install.sh | sh

# Start Ollama service
sudo systemctl start ollama
sudo systemctl enable ollama

# Verify it's running
ollama list
```

### 2. Pull the required model:
```bash
# Your DAG uses llama3.2:1b
ollama pull llama3.2:1b

# Test the model
ollama run llama3.2:1b "What should I wear in rainy weather?"
```

### 3. Verify Ollama is accessible:
```bash
# Test from host
curl http://localhost:11434/api/tags

# Should return something like:
{"models":[{"name":"llama3.2:1b","model":"llama3.2:1b",...}]}
```

### 4. Start Airflow container:
```bash
docker-compose -f docker-compose-standalone.yml up -d
```

### 5. Test connection from inside container:
```bash
# Enter the container
docker exec -it airflow-weather-llm-custom-airflow-standalone-1 bash

# Test Ollama connection
curl http://host.docker.internal:11434/api/tags
```

---

## Troubleshooting

### Connection Refused:
```bash
# Check if Ollama is running
sudo systemctl status ollama

# Check if port 11434 is listening
netstat -tlnp | grep 11434
```

### Model Not Found:
```bash
# List available models
ollama list

# Pull the required model
ollama pull llama3.2:1b
```

### Permission Issues:
```bash
# Add user to ollama group (if needed)
sudo usermod -a -G ollama $USER
# Then logout and login again
```

### Firewall Issues:
```bash
# Allow port 11434 (if using firewall)
sudo ufw allow 11434
```

---

## Verification

Once everything is running, your Airflow DAG should successfully:
1. Fetch weather data from Open-Meteo API
2. Send it to Ollama at `http://host.docker.internal:11434`
3. Get clothing suggestions from `llama3.2:1b` model
4. Send Pushover notification

You can test this manually:
```bash
# Test the full workflow
curl -X POST http://localhost:11434/api/generate \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llama3.2:1b",
    "prompt": "Weather: 15°C, rainy. What should I wear?",
    "stream": false
  }'
```

This setup gives you the best of both worlds - Ollama runs directly on your Pi hardware for maximum performance, while Airflow runs in Docker for isolation! 🚀