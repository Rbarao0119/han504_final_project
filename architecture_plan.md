## Service Mapping
| Layer                | Service (Cloud)                                | Role in Solution                                                                                     | Related Assignment / Module                    |
| -------------------- | ---------------------------------------------- | ---------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| Storage              | **Azure Blob Storage**                         | Store daily raw dispensing/refill CSV files and logs                                                 | Storage module (Blob basics & access patterns) |
| Compute (Serverless) | **Azure Functions**          | Ingest CSVs from Blob, clean data, compute MPR/PDC and flags                                         | Serverless / Functions lab                     |
| Database / SQL       | **Azure Database for MySQL – Flexible Server** | Store structured tables (patients, prescriptions, dispense_events, adherence_summary, interventions) | MySQL VM vs Managed MySQL assignment           |
| Compute (Containers) | **Azure Container Apps (Flask/FastAPI)**       | Run the refill work queue dashboard + simple API for pharmacy staff                                  | Containers / Cloud Run / Container Apps module |
| Analytics (Optional) | **Power BI or Azure ML Notebook**              | Build adherence dashboards; experiment with simple analytics or risk scoring                         | Analytics / Notebooks / BI module              |


### Data Flow Narrative
1. **Dispense data arrives**
    - Each day, the pharmacy dispensing system generates a CSV export of the previous day’s fills and refills.
    - The file is uploaded to Azure Blob Storage in a designated container (e.g., manually via portal, or automatically using a script).
2. **Ingestion and data cleaning (Azure Function 1)**
- Azure Function runs nightly:
    - Lists new blobs in the pharmacy-dispense-raw container.
    - Downloads each CSV and validates:
        - Required columns present (patient_id, drug_name, fill_date, days_supply, etc.)
        - Basic type and range checks (e.g., days_supply > 0).

3. **Adherence metrics and risk flagging (Azure Function 2)**
- A second Azure Function (also timer-triggered) runs after ingestion:
    - Queries for fills in the last 6–12 months.
    - For each patient + drug, calculates:
        - Number of gaps (days without on-hand supply beyond a small grace period).
        - Number of claim rejections in the last N days.

4. **Pharmacy dashboard (Containerized web app)**
- A Flask/FastAPI app, containerized and deployed to Azure Container Apps, provides:
    - Login-protected dashboard views using role-based access (pharmacist vs tech).
    - A work queue showing all patients with open flags, sorted by risk.

5. **Analytics & reporting**
- Power BI or an Azure ML Notebook connects to MySQL (read-only):
    - Builds charts for adherence by store, disease state, payer, or time period.
    Supports quality reporting

### Security,identity, and governance basics
        In this system, credentials would be controlled by either an encryption manager like Azure Key Vault in a production scenario or secure environment variables. This stops sensitive data from being hardcoded or saved in GitHub, such as database passwords, storage account credentials, or API tokens. In order to login to Blob Storage and the database without directly storing credentials, each compute component (Azure Functions and Azure Container Apps) would be given its own managed identity. This guarantees that the analytics and ingestion components only access the resources that they are specifically allowed to utilize.
        Least-privilege permissions would be enforced throughout the system by role-based access control (RBAC). For instance, the analytics layer would have read-only access, the dashboard backend would have limited read/write access to only the tables required for refill management, and the ingestion function would have write access to Blob Storage and the database. All testing and demonstration would employ totally synthetic or de-identified patient data to prevent genuine PHI from being exposed in public or unsafe settings. PHI would never leave protected Azure resources or reach any public environment in a true deployment thanks to further security measures including private networking, HTTPS-only endpoints, encryption at rest, and stringent access controls.

### Cost and operational considerations
        Instead of storage or analytics, the compute layer and managed database are the parts of this system that are most likely to increase costs. Because Azure Database for MySQL operates continuously and expands with CPU, memory, and storage requirements, it usually represents the biggest continuing expense. Depending on traffic and resource allotment, Azure Container Apps hosting the dashboard may also significantly increase costs. In comparison, the optional analytics layer (Power BI or an Azure ML Notebook) costs very little when utilized sporadically, while Blob Storage stays affordable even with frequent CSV uploads.
            Serverless components, like Azure Functions, are utilized for ingestion, adherence calculations, and reminders in order to keep the solution within a student budget because they only cost money when they are activated rather than continuously operating like a virtual machine (VM). Timer-based functions and scheduled jobs significantly lower operating overhead and eliminate the requirement for always-on computation. Using synthetic data sets that stay well inside free storage limitations, limiting Container Apps to minimal CPU and memory resources, and choosing the lowest-tier managed database SKU are further cost-saving choices. When taken as a whole, these decisions maintain the architecture's affordability while showcasing practical cloud workflows.