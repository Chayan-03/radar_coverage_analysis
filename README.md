# Radar Coverage Analysis System

A microservices-based system for computing and visualizing radar coverage areas using geospatial technologies. This project demonstrates a full-stack architecture with asynchronous message processing, containerization, and real-time map visualization.

## 🏗️ Architecture Overview

The system follows a microservices architecture with the following components:

```
┌─────────────┐
│   React     │  Frontend (Port 3000)
│  Frontend   │
└──────┬──────┘
       │ HTTP/REST
       ▼
┌─────────────────────┐
│  Analysis Manager   │  Node.js/Express (Port 3001)
│      Service        │
└──────┬──────────────┘
       │
       ├──► Kafka (analysis-requests)
       │
       ▼
┌─────────────────────┐
│ Radar Computation   │  Python/FastAPI (Port 3002)
│      Service        │
└──────┬──────────────┘
       │
       ├──► Kafka (computation-results)
       │
       ▼
┌─────────────────────┐
│   PostgreSQL +       │  Database (Port 5432)
│      PostGIS        │
└─────────────────────┘
```

## 🛠️ Technology Stack

### Frontend
- **React 18** - UI framework
- **Leaflet** - Interactive map visualization
- **Axios** - HTTP client
- **Nginx** - Web server (production)

### Backend Services
- **Analysis Manager Service** (Node.js/Express)
  - Orchestrates analysis workflow
  - Manages job status
  - REST API endpoints
  - Kafka producer/consumer

- **Radar Computation Service** (Python)
  - Core radar propagation calculations
  - Coverage polygon generation
  - Geospatial operations (Shapely, GeoPandas)

### Infrastructure
- **PostgreSQL 16** with **PostGIS** - Geospatial database
- **Apache Kafka** - Asynchronous message broker
- **Docker & Docker Compose** - Containerization
- **Zookeeper** - Kafka coordination

## 📋 Prerequisites

- Docker Desktop (or Docker Engine + Docker Compose)
- Git
- At least 4GB of available RAM
- Ports 3000, 3001, 3002, 5432, 9092, 2181 available

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone <repository-url>
cd radar-coverage-system
```

### 2. Start All Services

```bash
docker-compose up --build
```

This will:
- Build Docker images for all services
- Start PostgreSQL with PostGIS
- Start Zookeeper and Kafka
- Start Analysis Manager service
- Start Radar Computation service
- Start React frontend

### 3. Access the Application

- **Frontend**: http://localhost:3000
- **Analysis Manager API**: http://localhost:3001
- **Health Check**: http://localhost:3001/health

### 4. Stop All Services

```bash
docker-compose down
```

To also remove volumes (database data):

```bash
docker-compose down -v
```

## 📖 Usage Guide

### Creating a Radar Coverage Analysis

1. **Open the Frontend**: Navigate to http://localhost:3000

2. **Set Radar Location**:
   - Click on the map to set the radar location
   - Or manually enter latitude/longitude in the form

3. **Enter Radar Parameters**:
   - **Scenario Name**: A descriptive name for this analysis
   - **Frequency (MHz)**: Operating frequency (e.g., 2400 for 2.4 GHz)
   - **Transmit Power (dBm)**: Transmitter power in decibels-milliwatts
   - **Antenna Height (meters)**: Height of the antenna above ground
   - **Antenna Gain (dBi)**: Antenna gain in decibels-isotropic
   - **Minimum Signal Threshold (dBm)**: Minimum detectable signal level

4. **Submit Analysis**: Click "Calculate Coverage"

5. **View Results**: 
   - The system processes the request asynchronously
   - Coverage polygon appears on the map when computation completes
   - Status updates in real-time

### Example Parameters

```
Scenario Name: Test Radar Site
Latitude: 28.6139
Longitude: 77.2090
Frequency: 2400 MHz
Transmit Power: 30 dBm
Antenna Height: 50 meters
Antenna Gain: 10 dBi
Minimum Signal Threshold: -100 dBm
```

## 🔌 API Documentation

### Analysis Manager Service (Port 3001)

#### Health Check
```
GET /health
```
Returns service health status.

#### Create Analysis
```
POST /api/analysis
Content-Type: application/json

