# Understanding Data Engineering

## What is data engineering?

### Data flow

1. Data Collection & Storage
2. Data Preparation
3. Exploration & Visualization
4. Experimentation & Prediction

Data engineers are responsible for the 1st phase, preparing the data for:

- Data analyists
- Data Scientists
- ML engineers

### Data engineer's responsibilities

- ingest data from different sources
- oprtimize databases for analysis
- remove corrupted data
- develop, construct, test and maintain data architectures

### Big Data

#### The five Vs

- Volume: how much?
- Variety: what kind?
- Velocity: how frequently is the data generated and processed?
- Veracity: how accurate/trustworthy is the data?
- Value: how useful/actionable is the data?

### Data piplines

Data pipelines ensure an efficient flow of the data by:

- Automating:
  - extracting
  - transforming
  - combining
  - validating
  - loading
- To reduce:
  - human intervention
  - errors
  - time it takes for data to move through an organization

## Storing Data

### Data Structures

#### Structured data

#### Semi-structured data

#### Unstructured data

### Data warehouses and data lakes

#### Data Lake

- stores raw data
- large
- stores all data structures
- cost effective
- difficult to analyze
- requires up-to-date data catalog
- big data, real-time analytics

#### Data warehouse

- specific data for a specific use
- smaller
- stores mainly structured data
- costly to update
- optimized for data anlysis
- ad-hoc, read-only queries

#### Data catalog

- Helps compensate for the lack of structure in a data lake
- Documents:
  - the source of the data
  - where is the data used
  - who is the owner of the data
  - how often is the data updated
- good practice for data governance
- ensures reproducibility
- no catalog = data swamp
- good practice for any data moving through the organization to ensure:
  - reliability
  - autonomy
  - scalability
  - speed
