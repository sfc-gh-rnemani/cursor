# Data Platform Architecture Diagram

## Overview
This diagram illustrates the end-to-end data flow from multiple source systems (Salesforce, ServiceNow, Jira, GitHub) through Snowflake's medallion architecture (Bronze → Silver → Gold) to downstream consumption via Streamlit dashboards and ServiceNow integration.

## Architecture Diagram

```mermaid
graph TB
    %% Source Systems
    subgraph Sources["Data Sources"]
        SF[("Salesforce<br/>CRM Data")]
        SN[("ServiceNow<br/>ITSM Data")]
        JIRA[("Jira<br/>Project Data")]
        GH[("GitHub<br/>Code & Issues")]
    end

    %% Ingestion Layer
    subgraph Ingestion["Ingestion Layer"]
        SFDC_PIPE["Salesforce Connector<br/>(Fivetran/Airbyte)"]
        SN_PIPE["ServiceNow API<br/>(REST/Connector)"]
        JIRA_PIPE["Jira Connector<br/>(REST API)"]
        GH_PIPE["GitHub API<br/>(REST/Webhook)"]
    end

    %% Snowflake Medallion Architecture
    subgraph Snowflake["Snowflake Data Platform"]
        subgraph Bronze["🥉 Bronze Layer (Raw)"]
            BRONZE_DB[("BRONZE_DB<br/>Raw Ingested Data<br/>Minimal Transformation")]
            BRONZE_SF["salesforce_raw"]
            BRONZE_SN["servicenow_raw"]
            BRONZE_JIRA["jira_raw"]
            BRONZE_GH["github_raw"]
        end

        subgraph Silver["🥈 Silver Layer (Cleansed)"]
            SILVER_DB[("SILVER_DB<br/>Cleaned & Validated<br/>Standardized Schema")]
            SILVER_SF["salesforce_cleansed"]
            SILVER_SN["servicenow_cleansed"]
            SILVER_JIRA["jira_cleansed"]
            SILVER_GH["github_cleansed"]
        end

        subgraph Gold["🥇 Gold Layer (Business)"]
            GOLD_DB[("GOLD_DB<br/>Business Logic Applied<br/>Analytics-Ready")]
            GOLD_SALES["sales_metrics"]
            GOLD_INCIDENTS["incident_analytics"]
            GOLD_PROJECTS["project_kpis"]
            GOLD_DEV["developer_metrics"]
            GOLD_UNIFIED["unified_dashboard_view"]
        end
    end

    %% Consumption Layer
    subgraph Consumption["Consumption Layer"]
        STREAMLIT["Streamlit App<br/>📊 Interactive Dashboards<br/>Real-time Analytics"]
        SN_WRITE["ServiceNow<br/>📤 Case Updates<br/>Automated Insights"]
    end

    %% Data Flow - Sources to Ingestion
    SF --> SFDC_PIPE
    SN --> SN_PIPE
    JIRA --> JIRA_PIPE
    GH --> GH_PIPE

    %% Ingestion to Bronze
    SFDC_PIPE --> BRONZE_SF
    SN_PIPE --> BRONZE_SN
    JIRA_PIPE --> BRONZE_JIRA
    GH_PIPE --> BRONZE_GH

    %% Bronze to Silver
    BRONZE_SF --> SILVER_SF
    BRONZE_SN --> SILVER_SN
    BRONZE_JIRA --> SILVER_JIRA
    BRONZE_GH --> SILVER_GH

    %% Silver to Gold
    SILVER_SF --> GOLD_SALES
    SILVER_SN --> GOLD_INCIDENTS
    SILVER_JIRA --> GOLD_PROJECTS
    SILVER_GH --> GOLD_DEV
    SILVER_SF --> GOLD_UNIFIED
    SILVER_SN --> GOLD_UNIFIED
    SILVER_JIRA --> GOLD_UNIFIED
    SILVER_GH --> GOLD_UNIFIED

    %% Gold to Consumption
    GOLD_SALES --> STREAMLIT
    GOLD_INCIDENTS --> STREAMLIT
    GOLD_PROJECTS --> STREAMLIT
    GOLD_DEV --> STREAMLIT
    GOLD_UNIFIED --> STREAMLIT
    
    GOLD_INCIDENTS --> SN_WRITE
    GOLD_UNIFIED --> SN_WRITE

    %% Styling
    classDef sourceStyle fill:#e1f5ff,stroke:#0288d1,stroke-width:2px
    classDef ingestionStyle fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    classDef bronzeStyle fill:#fff8e1,stroke:#f9a825,stroke-width:2px
    classDef silverStyle fill:#e8eaf6,stroke:#5e35b1,stroke-width:2px
    classDef goldStyle fill:#fff9c4,stroke:#f9a825,stroke-width:3px
    classDef consumptionStyle fill:#e8f5e9,stroke:#43a047,stroke-width:2px

    class SF,SN,JIRA,GH sourceStyle
    class SFDC_PIPE,SN_PIPE,JIRA_PIPE,GH_PIPE ingestionStyle
    class BRONZE_DB,BRONZE_SF,BRONZE_SN,BRONZE_JIRA,BRONZE_GH bronzeStyle
    class SILVER_DB,SILVER_SF,SILVER_SN,SILVER_JIRA,SILVER_GH silverStyle
    class GOLD_DB,GOLD_SALES,GOLD_INCIDENTS,GOLD_PROJECTS,GOLD_DEV,GOLD_UNIFIED goldStyle
    class STREAMLIT,SN_WRITE consumptionStyle
```

## Layer Descriptions

### 🥉 Bronze Layer (Raw Data)
- **Purpose**: Store raw, unmodified data from source systems
- **Characteristics**: 
  - Exact copy of source data
  - Minimal to no transformation
  - Preserves data lineage
  - Supports data reprocessing

### 🥈 Silver Layer (Cleansed Data)
- **Purpose**: Clean, validate, and standardize data
- **Characteristics**:
  - Data quality checks applied
  - Standardized schemas
  - Deduplicated records
  - Null handling and type conversions

### 🥇 Gold Layer (Business-Ready Data)
- **Purpose**: Apply business logic and create analytics-ready datasets
- **Characteristics**:
  - Business metrics calculated
  - Data aggregated and enriched
  - Optimized for query performance
  - Ready for consumption by BI tools

## Data Flow Summary

1. **Ingestion**: Data extracted from Salesforce, ServiceNow, Jira, and GitHub using connectors/APIs
2. **Bronze**: Raw data lands in Snowflake with minimal transformation
3. **Silver**: Data cleansed, validated, and standardized
4. **Gold**: Business logic applied, metrics calculated, unified views created
5. **Consumption**: 
   - Streamlit dashboards consume Gold layer for interactive analytics
   - ServiceNow receives automated insights and updates from Gold layer

## Technologies

- **Data Sources**: Salesforce, ServiceNow, Jira, GitHub
- **Ingestion**: REST APIs, Webhooks, ETL Connectors (Fivetran/Airbyte)
- **Data Warehouse**: Snowflake
- **Visualization**: Streamlit
- **Automation**: ServiceNow API for write-back

