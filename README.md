# Sustainability Impact Analysis for Intel (SQL Project)

## Overview
This project analyzes corporate device inventory and energy metrics across regions to evaluate $\text{CO}_2$ emissions savings, identify device age buckets, and calculate regional contribution percentages.

## Core SQL Implementation

```sql
WITH based_data AS (SELECT
  a.*,
  b.*,
  CASE
    when (2024 - a.model_year) <= 3 then 'newer'
    when (2024 - a.model_year) > 3
    and (2024 - a.model_year) <= 6 then 'mid-age'
    else 'older'
  END as device_age_bucket,
  2024 - a.model_year as device_age
FROM
  intel.device_data as a FULL
  OUTER JOIN intel.impact_data as b on a.device_id = b.device_id),
  
  
 regional as(
  SELECT 
  region,
  device_type,
  COUNT(*) as device_in_categorey,
  SUM(energy_savings_yr) as  total_energy_saving_kwh,
  SUM(co2_saved_kg_yr) as total_co2_saving_kg
  FROM based_data
  GROUP BY
  region, 
  device_type
  )
SELECT 
region,
device_type,
device_in_categorey as total_device,
ROUND(total_energy_saving_kwh / device_in_categorey, 2) as avg_energy_saving_kwh,
ROUND((total_co2_saving_kg/device_in_categorey )/ 1000, 2) as avg_co2_saved_ton,
ROUND((total_energy_saving_kwh/ SUM(total_energy_saving_kwh) OVER(partition by region ) )* 100, 2) as pct_of_region_energy_contributed,
ROUND((total_co2_saving_kg/ SUM(total_co2_saving_kg) OVER(partition by region))* 100, 2)as pct_of_region_co2_contributed
FROM regional
order BY 
  region ASC, pct_of_region_co2_contributed desc
