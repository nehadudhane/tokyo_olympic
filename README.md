# Tokyo Olympics 2020 — Azure Data Factory Ingestion Pipeline

## Project Overview

This project documents a batch data-ingestion pipeline built with **Azure Data Factory (ADF)** and **Azure Data Lake Storage Gen2 (ADLS Gen2)** using five CSV datasets about the Tokyo 2020 Olympics.

The `data-ingestion` pipeline reads source files hosted in this GitHub repository over HTTP and copies them into a raw-data folder in ADLS Gen2. This repository contains the source datasets and the exported Azure Resource Manager (ARM) configurations for the factory and ingestion pipeline.

## Technology Stack

- **Orchestration:** Azure Data Factory
- **Storage:** Azure Data Lake Storage Gen2
- **Source:** GitHub-hosted CSV files accessed through HTTP linked services
- **Data format:** Delimited text / CSV
- **Infrastructure configuration:** ARM JSON templates

## Architecture

```text
GitHub-hosted CSV files
          |
          | HTTP source datasets
          v
Azure Data Factory
  data-ingestion pipeline
  5 sequential Copy activities
          |
          | ADLS Gen2 sink datasets
          v
ADLS Gen2
  tokyo-olympic-data/
    raw-data/
      athletes.csv
      coaches.csv
      egender.csv
      medals.csv
      teams.csv
```

The exported pipeline defines five activities connected by *Succeeded* dependencies: **Athletes → coaches → gender → medals → Teams**. This is a sequential batch ingestion workflow, rather than a streaming pipeline.

## Datasets

| Source file | Contains |
| --- | --- |
| `Athletes.csv` | Athlete participation information |
| `Coaches.csv` | Coach information |
| `EntriesGender.csv` | Entries by gender and discipline |
| `Medals.csv` | Medal standings |
| `Teams.csv` | Team participation information |

The CSV files are available under [`data/`](data/).

## Azure Data Factory Implementation

### Source and destination datasets

ADF uses delimited-text source datasets backed by HTTP linked services pointing at the raw CSV files in this repository. The destination datasets use an ADLS Gen2 linked service and write to the `raw-data` folder in the `tokyo-olympic-data` filesystem.

### Pipeline orchestration

The `data-ingestion` pipeline defines one **Copy** activity per CSV file. Each activity starts only after the previous one succeeds:

1. `Athletes`
2. `coaches`
3. `gender`
4. `medals`
5. `Teams`

The exported activities use HTTP reads and ADLS Gen2 writes. The templates describe the configured pipeline; this repository does not include independent run logs or validation results.

### ARM deployment artifacts

All exported ADF configuration files are grouped under [`adf/`](adf/):

```text
tokyo_olympic/
├── README.md
├── data/
│   ├── Athletes.csv
│   ├── Coaches.csv
│   ├── EntriesGender.csv
│   ├── Medals.csv
│   └── Teams.csv
└── adf/
    ├── README.md
    ├── factory/
    │   ├── tokyoadf21_ARMTemplateForFactory.json
    │   └── tokyoadf21_ARMTemplateParametersForFactory.json
    ├── standalone/
    │   ├── ARMTemplateForFactory.json
    │   └── ARMTemplateParametersForFactory.json
    └── linked-deployment/
        ├── ArmTemplate_0.json
        ├── ArmTemplate_master.json
        └── ArmTemplateParameters_master.json
```

- **`factory/`** — Factory resource template and parameters.
- **`standalone/`** — Ingestion pipeline with linked services, datasets, and parameters in a full export.
- **`linked-deployment/`** — Alternative master/child linked ARM deployment export.

See [`adf/README.md`](adf/README.md) for more details on these exports.

## Deployment Notes

The templates contain environment-specific factory and storage defaults and are **not guaranteed to deploy unchanged** into another Azure subscription.

- Update the target factory name and ADLS Gen2 storage URL for your environment.
- Supply the required ADLS account key securely at deployment time; never commit populated secrets.
- The linked-deployment option also requires a reachable templates-container URI and an appropriate SAS token.
- Confirm Azure access permissions, linked-service connections, and pipeline execution after deployment.

**Scope:** This repository documents the ADF ingestion layer. It does not contain Databricks transformation notebooks, Synapse models, or reporting artifacts.

## Author

**Neha Dudhane** — [GitHub](https://github.com/nehadudhane)
