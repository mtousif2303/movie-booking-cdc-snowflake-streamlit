# movie-booking-cdc-snowflake-streamlit
Real-time movie booking analytics powered by Snowflake CDC. Implements Streams, Tasks, and Dynamic Tables for change data capture with automated processing. Features derived fields, business logic transformations, and an interactive Streamlit dashboard for comprehensive analytics and insights.

# 🎬 Movie Booking CDC Analytics Platform

[![Snowflake](https://img.shields.io/badge/Snowflake-CDC-29B5E8?logo=snowflake)](https://www.snowflake.com/)
[![Streamlit](https://img.shields.io/badge/Streamlit-Dashboard-FF4B4B?logo=streamlit)](https://streamlit.io/)
[![Python](https://img.shields.io/badge/Python-3.8+-3776AB?logo=python)](https://www.python.org/)

> Real-time movie booking analytics powered by Snowflake CDC with Streams, Tasks, and Dynamic Tables

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Data Flow](#-data-flow)
- [Task Orchestration](#-task-orchestration)
- [Features](#-features)
- [Quick Start](#-quick-start)
- [Database Schema](#-database-schema)
- [Dashboard](#-dashboard)
- [Monitoring](#-monitoring)
- [Contributing](#-contributing)

---

## 🎯 Overview

This project demonstrates a production-ready **Change Data Capture (CDC)** solution using Snowflake's streaming architecture. It captures real-time changes to movie bookings, processes them through automated tasks, enriches data with business logic, and visualizes insights through an interactive Streamlit dashboard.

### What is CDC?

**Change Data Capture (CDC)** tracks and captures every change (INSERT, UPDATE, DELETE) made to your data in real-time, enabling:
- ⚡ Real-time analytics
- 📊 Audit trails
- 🔄 Data synchronization
- 📈 Historical tracking

---

## 🏗️ Architecture

### System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          SNOWFLAKE ECOSYSTEM                             │
│                                                                          │
│  ┌──────────────────┐         ┌─────────────────┐                      │
│  │  Raw Movie       │────────>│  Stream         │                       │
│  │  Bookings        │         │  (Captures      │                       │
│  │  (Source)        │         │   Changes)      │                       │
│  └──────────────────┘         └────────┬────────┘                       │
│         │                              │                                 │
│         │ INSERT/UPDATE/DELETE         │ METADATA$ACTION                │
│         │                              │ METADATA$ISUPDATE              │
│         ▼                              ▼                                 │
│  ┌──────────────────────────────────────────────────┐                  │
│  │         Task: consume_stream_task                 │                  │
│  │         Schedule: Every 1 minute                  │                  │
│  │         Action: Process stream → CDC Events       │                  │
│  └─────────────────────┬────────────────────────────┘                  │
│                        │                                                 │
│                        ▼                                                 │
│  ┌──────────────────────────────────────────────────┐                  │
│  │     CDC Events Table (Bronze Layer)              │                  │
│  │     • Raw stream data                            │                  │
│  │     • Change metadata                            │                  │
│  │     • Timestamp tracking                         │                  │
│  └─────────────────────┬────────────────────────────┘                  │
│                        │                                                 │
│                        ▼                                                 │
│  ┌──────────────────────────────────────────────────┐                  │
│  │  Dynamic Table: movie_bookings_filtered          │                  │
│  │  (Silver Layer - Target Lag: DOWNSTREAM)         │                  │
│  │  • Derived fields (categories)                   │                  │
│  │  • Business logic                                │                  │
│  │  • Data quality validation                       │                  │
│  │  • Revenue calculations                          │                  │
│  └─────────────────────┬────────────────────────────┘                  │
│                        │                                                 │
│                        ▼                                                 │
│  ┌──────────────────────────────────────────────────┐                  │
│  │  Dynamic Table: movie_booking_insights           │                  │
│  │  (Gold Layer - Target Lag: DOWNSTREAM)           │                  │
│  │  • Aggregated analytics                          │                  │
│  │  • KPIs and metrics                              │                  │
│  │  • Business categorizations                      │                  │
│  └─────────────────────┬────────────────────────────┘                  │
│                        │                                                 │
└────────────────────────┼─────────────────────────────────────────────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │  Streamlit Dashboard │
              │  • Real-time viz     │
              │  • Interactive       │
              │  • Filters & export  │
              └──────────────────────┘
```

### Layer Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     BRONZE LAYER (Raw Data)                      │
├─────────────────────────────────────────────────────────────────┤
│  • raw_movie_bookings (Source Table)                            │
│  • movie_bookings_stream (Change Capture)                       │
│  • movie_booking_cdc_events (Raw CDC Events)                    │
│                                                                  │
│  Purpose: Capture and preserve all changes                      │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   SILVER LAYER (Enriched Data)                   │
├─────────────────────────────────────────────────────────────────┤
│  • movie_bookings_filtered (Dynamic Table)                      │
│                                                                  │
│  Transformations:                                               │
│  ✓ Business categorizations (ACTIVE/INACTIVE)                  │
│  ✓ Size categories (SINGLE/GROUP/LARGE_GROUP)                  │
│  ✓ Price categories (BUDGET/STANDARD/PREMIUM)                  │
│  ✓ Revenue calculations (active_revenue, lost_revenue)         │
│  ✓ Data quality validation                                     │
│  ✓ Time-based fields (hour, day_of_week)                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    GOLD LAYER (Analytics)                        │
├─────────────────────────────────────────────────────────────────┤
│  • movie_booking_insights (Dynamic Table)                       │
│                                                                  │
│  Analytics:                                                     │
│  ✓ Aggregated KPIs                                             │
│  ✓ Revenue metrics by category                                 │
│  ✓ Cancellation rates                                          │
│  ✓ Booking patterns                                            │
│  ✓ Data quality scores                                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Data Flow

### Complete Data Flow Diagram

```
┌─────────────┐
│   User      │
│  Actions    │
└──────┬──────┘
       │
       │ INSERT booking
       │ UPDATE status
       │ DELETE booking
       ▼
┌──────────────────────────────────────────┐
│  raw_movie_bookings                      │
│  ┌────────────────────────────────────┐  │
│  │ booking_id, customer_id, movie_id  │  │
│  │ status, ticket_count, price        │  │
│  │ created_at, updated_at             │  │
│  └────────────────────────────────────┘  │
└──────────────┬───────────────────────────┘
               │
               │ Stream monitors changes
               ▼
┌──────────────────────────────────────────┐
│  movie_bookings_stream                   │
│  ┌────────────────────────────────────┐  │
│  │ All original fields +              │  │
│  │ METADATA$ACTION (INSERT/UPDATE/    │  │
│  │                  DELETE)            │  │
│  │ METADATA$ISUPDATE (TRUE/FALSE)     │  │
│  └────────────────────────────────────┘  │
└──────────────┬───────────────────────────┘
               │
               │ Task processes every 1 min
               ▼
┌──────────────────────────────────────────┐
│  consume_stream_task                     │
│  ┌────────────────────────────────────┐  │
│  │ INSERT INTO cdc_events             │  │
│  │ SELECT * FROM stream               │  │
│  │ WHERE METADATA$ACTION IS NOT NULL  │  │
│  └────────────────────────────────────┘  │
└──────────────┬───────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────┐
│  movie_booking_cdc_events                │
│  ┌────────────────────────────────────┐  │
│  │ All booking fields                 │  │
│  │ + change_action                    │  │
│  │ + is_update                        │  │
│  │ + change_timestamp                 │  │
│  └────────────────────────────────────┘  │
└──────────────┬───────────────────────────┘
               │
               │ Dynamic Table refreshes (2 min lag)
               ▼
┌──────────────────────────────────────────┐
│  movie_bookings_filtered                 │
│  ┌────────────────────────────────────┐  │
│  │ Original fields                    │  │
│  │ + booking_status_category          │  │
│  │ + booking_size_category            │  │
│  │ + price_category                   │  │
│  │ + active_revenue                   │  │
│  │ + lost_revenue                     │  │
│  │ + is_valid_booking                 │  │
│  │ + booking_hour, day_of_week        │  │
│  └────────────────────────────────────┘  │
└──────────────┬───────────────────────────┘
               │
               │ Dynamic Table aggregates
               ▼
┌──────────────────────────────────────────┐
│  movie_booking_insights                  │
│  ┌────────────────────────────────────┐  │
│  │ movie_id                           │  │
│  │ total_bookings                     │  │
│  │ total_active_revenue               │  │
│  │ cancellation_rate_percent          │  │
│  │ active_bookings                    │  │
│  │ category breakdowns                │  │
│  │ data_quality_score                 │  │
│  └────────────────────────────────────┘  │
└──────────────┬───────────────────────────┘
               │
               │ Query for dashboard
               ▼
        ┌──────────────┐
        │  Streamlit   │
        │  Dashboard   │
        └──────────────┘
```

---

## ⚙️ Task Orchestration

### Task Execution Flow

```
┌─────────────────────────────────────────────────────────────┐
│              Task: consume_stream_task                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Schedule: Every 1 minute                                   │
│  Warehouse: COMPUTE_WH                                      │
│  Status: STARTED (RESUME required)                          │
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Execution Steps:                                   │    │
│  │                                                      │    │
│  │  1. Check if stream has new data                   │    │
│  │     └─> METADATA$ACTION IS NOT NULL                │    │
│  │                                                      │    │
│  │  2. Read all fields from stream                    │    │
│  │     ├─> Original booking data                      │    │
│  │     ├─> METADATA$ACTION (INSERT/UPDATE/DELETE)     │    │
│  │     └─> METADATA$ISUPDATE (TRUE/FALSE)             │    │
│  │                                                      │    │
│  │  3. Insert into CDC events table                   │    │
│  │     └─> Add change_timestamp                       │    │
│  │                                                      │    │
│  │  4. Stream automatically advances                  │    │
│  │     └─> Processed records removed from stream      │    │
│  │                                                      │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
│  Error Handling:                                            │
│  • Automatic retry on transient failures                   │
│  • Error logged to task history                            │
│  • Stream preserves data until successfully processed      │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Dynamic Table Refresh Flow

```
┌───────────────────────────────────────────────────────────────┐
│           Dynamic Table Refresh Mechanism                      │
├───────────────────────────────────────────────────────────────┤
│                                                                │
│  movie_bookings_filtered                                      │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  Target Lag: DOWNSTREAM                               │    │
│  │  Trigger: When cdc_events has new data               │    │
│  │                                                        │    │
│  │  Refresh Process:                                     │    │
│  │  1. Detect changes in source (cdc_events)           │    │
│  │  2. Calculate derived fields                         │    │
│  │  3. Apply business logic                             │    │
│  │  4. Validate data quality                            │    │
│  │  5. Update materialized view                         │    │
│  └──────────────────────────────────────────────────────┘    │
│                          │                                     │
│                          ▼                                     │
│  movie_booking_insights                                       │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  Target Lag: DOWNSTREAM                               │    │
│  │  Trigger: When filtered table updates                │    │
│  │                                                        │    │
│  │  Refresh Process:                                     │    │
│  │  1. Detect changes in filtered table                 │    │
│  │  2. Aggregate metrics by movie_id                    │    │
│  │  3. Calculate KPIs and ratios                        │    │
│  │  4. Update analytics view                            │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                                │
│  Manual Refresh:                                              │
│  ALTER DYNAMIC TABLE movie_bookings_filtered REFRESH;        │
│  ALTER DYNAMIC TABLE movie_booking_insights REFRESH;         │
│                                                                │
└───────────────────────────────────────────────────────────────┘
```

### Change Processing Timeline

```
Time    │ Event
────────┼──────────────────────────────────────────────────────
00:00   │ User updates booking B001 status → CANCELLED
        │
00:01   │ ✓ Stream captures change
        │   └─> METADATA$ACTION = 'UPDATE'
        │   └─> METADATA$ISUPDATE = TRUE
        │
00:01   │ ✓ Task runs (1-minute schedule)
        │   └─> Processes stream data
        │   └─> Inserts into cdc_events
        │   └─> Stream advances (clears processed data)
        │
00:03   │ ✓ Dynamic Table: movie_bookings_filtered refreshes
        │   └─> Detects new record in cdc_events
        │   └─> Calculates derived fields
        │   └─> booking_status_category = 'INACTIVE'
        │   └─> lost_revenue = total_amount
        │   └─> active_revenue = 0
        │
00:03   │ ✓ Dynamic Table: movie_booking_insights refreshes
        │   └─> Detects change in filtered table
        │   └─> Recalculates aggregations
        │   └─> Updates cancellation_rate_percent
        │   └─> Updates total_lost_revenue
        │
00:04   │ ✓ Dashboard queries insights table
        │   └─> Displays updated metrics
        │   └─> Shows increased cancellation rate
        │
Total   │ 4 minutes from change to dashboard visibility
Latency │
```

---

## ✨ Features

### 🔄 Real-time CDC Processing
- **Automatic Change Detection**: Captures all INSERT, UPDATE, DELETE operations
- **Complete Change History**: Preserves raw stream data with metadata
- **Near Real-time Updates**: 1-minute task execution, 2-minute table refresh
- **Audit Trail**: Full timestamp tracking (created_at, updated_at, change_timestamp)

### 📊 Enhanced Analytics
- **Business Categorizations**:
  - Status: ACTIVE (BOOKED) vs INACTIVE (CANCELLED)
  - Size: SINGLE, GROUP, LARGE_GROUP
  - Price: BUDGET, STANDARD, PREMIUM
- **Revenue Analysis**: Active revenue vs lost revenue tracking
- **Data Quality**: Built-in validation and quality scoring
- **Time-based Insights**: Hour, day of week patterns

### 🎨 Interactive Dashboard
- **Essential Filters**: Date range, status, movie selection
- **Key Metrics**: Bookings, revenue, active/lost revenue
- **Visualizations**: Bar charts, pie charts, performance tables
- **Export**: CSV download with timestamps

---

## 🚀 Quick Start

### Prerequisites
- Snowflake account with appropriate privileges
- Access to `COMPUTE_WH` warehouse
- Python 3.8+ with Streamlit

### Installation

1. **Clone the repository**
```bash
git clone https://github.com/yourusername/snowflake-cdc-streamlit-dashboard.git
cd snowflake-cdc-streamlit-dashboard
```

2. **Execute SQL script in Snowflake**
```sql
-- Run snowflake_dynamic_tables.sql
-- This creates all tables, streams, tasks, and dynamic tables
```

3. **Verify setup**
```sql
-- Check objects
SHOW TABLES;
SHOW STREAMS;
SHOW TASKS;
SHOW DYNAMIC TABLES;

-- Verify task is running
SELECT * FROM TABLE(INFORMATION_SCHEMA.TASK_HISTORY(
    TASK_NAME => 'consume_stream_task'
)) ORDER BY SCHEDULED_TIME DESC LIMIT 5;
```

4. **Launch Streamlit dashboard**
```bash
streamlit run streamlit_app.py
```

### Sample Data
The project includes 5 initial bookings across different movies:
- Booking IDs: B001-B005
- Movies: M001-M005
- Price range: $10-$25
- September 2025 data

---

## 📊 Database Schema

### Core Tables

#### raw_movie_bookings (Source)
```sql
booking_id        STRING           -- Unique identifier
customer_id       STRING           -- Customer identifier
movie_id          STRING           -- Movie identifier
booking_date      TIMESTAMP        -- Booking timestamp
status            STRING           -- BOOKED, CANCELLED
ticket_count      INT              -- Number of tickets
ticket_price      NUMBER(10,2)     -- Price per ticket
total_amount      NUMBER(10,2)     -- Computed: count × price
created_at        TIMESTAMP        -- Auto: creation time
updated_at        TIMESTAMP        -- Auto: last update time
```

#### movie_booking_cdc_events (Bronze)
```sql
-- All fields from source table plus:
change_action     STRING           -- INSERT, UPDATE, DELETE
is_update         BOOLEAN          -- TRUE for updates
change_timestamp  TIMESTAMP        -- When change captured
```

#### movie_bookings_filtered (Silver)
```sql
-- All CDC fields plus derived fields:
booking_status_category   STRING   -- ACTIVE, INACTIVE
booking_size_category     STRING   -- SINGLE, GROUP, LARGE_GROUP
price_category            STRING   -- BUDGET, STANDARD, PREMIUM
active_revenue           NUMBER    -- Revenue if BOOKED
lost_revenue             NUMBER    -- Revenue if CANCELLED
is_valid_booking         BOOLEAN   -- Data quality flag
booking_hour             INT       -- Hour of booking
day_of_week              INT       -- Day of week
```

#### movie_booking_insights (Gold)
```sql
movie_id                      STRING
total_bookings                INT
total_active_revenue          NUMBER
total_lost_revenue            NUMBER
active_bookings               INT
cancelled_bookings            INT
cancellation_rate_percent     NUMBER
-- Plus category breakdowns and metrics
```

---

## 📈 Dashboard

### Features
- **Real-time Metrics**: Total bookings, revenue, active/lost revenue
- **Interactive Filters**: Date range, status, movie selection
- **Visualizations**:
  - Revenue by status (bar chart)
  - Booking distribution (pie chart)
  - Movie performance (table)
  - Real-time insights (dynamic table)
- **Export**: CSV download with timestamp

### Screenshots

```
┌────────────────────────────────────────────────────────────┐
│  🎬 Movie Booking Analytics Dashboard                      │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  📊 Key Metrics                                            │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │
│  │  Total   │ │  Total   │ │  Active  │ │   Lost   │    │
│  │Bookings  │ │ Revenue  │ │ Revenue  │ │ Revenue  │    │
│  │   150    │ │ $2,500   │ │ $2,100   │ │  $400    │    │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘    │
│                                                             │
│  📈 Revenue by Status          🎫 Booking Distribution    │
│  ┌─────────────────┐           ┌─────────────────┐       │
│  │   Bar Chart     │           │   Pie Chart     │       │
│  │                 │           │                 │       │
│  └─────────────────┘           └─────────────────┘       │
│                                                             │
│  🎬 Movie Performance                                      │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ Movie │ Bookings │ Revenue │ Tickets │            │  │
│  │ M001  │    45    │  $750   │   90    │            │  │
│  └─────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

---

## 🔍 Monitoring

### Task Monitoring
```sql
-- Check task execution history
SELECT * FROM TABLE(INFORMATION_SCHEMA.TASK_HISTORY(
    TASK_NAME => 'consume_stream_task'
)) ORDER BY SCHEDULED_TIME DESC LIMIT 10;

-- Check task status
SHOW TASKS LIKE 'consume_stream_task';
```

### Stream Monitoring
```sql
-- Check stream contents
SELECT * FROM movie_bookings_stream;

-- Count pending changes
SELECT COUNT(*) as pending_changes 
FROM movie_bookings_stream;
```

### Dynamic Table Monitoring
```sql
-- Check refresh history
SELECT * FROM TABLE(
    INFORMATION_SCHEMA.DYNAMIC_TABLE_REFRESH_HISTORY(
        TABLE_NAME => 'movie_bookings_filtered'
    )
) ORDER BY REFRESH_START_TIME DESC LIMIT 5;

-- Check table status
SHOW DYNAMIC TABLES LIKE 'movie_booking%';
```

---

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

### Development Setup
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

---

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

## 🙏 Acknowledgments

- Snowflake for powerful CDC capabilities
- Streamlit for easy dashboard creation
- The data engineering community

---

**Built with ❤️ using Snowflake, Streamlit, and Python**

*Last Updated: January 2026*
