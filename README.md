# SQL Codes

##### To aggregate period 1 (3rd to 4th) tables, the same code is repeated for period 2 (4th to 5th):
CREATE TABLE  groovy-patrol-473812-c8.3_to_4_2016.fitbit_tracker_data_period1 AS
WITH cal_min AS (
  SELECT
    Id,
    DATE(ActivityMinute) AS ActivityDate,
    sum(Calories) AS total_calories
    FROM  `groovy-patrol-473812-c8.3_to_4_2016.calories_minute`
    GROUP BY ActivityDate, Id
  ),
  intensity AS  (
    SELECT
      Id,
      DATE(ActivityMinute) AS ActivityDate,
      avg(Intensity) AS intensity_avg_perMin
      FROM `groovy-patrol-473812-c8.3_to_4_2016.intensities_minute`
      GROUP BY ActivityDate, Id
  ),
  sleep_min AS (
    SELECT
      Id,
      DATE(date) AS ActivityDate,
      sum(value)  AS  sleep_mins
      FROM  `groovy-patrol-473812-c8.3_to_4_2016.sleep_minute`
      GROUP BY  ActivityDate, Id
  ),
  steps_min AS (
    SELECT
      Id,
      DATE(ActivityMinute) AS ActivityDate,
      sum(Steps) AS total_steps_min
      FROM  `groovy-patrol-473812-c8.3_to_4_2016.steps_minute`
      GROUP BY Id, ActivityDate
  ),
  weight AS (
    SELECT
      Id,
      WeightKg,
      BMI,
      DATE(date) AS ActivityDate,
      FROM  `groovy-patrol-473812-c8.3_to_4_2016.weight_log` 
  )

SELECT 
  da.Id,da.ActivityDate,da.TotalSteps,da.LoggedActivitiesDistance,da.SedentaryMinutes,da.LightlyActiveMinutes,da.FairlyActiveMinutes,da.VeryActiveMinutes, cal_min.total_calories, intensity.intensity_avg_perMin, sleep_min.sleep_mins, steps_min.total_steps_min, weight.weightKg, weight.BMI, SUM(MET.METs) AS total_METs

FROM `groovy-patrol-473812-c8.3_to_4_2016.DailyActivity` da
LEFT JOIN `groovy-patrol-473812-c8.3_to_4_2016.MET_minute` MET
  ON da.Id = MET.Id AND da.ActivityDate = DATE(MET.ActivityMinute)
LEFT JOIN cal_min
  ON da.Id = cal_min.Id AND da.ActivityDate = cal_min.ActivityDate
LEFT JOIN intensity
  ON da.Id = intensity.Id AND da.ActivityDate = intensity.ActivityDate
LEFT JOIN sleep_min
  ON da.Id = sleep_min.Id AND da.ActivityDate = sleep_min.ActivityDate
LEFT JOIN steps_min
  ON da.Id = steps_min.Id AND da.ActivityDate = steps_min.ActivityDate
LEFT JOIN weight
  ON da.Id = weight.Id AND da.ActivityDate = weight.ActivityDate
GROUP BY  
  da.Id,
  da.ActivityDate,
  da.TotalSteps,
  da.LoggedActivitiesDistance,
  da.SedentaryMinutes,
  da.LightlyActiveMinutes,
  da.FairlyActiveMinutes,
  da.VeryActiveMinutes,
  cal_min.total_calories,
  intensity.intensity_avg_perMin,
  sleep_min.sleep_mins,
  steps_min.total_steps_min,
  weight.WeightKg,
  weight.BMI;

##### To aggreate both periods into one table:
CREATE TABLE groovy-patrol-473812-c8.fitbit.both_periods  AS
  SELECT  *
  FROM `groovy-patrol-473812-c8.3_to_4_2016.fitbit_tracker_data_period1`
  UNION ALL 
  SELECT  *
  FROM `groovy-patrol-473812-c8.4_to_5_2016.fitbit_tracker_data_period2`
  
##### To calculate duplicates and view duplicates: 
WITH dupes AS (
  SELECT Id, ActivityDATE,
  FROM `groovy-patrol-473812-c8.fitbit.both_periods`
  GROUP BY Id, ActivityDate
  HAVING COUNT(*)>1
)
SELECT t.*
FROM  `groovy-patrol-473812-c8.fitbit.both_periods` AS t
JOIN dupes
ON t.Id = dupes.Id AND t.ActivityDate = dupes.ActivityDate
ORDER BY Id, ActivityDate




## R
