# Formula_1# 🏎️  F1 Insights: Real-Time Replay & Historical Analytics 🏁

# 📖 Project Overview

A real-time Formula 1 telemetry system and historical data analysis system that captures, processes, and visualizes car data during race sessions, enabling both live replay and historical analysis.

The project involves a two fold analysis approach

- Realtime Race replay - To replay how the Race would pan out, and dive deep into what and how things changed that led to an outcome. (Would work during the race as well :) )
- Historical analysis - To answer important questions which involves post race analysis, about how weather impacts the outcome, tyre choices , race strategies and pit strategies

The system serves two primary analytical purposes:

### Real-Time Race Replay

Using live telemetry data, the system reconstructs race sessions in real-time, allowing viewers to experience the race as it unfolds. This feature provides deep insights into crucial moments by visualizing:

- Driver performance through speed, throttle, and brake data
- Strategic DRS (Drag Reduction System) usage
- Gear selection patterns across different track sections
- Dynamic position changes and overtaking maneuvers

### Historical Analysis & Strategic Insights

Beyond real-time replay, the system enables comprehensive post-race analysis to understand the intricate factors that influence race outcomes:

- Weather impact on tire degradation and pit stop timing
- Effectiveness of different pit strategies across varying track conditions
- Analysis of tire compound choices and their influence on race pace

By combining real-time telemetry with historical data, this platform serves as both an engaging visualization tool and a powerful analytical resource for understanding the technical and strategic elements that define Formula 1 racing.

# Motivation

Formula 1 racing represents the pinnacle of motorsport technology, where split-second decisions and minute performance differences determine victory. This project presents a comprehensive Formula 1 telemetry analysis system that brings together real-time data streaming and historical insights to unlock the complexities of race strategy and performance.

This has been a passion project of mine since long and i hope to keep improving and deploying it down the line for anyone to use and make sense of what factors led to the Race outcome

![High Level Architecture](./images/High_level_architecture_diagram_image_1.jpeg)

# Project Scope

- The project comprises 3 major sections  :
    - Real-time Streaming Pipeline using Confluent Kafka, SingleStore , AWS ec2 deployment
    - Historical Trend Analysis using DBT , Snowflake , Airflow (WAP maintaining idempodency)
    - Grafana (Interactive visualizations and deep dive)

## Dataset Choices

