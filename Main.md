# Project Overview
**Project Title:** Cyclistic Data Analysis: Case Study 1  
**Objective:** Understand the variations in bike usage patterns between annual members and casual riders, informing a strategic marketing approach to convert casual riders into annual members.

## Background
Cyclistic, a Chicago-based bike-share program, boasts a diverse fleet that includes reclining bikes, hand tricycles, and cargo bikes, promoting inclusivity for riders with disabilities. Despite the popularity of traditional bikes, assistive options account for about 8% of users. While leisure rides dominate, approximately 30% of users rely on Cyclistic for daily commuting. The company aims to shift its marketing focus to maximize annual memberships, known to be more profitable.

## Scenario
As a junior data analyst at Cyclistic, I am tasked with understanding the nuances of how annual members and casual riders utilize Cyclistic bikes. This exploration serves as a foundation for crafting a compelling marketing strategy to convert casual riders into annual members. The success of these recommendations hinges on robust data insights and professional visualizations, crucial for obtaining executive approval.

## Data Analysis Process
In line with the Google Data Analytics Professional Capstone, the following steps will guide the analysis:

### Step 1: Ask
**Key Task:** Devise marketing tactics for converting casual riders into Cyclistic members.
**Analysis Questions:**
- How do annual members and casual riders differ in their usage of Cyclistic bikes?
- What motivates casual riders to opt for Cyclistic annual memberships?
- How can Cyclistic leverage digital media to facilitate the conversion of casual riders into members?

**Focus Question:** How do annual members and casual riders use Cyclistic bikes differently?

### Step 2: Prepare
**Data Source:** Cyclist's historical trip data, a publicly available dataset from Motivate International Inc., spanning from January 2020 to December 2023, consisting of 36 files, each following the naming convention "YYYYMM-divvy-tripdata."

**Data Information:**
- ride_id
- rideable_type
- started_at
- ended_at
- start_station_name
- start_station_id
- end_station_name
- end_station_id
- start_lat
- start_lng
- end_lat
- end_lng
- Member_casual

**Organization and Naming Convention:** The files are structured with a consistent naming convention, facilitating easy identification and retrieval. The analysis will involve merging and processing these files to derive insights into the differences in bike usage patterns between annual members and casual riders.

### Merge All Files into One Data Table
```sql
DROP TABLE FINAL20
CREATE TABLE FINAL20
(
    ride_id NVARCHAR(50) NULL,
    rideable_type NVARCHAR(50) NULL,
    started_at date NULL,
    ended_at date NULL,
    start_station_name VARCHAR(MAX) NULL,
    start_station_id VARCHAR(MAX) NULL,
    end_station_name VARCHAR(MAX) NULL,
    end_station_id VARCHAR(MAX) NULL,
    start_lat FLOAT NULL,
    start_lng FLOAT NULL,
    end_lat FLOAT NULL,
    end_lng FLOAT NULL,
    member_casual NVARCHAR(50) NULL
);

-- INSERT INTO FINAL
INSERT INTO FINAL20
SELECT *
FROM (
    SELECT * FROM C1..z013 
    UNION ALL    
    SELECT * FROM C1..z04
	UNION ALL
	SELECT * FROM C1..z05
	UNION ALL
    SELECT * FROM C1..z06
	UNION ALL
    SELECT * FROM C1..z07
	UNION ALL
    SELECT * FROM C1..z08
	UNION ALL
    SELECT * FROM C1..z09
	UNION ALL
    SELECT * FROM C1..z10
	UNION ALL
    SELECT * FROM C1..z11
	UNION ALL
    SELECT * FROM C1..z12
) AS CombinedData20;

SELECT *
FROM FINAL20

```

## Step 3: Process

## Data Exploration Queries 

### Missing Values Analysis 

```sql
-- Check missing values for each column in the dataset
SELECT 
    COUNT(*) - COUNT(ride_id) AS ride_id,
    COUNT(*) - COUNT(rideable_type) AS rideable_type,
    COUNT(*) - COUNT(started_at) AS started_at,
    COUNT(*) - COUNT(ended_at) AS ended_at,
    COUNT(*) - COUNT(start_station_name) AS start_station_name,
    COUNT(*) - COUNT(start_station_id) AS start_station_id,
    COUNT(*) - COUNT(end_station_name) AS end_station_name,
    COUNT(*) - COUNT(end_station_id) AS end_station_id,
    COUNT(*) - COUNT(start_lat) AS start_lat,
    COUNT(*) - COUNT(start_lng) AS start_lng,
    COUNT(*) - COUNT(end_lat) AS end_lat,
    COUNT(*) - COUNT(end_lng) AS end_lng,
    COUNT(*) - COUNT(member_casual) AS member_casual
FROM FINAL20;


```
## Remove Duplicate Rows.
```
SELECT COUNT(ride_id) - COUNT(DISTINCT ride_id) AS duplicate_rows
FROM FINAL20;

WITH DuplicatesCTE AS (
    SELECT ride_id,
           ROW_NUMBER() OVER (PARTITION BY ride_id ORDER BY ride_id) AS RowNum
    FROM FINAL20
)
DELETE FROM DuplicatesCTE WHERE RowNum > 1;
```

