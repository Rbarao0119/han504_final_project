# han504_final_project

### Instructions for assignment: 

This project uses Azure services to construct a small, practical healthcare cloud system (with analogies to GCP and OCI, which were also discussed in the course). The objective is to provide a working end-to-end pipeline that incorporates computation, storage, a controlled database, and an analytics component while adhering to the architectural patterns that have been practiced all semester.

## 1. Understand the Use Case

Begin by reviewing the use_case.md file. This document defines the pharmacy scenario, identifies the users, describes the problem being solved, and outlines the data sources and workflow. Make sure you understand how the daily dispense CSV files flow into the system and how adherence metrics are produced for the pharmacy dashboard.

## 2. Review the Architecture Diagram

- Open the generated PNG diagram. 

- Included in the image is: 
    - Frontend / Access layer
    - Serverless and container-based compute layer
    - Object storage and relational database
    - Optional analytics/AI component

## 3. Read the Architecture & Implementation Plan
- Which cloud services are used and why
- How credentials, RBAC, and governance are handled
- Cost considerations and where serverless is used to minimize spending
- The detailed step-by-step data flow from ingestion all the way to analytics
This plan ties the architecture back to the concepts taught in Azure, GCP, and OCI labs.