{
  "name": "Radar Site 1",
  "latitude": 28.6139,
  "longitude": 77.2090,
  "frequency_mhz": 2400,
  "transmit_power_dbm": 30,
  "antenna_height_m": 50,
  "antenna_gain_dbi": 10,
  "minimum_signal_threshold_dbm": -100
}
```

Response:
```json
{
  "jobId": "uuid-here",
  "scenarioId": 1,
  "status": "pending",
  "message": "Analysis request submitted"
}
```

#### Get Job Status
```
GET /api/analysis/:jobId/status
```

Response:
```json
{
  "jobId": "uuid-here",
  "scenarioId": 1,
  "status": "completed",
  "result": {
    "coverage_radius_km": 15.5,
    "status": "completed"
  },
  "createdAt": "2024-01-01T00:00:00.000Z",
  "completedAt": "2024-01-01T00:00:05.000Z"
}
```

#### Get All Scenarios
```
GET /api/scenarios
```

#### Get Scenario with Coverage
```
GET /api/scenarios/:id
```

#### Get All Coverage Results
```
GET /api/coverage
```

## 🔬 Technical Details

### Radar Coverage Calculation

The system uses a simplified free-space path loss model:

**Path Loss Formula:**
```
L = 20*log10(d) + 20*log10(f) + 32.44
```
Where:
- `d` = distance in kilometers
- `f` = frequency in MHz

**Received Power:**
```
Pr = Pt + Gt - L
```
Where:
- `Pr` = Received power (dBm)
- `Pt` = Transmit power (dBm)
- `Gt` = Antenna gain (dBi)
- `L` = Path loss (dB)

**Coverage Radius:**
The maximum distance where received power ≥ minimum signal threshold is calculated using binary search.

### Geospatial Processing

- Coverage polygons are generated as circles using Shapely
- Polygons are stored in PostGIS as `GEOMETRY(POLYGON, 4326)` (WGS84)
- Frontend displays polygons using Leaflet with GeoJSON conversion

### Asynchronous Processing Flow

1. Frontend submits analysis request → Analysis Manager
2. Analysis Manager creates scenario in database
3. Analysis Manager publishes message to Kafka topic `analysis-requests`
4. Radar Computation Service consumes message
5. Computation Service calculates coverage and saves to database
6. Computation Service publishes result to Kafka topic `computation-results`
7. Analysis Manager consumes result and updates job status
8. Frontend polls job status and displays result on map

## 📁 Project Structure

```
radar-coverage-system/
├── frontend/                 # React frontend application
│   ├── src/
│   │   ├── App.js           # Main React component
│   │   ├── App.css          # Styles
│   │   └── index.js         # Entry point
│   ├── public/
│   ├── Dockerfile           # Frontend container
│   ├── nginx.conf           # Nginx configuration
│   └── package.json
│
├── analysis-manager/        # Node.js orchestration service
│   ├── src/
│   │   └── index.js         # Express server with Kafka
│   ├── Dockerfile
│   └── package.json
│
├── radar-computation/        # Python computation service
│   ├── src/
│   │   └── main.py          # Kafka consumer & calculations
│   ├── Dockerfile
│   └── requirements.txt
│
├── database/
│   └── init.sql             # PostGIS schema initialization
│
├── docker-compose.yml       # Service orchestration
└── README.md
```

## 🐛 Troubleshooting

### Services Not Starting

1. **Check Docker**: Ensure Docker is running
   ```bash
   docker ps
   ```

2. **Check Ports**: Ensure required ports are not in use
   ```bash
   # Windows
   netstat -ano | findstr :3000
   
   # Linux/Mac
   lsof -i :3000
   ```

3. **View Logs**: Check service logs
   ```bash
   docker-compose logs analysis-manager
   docker-compose logs radar-computation
   docker-compose logs frontend
   ```

### Database Connection Issues

1. **Wait for Database**: PostgreSQL may take 10-20 seconds to initialize
2. **Check Database Logs**:
   ```bash
   docker-compose logs postgres
   ```

### Kafka Connection Issues

1. **Check Kafka Health**:
   ```bash
   docker-compose logs kafka
   ```

2. **Restart Kafka**:
   ```bash
   docker-compose restart kafka zookeeper
   ```

### Frontend Not Loading

1. **Check Build**: Frontend may need to rebuild
   ```bash
   docker-compose up --build frontend
   ```

2. **Check Browser Console**: Open developer tools for errors

## 🔒 Security Considerations

This is a demonstration project. For production use, consider:

- Authentication and authorization (JWT, OAuth)
- HTTPS/TLS encryption
- Input validation and sanitization
- Rate limiting
- Secrets management (HashiCorp Vault)
- Container security scanning
- Network policies
- Database connection encryption

## 🚧 Future Enhancements

- [ ] User authentication and authorization
- [ ] Multiple radar scenario comparison
- [ ] Terrain-aware propagation models
- [ ] Real-time WebSocket updates
- [ ] Export coverage data (GeoJSON, KML)
- [ ] Historical analysis tracking
- [ ] Advanced visualization options
- [ ] Redis caching for performance
- [ ] Prometheus/Grafana monitoring
- [ ] Kubernetes deployment manifests

## 📝 License

This project is for educational and demonstration purposes.

## 👥 Contributing

This is a learning project. Feel free to fork and modify for your own use.

## 📚 References

- [PostGIS Documentation](https://postgis.net/documentation/)
- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [React Leaflet](https://react-leaflet.js.org/)
- [Free Space Path Loss](https://en.wikipedia.org/wiki/Free-space_path_loss)

## 🎓 Learning Outcomes

This project demonstrates:

- Microservices architecture design
- Asynchronous message processing with Kafka
- Geospatial data processing and visualization
- Docker containerization and orchestration
- Full-stack development (React + Node.js + Python)
- REST API design
- Database schema design with PostGIS
- Error handling and logging
- Service-to-service communication patterns

---

**Note**: This is a simplified implementation for educational purposes. Real-world radar systems require more sophisticated propagation models, terrain data, and atmospheric considerations.

