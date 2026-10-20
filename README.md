<p align="center">
  <img src="assets/banner.svg" alt="AirBnB Cloudera Pipeline Banner" width="100%">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Cloudera_Hadoop_CDH-D97706?style=flat-square" alt="Cloudera">
  <img src="https://img.shields.io/badge/Engine-MapReduce_HDFS-3B82F6?style=flat-square" alt="HDFS">
  <img src="https://img.shields.io/badge/Dataset-74%2C111_Records-10B981?style=flat-square" alt="Dataset">
  <img src="https://img.shields.io/badge/Language-Python_3-3776AB?style=flat-square&logo=python" alt="Python">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-gray?style=flat-square" alt="License"></a>
</p>

## Overview

Analyzing rental market price dynamics across large urban datasets requires distributed batch processing to avoid single-node memory bottlenecks. This project implements an end-to-end distributed data processing pipeline on **Apache Hadoop** and **Cloudera CDH**, crunching 74,111 Airbnb listing records to extract pricing determinants, room-type distributions, and geographic fee variations.

The pipeline stages raw records on the Hadoop Distributed File System (HDFS), runs parallel MapReduce aggregation jobs in Python using Hadoop Streaming, and produces structured analytical summaries for econometric evaluation.

---

## Architecture Flow

```text
Raw Airbnb CSV (74,111 rows)
       │
       ▼
HDFS Cluster Staging (/user/cloudera/airbnb/input)
       │
       ▼
MapReduce Job 1: Mapper (Key extraction: city, room_type, accommodates)
       │
       ▼
Shuffle and Sort (Distributed grouping across worker nodes)
       │
       ▼
MapReduce Job 2: Reducer (Statistical mean, standard deviation, count)
       │
       ▼
HDFS Export (/user/cloudera/airbnb/output) ──► Pandas Analysis & Tuning
```

---

## Core Capabilities

- **Distributed HDFS Storage**: Ingests multi-megabyte structured rental datasets with replication fault tolerance across Cloudera virtual machine nodes.
- **Hadoop Streaming Execution**: Executes custom Python mapper and reducer scripts across commodity nodes without Java boilerplate.
- **Outlier Mitigation**: Filters zero-dollar listings and malformed bathroom records during mapping phases to prevent skewed aggregates.
- **Price Metric Extraction**: Computes average night rates, cleaning fee ratios, and review-to-price correlations across major metropolitan regions.

---

## Quickstart

### Prerequisites

- Cloudera QuickStart VM or Apache Hadoop 2.x/3.x cluster
- Python 3.6+ on cluster nodes

### Execution Steps

1. Clone this repository onto the Hadoop edge node:
   ```bash
   git clone https://github.com/MasterPandaa/AirBnB-Cloudera.git
   cd AirBnB-Cloudera
   ```

2. Load the dataset into HDFS:
   ```bash
   hdfs dfs -mkdir -p /user/cloudera/airbnb/input
   hdfs dfs -put dataset/Airbnb_Data.csv /user/cloudera/airbnb/input/
   ```

3. Run the MapReduce job via Hadoop Streaming:
   ```bash
   hadoop jar /usr/lib/hadoop-mapreduce/hadoop-streaming.jar \
     -input /user/cloudera/airbnb/input/Airbnb_Data.csv \
     -output /user/cloudera/airbnb/output \
     -mapper mapper.py \
     -reducer reducer.py \
     -file mapper.py \
     -file reducer.py
   ```

4. Inspect the aggregated results:
   ```bash
   hdfs dfs -cat /user/cloudera/airbnb/output/part-00000 | head -n 25
   ```

---

## Repository Structure

```text
AirBnB-Cloudera/
├── dataset/                # Airbnb sample records and schema definitions
├── mapper.py               # Hadoop streaming mapper script for feature extraction
├── reducer.py              # Statistical reducer aggregating price metrics
├── analysis_notebook.ipynb # Exploratory data analysis and regression modeling
└── README.md               # Pipeline documentation
```

---

## License

This project is licensed under the [MIT License](LICENSE).
