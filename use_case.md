# Healthcare Scenario
### Problem Statetment: 
A busy retail pharmacy finds it difficult to identify patients who are not following their routine and to keep up with refills for chronic medications (such as those for asthma, diabetes, and hypertension). Pharmacy staff members now have to manually mark patients who are past due, print reports from the dispensing system, and call each patient individually, which presents a number of difficulties.

#### Primary Users: 
- **Pharmacists** : review high-risk patients, clinical notes, and decide on interventions.
- **Pharmacy technicians** : use a daily queue to call/text patients and resolve refill issues.
- **District Managers/ Store Managers** : view aggregate adherence metrics for quality programs.


#### Data sources:
- Dispensing and refill data (CSV → Blob → Database)
- Patient panel / risk tiers (CSV → Database) 
- Intervention log 
- Derived adherence metrics (database tables)

#### Basic Workflow
1. **Daily data export and upload**  
   - Once per day, the pharmacy (or a simulated script) exports a **dispense/refill CSV** from the pharmacy system.  
   - A staff member uploads the file to a secure **Azure Blob Storage** container 

2. **Ingestion and cleaning (Serverless compute)**  
   - An Azure function that operates according to a schedule (e.g., nightly). 

3. **Adherence calculation and flagging**  
   - The same function, or a second Azure Function, computes:
     - Rolling per patient and drug (e.g., last 180 days).
     - “Refill overdue” flags based on days_supply and last fill_date.
     - “Claim trouble” flags for repeated claim rejections.  

4. **Pharmacy work queue dashboard (Containerized web app)**  
   - A small **Flask or FastAPI app** runs in **Azure Container Apps**.  
     - Filters: store/location, risk tier, condition, “overdue days,” payer program.  
     - Clicking a patient row shows basic history and recent interventions.

5. **Analytics and reporting**  
   - A **Power BI report** or **Azure ML notebook** connects to the MySQL database to:
     - Track overall adherence trends by store, condition, or payer.  
     - Evaluate the effect of outreach (e.g., change in PDC after intervention).  
     - Optionally experiment with simple models to predict which patients are most likely to become nonadherent.