**Airflow-Docker-Prod 🚀**
Production-grade Apache Airflow setup running on Docker. This repository orchestrates the ETL pipelines for the MarTech Analytics Warehouse, specifically focusing on Google Ads, Meta Ads, LinkedIn Ads, TikTok Ads Data Extraction.

**🏗 Architecture:**

- This setup is optimized for stability and resource efficiency on a single node (Docker Desktop).
- Airflow Version: 2.9+ (Python 3.10)
- Transformation Engine: dbt Core 1.7.8 (BigQuery Adapter).
- Executor: LocalExecutor (Enables parallelism without the overhead of Redis/Celery workers).
- Metadata Database: PostgreSQL 14.
- Infrastructure: Docker Compose.

**📂 Project Structure:**
Plaintext
```
Airflow-Docker-Prod/
├── dags/                  # ETL Workflows (Ads pipelines)
│   ├── google_ads_*.py    # Specific extraction DAGs
├── db/                    # Mounted volume for PostgreSQL (Legacy/Backup)(Gitignored)
├── logs/                  # Airflow task logs (Persisted locally)(Gitignored)
├── plugins/               # Custom Airflow plugins/hooks
├── secrets/               # Directory for business logic secrets (JSON keys, params.env)(Gitignored)
├── .env                   # Infrastructure Environment Variables (Gitignored)
├── docker-compose.yaml    # Container orchestration config (Gitignored)
├── Dockerfile             # Custom Airflow image definition
├── requirements.txt       # Python dependencies (pandas, google-ads, etc.)
└── best_practices.sh      # Guide for production standards
```

**🛠️ Prerequisites:**

- Docker Desktop installed and running.
- Recommendation: Allocate at least 4GB - 6GB RAM to Docker in settings.
- Git configured.
- PyCharm (Recommended IDE).

# --- Core Config ---
AIRFLOW_UID=50000
AIRFLOW__CORE__FERNET_KEY=<Paste_Fernet_Key_Here>
AIRFLOW__CORE__EXECUTOR=LocalExecutor
AIRFLOW__CORE__LOAD_EXAMPLES=False
AIRFLOW__CORE__DEFAULT_TIMEZONE=Europe/Paris

# --- Database ---
POSTGRES_USER=xxxx_prod
POSTGRES_PASSWORD=<Secure_Password>
POSTGRES_DB=airflow_prod_xxxxx
AIRFLOW__DATABASE__SQL_ALCHEMY_CONN=postgresql+psycopg2://xxxx_prod:<Secure_Password>@postgres:5432/airflow_prod_xxxxx

# --- Performance ---
AIRFLOW__WEBSERVER__WORKERS=2
AIRFLOW__CORE__PARALLELISM=4

**3. Business Logic Secrets:**
Placed Google Cloud credentials and extraction parameters in secured place.

**4. Build and Run:**
Run the following command to build the image and start the services:

Bash

`docker compose up -d --build`

**5. Access Airflow:**

- UI: http://localhost:8080
- User: admin_xxxx
- Password: (The password you set in .env)

**💻 Development Workflow:** Adding New Dependencies If your DAG needs a new Python library:

- Add the library to requirements.txt.

**Rebuild the container:**

Bash

`docker compose up -d --build`

**Adding New DAGs**

- Place new Python files in the dags/ folder. The Airflow Scheduler picks them up automatically (within ~30 seconds).

**Best Practices (best_practices.sh)**

- Please refer to the best_practices.sh file in this repository for coding standards. Key rules:

**No Top-Level Code:** Avoid database connections or API calls outside of Tasks/Operators.

**Idempotency:** Ensure DAGs can be re-run without duplicating data.

**Secrets:** Never hardcode passwords in Python files; use Variables or Connections.

**dbt Specific Standards** (dbt_best_practices.sh)

- Refer to the dbt_best_practices.sh file for specific CLI commands.


- **Materialization:** Staging (Silver) layers are Views (for instant CDC). Marts (Gold) layers are Tables (for performance).


- **Environment:** Prod targets marketing_prod dataset; Dev targets marketing_dev.

**🔧 Troubleshooting:**

**Issue:** "Worker was sent SIGKILL"

**Cause:** Docker ran out of RAM.

**Fix:** Increase Docker Desktop memory to 6GB or reduce AIRFLOW__CORE__PARALLELISM in .env.

**Issue:** "FATAL: password authentication failed"

**Cause:** You changed the password in .env but the Database volume still has the old one.

**Fix:** Reset the volume (Warning: Deletes DAG history):

Bash

`docker compose down --volumes
docker compose up -d`

**Issue:** "dbt: command not found"

**Cause:** You are trying to run dbt from Windows, or the image wasn't rebuilt after adding dbt-bigquery to requirements.

**Fix:** Ensure you run commands inside the container (see Workflow above) and rebuild using docker compose up -d --build.

**🧠 Author & Maintainer**

Prasanna

📧 prasanna.uthamaraj@informa.com

🌐 GitHub: PrasannaDataBus

**🏁 License**

- This repository is proprietary to Informa PLC / PrasannaDataBus.
- Unauthorized redistribution or public sharing of API credentials, business logic, or proprietary connectors is prohibited.
