# Analysis of Electric Vehicle Charging Habits

## Understanding EV Charging Behavior

As electric vehicles (EVs) become more popular, providing adequate charging infrastructure is crucial, especially in residential buildings. Many apartment complexes are now installing shared charging stations for their tenants, but this can lead to competition for limited resources.
This analysis explores a dataset of charging sessions from several apartment garages to help building managers understand tenant habits. By identifying patterns in usage, we can provide recommendations to optimize station availability and ensure fair access for all residents.

## The Dataset: `charging_sessions`

The analysis is performed on a single table named `charging_sessions`, which logs individual charging events. The table contains the following columns:

| Column | Definition | Data type |
|---|---|---|
|`garage_id`| Identifier for the garage/building|`VARCHAR`|
|`user_id` | Identifier for the individual user|`VARCHAR`|
|`user_type`|Indicating whether the station is `Shared` or `Private`| `VARCHAR` |
|`start_plugin`|The date and time the session started |`DATETIME`|
|`start_plugin_hour`|The hour (in military time) that the session started | `NUMERIC`|
|`end_plugout`|The date and time the session ended | `DATETIME` |
|`end_plugout_hour`|The hour (in military time) that the session ended | `NUMERIC`|
|`duration_hours`| The length of the session, in hours|`NUMERIC`|
|`el_kwh`| Amount of electricity used (in Kilowatt hours)|`NUMERIC`|
|`month_plugin`| The month that the session started |`VARCHAR`|
|`weekdays_plugin`| The day of the week that the session started|`VARCHAR`|

## Analysis and Findings

To better understand usage patterns, three key questions were investigated.

### 1\. Which garages have the most users?

The first step was to identify which garages have the highest number of unique users for shared stations. This helps pinpoint high-demand locations that might require more resources.

```sql
-- unique_users_per_garage
SELECT COUNT(DISTINCT user_id) AS num_unique_users, garage_id
FROM charging_sessions
WHERE user_type = 'Shared'
GROUP BY garage_id
ORDER BY num_unique_users DESC;
```

The results show a significant concentration of users in three main garages: **Bl2 (18 users)**, **AsO2 (17 users)**, and **UT9 (16 users)**. In contrast, several other garages have only one or two shared users.

### 2\. What are the peak charging times?

Next, we identified the most popular times for tenants to start charging their vehicles. Understanding peak hours is essential for managing station availability and preventing congestion.

```sql
-- most_popular_shared_start_times
SELECT weekdays_plugin, start_plugin_hour, COUNT(*) AS num_charging_sessions
FROM charging_sessions
WHERE user_type = 'Shared'
GROUP BY weekdays_plugin,
	start_plugin_hour
ORDER BY num_charging_sessions DESC
LIMIT 10;
```

The data reveals that **Sunday at 5 PM (17:00)** is the most popular time to begin a charging session. Generally, the busiest periods are late afternoons and evenings, particularly between **3 PM (15:00) and 7 PM (19:00)**, which aligns with typical end-of-workday schedules.

### 3\. Who are the 'super-users'?

Finally, the analysis identified "super-users", individuals who occupy shared chargers for extended periods. These long sessions can reduce availability for other tenants. The query searched for users whose average charging duration exceeds 10 hours.

```sql
-- long_duration_shared_users
SELECT user_id, AVG(duration_hours) AS avg_charging_duration
FROM charging_sessions
WHERE user_type = 'Shared'
GROUP BY user_id
HAVING AVG(duration_hours) > 10
ORDER BY avg_charging_duration DESC
LIMIT 10;
```

Several users were found to have very long average charging times, with the top user, **Share-9**, averaging nearly **17 hours** per session. This suggests that some tenants may be using the stations for overnight charging, which could be a factor in station unavailability.

## Conclusion

The data provides valuable insights for building managers. The high concentration of users in garages **Bl2**, **AsO2**, and **UT9** suggests that these locations may be candidates for additional charging infrastructure. The identification of peak charging hours, primarily in the late afternoon and evening, can help in managing demand, perhaps by implementing a reservation system or time-based pricing. Lastly, the presence of users with very long charging durations highlights a potential need for policies, such as idle fees or maximum charging times, to ensure fair and efficient use of these shared resources.

## Tools Used

  - **SQL (PostgreSQL)**
