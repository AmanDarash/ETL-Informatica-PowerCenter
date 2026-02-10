## Question: Create a mapping to insert the first and last row of a table

### Strategy 1 (Ranking Approach)
1. **Source Qualifier (SQ)** → Read data from source.
2. **Sorter (SRT)** → Sort by the column that defines order (e.g., employee_id, date).
3. **Rank Transformation**:
   - Rank 1 → Select top row (first record).
   - Rank 2 → Select bottom row (last record).
4. **Union Transformation** → Combine both pipelines.
5. **Target** → Insert into target table.

---

### Optimized Strategy (Sequence + Aggregator Approach)
1. **Source Qualifier (SQ)** → Read data from source.
2. **Sorter (SRT)** → Sort by the column that defines order.
3. **Sequence Generator (SG)** → Generate incremental sequence numbers.
4. **Expression Transformation (EXP)** → Pass sequence number along with data.
A → 1 B → 2 C → 3 D → 4 E → 5

---
5. **Aggregator Transformation** → Compute `MIN(seq_val)` and `MAX(seq_val)`.
6. **Expression Transformation (EXP)** → Add min and max values to each row.
A 1 1 5 B 2 1 5 C 3 1 5 D 4 1 5 E 5 1 5

---

7. **Filter Transformation (FLTR)** → Keep only rows where `seq_val = MIN` or `seq_val = MAX`.
8. **Target** → Insert first and last rows into target.

---

### Mapping Name
**FirstLastRowExtractor**

