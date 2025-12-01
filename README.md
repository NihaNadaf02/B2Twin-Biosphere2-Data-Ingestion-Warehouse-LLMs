# B2Twin - Biosphere 2 Data Ingestion & Warehouse Pipeline

A comprehensive data pipeline for extracting, transforming, and serving Biosphere 2 environmental sensor data through a RESTful API.

## 📁 Project Structure

```
B2Twin-Biosphere2-Data-Ingestion-Warehouse-LLMs/
├── src/                          # Source code
│   ├── config/                   # Configuration management
│   │   ├── __init__.py
│   │   └── config.py            # Centralized configuration
│   ├── extraction/              # Data extraction from Oracle
│   │   ├── __init__.py
│   │   └── bio2Oracle.py        # Oracle to MySQL extraction
│   ├── transformation/          # Data transformation & aggregation
│   │   ├── __init__.py
│   │   └── join_rainforest_tables.py
│   ├── api/                     # REST API server
│   │   ├── __init__.py
│   │   ├── api_server.py        # FastAPI server
│   │   └── api_client.py        # API testing client
│   ├── monitoring/              # Pipeline monitoring
│   │   ├── __init__.py
│   │   └── pipeline_monitor.py  # Health dashboard
│   └── biosphere_pipeline.py    # Main pipeline orchestrator
├── tests/                       # Test suite
│   ├── __init__.py
│   └── test_config.py          # Configuration tests
├── docs/                        # Documentation
│   └── README.md
├── scripts/                     # Legacy scripts location
├── data/                        # Data storage (created at runtime)
├── logs/                        # Log files (created at runtime)
├── requirements_api.txt         # API dependencies
├── start_api.bat               # Windows API launcher
└── last_run_date.txt           # Pipeline execution tracking

```

## 🚀 Features

- **Incremental Data Extraction**: Efficiently pulls only new data from Oracle database
- **Rolling Window Management**: Maintains 30-day data window in MySQL
- **Data Categorization**: Organizes tables into 5 categories (type1, type2, less50, between50and100, other)
- **RESTful API**: FastAPI-based server with automatic documentation
- **Monitoring Dashboard**: Real-time pipeline health and statistics
- **Comprehensive Logging**: Detailed execution logs for troubleshooting

## 🛠️ Installation

1. **Clone the repository**
```bash
git clone https://github.com/panditpooja/B2Twin-Biosphere2-Data-Ingestion-Warehouse-LLMs.git
cd B2Twin-Biosphere2-Data-Ingestion-Warehouse-LLMs
```

2. **Create virtual environment**
```bash
python -m venv venv
venv\Scripts\activate  # Windows
```

3. **Install dependencies**
```bash
pip install -r requirements_api.txt
```

4. **Configure databases**
Edit `src/config/config.py` with your database credentials.

## 📊 Usage

### Run Complete Pipeline
```bash
python src/biosphere_pipeline.py
```

### Start API Server
```bash
# Windows
start_api.bat

# Or manually
python src/api/api_server.py
```

Access API documentation at:
- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc

### Monitor Pipeline
```bash
python src/monitoring/pipeline_monitor.py
```

### Test Configuration
```bash
python tests/test_config.py
```

## 🔌 API Endpoints

- `GET /` - API information
- `GET /health` - Health check
- `GET /tables` - List available tables
- `GET /data/{category}` - Retrieve data by category
- `GET /data/{category}/stats` - Get statistics
- `GET /data/{category}/unique_ids` - List unique identifiers

## 🏗️ Architecture

### Phase 1: Extraction & Staging
- Connects to Oracle database (Biosphere 2)
- Extracts rainforest sensor data incrementally
- Assigns unique sequential IDs
- Stores in MySQL staging database

### Phase 2: Transformation & Aggregation
- Joins related tables by category
- Creates optimized views for API consumption
- Maintains data integrity across joins

### Phase 3: API Serving
- FastAPI server exposes data via REST endpoints
- Supports pagination, filtering, and statistics
- CORS-enabled for web applications

## 🔧 Configuration

Key configuration parameters in `src/config/config.py`:
- Database connection strings
- Rolling window duration (30 days default)
- File paths and directories
- Logging settings

## 📝 License

[Add your license here]

## 👥 Contributors

- Pooja Pandit

## 📞 Support

For issues or questions, please open an issue on GitHub.
