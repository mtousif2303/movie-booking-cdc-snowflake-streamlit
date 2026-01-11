# movie-booking-cdc-snowflake-streamlit
Real-time movie booking analytics powered by Snowflake CDC. Implements Streams, Tasks, and Dynamic Tables for change data capture with automated processing. Features derived fields, business logic transformations, and an interactive Streamlit dashboard for comprehensive analytics and insights.

This  demonstrates a production-ready **Change Data Capture (CDC)** solution using Snowflake's streaming architecture. It captures real-time changes to movie bookings, processes them through automated tasks, enriches data with business logic, and visualizes insights through an interactive Streamlit dashboard.

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

## Analytics dashboard

<img width="2874" height="1758" alt="image" src="https://github.com/user-attachments/assets/24691e50-297d-4afb-ae35-406a55a272c9" />

The Dynamic silver and Gold layer tables

<img width="2968" height="1762" alt="image" src="https://github.com/user-attachments/assets/7f7272fc-934b-4070-8673-6b0f7f645851" />



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

### Enhanced Data Flow

```
Raw Bookings → Stream → CDC Events → Enhanced Filtered Table → Analytics Dashboard
     ↓              ↓         ↓              ↓                    ↓
  INSERT/      Captures   Raw Stream    Derived Fields        Interactive
 UPDATE/       Changes    Data +        + Business Logic      Visualization
DELETE                     Metadata     + Data Quality
```

### Key Enhancements

- **Derived Fields**: Business categorizations (ACTIVE/INACTIVE, SINGLE/GROUP, BUDGET/PREMIUM)
- **Data Quality**: Built-in validation and quality scoring
- **Enhanced Analytics**: Rich metrics with business context
- **Simplified Dashboard**: Clean interface focused on essential features

## 📊 Key Features

### Real-time CDC Processing
- **Automatic Change Detection**: Streams capture all INSERT, UPDATE, DELETE operations
- **Raw Stream Data**: Complete change history with metadata
- **Near Real-time Processing**: Tasks run every minute, dynamic tables refresh every 2 minutes
- **Timestamp Tracking**: Automatic created_at and updated_at management

### Enhanced Analytics with Derived Fields
- **Business Categorizations**: 
  - Status Categories: ACTIVE (BOOKED) vs INACTIVE (CANCELLED)
  - Size Categories: SINGLE, GROUP, LARGE_GROUP based on ticket count
  - Price Categories: BUDGET, STANDARD, PREMIUM based on ticket price
- **Revenue Analysis**: Active revenue vs lost revenue tracking
- **Data Quality Metrics**: Built-in validation and quality scoring
- **Time-based Analysis**: Booking hour, day of week patterns

### Simplified Dashboard
- **Essential Filters**: Date range, booking status, movie selection
- **Key Metrics**: Total bookings, revenue, active/lost revenue
- **Core Visualizations**: Revenue by status, booking distribution, movie performance
- **Clean Interface**: Beginner-friendly design with focused functionality
- **Export Capabilities**: Download filtered data as CSV

## 🚀 Getting Started

### Prerequisites

- Snowflake account with appropriate privileges
- Access to `COMPUTE_WH` warehouse
- Streamlit environment (for dashboard)

### Setup Instructions

1. **Execute SQL Script**:
   ```sql
   -- Run the complete snowflake_dynamic_tables.sql script
   -- This will create all tables, streams, tasks, and dynamic tables
   ```

2. **Verify Setup**:
   ```sql
   -- Check that all objects are created successfully
   SHOW TABLES;
   SHOW STREAMS;
   SHOW TASKS;
   SHOW DYNAMIC TABLES;
   ```

3. **Run Streamlit Dashboard**:
   ```bash
   streamlit run streamlit_app.py
   ```

### Sample Data

The project includes realistic sample data with:
- **5 initial bookings** across 5 different movies (September 2025)
- **Booking statuses**: BOOKED and CANCELLED (simplified for clarity)
- **Various ticket prices** ($10-$25) and quantities (1-4 tickets)
- **Time-stamped transactions** with automatic created_at/updated_at tracking
- **Realistic movie data**: Popular movies with different price points

## 📋 Database Schema