- For the most part i came across multiple data sources that help
    
    
    | Data_source | Data_source_type | Cost | Longevity | Support and community | Advantages |
    | --- | --- | --- | --- | --- | --- |
    | [Fast-F1](https://github.com/theOehrly/Fast-F1/tree/master?tab=readme-ov-file) | Python / pandas-based | Free | Din’t offer a robust API | 2.5k stars on github | 
    • Implements caching for all API requests to speed up your scripts |
    | [Ergast](http://ergast.com/mrd/) | REST-API | Free | will be [Depricated](http://ergast.com/mrd/limit-parameter-restriction/) (End of 2024) |  | Been around for a while, quite relaible with the data |
    | [OpenF1](https://openf1.org/?python#pit) | open-source API | Free | Quite active (as of today) | 368 stars |  |
    | [SportMonks](https://my.sportmonks.com/subscriptions/create/sport/2/plans) | API | Paid (65 euros/month) | Reliable | -  | Accurate well maintained and can be banked on  |
    
    My choice was OpenF1 for the following reasons:
    
    - The API is directly sourced from [F1 LiveTiming](https://www.formula1.com/en/timing/) , So in future after i subscribe the data json would look similar and i can directly source from the official site
    - Offers multiple data in depth with different API endpoints. essentially 1 API with multiple API endpoints that give different data (High frequency data and low frequency generic info)
    - Had structure and relationships between endpoints, that would let me correlate the effect of one over the other

## High level Architecture Diagram

![Image 06-12-24 at 8.31 PM.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d8acfbb-b3a9-4c9e-b434-052e2a7be4b5/1184628d-fa69-4644-9fec-52b55858ca47/Image_06-12-24_at_8.31_PM.jpeg)

## Technology choices:

### Real-time Streaming Pipeline

The real-time component leverages 

Confluent Kafka hosted on AWS EC2, paired with SingleStore as our operational database. This architecture was chosen for several compelling reasons:

- **Confluent Kafka** serves as our message broker because it excels at handling high-throughput, real-time data streams with minimal latency.
- Formula 1 telemetry generates thousands of data points per second across multiple cars especially the telemetry metrics like speed , throttle metrics, and Kafka's publish-subscribe model perfectly suits this use case. The platform's robust partitioning and fault tolerance ensure we never miss critical race data. Four topics in my producer that is consumed by singlestore pipelines
    
    ![Image 06-12-24 at 8.02 PM.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d8acfbb-b3a9-4c9e-b434-052e2a7be4b5/1d3facbd-76a9-430c-8fd0-0cd65f6ec728/Image_06-12-24_at_8.02_PM.jpeg)
    
    ## **SingleStore**
    
    **SingleStore** was selected as our operational database due to its unique ability to handle both real-time data ingestion and analytical queries simultaneously. Its columnar storage format and vector processing capabilities make it ideal for processing time-series telemetry data while maintaining sub-second query response times for our dashboards. Serving as both OLTP and OLAP database in one, which can perform high fidelity data ingests, analysis over millions of rows with ms latency  
    
    ![Image 06-12-24 at 8.05 PM.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d8acfbb-b3a9-4c9e-b434-052e2a7be4b5/a077e55d-563c-4e24-840b-9984d05027cc/65a9c105-720c-43e4-80f1-b0f49a519065.png)
    
- **AWS EC2** provides the scalable infrastructure needed to handle variable workloads during race weekends versus off-peak periods. The cloud deployment ensures high availability and enables easy scaling during peak racing events.

### Historical Trend Analysis

For deeper historical analysis, we implemented a robust data warehouse solution using:

- DBT (Data Build Tool) manages our data transformations, enabling us to create clean, tested, and documented data models. Its version control and modularity allow us to maintain complex racing analytics while ensuring data quality through built-in testing.
    
    
- Snowflake serves as our data warehouse, chosen for its separation of storage and compute resources. This architecture is particularly valuable for F1 analysis, where we might need to process years of historical race data while simultaneously handling real-time queries.
- Apache Airflow orchestrates our ETL workflows using the Write Audit Publish (WAP) pattern to maintain idempotency. This ensures our historical analysis remains accurate and reproducible, even when processing data from multiple racing seasons.

### Interactive Visualization

Grafana ties our entire solution together through interactive dashboards that serve both real-time and historical analysis needs. We chose Grafana for its:

- Ability to handle real-time data streams with  refresh rates as low as 1s
- Support for complex time-series visualizations essential for race telemetry
- Flexible query builders that work seamlessly with both SingleStore and Snowflake

## API responses converted into Tables (ERD)

```mermaid
erDiagram
    MEETINGS ||--o{ SESSIONS : contains
    MEETINGS ||--o{ DRIVERS : participates-in
    MEETINGS ||--o{ RACE_CONTROL : records
    MEETINGS ||--o{ WEATHER : tracks
    
    SESSIONS ||--o{ LAPS : includes
    SESSIONS ||--o{ CAR_DATA : records
    SESSIONS ||--o{ INTERVALS : tracks
    SESSIONS ||--o{ POSITION : monitors
    SESSIONS ||--o{ PIT : records
    SESSIONS ||--o{ STINTS : tracks
    SESSIONS ||--o{ TEAM_RADIO : captures
    SESSIONS ||--o{ LOCATION : tracks

    DRIVERS {
        int driver_number PK
        string first_name
        string last_name
        string full_name
        string country_code
        string team_name
        string team_colour
    }

    MEETINGS {
        int meeting_key PK
        int year
        string country_name
        string circuit_short_name
        string location
        datetime date_start
    }

    SESSIONS {
        int session_key PK
        int meeting_key FK
        string session_name
        string session_type
        datetime date_start
        datetime date_end
    }

    CAR_DATA {
        int driver_number FK
        int session_key FK
        datetime date
        int speed
        int rpm
        int throttle
        int brake
        int n_gear
        int drs
    }

    LAPS {
        int driver_number FK
        int session_key FK
        int lap_number
        float lap_duration
        float duration_sector_1
        float duration_sector_2
        float duration_sector_3
        int st_speed
    }

    INTERVALS {
        int driver_number FK
        int session_key FK
        datetime date
        float gap_to_leader
        float interval
    }

    POSITION {
        int driver_number FK
        int session_key FK
        datetime date
        int position
    }

    PIT {
        int driver_number FK
        int session_key FK
        datetime date
        int lap_number
        float pit_duration
    }

    RACE_CONTROL {
        int driver_number FK
        int session_key FK
        datetime date
        string category
        string flag
        string message
        string scope
    }

    STINTS {
        int driver_number FK
        int session_key FK
        string compound
        int lap_start
        int lap_end
        int tyre_age_at_start
    }

    TEAM_RADIO {
        int driver_number FK
        int session_key FK
        datetime date
        string recording_url
    }

    LOCATION {
        int driver_number FK
        int session_key FK
        datetime date
        int x
        int y
        int z
    }

    WEATHER {
        int session_key FK
        datetime date
        float air_temperature
        float track_temperature
        int humidity
        float wind_speed
        int wind_direction
    }
```

The processes have the following stages of data architecture:

## Data Ingestion

![Image 06-12-24 at 8.42 PM.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d8acfbb-b3a9-4c9e-b434-052e2a7be4b5/2ad164da-86df-4f0f-8639-1d01084704ac/Image_06-12-24_at_8.42_PM.jpeg)

I made sure during my ingestion these 5 properties are always maintained

### Idempodency ( Merge keys )

- No two non-unique recods are added to the table also ensuring upserts only
- Created a dynamic since the structure of the statement remains same
    
    ![Image 06-12-24 at 9.04 PM.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d8acfbb-b3a9-4c9e-b434-052e2a7be4b5/02fdc42a-1002-4061-87fa-4305c3307bfa/Image_06-12-24_at_9.04_PM.jpeg)
    
    ![Image 06-12-24 at 9.05 PM.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d8acfbb-b3a9-4c9e-b434-052e2a7be4b5/92296ff2-8356-4b9a-a427-2061e59cd4ff/Image_06-12-24_at_9.05_PM.jpeg)
    

## API call retires and Fallbacks

- So API’s are unreliable sometimes, hence having retry feature to make sure, no data is left behind during ingest
    
    ![Image 06-12-24 at 9.12 PM.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d8acfbb-b3a9-4c9e-b434-052e2a7be4b5/dacbce70-d682-485a-907b-c9f70367a55d/Image_06-12-24_at_9.12_PM.jpeg)
    
- Handling high frequency data
    - For telemetry high frequency data, i broke my api calls into 5 min chunks
    - Let’s say the API still produces too much data to handle, then it fallback into 1/2 the window size and then continues to maintain 5 min windows for the next call
        
        ![Image 06-12-24 at 8.55 PM.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d8acfbb-b3a9-4c9e-b434-052e2a7be4b5/e91140bf-b3e6-45b6-a50c-ce8bc52dd484/Image_06-12-24_at_8.55_PM.jpeg)
        

## Always ingest into a temp staging table

- Made sure the production table is not impacted by ingestion issues (always ingesting to a staging table and dropping it immediately after succesful merge)
    
    ![Image 06-12-24 at 8.49 PM.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d8acfbb-b3a9-4c9e-b434-052e2a7be4b5/907c8249-5113-4bdc-8e7d-febfb71b2cac/Image_06-12-24_at_8.49_PM.jpeg)
    

## Compute resources ( save costs)

- There is no Race every single day so i made sure i don’t exhaust the compute resources on a non race day
    
    ![Image 06-12-24 at 8.44 PM.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d8acfbb-b3a9-4c9e-b434-052e2a7be4b5/f277cc50-e3cf-407c-8d78-5cb6766e3e47/Image_06-12-24_at_8.44_PM.jpeg)
    

## Data Transformation

These are all my data transformations

- Anything prefixed with stg are my staging models.
- Prefixed with “f1_” are my data marts and used in final dashboards
- Prefixed with “int_” are my intermediate models used a lot in multiple marts as well
    
    ![Image 07-12-24 at 6.02 PM.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d8acfbb-b3a9-4c9e-b434-052e2a7be4b5/1eb8ba28-fca1-47b1-aecc-3e27920cce37/Image_07-12-24_at_6.02_PM.jpeg)
    

My Transformations on a high level include:

### Staging models ( Pulled from tested source  table ):

`dbt_project/models/staging`

- Things like (Changing datatype and names of my columns)
    - Changing datatype and names of my columns
        - `driver_number::VARCHAR(50) as driver_number,`
        - `TO_TIMESTAMP_NTZ(date) as timestamp,`
        - `NULLIF(gap_to_leader, 'None') as gap_to_leader`
        - `DATEADD(second, lap_duration, DATE_TRUNC('second', date_start)) AS date_end,`
    - Adding columns for time ranges
        - `DATE_TRUNC('second', date_start) AS date_start,`
    
    ### Intermediate models ( Pulled from tested staging  models and other int_models alone ): few example below
    
    `dbt_project/models/intermediate`
    
    - Weather (Computing averages and Categorizing my dashboards)
        
        ![Image 07-12-24 at 6.13 PM.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d8acfbb-b3a9-4c9e-b434-052e2a7be4b5/bb99c414-e048-4183-bfc9-626073aeb037/Image_07-12-24_at_6.13_PM.jpeg)
        
    - Computing lap wise start and end positions
        
        ![Image 07-12-24 at 6.15 PM.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d8acfbb-b3a9-4c9e-b434-052e2a7be4b5/c63047f3-0644-4c1a-bbf2-77d64322c563/Image_07-12-24_at_6.15_PM.jpeg)
        
        ## Marts (`dbt_project/models/marts`)
        
        Used directly in dashboards
        
        - There were only position changes , so i had to fill positions for the laps where there was no position change , to understand how the position evolved
            
            ![Image 07-12-24 at 6.22 PM.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d8acfbb-b3a9-4c9e-b434-052e2a7be4b5/b1426598-3365-4ef2-8cf3-e9e275ed9790/Image_07-12-24_at_6.22_PM.jpeg)
            
        - A mart for dashboard filters as well
            
            ![Image 07-12-24 at 6.23 PM.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d8acfbb-b3a9-4c9e-b434-052e2a7be4b5/4be70f81-3ec9-4d21-bf15-f3e20e64c0a8/Image_07-12-24_at_6.23_PM.jpeg)
            
    

## Data Validation

![Image 07-12-24 at 6.19 PM.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/3d8acfbb-b3a9-4c9e-b434-052e2a7be4b5/414b3a69-5465-4ae5-8b39-5e4fe9ecda86/Image_07-12-24_at_6.19_PM.jpeg)

## Data Storage

## Data Visualization

```sql
version: 2

sources:
  - name: HITESH
    database: DATAEXPERT_STUDENT
    schema: HITESH
    tables:
      - name: F1_POSITION
      - name: F1_LAPS
      - name : F1_CAR_DATA
      - name : F1_INTERVALS
      - name : F1_PIT
      - name : F1_STINT
      - name : F1_MEETINGS
      - name : F1_SESSIONS
      - name : F1_DRIVERS

```
