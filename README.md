
# 🧠 Batch Model Execution with Control-M and Jenkins CI/CD

This project automates the execution of a machine learning model using **Control-M** for batch orchestration and **Jenkins** for CI/CD. The workflow connects to a source database, runs a model via an external service, and saves the output to a destination file. It includes preprocessing, model execution, and post-processing stages.

## 📁 Project Structure

```
project-root/
│
├── controlm/
│   └── batch_model.json         # Control-M job definition (with pre, model, post stages)
│
├── scripts/
│   ├── preprocess.py            # Extract and preprocess data from source DB
│   ├── run_model.py             # Run model via external service
│   └── postprocess.py           # Save/format/export final result
│
├── tests/
│   └── test_model.py            # Optional unit tests
│
├── Jenkinsfile                 # Jenkins CI/CD pipeline
└── README.md                   # This documentation
```

## 🔧 Workflow Overview

### 1. **Preprocessing Stage**
- Extracts raw data from the source database
- Performs transformations or filtering

### 2. **Model Execution**
- Calls an external service or local script that performs ML predictions

### 3. **Postprocessing**
- Formats the output and saves it to a destination file or system

## ⚙️ Jenkins Pipeline Details

### 💡 Stages

| Stage                    | Description                                                   |
|--------------------------|---------------------------------------------------------------|
| `Build`                  | Runs `ctm build` to validate JSON and create deployable ZIP   |
| `Test`                   | Runs any Python unit tests                                    |
| `Pre-Copy`               | Securely transfers scripts to the execution server via SCP    |
| `Deploy & Run`           | Logs in to Control-M CLI, deploys job, and triggers execution |

### 🔐 Required Jenkins Credentials
- `controlm_user` – Control-M user (stored as credential ID)
- `controlm_password` – Control-M password
- Make sure `batchuser@execution-host` has SSH access enabled

## 📄 Control-M Job Breakdown

Defined in `controlm/batch_model.json`, the folder contains three dependent jobs:

### `Preprocess_Data`
- Extracts and cleans data
- Script: `preprocess.py`

### `Run_Model`
- Executes model using external service
- Script: `run_model.py`
- Depends on `Preprocess_Data`

### `Postprocess_Data`
- Formats and saves result (e.g., CSV, DB write)
- Script: `postprocess.py`
- Depends on `Run_Model`

### Example Command in Job
```json
"Command": "python3 /opt/batch_model/preprocess.py"
```

## 🛠️ Deployment Steps

### ✅ Prerequisites

- Control-M CLI installed on Jenkins agent: `/opt/controlm/ctmcli`
- Execution server (e.g., EC2, VM) accessible via SSH
- Python installed on the execution server
- `scp` and `ssh` enabled from Jenkins to host

### 🚀 Triggering the Pipeline

```bash
git push origin main   # or trigger via Jenkins UI
```

This will:
1. Build the Control-M ZIP
2. Run tests
3. Copy scripts to execution server
4. Deploy and run the job in Control-M

## 🧪 Sample Python Script (`run_model.py`)

```python
import pandas as pd
import psycopg2
import requests

# Extract from DB
conn = psycopg2.connect(...)
df = pd.read_sql("SELECT * FROM source_table", conn)
conn.close()

# Call external model
response = requests.post("https://model-service/predict", json=df.to_dict(orient="records"))
predictions = response.json()

# Save to CSV
pd.DataFrame(predictions).to_csv("/opt/batch_model/output.csv", index=False)
```

## 🔍 Monitoring

You can monitor job status using:

```bash
ctm run status <run-id>
```

## 📬 Contact & Maintenance

- **Owner**: [Your Name or Team]
- **Support Email**: support@example.com
- **Version**: 1.0.0
- **License**: Internal/Proprietary

## 📌 Future Improvements

- Add retry and failure hooks to Control-M jobs
- Add Slack/email notifications on failure
- Containerize the model logic using Docker
- Add data validation and quality checks