### Source Table: `raw_movie_bookings`
```sql
CREATE TABLE raw_movie_bookings (
    booking_id STRING,                    -- Unique booking identifier
    customer_id STRING,                   -- Customer identifier  
    movie_id STRING,                      -- Movie identifier
    booking_date TIMESTAMP,               -- When booking was made
    status STRING,                        -- BOOKED, CANCELLED (simplified)
    ticket_count INT,                     -- Number of tickets
    ticket_price NUMBER(10, 2),           -- Price per ticket
    total_amount NUMBER(10, 2) AS (ticket_count * ticket_price), -- Computed total
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP(),
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP()
);
```

### CDC Events Table: `movie_booking_cdc_events`
```sql
CREATE TABLE movie_booking_cdc_events (
    -- All original booking fields
    booking_id STRING,
    customer_id STRING,
    movie_id STRING,
    booking_date TIMESTAMP,
    status STRING,
    ticket_count INT,
    ticket_price NUMBER(10, 2),
    total_amount NUMBER(10, 2),
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    -- CDC metadata
    change_action STRING,                 -- INSERT, UPDATE, DELETE
    is_update BOOLEAN,                    -- TRUE for updates
    change_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP()
);
```

### Enhanced Filtered Table: `movie_bookings_filtered`
```sql
-- Dynamic table with derived fields and business logic
CREATE DYNAMIC TABLE movie_bookings_filtered AS
SELECT
    -- Original fields
    booking_id, customer_id, movie_id, booking_date, status,
    ticket_count, ticket_price, total_amount, created_at, updated_at,
    change_action, is_update, change_timestamp,
    
    -- Derived business fields
    CASE 
        WHEN status = 'BOOKED' THEN 'ACTIVE'
        WHEN status = 'CANCELLED' THEN 'INACTIVE'
    END AS booking_status_category,
    
    CASE 
        WHEN ticket_count = 1 THEN 'SINGLE'
        WHEN ticket_count BETWEEN 2 AND 4 THEN 'GROUP'
        WHEN ticket_count >= 5 THEN 'LARGE_GROUP'
    END AS booking_size_category,
    
    CASE 
        WHEN ticket_price < 10 THEN 'BUDGET'
        WHEN ticket_price BETWEEN 10 AND 20 THEN 'STANDARD'
        WHEN ticket_price > 20 THEN 'PREMIUM'
    END AS price_category,
    
    -- Revenue analysis
    CASE WHEN status = 'BOOKED' THEN total_amount ELSE 0 END AS active_revenue,
    CASE WHEN status = 'CANCELLED' THEN total_amount ELSE 0 END AS lost_revenue,
    
    -- Data quality
    CASE 
        WHEN booking_id IS NULL OR customer_id IS NULL OR movie_id IS NULL THEN FALSE
        WHEN ticket_count <= 0 OR ticket_price <= 0 THEN FALSE
        ELSE TRUE
    END AS is_valid_booking

FROM movie_booking_cdc_events
WHERE booking_id IS NOT NULL AND customer_id IS NOT NULL;
```

## 🔄 CDC Processing Logic

### Stream Processing
- **Automatic Capture**: Streams automatically detect all changes to source table
- **Metadata Addition**: Adds `METADATA$ACTION` and `METADATA$ISUPDATE` columns
- **Raw Data Storage**: Complete change history preserved in CDC events table

### Task Automation (`consume_stream_task`)
- **Scheduled Execution**: Runs every minute to process new changes
- **Raw Stream Consumption**: Populates CDC events table with complete stream data
- **Metadata Preservation**: Maintains all original fields plus change metadata
- **Error Handling**: Built-in retry and error logging capabilities

### Dynamic Table Processing
- **Enhanced Filtered Table**: 2-minute refresh lag, consumes from CDC events
- **Derived Field Calculation**: Business logic applied during refresh
- **Data Quality Filtering**: Invalid records filtered out automatically
- **Analytics Table**: Downstream refresh, aggregates from filtered table

## 📈 Analytics Capabilities

### Enhanced Key Performance Indicators
- **Total Bookings**: Count of all valid booking transactions
- **Active Revenue**: Revenue from BOOKED status bookings
- **Lost Revenue**: Revenue from CANCELLED status bookings
- **Data Quality Score**: Percentage of valid bookings
- **Cancellation Rate**: Percentage of cancelled bookings

### Business Categorization Analytics
- **Status Categories**: ACTIVE vs INACTIVE booking analysis
- **Size Categories**: SINGLE, GROUP, LARGE_GROUP booking patterns
- **Price Categories**: BUDGET, STANDARD, PREMIUM revenue analysis
- **Change Tracking**: INSERT, UPDATE, DELETE operation metrics

