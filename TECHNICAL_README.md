# Customer Loyalty Segmentation — Technical Documentation (TravelTide)

This technical documentation explains the SQL-based segmentation pipeline used to engineer behavioral features, compute segment scores, and generate the final customer segment assignments for TravelTide.  
All steps are executed within a **single, self-contained SQL workflow** built using sequential CTEs, ensuring that the entire logic can be reproduced consistently on every run.

The full end-to-end query is available here:  
👉 [`sql/00_master_version_complete.sql`](sql/00_master_version_complete.sql)

---

## 1. Data Preparation

Users, sessions, flights, and hotels are joined into a unified **session-level table**.  
Several filters ensure analytical relevance:

- Only sessions from `2023-01-04` onwards  
- Users with more than 7 sessions (active user cohort)  

```sql
-- Cohort filter
WITH filtered_sessions AS (
  SELECT *
  FROM sessions
  WHERE session_start >= '2023-01-04'
),
active_users AS (
  SELECT user_id
  FROM filtered_sessions
  GROUP BY user_id
  HAVING COUNT(*) > 7
)
```

This forms the base for feature engineering at both session and user level.

---

## 2. Feature Engineering

A series of transformations creates standardized, comparable behavioral metrics.

### Key transformations include

- Cleaning of negative/zero nights  
- Fixing discount and booking flags  
- Deriving trip status (`booked`, `cancelled`, `none`)  
- Session duration and booking lead times  
- Flight distance and hotel night metrics
- Categorizing age, trip length, destination type  

```sql
-- Example: Nights cleaning
CASE 
  WHEN nights < 0 AND return_time IS NULL THEN nights * -1
  WHEN nights = 0 AND return_time IS NULL THEN nights + 1
  ELSE date(check_out_time) - date(check_in_time)
END AS nights_cleaned
```

These enrichments are collected in the `session_based_final` CTE, which is the cleaned session-level dataset.

---

## 3. User-Level Aggregations

Session metrics are aggregated into **user-level behavior profiles**, including:

- Total bookings, cancellations, and discount usage
- Average session minutes and clicks
- Booking speed and lead-time indicators
- Flight cost per km, hotel cost per night  
- Seats, bags, and room usage per booking  
- Night counts and travel frequency
- Weekday vs. weekend travel patterns  

```sql
-- Example: average minutes per session
ROUND(SUM(session_duration_min)::numeric / NULLIF(COUNT(session_id), 0), 4) AS avg_min_per_session
```

The result d´feed into the `user_based_avg` CTE, which provides normalized per-user features.

---

## 4. Features Normalization & Behavioral Flags

To create comparable dimensions across all users, several normalization steps and percentile-based flags are introduced:

- **Percentile flags**: quick/slow bookers, low/high clickers, short/long lead times  
- **Price sensitivity**: based on p20/p80 of flight or hotel costs  
- **Demographic indicators**: age brackets, new/young customers  
- **Behavioral condition flags**: e.g., Dreamers (only cancelled/no bookings)

```sql
-- Example: quick booker flag
CASE 
  WHEN avg_min_booking < (
    SELECT percentile_cont(0.2) WITHIN GROUP (ORDER BY avg_min_booking)
    FROM user_based_avg
  ) THEN 1 ELSE 0 
END AS is_quick_booker_p20
```

---

## 5. Segmentation Model

A weighted scoring model evaluates each user across a set of behavioral and demographic features.

Example (Business segment):

```sql
-- Business score
CASE 
  WHEN is_in_working_age = 0 OR weekdays_travel_quote < 0.5 THEN 0
  ELSE avg_bags_norm_invert * 0.2 
     + avg_seats_norm_invert * 0.2 
     + weekdays_travel_quote * 0.6 
END AS score_business
```

All segment scores are computed in a unified scoring CTE.  
Each user is then assigned to the **segment with the highest score**, with an additional rule that assigns users to *Others* if all scores fall below a minimum threshold of `0.3`.

Segments include:  
Business, Family, Frequent Traveler, Premium, Budget, Deal Hunter, Dreamer, New, Young, Others.

These segments match the definitions in the accompanying project reports.

---

## 6. Perk Assignment (Hypothesis-Based)

Each segment is linked to an **initial perk hypothesis** derived from:

- the behavioral and demographic properties of the segment  
- patterns typical in travel loyalty programs  
- inferred user needs  

Because no experimental or causal data is available in this fictional case, the perk assignment represents an **educated guess**, meant to guide marketing strategy — not a validated effect.

Validation would require A/B testing and longitudinal analysis, which is beyond the scope of this project setup.

---

## 7. Final Output

The final query produces the consolidated results (segment sizes, shares, perk mapping) used in the dashboards and reports:

```sql
SELECT
  final_segment,
  COUNT(*) AS users,
  ROUND(COUNT(*)::numeric / SUM(COUNT(*)) OVER (), 2) AS share_perc,
  CASE 
    WHEN final_segment = 'Business' THEN 'Free Cancellation'
    WHEN final_segment = 'Family' THEN 'Free Hotel Meal'
    WHEN final_segment = 'Frequent Traveler' THEN 'Free Cancellation & Lounge'
    WHEN final_segment = 'Premium Traveler' THEN 'Priority Check-In & Lounge'
    -- etc. for other segments
    ELSE 'Baseline Discount - TBD'
  END AS perk
FROM assigned_cleaned
GROUP BY final_segment
ORDER BY users DESC;
```

This output aligns with the visualizations and conclusions in the executive summary and full report.

---

## 8. Validation & Debugging

Debugging queries are included to inspect intermediate results:

- Per-user assignment (`assigned`)  
- Feature distributions (`features_norm`)  
- User-level averages (`user_based_avg`)  
- Session-level cleaning (`session_based_final`)  

```sql
-- Inspect per-user assignment
SELECT * 
FROM assigned
ORDER BY score DESC;
```

These queries help ensure consistency across transformations and transparency in scoring behavior.

---

## 9. Reproducibility Notes

- The Pipeline is fully **idempotent** (reruns produce identical results).  
- No temp tables are created; all logic is defined in CTEs.
- All `JOIN`s on `trip_id` use `USING(trip_id)` to avoid duplicates.  
- Percentiles are computed per cohort and depend on the dataset's distribution.
- Tie-breaks in equal scores follow alphabetical order.  
- Segment definitions and scoring logic correspond to those documented in the project reports.

---

## 10. Tuning Guide

Parameters that can be adjusted:  

- **Cohort date** → change in `filtered_sessions`  
- **Active user threshold** → default = 7 sessions  
- **Percentile cut-offs** → adjust in `features_norm`  
- **Segment weights** → formulas in `user_scores`  
- **Others threshold** → default = score < 0.3  

---

## 👤 Author

**Thomas Jortzig**  
TravelTide Segmentation Pipeline — Technical Documentation (09/2025)
