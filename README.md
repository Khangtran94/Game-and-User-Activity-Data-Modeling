# Game and User Activity Data Modeling

Imagine a dynamic sports and digital analytics arena where game performances and user interactions are captured in raw datasets. This repository transforms the `game_details` and `events` datasets into structured, analytical models using SQL, enabling insights into player performance and user activity trends.

## Project Overview

This project processes two datasets: `game_details` and `game` for NBA basketball player statistics and `events`, `devices` for user device and host activity. Using PostgreSQL, we create fact and cumulative tables to deduplicate data, track activity over time, and generate monthly aggregated metrics for efficient analysis.

### Datasets Description

- **game_details**: Tracks player performance per game
  - Fields: Game ID, team ID, team_name, player ID, player name, minutes played, field goals, rebounds, assists, etc.
  <img width="1415" height="751" alt="Screenshot 2025-09-03 at 13 28 46" src="https://github.com/user-attachments/assets/673c1410-52dd-425d-99a1-a63dd8f6d609" />

- **games**: Record matches History from 2016-01-01	to 2022-12-22
  - Fields: game_date, game_id, game_status, home_team, away_team, pts_home, pts_away, etc.
  <img width="1470" height="745" alt="Screenshot 2025-09-03 at 13 32 18" src="https://github.com/user-attachments/assets/999662f1-47f7-411c-82ce-24f73d9c937b" />
  
- **events**: Captures user interactions across devices and hosts from 2023-01-01 to 2023-01-31
  - Fields: User ID, device ID, host, event time.
  <img width="1215" height="742" alt="Screenshot 2025-09-03 at 13 40 45" src="https://github.com/user-attachments/assets/974f681e-f413-4cf1-9e88-147cb85c3a54" />

- **devices**: Captures device, browser type, os type information across each device id. 
  - Fields: device_id, device_type, os_type, etc.
  <img width="1452" height="741" alt="Screenshot 2025-09-03 at 13 42 48" src="https://github.com/user-attachments/assets/ce780b4a-fbcc-4e11-9644-c46eced622c0" />

## Key Concepts

- **SCD Type 2**: Tracks historical changes by adding new rows with `start_date` and `end_date`, preserving history.
- **Backfill**: Loads all historical data into a table at once (e.g., initializing `user_devices_cumulated`).
  - **Use When**: Setting up new tables or rebuilding history.
  - **Pros**: Comprehensive, ensures consistency.
  - **Cons**: Resource-intensive, slow for large datasets.
- **Incremental**: Updates tables with new/changed data (e.g., daily updates for `hosts_cumulated`).
  - **Use When**: Adding periodic updates to existing data.
  - **Pros**: Efficient, scalable for frequent updates.
  - **Cons**: Requires existing data, risks missing records if prior steps fail.
- **BIT_COUNT**:
  - **Definition**: A PostgreSQL function that counts the number of `1` bits in a bit string (e.g., `BIT(32)`).
  - **Usage**: Used in Task 4 to analyze `day_actives` (32-bit integer) to determine active days and check daily, weekly, and monthly activity via bitwise operations.
  - **Examples**:
    - **Active Days Count**: For `day_actives = '11000000000000000000000000000000'` (active on days 1 and 2 of January 2023), `BIT_COUNT(day_actives)` returns `2`, indicating 2 active days.
    - **Daily Activity Check**: `BIT_COUNT('10000000000000000000000000000000' & day_actives) > 0` checks if day 1 is active (returns `true` for `11000000000000000000000000000000`).
    - **Weekly Activity Check**: `BIT_COUNT('11111110000000000000000000000000' & day_actives) > 0` checks activity in the first 7 days (returns `true` if any of days 1–7 are active).
      
- **Why Create New Tables?**:
  - Raw datasets (`game_details`, `games`, `events`, `devices`) are transactional, not optimized for analysis.
  - New tables (`FCT_GAME_DETAILS`, `user_devices_cumulated`, `hosts_cumulated`) aggregate and deduplicate data for faster queries and trend analysis.

## Assignment Tasks

1. **Deduplicate `game_details`**:
   - Creates `FCT_GAME_DETAILS` table, deduplicates game data, and calculates metrics like minutes played and home/away status.
2. **DDL for `user_devices_cumulated`**:
   - Defines table to store user ID, browser type, and array of activity dates.
3. **Cumulative Query for `user_devices_cumulated`**:
   - Aggregates deduplicated `events` and `devices` data, building daily activity date lists for 2023-01-01.
4. **Datelist Integer Conversion**:
   - Converts `device_activity_datelist` into a 32-bit integer (`day_actives`) for January 2023, using `BIT_COUNT` to calculate daily, weekly, and monthly activity flags.
5. **DDL for `hosts_cumulated`**:
   - Defines table to store host and activity date lists.
6. **Incremental Query for `hosts_cumulated`**:
   - Updates host activity for 2023-01-31, appending new dates to existing lists.
7. **DDL for `host_activity_reduced`**:
   - Creates table for monthly host metrics (hits, unique visitors).
8. **Incremental Query for `host_activity_reduced`**:
   - Uses `array_metrics` to aggregate daily hits and unique visitors for January 2023, updating `host_activity_reduced`.

## Notes

- Assumes `game_details` and `events` data for 2023.
- Incremental queries focus on January 2023 updates.
- Ensure deduplication in `events` and `devices` for accurate results.
- `BIT_COUNT` enables compact storage and efficient activity analysis in `day_actives`.

## Resume Highlights

- **Developed Analytical Data Model**: Designed SQL tables (`FCT_GAME_DETAILS`, `user_devices_cumulated`, `host_activity_reduced`) to deduplicate and aggregate game and user activity data for efficient analysis.
- **Implemented Incremental Updates**: Built queries with `BIT_COUNT` for compact tracking of user and host activity in PostgreSQL, optimizing daily, weekly, and monthly metrics.

## Output:
- fct_game_details (already deduplicate):
  <img width="1433" height="754" alt="Screenshot 2025-09-03 at 13 37 59" src="https://github.com/user-attachments/assets/47b9acb5-b8a7-4597-8337-6b44d92c7af1" />

- user_devices_cumulated: a device_activity_datelist which tracks a users active days by browser_type:
  <img width="897" height="759" alt="Screenshot 2025-09-03 at 13 56 29" src="https://github.com/user-attachments/assets/f1558635-54d2-4f17-bf8c-502cfbf99995" />

=> Then convert to datelist_int (to flag user activity):
    <img width="913" height="746" alt="Screenshot 2025-09-03 at 13 59 46" src="https://github.com/user-attachments/assets/1aaae1b1-e086-40c7-9e7c-74bddfcdf8ef" />

- host_activity_reduced: A monthly, reduced fact table:
  <img width="1451" height="120" alt="Screenshot 2025-09-03 at 14 38 22" src="https://github.com/user-attachments/assets/87485e36-6c59-4de2-b12e-cb01679d4701" />


