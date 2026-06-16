## Analysing Kickstarter projects

1. This database consists of one table, ksprojects.
2. It creates a new field called funding_status using a CASE statement that applies conditional logic based on the percentage pledged (pledged divided by goal).
3. It categorizes projects as "Fully funded," "Nearly funded," or "Not nearly funded" according to the given thresholds.
4. It filters the results to show only projects that have failed but still have a significant number of backers (100 or more) and pledged amount (20,000 or more).
5. It also orders the output by main category and percentage pledged, limiting the results to 10 rows for clarity.

```
  SELECT main_category, backers, pledged, goal,
         pledged / goal AS pct_pledged,
         CASE
            WHEN pledged / goal >=1 THEN 'Fully funded'
            WHEN pledged / goal BETWEEN 0.75 AND 1 THEN 'Nearly funded'
            ELSE 'Not nearly funded'
            END AS funding_status
    FROM ksprojects
   WHERE state IN ('failed')
     AND backers >= 100 AND pledged >= 20000
ORDER BY main_category, pct_pledged DESC
   LIMIT 10;
   ```
