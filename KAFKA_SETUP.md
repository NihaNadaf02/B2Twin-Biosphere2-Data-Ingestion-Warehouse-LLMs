# Kafka Integration for Biosphere 2 Pipeline

## 📋 Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Quick Start](#quick-start)
- [For Pipeline Operators](#for-pipeline-operators)
- [For LLM Team](#for-llm-team)
- [For Omniverse Team](#for-omniverse-team)
- [Troubleshooting](#troubleshooting)

---

## Overview

The Biosphere 2 pipeline now streams cleaned sensor data in real-time via Apache Kafka. This enables multiple teams to consume the same data simultaneously for different purposes (LLM analysis, 3D visualization, analytics, etc.).

### What Changed?
- ✅ **Before**: CSV files generated after transformation
- ✅ **Now**: CSV files **AND** real-time Kafka streaming

### Benefits
- 🚀 **Real-time**: Data available immediately after transformation
- 🔄 **Decoupled**: Teams work independently without affecting each other
- 📦 **Reliable**: Kafka ensures no data loss
- ⏪ **Replay**: Can reprocess historical data anytime
- 📈 **Scalable**: Handles high data volumes effortlessly

---

## Architecture

```
Oracle DB → Extract → MySQL → Transform → Kafka Topics
                                               ↓
                                   ┌───────────┼───────────┐
                                   ↓           ↓           ↓
                              LLM Team   Omniverse   Analytics
                           (Analysis)   (3D Viz)    (Dashboards)
```

### Kafka Topics

| Topic | Description | Data Type |
|-------|-------------|-----------|
| `biosphere.rainforest.type1` | Consistent sensors (e.g., temp) | Single timestamp |
| `biosphere.rainforest.type2` | Consistent sensors (e.g., humidity) | Single timestamp |
| `biosphere.rainforest.less50` | Small tables (<50 rows) | Multiple timestamps |
| `biosphere.rainforest.between50and100` | Medium tables (50-100 rows) | Multiple timestamps |
| `biosphere.rainforest.other` | Variable sensors | Multiple timestamps |

### Message Format

#### Type1/Type2 (Single Timestamp)
```json
{
  "event_id": "550e8400-e29b-41d4-a716-446655440000",
  "unique_id": 12345,
  "timestamp": "2025-12-02T10:30:00.000Z",
  "category": "type1",
  "sensors": {
    "ahur_air_temperature": 23.5,
    "ahur_relative_humidity": 65.2,
    "ahur_co2_concentration": 410.8
  },
  "metadata": {
    "processed_at": "2025-12-02T10:35:00.000Z",
    "pipeline_version": "1.0",
    "table_count": 3,
    "source_tables": ["table1", "table2", "table3"]
  }
}
```

#### Other Categories (Multiple Timestamps)
```json
{
  "event_id": "660e8400-e29b-41d4-a716-446655440001",
  "unique_id": 12346,
  "category": "other",
  "sensors": {
    "ahur_sensor_1": {
      "value": 23.5,
      "timestamp": "2025-12-02T10:30:00.000Z"
    },
    "ahur_sensor_2": {
      "value": 65.2,
      "timestamp": "2025-12-02T10:30:15.000Z"
    }
  },
  "metadata": {
    "processed_at": "2025-12-02T10:35:00.000Z",
    "pipeline_version": "1.0",
    "table_count": 2
  }
}
```

---

## Quick Start

### Prerequisites
- ✅ Docker Desktop installed and running
- ✅ Python 3.11+ with virtual environment
- ✅ `kafka-python` library installed

### Start Kafka

```powershell
# Start Kafka containers
docker-compose up -d

# Verify containers are running
docker ps

# Should see:
# - biosphere-kafka (port 9092)
# - biosphere-zookeeper (port 2181)
```

### Stop Kafka

```powershell
# Stop containers
docker-compose down

# Stop and remove all data
docker-compose down -v
```

### View Kafka Topics

```powershell
docker exec biosphere-kafka kafka-topics --list --bootstrap-server localhost:9092
```

---

## For Pipeline Operators

### Running the Pipeline with Kafka

The pipeline automatically publishes to Kafka when enabled in config:

```python
# In config.py
KAFKA_ENABLE_PRODUCER = True  # Enable Kafka publishing
KAFKA_ENABLE_CONSUMERS = True  # Enable consumers
```

### Run Pipeline

```powershell
# Activate virtual environment
.\venv\Scripts\Activate.ps1

# Run pipeline (Kafka publishes automatically)
python src/biosphere_pipeline.py
```

### Monitor Kafka Messages

```powershell
# View live messages (simple viewer)
python src/streaming/consumers/simple_consumer.py
```

### Disable Kafka (Optional)

If you want to run without Kafka:

```python
# In config.py
KAFKA_ENABLE_PRODUCER = False
```

Pipeline will work normally, just won't publish to Kafka.

---

## For LLM Team

### Connection Information

**Kafka Broker:** `localhost:9092`

**Topics:**
- `biosphere.rainforest.type1`
- `biosphere.rainforest.type2`
- `biosphere.rainforest.less50`
- `biosphere.rainforest.between50and100`
- `biosphere.rainforest.other`

### Install Library

```bash
pip install kafka-python
```

### Example Consumer (Python)

```python
from kafka import KafkaConsumer
import json

# Create consumer
consumer = KafkaConsumer(
    'biosphere.rainforest.type1',  # Choose your topic(s)
    bootstrap_servers='localhost:9092',
    group_id='llm_analysis_team',
    auto_offset_reset='latest',
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)

print("✅ Connected! Listening for sensor data...")

# Process messages
for message in consumer:
    data = message.value
    
    # Extract sensor data
    sensors = data['sensors']
    timestamp = data['timestamp']
    
    # Build LLM prompt
    prompt = f"""
    Biosphere 2 Environmental Data:
    Timestamp: {timestamp}
    Sensors: {sensors}
    
    Analyze and provide insights...
    """
    
    # Send to LLM (OpenAI, Claude, etc.)
    # response = openai.chat.completions.create(...)
    print(f"📊 Received data: {sensors}")
```

### Template Consumer

We've provided a template consumer you can customize:

```powershell
# Run template LLM consumer
python src/streaming/consumers/llm_consumer.py
```

Edit `src/streaming/consumers/llm_consumer.py` to add your LLM API calls!

### Consumer Options

**Subscribe to Specific Topics:**
```python
consumer = KafkaConsumer(
    'biosphere.rainforest.type1',
    'biosphere.rainforest.type2',
    bootstrap_servers='localhost:9092'
)
```

**Subscribe to All Topics:**
```python
consumer = KafkaConsumer(
    'biosphere.rainforest.type1',
    'biosphere.rainforest.type2',
    'biosphere.rainforest.less50',
    'biosphere.rainforest.between50and100',
    'biosphere.rainforest.other',
    bootstrap_servers='localhost:9092'
)
```

**Start from Beginning (Historical Data):**
```python
consumer = KafkaConsumer(
    'biosphere.rainforest.type1',
    bootstrap_servers='localhost:9092',
    auto_offset_reset='earliest'  # Start from oldest message
)
```

---

## For Omniverse Team

Same setup as LLM team! Just connect to the topics you need and process the JSON messages for 3D visualization.

### Example: Update Avatar

```python
from kafka import KafkaConsumer
import json

consumer = KafkaConsumer(
    'biosphere.rainforest.type1',
    bootstrap_servers='localhost:9092',
    group_id='omniverse_team',
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)

for message in consumer:
    data = message.value
    sensors = data['sensors']
    
    # Update Omniverse avatar
    omniverse_client.update_avatar(
        temperature=sensors.get('ahur_air_temperature'),
        humidity=sensors.get('ahur_relative_humidity')
    )
```

---

## Troubleshooting

### Kafka Not Starting

**Check Docker Desktop is running:**
```powershell
docker --version
docker ps
```

**Restart containers:**
```powershell
docker-compose down
docker-compose up -d
```

### Consumer Not Receiving Messages

**Check topic exists:**
```powershell
docker exec biosphere-kafka kafka-topics --list --bootstrap-server localhost:9092
```

**Check for messages in topic:**
```powershell
docker exec biosphere-kafka kafka-console-consumer --bootstrap-server localhost:9092 --topic biosphere.rainforest.type1 --from-beginning --max-messages 1
```

### Connection Refused

**Ensure Kafka is listening on port 9092:**
```powershell
docker ps
# Should show: 0.0.0.0:9092->9092/tcp
```

**Check firewall isn't blocking port 9092**

### Messages Not Being Published

**Check config:**
```python
# In config.py
KAFKA_ENABLE_PRODUCER = True  # Should be True
```

**Check pipeline logs:**
```
✅ Kafka producer initialized
📤 Publishing X records to topic...
```

---

## Advanced Topics

### Change Kafka Configuration

Edit `docker-compose.yml` to change:
- Port numbers
- Message retention (default: 7 days)
- Memory limits
- Replication settings

### Add More Topics

```powershell
docker exec biosphere-kafka kafka-topics --create \
  --topic new.topic.name \
  --bootstrap-server localhost:9092 \
  --partitions 1 \
  --replication-factor 1
```

### Monitor Consumer Groups

```powershell
docker exec biosphere-kafka kafka-consumer-groups --list --bootstrap-server localhost:9092
```

### Reset Consumer Offset (Replay Data)

```powershell
docker exec biosphere-kafka kafka-consumer-groups \
  --bootstrap-server localhost:9092 \
  --group YOUR_GROUP_NAME \
  --reset-offsets \
  --to-earliest \
  --execute \
  --topic biosphere.rainforest.type1
```

---

## Support

**Issues?** Contact the pipeline team or check logs:
- Pipeline logs: `logs/biosphere_pipeline_*.log`
- Kafka logs: `docker logs biosphere-kafka`
- Zookeeper logs: `docker logs biosphere-zookeeper`

---

## Summary

✅ Kafka is running at `localhost:9092`
✅ 5 topics created for different sensor categories
✅ Messages are JSON format with structured sensor data
✅ Multiple teams can consume simultaneously
✅ Data persists for 7 days (configurable)
✅ Can replay historical data anytime

**Get started:** Connect to `localhost:9092` and subscribe to your topics!
