# mydrnow-medication-adherance
A Python‐based analysis and export of medication adherence for MY DR NOW, including data cleaning, feature engineering, provider &amp; clinic benchmarking, gap‐bin and weekly trend analyses, and composite scoring; ready for dashboarding.

# Known Issues & Lessons Learned

Below is a concise list of the main problems I ran into during the analysis and export pipelines.  Irrespective of these setbacks, I’m committed to addressing each point, refining the workflow, and sharing updated progress and insights as I go.

---

## 1. Data Cleaning & Preprocessing

- **Imperfect date parsing**  
  - I coalesced missing `absolute_fail_date` to `next_refill_date`, but didn’t fully audit rows where both dates were null → led to spurious “on-time” flags.  
- **Duplicate‐drop logic**  
  - My deduplication on `['patient_id','drug_name','next_refill_date','quantity']` masked cases where quantity differed but should have been merged (e.g. split fills).  
- **Swap‐logic edge cases**  
  - When swapping `date_of_last_refill` > `next_refill_date`, I didn’t re-validate downstream, so some `days_supply` came out negative and got nulled incorrectly.

## 2. Merging Encounters & Refills

- **Incorrect join key**  
  - I merged on `next_refill_date` ≈ `contact_date` (backward asof), but didn’t enforce a maximum gap—so “last encounter” sometimes came months earlier or was missing entirely.  
- **Many-to-many collisions in BI**  
  - In Power BI I attempted a 1:1 or 1:* relationship between the pharmacy and encounters tables on `patient_id` and date, but duplicate keys prevented a clean model.

## 3. Metric Validation & Assertions

- **Assertion failures**  
  - `assert prov_metrics['total_refills'].sum() == len(pharmacy)` failed—total counts didn’t match because some refills lacked a mapped encounter or category.  
- **Null buckets**  
  - I dropped rows with missing `drug_name` / `next_refill_date`, inadvertently discarding ~3% of data that may be clinically relevant.

## 4. Time-Series Trend Bugs

- **Incomplete weeks**  
  - The “Weekly adherence” graph dipped sharply at the end—this was purely an artifact of including partial final weeks rather than a true clinical drop-off.  
- **Period grouping**  
  - Using `dt.to_period('W')` without standardizing week start day caused overlapping bins across months.

## 5. Power BI Modeling & DAX

- **Missing columns on load**  
  - Key fields (`provider_id`, `encounter_gap`) did not appear in the PBI data pane after refresh, due to inconsistent query steps in Power Query.  
- **Circular DAX dependencies**  
  - Attempts to define `On-Time Refills` and `Adherence Rate` measures in the same table produced circular reference errors.

---

## Next Steps

1. **Refine join logic**  
   - Only merge encounters within a realistic window (e.g. ≤ 90 days before refill).  
2. **Revisit dropped rows**  
   - Surface “unknown-category” and “no-encounter” cases as separate buckets.  
3. **Stabilize time-series**  
   - Exclude incomplete weeks/months or pad to full intervals.  
4. **Power Query rebuild**  
   - Consolidate pharmacy + encounter + med_ref into a single fact table in PQ, then load into PBI with clear 1:N relationships.  
5. **Iterate DAX**  
   - Move pre-aggregations out of the model where possible and compute only in Python, leaving simple SUM/COUNT measures in PBI.

I’ll tackle these systematically and will update this repo (and my Power BI file) with corrected pipelines, new sample dashboards, and fresh insights as soon as they’re ready.

## Acknowledgements

Thank you for believing in me and giving me the opportunity to work on this important project. I recognize there are areas in this submission that could be stronger, and I remain committed to refining and improving these analyses. This is not the end of the journey—I will continue iterating on the data pipelines, metrics, and dashboards, and will share updated progress and insights with the team in the coming days.  

I am grateful for your support and feedback, and I look forward to delivering a more polished, robust solution soon.

— Rahul Muddhapuram 
 
