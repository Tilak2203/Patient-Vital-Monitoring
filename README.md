# Patient Vital Monitoring System

A real-time streaming data pipeline built on Google Cloud Platform that processes patient vital signs, performs data quality validation, calculates risk scores, and provides healthcare monitoring insights through Power BI dashboards.

## 🏥 Project Overview

This project simulates a hospital IoT environment where patient vital signs (heart rate, SpO2, temperature, blood pressure) are continuously streamed, processed, and analyzed in real-time. The system implements a **medallion architecture** (Bronze → Silver → Gold) to progressively refine data quality and deliver actionable healthcare insights.

## 🎯 Key Features

- **Real-time Data Streaming**: Simulates 20+ patient devices publishing vital signs every 2 seconds via Cloud Pub/Sub
- **Automated Data Quality**: Validates incoming records, filters invalid data, and handles missing/out-of-range values
- **Risk Assessment**: Calculates patient risk scores based on vital sign thresholds (Low/Moderate/High)
- **Medallion Architecture**: Implements Bronze (raw), Silver (cleaned), and Gold (aggregated) data layers
- **Scalable Processing**: Uses Apache Beam with Dataflow for distributed stream processing
- **Interactive Dashboards**: Power BI visualizations for real-time patient monitoring and risk alerts

## 🏗️ Architecture

```
Patient Devices (Simulator)
         ↓
    Cloud Pub/Sub
         ↓
   Apache Beam Pipeline (Dataflow)
         ↓
    ┌────────────────────────────┐
    │  Bronze Layer (Raw Data)   │ → Cloud Storage (JSON)
    │  Silver Layer (Validated)  │ → Cloud Storage (JSON)
    │  Gold Layer (Aggregated)   │ → BigQuery
    └────────────────────────────┘
         ↓
    Power BI Dashboard
```

### Data Flow Layers

1. **Bronze Layer**: Raw vital signs stored as-is in Cloud Storage
2. **Silver Layer**: Validated, enriched data with risk scores
3. **Gold Layer**: Patient-level aggregations (avg vitals, max risk level) in BigQuery

## 🛠️ Tech Stack

- **Streaming**: Google Cloud Pub/Sub
- **Processing**: Apache Beam, Cloud Dataflow
- **Storage**: Cloud Storage (GCS), BigQuery
- **Visualization**: Power BI
- **Language**: Python 3.9+
  
## 📋 Prerequisites

- Google Cloud Platform account with billing enabled
- Python 3.9 or higher
- GCP CLI (`gcloud`) installed and configured
- Required GCP APIs enabled:
  - Cloud Pub/Sub API
  - Cloud Dataflow API
  - BigQuery API
  - Cloud Storage API

## 🚀 Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/Tilak2203/patient-vital-monitoring.git
cd patient-vital-monitoring
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Configure Environment Variables

Create a `.env` file in the project root:

```env
# GCP Project Configuration
GCP_PROJECT=your-project-id
REGION=us-central1

# Pub/Sub Configuration
PUBSUB_TOPIC=patient-vitals-topic
PUBSUB_SUBSCRIPTION=patient-vitals-subscription

# Storage Paths
BRONZE_PATH=gs://your-bucket/bronze/
SILVER_PATH=gs://your-bucket/silver/
TEMP_LOCATION=gs://your-bucket/temp/
STAGING_LOCATION=gs://your-bucket/staging/

# BigQuery Configuration
BIGQUERY_TABLE=your-project:dataset.patient_vitals_gold

# Simulator Configuration
PATIENT_COUNT=20
STREAM_INTERVAL=2
ERROR_RATE=0.1
```

### 4. Create GCP Resources

```bash
# Create Pub/Sub topic and subscription
gcloud pubsub topics create patient-vitals-topic
gcloud pubsub subscriptions create patient-vitals-subscription \
    --topic=patient-vitals-topic

# Create Cloud Storage bucket
gsutil mb -l us-central1 gs://your-bucket-name

# Create BigQuery dataset
bq mk --dataset your-project:patient_vitals
```

### 5. Run the Data Simulator

```bash
python patient_simulator.py
```

This will start publishing simulated patient vitals to Pub/Sub.

## 📊 Data Schema

### Patient Vitals (Input)
```json
{
  "patient_id": "P001",
  "timestamp": "2024-01-27T10:30:00Z",
  "heart_rate": 78.5,
  "spo2": 97.2,
  "temperature": 37.1,
  "bp_systolic": 120.0,
  "bp_diastolic": 80.0
}
```

### Silver Layer (Enriched)
```json
{
  "patient_id": "P001",
  "timestamp": "2024-01-27T10:30:00Z",
  "heart_rate": 78.5,
  "spo2": 97.2,
  "temperature": 37.1,
  "bp_systolic": 120.0,
  "bp_diastolic": 80.0,
  "risk_score": 0.23,
  "risk_level": "Low"
}
```

### Gold Layer (Aggregated - BigQuery)
```json
{
  "patient_id": "P001",
  "avg_heart_rate": 75.3,
  "avg_spo2": 96.8,
  "avg_temperature": 37.0,
  "max_risk_level": "Moderate"
}
```

## 🔍 Data Quality Rules

The pipeline validates incoming data against these criteria:

- **Heart Rate**: 0 < HR < 200 bpm
- **SpO2**: 0 < SpO2 ≤ 100%
- **Temperature**: 30°C ≤ Temp ≤ 45°C
- **Required Fields**: All vital sign fields must be present
- **Error Injection**: 10% of simulated records contain intentional errors for testing

## 📈 Risk Score Calculation

```python
risk_score = (heart_rate/200) * 0.4 + 
             (temperature/40) * 0.3 + 
             (1 - spo2/100) * 0.3

Risk Levels:
- Low: risk_score < 0.3
- Moderate: 0.3 ≤ risk_score < 0.6
- High: risk_score ≥ 0.6
```

## 📊 Power BI Dashboard

The Power BI dashboard connects to BigQuery and provides:

1. **Real-time Patient Monitoring**: Live vital signs for all patients
2. **Risk Alert Panel**: Patients with High/Moderate risk levels
3. **Trend Analysis**: Historical vital sign patterns
4. **Anomaly Detection**: Sudden changes in patient condition
5. **KPI Cards**: Average vitals, total patients, risk distribution



---

**⚠️ Disclaimer**: This is a demonstration project using simulated data. It is not intended for actual clinical use and should not be used for real patient monitoring without proper medical device certification and regulatory compliance.
