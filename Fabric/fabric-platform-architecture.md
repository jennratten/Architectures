# Microsoft Fabric Architecture

The unified data platform for AI transformation: how data is ingested, stored in OneLake, prepared, visualized, acted on, analyzed and shared with the wider Microsoft ecosystem.

The source picture is too dense for one readable diagram, so it is presented as an overview followed by four focused drill-downs.

## Contents

1. [Platform overview](#1-platform-overview)
2. [Get data: sources and ingestion](#2-get-data-sources-and-ingestion)
3. [OneLake: store and prepare data](#3-onelake-store-and-prepare-data)
4. [Track, visualize and act](#4-track-visualize-and-act)
5. [Analyze, train and integrate](#5-analyze-train-and-integrate)

## Legend

| Line | Meaning |
| --- | --- |
| Solid arrow (`-->`) | Data movement or ingestion |
| Dashed arrow (`-.->`) | Data consumption (no data movement) |

| Border colour | Category |
| --- | --- |
| Slate `#64748B` | Sources and external systems |
| Orange `#FF8C00` | Ingestion and processing |
| Teal `#0099BC` | Data and storage |
| Blue `#0078D4` | Analytics, visuals and actions |
| Red `#E74856` | AI, agents and ML |
| Purple `#5C2D91` | Developer and DevOps tools |
| Green `#107C10` | Identity, security and governance |

A thicker border marks a primary component (OneLake, Semantic model, Activator).

## 1. Platform overview

**Microsoft Fabric: end-to-end overview**

Four families of source data enter Fabric through the Get data layer and land in OneLake. Every workload then consumes OneLake in place to track, visualize, analyze and train, and results flow out to Microsoft 365, agent, developer and partner tools. Azure, Entra ID, GitHub, Azure DevOps and Purview form the platform foundation.

```mermaid
%%{init: {"theme": "base", "flowchart": {"curve": "basis", "nodeSpacing": 45, "rankSpacing": 65, "htmlLabels": true}, "themeVariables": {"fontFamily": "Segoe UI, Arial, sans-serif", "fontSize": "16px", "primaryColor": "#FFFFFF", "primaryTextColor": "#111827", "primaryBorderColor": "#0078D4", "lineColor": "#64748B", "clusterBkg": "#F8FAFC", "clusterBorder": "#CBD5E1", "edgeLabelBackground": "#FFFFFF"}}}%%
flowchart LR
    subgraph Sources["Data sources"]
        direction TB
        RT["Real-time data<br/>Streams and events"]
        BATCH["Batch datasets<br/>Unstructured, semi-structured,<br/>relational"]
        EXT["External storage<br/>Cloud storage, SharePoint,<br/>OneDrive"]
        REPL["Replication sources<br/>Azure, 3rd-party,<br/>open mirroring partners"]
    end

    subgraph Fabric["Microsoft Fabric"]
        direction LR
        GET["Get data<br/>Eventstream, Pipeline,<br/>Copy Job, Dataflow Gen2"]
        LINK["Shortcuts and Mirroring<br/>Zero-copy and replication"]
        ONELAKE[("OneLake<br/>Store data<br/>Delta + Iceberg formats")]
        PREP["Prepare data<br/>Notebook, User data function"]
        TRACK["Track and visualize<br/>Dashboards, Report,<br/>Activator"]
        ANALYZE["Analyze and train<br/>Agents, ML, GraphQL,<br/>Power BI Copilot"]
    end

    subgraph Integration["Integration"]
        direction TB
        ACT["Actions<br/>Teams, Outlook,<br/>Power Automate, Fabric"]
        COLLAB["Collaboration<br/>Teams, SharePoint,<br/>Excel, PowerPoint"]
        AGENT["Agent AI<br/>Copilot, Copilot Studio,<br/>Foundry"]
        DEV["Developers<br/>Visual Studio"]
        ESTATE["Data estate<br/>Azure Databricks, Snowflake"]
    end

    subgraph Platform["Platform foundation"]
        direction LR
        AZ["Azure"]
        ENTRA["Microsoft Entra ID"]
        GH["GitHub"]
        ADO["Azure DevOps"]
        PURV["Microsoft Purview"]
    end

    RT -->|"Stream"| GET
    BATCH -->|"Batch load"| GET
    EXT -.->|"Shortcut"| LINK
    REPL -->|"Mirror"| LINK
    GET -->|"Land"| ONELAKE
    LINK --> ONELAKE
    ONELAKE <-.-> PREP
    ONELAKE -.-> TRACK
    ONELAKE -.-> ANALYZE
    TRACK -.->|"Alerts, triggers"| ACT
    TRACK -.->|"Share reports"| COLLAB
    ANALYZE -.->|"Grounding"| AGENT
    ANALYZE -.->|"APIs, models"| DEV
    ONELAKE <-.->|"Open formats"| ESTATE
    Platform ~~~ Fabric

    classDef neutralStyle fill:#F8FAFC,stroke:#64748B,stroke-width:2px,color:#111827
    classDef appStyle fill:#FFFFFF,stroke:#0078D4,stroke-width:3px,color:#111827
    classDef dataStyle fill:#FFFFFF,stroke:#0099BC,stroke-width:3px,color:#111827
    classDef primaryData fill:#FFFFFF,stroke:#0099BC,stroke-width:5px,color:#111827
    classDef aiStyle fill:#FFFFFF,stroke:#E74856,stroke-width:3px,color:#111827
    classDef securityStyle fill:#FFFFFF,stroke:#107C10,stroke-width:3px,color:#111827
    classDef toolStyle fill:#FFFFFF,stroke:#FF8C00,stroke-width:3px,color:#111827
    classDef devopsStyle fill:#FFFFFF,stroke:#5C2D91,stroke-width:3px,color:#111827

    class RT,BATCH,EXT,REPL,ACT,COLLAB,ESTATE neutralStyle
    class GET,LINK,PREP toolStyle
    class ONELAKE primaryData
    class TRACK appStyle
    class ANALYZE,AGENT aiStyle
    class DEV,AZ,GH,ADO devopsStyle
    class ENTRA,PURV securityStyle
```

- Only ingestion, mirroring and replication physically move data; shortcuts and every downstream workload read OneLake in place.
- OneLake supports any file type, structured or unstructured, stored in Delta and Iceberg-compatible formats, so partner engines such as Azure Databricks and Snowflake can share the same data.

## 2. Get data: sources and ingestion

**Sources and ingestion paths into OneLake**

Shows which Fabric ingestion item handles each kind of source: Eventstream for streams, Pipeline, Copy Job and Dataflow Gen2 for batch data, Shortcuts for external storage without moving data, and Mirroring for replicated databases.

```mermaid
%%{init: {"theme": "base", "flowchart": {"curve": "basis", "nodeSpacing": 45, "rankSpacing": 65, "htmlLabels": true}, "themeVariables": {"fontFamily": "Segoe UI, Arial, sans-serif", "fontSize": "16px", "primaryColor": "#FFFFFF", "primaryTextColor": "#111827", "primaryBorderColor": "#0078D4", "lineColor": "#64748B", "clusterBkg": "#F8FAFC", "clusterBorder": "#CBD5E1", "edgeLabelBackground": "#FFFFFF"}}}%%
flowchart LR
    subgraph RealTime["Real-time data"]
        direction TB
        MSSRC["Microsoft sources<br/>Event Grid, IoT Hub, Event Hub<br/>Azure Data Services CDC feed"]
        EXTSTREAM["External streaming sources<br/>GCP Pub/Sub, Kafka, AWS Kinesis<br/>MQTT, Solace"]
        DISCRETE["Discrete events<br/>Azure Blob Storage events<br/>OneLake, Workspace, Job events"]
    end

    subgraph Batch["Batch datasets"]
        direction TB
        UNSTR["Unstructured<br/>Images, video, audio, free text"]
        SEMI["Semi-structured<br/>CSV, logs, JSON, XML"]
        REL["Relational databases<br/>Structured"]
    end

    subgraph ExtStorage["External storage"]
        direction TB
        CLOUD["Cloud storage<br/>Azure Storage, ADLS Gen2,<br/>AWS S3, Google Storage"]
        FILES["File collaboration<br/>SharePoint, OneDrive"]
    end

    subgraph Replication["Data replication sources"]
        direction TB
        AZDS["Azure Data Services<br/>SQL Database, PostgreSQL, Cosmos DB,<br/>Databricks catalog, SQL Server"]
        THIRD["3rd-party data services<br/>Google BigQuery, Snowflake, Oracle"]
        OPENMIR["Open mirroring partners<br/>ASAPIO, Striim, Oracle GoldenGate,<br/>MongoDB"]
    end

    subgraph GetData["Get data"]
        direction TB
        ES["Eventstream<br/>Real-time ingestion"]
        PIPE["Pipeline<br/>Orchestration"]
        DFG2["Dataflow Gen2<br/>Low-code transformation"]
        COPY["Copy Job<br/>Bulk and incremental copy"]
        SHORT["Shortcut<br/>No data movement"]
        MIRROR["Data replication<br/>Mirroring"]
    end

    subgraph Land["OneLake landing items"]
        direction TB
        EH[("Eventhouse<br/>Time series")]
        LHWH[("Lakehouse and Warehouse")]
        MIRDB[("Mirrored database<br/>Read-only")]
    end

    MSSRC -->|"Events"| ES
    EXTSTREAM -->|"Streams"| ES
    ES -->|"Ingest"| EH
    DISCRETE -->|"Fabric events"| EH
    UNSTR --> PIPE
    SEMI --> COPY
    REL --> COPY
    PIPE -->|"Orchestrate"| DFG2
    DFG2 -->|"Load"| LHWH
    COPY -->|"Copy"| LHWH
    CLOUD -.->|"Reference"| SHORT
    FILES -.->|"Reference"| SHORT
    SHORT -.->|"Virtualize"| LHWH
    AZDS -->|"Replicate"| MIRROR
    THIRD -->|"Replicate"| MIRROR
    OPENMIR -->|"Replicate"| MIRROR
    MIRROR -->|"Near real-time"| MIRDB

    classDef neutralStyle fill:#F8FAFC,stroke:#64748B,stroke-width:2px,color:#111827
    classDef dataStyle fill:#FFFFFF,stroke:#0099BC,stroke-width:3px,color:#111827
    classDef toolStyle fill:#FFFFFF,stroke:#FF8C00,stroke-width:3px,color:#111827

    class MSSRC,EXTSTREAM,DISCRETE,UNSTR,SEMI,REL,CLOUD,FILES,AZDS,THIRD,OPENMIR neutralStyle
    class ES,PIPE,DFG2,COPY,SHORT,MIRROR toolStyle
    class EH,LHWH,MIRDB dataStyle
```

- Eventstream also feeds Activator directly for real-time triggers (see diagram 4).
- Shortcuts expose Cloud storage and SharePoint/OneDrive content inside OneLake without copying it.

## 3. OneLake: store and prepare data

**OneLake storage items, semantic model and data preparation**

All Fabric storage items live in OneLake and are exposed to reporting through a semantic model using Direct Lake, Import or DirectQuery. Notebooks and User data functions prepare data in place, and Reports can write back through User data functions (translytical).

```mermaid
%%{init: {"theme": "base", "flowchart": {"curve": "basis", "nodeSpacing": 45, "rankSpacing": 65, "htmlLabels": true}, "themeVariables": {"fontFamily": "Segoe UI, Arial, sans-serif", "fontSize": "16px", "primaryColor": "#FFFFFF", "primaryTextColor": "#111827", "primaryBorderColor": "#0078D4", "lineColor": "#64748B", "clusterBkg": "#F8FAFC", "clusterBorder": "#CBD5E1", "edgeLabelBackground": "#FFFFFF"}}}%%
flowchart LR
    subgraph Prepare["Prepare data"]
        direction TB
        NB["Notebook<br/>Spark, Python"]
        UDF["User data function<br/>Custom logic"]
    end

    subgraph OneLake["OneLake: store data"]
        direction TB
        EH[("Eventhouse<br/>Time series")]
        LH[("Lakehouse")]
        WH[("Warehouse")]
        MIRDB[("Mirrored database<br/>Read-only")]
        SQLDB[("SQL Database")]
        COSMOS[("Azure Cosmos DB")]
    end

    SM["Semantic model<br/>Direct Lake, Import,<br/>DirectQuery"]
    RPT["Report<br/>Power BI"]
    ESTATE["Partner data estate<br/>Azure Databricks, Snowflake"]

    NB <-.->|"Invoke"| UDF
    NB <-.->|"Read, write"| OneLake
    UDF <-.->|"Read, write"| OneLake
    EH -.-> SM
    LH -.-> SM
    WH -.-> SM
    MIRDB -.-> SM
    SQLDB -.-> SM
    COSMOS -.-> SM
    SM -.->|"Model"| RPT
    RPT -.->|"Translytical write-back"| UDF
    OneLake <-.->|"Delta, Iceberg"| ESTATE

    classDef neutralStyle fill:#F8FAFC,stroke:#64748B,stroke-width:2px,color:#111827
    classDef appStyle fill:#FFFFFF,stroke:#0078D4,stroke-width:3px,color:#111827
    classDef dataStyle fill:#FFFFFF,stroke:#0099BC,stroke-width:3px,color:#111827
    classDef primaryData fill:#FFFFFF,stroke:#0099BC,stroke-width:5px,color:#111827
    classDef toolStyle fill:#FFFFFF,stroke:#FF8C00,stroke-width:3px,color:#111827

    class EH,LH,WH,MIRDB,SQLDB,COSMOS dataStyle
    class SM primaryData
    class NB,UDF toolStyle
    class RPT appStyle
    class ESTATE neutralStyle
```

- The semantic model is the single business layer over every OneLake item; Direct Lake reads Delta tables without import.
- Translytical flow: a Report triggers a User data function that writes changes back to OneLake.

## 4. Track, visualize and act

**Real-time intelligence, reporting and Activator actions**

Visual and semantic items read OneLake in place and feed Activator, which turns detected conditions into actions in Teams, Outlook, Power Automate and Fabric. Reports are shared through Microsoft 365 collaboration tools.

```mermaid
%%{init: {"theme": "base", "flowchart": {"curve": "basis", "nodeSpacing": 45, "rankSpacing": 65, "htmlLabels": true}, "themeVariables": {"fontFamily": "Segoe UI, Arial, sans-serif", "fontSize": "16px", "primaryColor": "#FFFFFF", "primaryTextColor": "#111827", "primaryBorderColor": "#0078D4", "lineColor": "#64748B", "clusterBkg": "#F8FAFC", "clusterBorder": "#CBD5E1", "edgeLabelBackground": "#FFFFFF"}}}%%
flowchart LR
    ES["Eventstream<br/>Real-time ingestion"]
    ONELAKE[("OneLake<br/>Eventhouse and stores")]
    SM["Semantic model"]

    subgraph Track["Track and visualize data"]
        direction TB
        RTD["Real-Time dashboard"]
        ANOM["Anomaly detector"]
        MAP["Map<br/>Geospatial"]
        GRAPH["Graph model<br/>Relationships"]
        ONTO["Ontology<br/>Business entities"]
        RPT["Report<br/>Power BI"]
    end

    ACTV["Activator<br/>Detect and trigger"]

    subgraph Actions["Actions"]
        direction TB
        A_TEAMS["Teams"]
        A_OUT["Outlook"]
        A_PA["Power Automate"]
        A_FAB["Fabric items"]
    end

    subgraph Collab["Collaboration"]
        direction TB
        C_TEAMS["Teams"]
        C_SP["SharePoint"]
        C_XL["Excel"]
        C_PPT["PowerPoint"]
    end

    ES -.->|"Live events"| ACTV
    ONELAKE -.-> RTD
    ONELAKE -.-> ANOM
    ONELAKE -.-> MAP
    ONELAKE -.-> GRAPH
    ONELAKE -.-> ONTO
    SM -.-> RPT
    RTD -.-> ACTV
    ANOM -.-> ACTV
    ONTO -.-> ACTV
    RPT -.->|"Data alerts"| ACTV
    ACTV -.->|"Notify, run"| Actions
    RPT -.->|"Share, embed"| Collab

    classDef appStyle fill:#FFFFFF,stroke:#0078D4,stroke-width:3px,color:#111827
    classDef primaryApp fill:#FFFFFF,stroke:#0078D4,stroke-width:5px,color:#111827
    classDef dataStyle fill:#FFFFFF,stroke:#0099BC,stroke-width:3px,color:#111827
    classDef toolStyle fill:#FFFFFF,stroke:#FF8C00,stroke-width:3px,color:#111827
    classDef neutralStyle fill:#F8FAFC,stroke:#64748B,stroke-width:2px,color:#111827

    class RTD,ANOM,MAP,GRAPH,ONTO,RPT appStyle
    class ACTV primaryApp
    class ONELAKE,SM dataStyle
    class ES toolStyle
    class A_TEAMS,A_OUT,A_PA,A_FAB,C_TEAMS,C_SP,C_XL,C_PPT neutralStyle
```

- Activator is the hub for automated responses: it receives Eventstream data and signals from dashboards, Anomaly detector, Ontology and Report.
- The Ontology also grounds the Operations agent (see diagram 5).

## 5. Analyze, train and integrate

**AI, machine learning and developer integration**

Agents, experiments, ML models, the GraphQL API and Power BI Copilot all consume OneLake and the semantic model in place. Their outputs reach Copilot, Copilot Studio, Foundry and Visual Studio.

```mermaid
%%{init: {"theme": "base", "flowchart": {"curve": "basis", "nodeSpacing": 45, "rankSpacing": 65, "htmlLabels": true}, "themeVariables": {"fontFamily": "Segoe UI, Arial, sans-serif", "fontSize": "16px", "primaryColor": "#FFFFFF", "primaryTextColor": "#111827", "primaryBorderColor": "#0078D4", "lineColor": "#64748B", "clusterBkg": "#F8FAFC", "clusterBorder": "#CBD5E1", "edgeLabelBackground": "#FFFFFF"}}}%%
flowchart LR
    ONELAKE[("OneLake<br/>Store data")]
    SM["Semantic model"]
    ONTO["Ontology"]

    subgraph Analyze["Analyze and train data"]
        direction TB
        OPSAG["Operations agent<br/>Monitor and act"]
        DATAAG["Data agent<br/>Natural-language questions"]
        EXP["Experiments<br/>Model training"]
        ML["ML model<br/>Scoring"]
        GQL["API for GraphQL<br/>Data access API"]
        PBIC["Power BI Copilot"]
    end

    subgraph AgentAI["Agent AI"]
        direction TB
        COPILOT["Microsoft Copilot"]
        CPS["Copilot Studio"]
        FOUNDRY["Foundry"]
    end

    subgraph Devs["Developers"]
        VS["Visual Studio"]
    end

    ONTO -.->|"Business context"| OPSAG
    ONELAKE -.-> OPSAG
    ONELAKE -.-> DATAAG
    ONELAKE -.-> EXP
    ONELAKE -.-> GQL
    SM -.-> PBIC
    EXP -.->|"Register"| ML
    DATAAG -.->|"Grounding"| AgentAI
    ML -.->|"Consume"| Devs
    GQL -.->|"Query"| Devs

    classDef aiStyle fill:#FFFFFF,stroke:#E74856,stroke-width:3px,color:#111827
    classDef dataStyle fill:#FFFFFF,stroke:#0099BC,stroke-width:3px,color:#111827
    classDef appStyle fill:#FFFFFF,stroke:#0078D4,stroke-width:3px,color:#111827
    classDef devopsStyle fill:#FFFFFF,stroke:#5C2D91,stroke-width:3px,color:#111827

    class OPSAG,DATAAG,EXP,ML,PBIC,COPILOT,CPS,FOUNDRY aiStyle
    class GQL appStyle
    class ONELAKE,SM dataStyle
    class ONTO appStyle
    class VS devopsStyle
```

- The Data agent can be published to Copilot, Copilot Studio and Foundry as a grounded data source.
- ML models and the GraphQL API give developers programmatic access from Visual Studio.

---

*Diagrams render natively on GitHub. Exact appearance may vary slightly with GitHub's Mermaid version and light/dark theme.*