##Trip Type Distribution ##
```
SELECT COUNT(*) AS longer_than_1_day
FROM FINAL20
WHERE (
    DATEDIFF(MINUTE, started_at, ended_at) >= 125
);

SELECT COUNT(*) AS less_than_a_minute
FROM FINAL20
WHERE DATEDIFF(MINUTE, started_at, ended_at) <= 1;
```

##Member Type Distribution
```

SELECT DISTINCT member_casual, COUNT(*) AS count_member_type
FROM FINAL20
GROUP BY member_casual;
```
##Station Information
------------------------------------------------------------------------------------------------------
```
SELECT DISTINCT start_station_name
FROM FINAL20
ORDER BY start_station_name;

SELECT COUNT(ride_id) AS start_station_null   
FROM FINAL20
WHERE start_station_name IS NULL OR start_station_id IS NULL;

DELETE FROM FINAL20
WHERE start_station_name IS NULL OR start_station_id IS NULL;  -----(83583 rows affected)--------
```
--------------------------------------------------------------------------------------------------------
```
SELECT DISTINCT end_station_name
FROM FINAL20
ORDER BY end_station_name;

SELECT COUNT(ride_id) AS end_station_null
FROM FINAL20
WHERE end_station_name IS NULL OR end_station_id IS NULL;

DELETE FROM FINAL20
WHERE end_station_name IS NULL OR end_station_id IS NULL; ------(51057 rows affected)--------
```
------------------------------------------------------------------------------------------------------------------
```
SELECT COUNT(ride_id) AS end_loc_null
FROM FINAL20
WHERE end_lat IS NULL OR end_lng IS NULL;

DELETE FROM FINAL20
WHERE end_station_name IS NULL OR end_station_id IS NULL;
```

## Using Temp table created a column for ride length and ADD to original Table.
```
WITH RideLengthCTE AS (
  SELECT
    ride_id,
    DATEDIFF(MINUTE, started_at, ended_at) AS ride_length_MIN
  FROM FINAL20
)

SELECT 
  a.ride_id,
  a.rideable_type,
  a.started_at,
  a.ended_at,
  rlc.ride_length_MIN,
  a.start_station_name,
  a.end_station_name,  
  a.start_lng,  
  a.end_lng,
  a.member_casual,
  a.new_month,
  a.day_of_week
FROM FINAL20 a
JOIN RideLengthCTE rlc ON a.ride_id = rlc.ride_id
WHERE 
  a.start_station_name IS NOT NULL AND
  a.end_station_name IS NOT NULL AND
  a.end_lat IS NOT NULL AND
  a.end_lng IS NOT NULL AND
  rlc.ride_length_MIN > 1 AND
  rlc.ride_length_MIN < 1440;
```

### This Is For data 2020 After I repeat this process with 2021,2022, And 2023 And Merged All Data ( DATA CLEANING)
```
CREATE TABLE All_CLEANED_DATA
(
    ride_id NVARCHAR(50) NULL,
    rideable_type NVARCHAR(50) NULL,
    started_at DATE NULL,
    ended_at DATE NULL,
    start_station_name VARCHAR(MAX) NULL,
    start_station_id NVARCHAR(50) NULL,
    end_station_name VARCHAR(MAX) NULL,
    end_station_id NVARCHAR(50) NULL,
    start_lat FLOAT NULL,
    start_lng FLOAT NULL,
    end_lat FLOAT NULL,
    end_lng FLOAT NULL,
    member_casual NVARCHAR(50) NULL,
    new_month VARCHAR(50) NULL,
    day_of_week VARCHAR(50) NULL,
    ride_length_MIN NUMERIC NULL
);


-- INSERT INTO All_CLEANED_DATA
INSERT INTO All_CLEANED_DATA (ride_id, rideable_type, started_at, ended_at, start_station_name, start_station_id, end_station_name, end_station_id, start_lat, start_lng,end_lat,
    end_lng, member_casual, new_month, day_of_week, ride_length_MIN)
SELECT
    ride_id,
    rideable_type, 
    started_at, 
    ended_at, 
    start_station_name, 
    start_station_id, 
    end_station_name, 
    end_station_id, 
    start_lat, 
    start_lng,
	end_lat,
    end_lng,
    member_casual,
    new_month,
    day_of_week,
    ride_length_MIN
FROM (
    SELECT
		
        ride_id,
        rideable_type, 
        started_at, 
        ended_at, 
        start_station_name, 
        start_station_id, 
        end_station_name, 
        end_station_id, 
        start_lat, 
        start_lng,
		end_lat,
	    end_lng,
        member_casual,
        new_month,
        day_of_week,
        ride_length_MIN
    FROM FINAL20

    UNION ALL

    SELECT
		
        ride_id,
        rideable_type, 
        started_at, 
        ended_at, 
        start_station_name, 
        start_station_id, 
        end_station_name, 
        end_station_id, 
        start_lat, 
        start_lng,
		end_lat,
	    end_lng,
        member_casual,
        new_month,
        day_of_week,
        ride_length_MIN
    FROM FINAL21

    UNION ALL

    SELECT
		
        ride_id,
        rideable_type, 
        started_at, 
        ended_at, 
        start_station_name, 
        start_station_id, 
        end_station_name, 
        end_station_id, 
        start_lat, 
        start_lng,
		end_lat,
        end_lng,
        member_casual,
        new_month,
        day_of_week,
        ride_length_MIN
    FROM FINAL22

    UNION ALL

    SELECT
		
        ride_id,
        rideable_type, 
        started_at, 
        ended_at, 
        start_station_name, 
        start_station_id, 
        end_station_name, 
        end_station_id, 
        start_lat, 
        start_lng,
		end_lat,
        end_lng,
        member_casual,
        new_month,
        day_of_week,
        ride_length_MIN
    FROM FINAL
) AS combinedataclean;----16564930


SELECT *
FROM All_CLEANED_DATA
```
##Step 5: Share

