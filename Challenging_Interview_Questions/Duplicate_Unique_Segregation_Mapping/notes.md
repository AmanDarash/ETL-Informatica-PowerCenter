# Duplicate vs Unique Segregation Mapping

## 🎯 Objective
To separate duplicate records into one target and unique records into another target using Informatica PowerCenter.

---

## ⚙️ Mapping Strategy

1. **Source Qualifier (SQ)**  
   - Reads data from the source table.

2. **Sorter (SRT)**  
   - Sorts the data based on the column(s) that may contain duplicates.

3. **Aggregator (AGG)**  
   - Groups by the key column and counts occurrences.  
   - Example output:  
     ```
     A → 2  
     B → 1  
     C → 3  
     D → 4
     ```

4. **Joiner (Sorted Input)**  
   - Joins the original sorted data with the aggregated counts.  
   - Result:  
     ```
     A 1 2  
     A 1 2  
     B 1 1  
     C 1 3  
     C 1 3  
     C 1 3  
     D 1 4  
     D 1 4  
     D 1 4  
     D 1 4
     ```

5. **Router Transformation**  
   - **Condition 1:** `exp_val == agg_val` → Unique records group  
   - **Condition 2:** `exp_val != agg_val` → Duplicate records group

---

## 🛠️ Targets
- **Target 1 (Unique_Records)** → Stores unique rows.  
- **Target 2 (Duplicate_Records)** → Stores duplicate rows.

---

## ✅ Key Takeaways
- Router is used to split data flows based on conditions.  
- Aggregator + Joiner combination ensures accurate duplicate detection.  
- This mapping is a common interview scenario to test logical thinking in ETL design.