### Movie Performance Metrics
- **Revenue Analysis**: Active revenue vs lost revenue by movie
- **Booking Volume**: Total bookings with validity checks
- **Category Breakdown**: Bookings by size and price categories
- **Change Metrics**: New bookings, status changes, deletions

### Time-based Analysis
- **Date Range Filtering**: Flexible date range selection
- **Booking Hour Analysis**: Peak booking times
- **Day of Week Patterns**: Weekly booking trends
- **Real-time Updates**: 2-minute refresh for current insights

## 🎯 Use Cases

### Business Intelligence
- **Revenue Optimization**: Identify high-performing movies and time slots
- **Customer Behavior**: Analyze booking patterns and preferences
- **Operational Efficiency**: Monitor cancellation rates and booking trends

### Real-time Monitoring
- **Live Dashboard**: Monitor booking activity in real-time
- **Alert Systems**: Set up notifications for unusual patterns
- **Performance Tracking**: Track key metrics as they change

### Data Quality
- **Change Tracking**: Complete audit trail of all data modifications
- **Data Lineage**: Track data flow from source to analytics
- **Compliance**: Maintain historical records for regulatory requirements

## 🔧 Configuration Options

### Task Scheduling
```sql
-- Modify task frequency
ALTER TASK consume_stream_task 
SET SCHEDULE = '30 SECONDS';  -- More frequent processing

-- Suspend/resume tasks
ALTER TASK consume_stream_task SUSPEND;
ALTER TASK consume_stream_task RESUME;
```

### Dynamic Table Settings
```sql
-- Modify filtered table refresh frequency
ALTER DYNAMIC TABLE movie_bookings_filtered 
SET TARGET_LAG = '1 MINUTE';  -- More frequent refresh

-- Manual refresh
ALTER DYNAMIC TABLE movie_bookings_filtered REFRESH;
ALTER DYNAMIC TABLE movie_booking_insights REFRESH;
```

### Warehouse Configuration
```sql
-- Use different warehouse for processing
ALTER TASK consume_stream_task 
SET WAREHOUSE = 'ANALYTICS_WH';
```

## 📊 Dashboard Features

### Essential Interactive Filters
- **Date Range Selection**: Default September 2025, flexible date range
- **Status Filtering**: BOOKED, CANCELLED, or All bookings
- **Movie Selection**: Individual movie analysis or All movies
- **Refresh Button**: Manual data refresh capability

### Core Visualizations
- **Revenue by Status**: Bar chart showing active vs lost revenue
- **Booking Distribution**: Pie chart of booking status breakdown
- **Movie Performance Table**: Detailed metrics by movie
- **Real-time Insights**: Live analytics from dynamic tables

### Export and Navigation
- **CSV Download**: Export filtered data with timestamp
- **Raw Data View**: Expandable section for detailed data inspection
- **Clean Interface**: Beginner-friendly design with essential features
- **Responsive Layout**: Optimized for different screen sizes

## 🚨 Monitoring and Troubleshooting

### Task Monitoring
```sql
-- Check task execution history
SELECT * FROM TABLE(INFORMATION_SCHEMA.TASK_HISTORY(
    TASK_NAME => 'consume_stream_task'
)) ORDER BY SCHEDULED_TIME DESC;

-- Check task status
SHOW TASKS;
```

### Stream and CDC Monitoring
```sql
-- Check stream data
SELECT * FROM movie_bookings_stream;

-- Check CDC events (raw stream data)
SELECT * FROM movie_booking_cdc_events 
ORDER BY change_timestamp DESC 
LIMIT 10;

-- Check filtered data with derived fields
SELECT booking_id, status, booking_status_category, 
       booking_size_category, price_category, active_revenue, lost_revenue
FROM movie_bookings_filtered 
ORDER BY change_timestamp DESC;
```

### Dynamic Table Status
```sql
-- Check dynamic table refresh status
SHOW DYNAMIC TABLES;

-- Check refresh history for filtered table
SELECT * FROM TABLE(INFORMATION_SCHEMA.DYNAMIC_TABLE_REFRESH_HISTORY(
    TABLE_NAME => 'movie_bookings_filtered'
)) ORDER BY REFRESH_START_TIME DESC LIMIT 5;

-- Check analytics table refresh
SELECT * FROM TABLE(INFORMATION_SCHEMA.DYNAMIC_TABLE_REFRESH_HISTORY(
    TABLE_NAME => 'movie_booking_insights'
)) ORDER BY REFRESH_START_TIME DESC LIMIT 5;
```
