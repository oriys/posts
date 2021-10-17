---
title: "Database Internals 摘录"
date: 2021-02-18T22:30:43+08:00
draft: true
---

## Storage Engines

- Introduction and Overview
  - serve different purposes
    - store temporary hot data
    - long-lived cold storage
    - complex analytical queries
    - accessing values by the key
    - optimized to store time-series data
    - store large blobs efficiently
  - three major categories
    - Online transaction processing (OLTP) databases
    - Online analytical processing (OLAP) databases
    - Hybrid transactional and analytical processing (HTAP)
  - DBMS Architecture
    - Transport
      - Cluster Communication
      - Client Communication
    - Query Processor
      - Query Parser
      - Query Optimizer
    - Execution Engine
      - Remote Execution
      - Local Execution
    - Storae Engine
      - Transaction Manage
      - Lock Manage
      - Access Methods
      - Buffer Manager
      - Recovery Manager
  - Memory-Versus Disk-Based DBMS
    - Distinctions
      - In-memory database management systems store data primarily in memory and use the disk for recovery and logging
      - Disk-based DBMS hold most of the data on disk and use memory for caching disk contents or as a temporary storage
      - Main memory database systems are different from their disk-based counterparts not only in terms of a primary storage medium, but also in which data structures, organi‐ zation, and optimization techniques they use
    - limiting factors on the growth of in-memory databases
      - RAM volatility and costs
