# Duplicate vs Unique Segregation using Dense Rank

## 🎯 Objective
To separate duplicate records into one target and unique records into another target using **dense_rank logic** implemented in an Expression transformation.

---

## ⚙️ Mapping Strategy

1. **Source Qualifier (SQ)**  
   - Reads data from the source table.  
   Example input:
A
A
B
C
D
D
D
E
E

---

2. **Sorter (SRT)**  
- Sorts the data by the key column(s) that may contain duplicates.

3. **Expression Transformation (EXP)**  
- Implements **dense_rank logic** using variables and output ports:
  - `v_repeat` → `IIF(pre_val = val, v_repeat + 1, 1)`
  - `v_pre_val` → `val`
  - `o_dense_rnk` → `v_repeat`

Example output after Expression:  
A → 1
A → 2
B → 1
C → 1
D → 1
D → 2
D → 3
E → 1
E → 2

---

4. **Router Transformation (ROU)**  
- **Condition 1:** `dense_rnk = 1` → First occurrence (unique record group)  
- **Condition 2:** `dense_rnk != 1` → Subsequent occurrences (duplicate record group)  
  *(You can also use the default group for duplicates.)*

---

## 🛠️ Targets
- **Target 1 (Unique_Records)** → Stores the first occurrence of each value.  
- **Target 2 (Duplicate_Records)** → Stores all subsequent occurrences.

---

## ✅ Key Takeaways
- Dense Rank logic avoids the need for Aggregator + Joiner combination.  
- Expression transformation provides flexibility for ranking and conditional routing.  
- This approach is efficient for scenarios where duplicates must be identified and split into separate targets.  
- Demonstrates alternative ETL design strategies, useful for interviews and real-world problem solving.




