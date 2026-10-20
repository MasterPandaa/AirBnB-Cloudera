# AirBnB Big Data Analytics — Cloudera Hadoop & MapReduce Pipeline

[![Hadoop](https://img.shields.io/badge/Big_Data-Apache_Hadoop_3.x-66CCFF?style=flat&logo=apache-hadoop&logoColor=black)](#)
[![Platform](https://img.shields.io/badge/Environment-Cloudera_CDH-FF6600?style=flat)](#)
[![Engine](https://img.shields.io/badge/Computing-HDFS%20%7C%20MapReduce-yellow.svg)](#)
[![Dataset](https://img.shields.io/badge/Dataset-74%2C111%20Rows%20(101.55%20MB)-success.svg)](#)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A distributed big data processing and pricing analytics pipeline executed on the **AirBnB dataset (74,111 records, 101.55 MB)** utilizing **Cloudera CDH**, **HDFS**, and **Hadoop MapReduce Streaming**.

This project analyzes rental rate determinants by isolating features such as room counts, property classifications, and amenity sets across US metropolitan markets, producing aggregated data ready for BI dashboards and econometric modeling.

---

## Overview

Processing high-cardinality multi-feature rental datasets on single-node environments often encounters I/O and memory bottlenecks. This pipeline utilizes Hadoop's distributed computing architecture:

- **HDFS Distributed Storage**: Distributes raw listing records across HDFS blocks, enabling resilient parallel reads.
- **MapReduce Streaming Paradigm**: Custom Python mappers and reducers parse semi-structured amenity arrays and calculate grouped pricing statistics.
- **Amenity Valuation Metrics**: Quantifies the price premiums associated with high-demand amenities (High-Speed WiFi, Air Conditioning, Pool, Kitchen, Dedicated Parking).
- **Bedroom Capacity Segmentation**: Aggregates mean and median pricing tiers from studios to multi-bedroom estates.
- **Export Artifacts**: Produces clean CSV exports (`price_by_amenity.csv`, `price_by_bedrooms.csv`) for consumption in visualization tools.

---

## Dataset Specifications

| Metric | Specification |
| :--- | :--- |
| **Source** | [Kaggle - Airbnb Price Dataset](https://www.kaggle.com/datasets/rupindersinghrana/airbnb-price-dataset) |
| **Record Volume** | 74,111 rows |
| **Feature Dimensions** | 29 columns |
| **Raw File Size** | 101.55 MB (`.csv`) |
| **Key Attributes** | `id`, `log_price`, `property_type`, `room_type`, `amenities`, `accommodates`, `bathrooms`, `bedrooms`, `city`, `review_scores_rating` |

---

## Pipeline Architecture

```text
┌─────────────────────────────────┐
│     Airbnb_Data.csv (101 MB)    │  Raw Input Dataset in HDFS
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│         HDFS INGESTION          │  hdfs dfs -put Airbnb_Data.csv /user/cloudera/airbnb/
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│       MAPPER PROCESSING         │  CSV row parsing, Key-Value pair extraction (Bedrooms/Amenities -> Price)
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│      SHUFFLE & SORT PHASE       │  Cluster-wide key partitioning and sorting
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│       REDUCER AGGREGATION       │  Statistical calculation (Record count, Average price, Median)
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│      OUTPUT CSV ARTIFACTS       │  Aggregated analytics exported from HDFS to local storage
└─────────────────────────────────┘
```

---

## Execution Guide (Cloudera CDH)

### 1. Ingest Data into HDFS

```bash
# Create working directory on HDFS
hdfs dfs -mkdir -p /user/cloudera/airbnb/input

# Upload dataset to HDFS
hdfs dfs -put Airbnb_Data.csv /user/cloudera/airbnb/input/

# Verify upload
hdfs dfs -ls /user/cloudera/airbnb/input/
```

### 2. Execute MapReduce Jobs

Run Hadoop Streaming job with custom Python mappers and reducers:

```bash
# Example: Aggregating price by bedroom count
hadoop jar /usr/lib/hadoop-mapreduce/hadoop-streaming.jar \
  -files mapper_bedrooms.py,reducer_bedrooms.py \
  -mapper "python3 mapper_bedrooms.py" \
  -reducer "python3 reducer_bedrooms.py" \
  -input /user/cloudera/airbnb/input/Airbnb_Data.csv \
  -output /user/cloudera/airbnb/output_bedrooms
```

### 3. Extract Results

```bash
# Inspect outputs on HDFS
hdfs dfs -cat /user/cloudera/airbnb/output_bedrooms/part-00000 | head -n 20

# Export to local filesystem
hdfs dfs -getmerge /user/cloudera/airbnb/output_bedrooms price_by_bedrooms.csv
```

---

## License

This project is licensed under the [MIT License](LICENSE).