## Sheet 1: Consumer Types Over the Years (Pie Chart)

### Visualization Overview

Sheet 1 presents a comprehensive overview of the distribution between casual and member types of consumers over the four-year period. The primary visualization is a pie chart, allowing for a quick and intuitive understanding of the proportion of each consumer type in Cyclistic's user base.

### Key Insights

- **Yearly Comparison:** The Pie Chart visualizes the percentage split between casual and member consumers for each year from the historical data (Jan 2022 to Dec 2022).
- **Overall Trend:** Observe the overall trend in the composition of consumer types, highlighting any shifts or consistent patterns over the years.


![PIE](https://github.com/vraj9860/G00gle-Capstone/assets/141504835/63e8ef9a-aeaa-4be5-9312-fa1d947a3518)

# Interpretation
The pie chart provides a clear and visually appealing representation of the consumer landscape, aiding in the identification of any significant changes in the distribution of casual and member consumers over the analyzed period.

## Sheet 2: Trend Line Usage of Different Cycle Types

### Visualization Overview

Sheet 2 delves into the usage patterns of different cycle types by casual and member consumers over the four-year period. The primary visualization is a trend line chart, allowing for a detailed analysis of how usage trends have evolved for various cycle types.

### Key Insights

- **Cycle Type Trends:** The Trend Line chart illustrates the usage trends of distinct cycle types (e.g., reclining bikes, hand tricycles, cargo bikes) by both casual and member consumers.
- **Annual Patterns:** Identify any annual patterns or fluctuations in the usage of different cycle types, providing insights into the popularity and seasonality of specific bikes.

![LINE](https://github.com/vraj9860/G00gle-Capstone/assets/141504835/85c294c6-0fb2-4aab-a574-0e9aeae47f3e)


### Interpretation

The trend line chart offers a dynamic view of the changing preferences and usage patterns of cycle types, facilitating a nuanced understanding of how each type resonates with casual and member consumers over the years.

## Sheet 3: Preference of Cycles by Different Consumer Types

### Visualization Overview

Sheet 3 explores the preference of cycle types by different consumer types (casual and member). The primary visualization is a stacked bar chart, providing a detailed breakdown of the proportion of cycle types preferred by each consumer category.

### Key Insights

- **Stacked Representation:** The stacked bar chart visually represents the distribution of cycle types preferred by casual and member consumers side by side.
- **Relative Preferences:** Identify the relative popularity of each cycle type within the casual and member consumer segments.

  ![BAR](https://github.com/vraj9860/G00gle-Capstone/assets/141504835/86e4b1ed-f013-4219-b008-64e82f76e33c)


### Interpretation

The stacked bar chart enables a nuanced analysis of consumer preferences, offering insights into the specific types of cycles that attract casual and member consumers. This information is crucial for tailoring marketing strategies and enhancing the overall user experience.

# KEY FINDINGS

1. **TRIP DURATION**

   The data shows that annual members tend to have shorter trip durations, with a significant portion of their rides lasting between 15 to 30 minutes. Annual members mainly use the bikes for quick and convenient commuting or short leisure rides.

   For casual riders, a significant percentage of their journeys last more than 30 minutes, indicating that the bikes are more likely to be used for leisurely exploration or longer recreational outings.

2. **TRIP PATTERN**

   - **Annual Members:**
     During regular commuting hours (early morning and late afternoon), there is a noticeable pattern of increased bike utilization. This validates the assumption that the majority of annual members travel to work on bicycles.

   - **Casual Riders:**
     Throughout the day, casual riders use the system more irregularly, with a noticeable peak in the late morning and early afternoon. This implies that casual riders may be using their bikes for leisure activities, including going on trips or running errands.

# IMPLEMENTING STRATEGY

## Targeted Marketing Campaigns:

Design specialized annual membership promotions that highlight the benefits of time-saving and convenience for people who commute every day. Promote bikes with an emphasis on their versatility and adaptability for leisure and adventure to target casual riders.

## Campaigns for Digital Media:

Adapt digital media campaigns to the discovered usage trends. Encourage yearly memberships, for instance, through channels targeted at commuters. Make use of eye-catching content that highlights the many uses of bikes for non-riders, enticing people to think about memberships for their various requirements.



