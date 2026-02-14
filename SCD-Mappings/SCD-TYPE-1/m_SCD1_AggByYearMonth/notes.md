## Question: Create SCD Type 1 mapping to upload aggregated data by year and month

### Input
Transaction table with transaction date and amount.

### Expected Output
Aggregated data by year and month, inserted or updated into the target table:
- If record exists → Update
- If record does not exist → Insert

---

### Mapping Name
**m_SCD1_AggByYearMonth**

---

### Strategy
1. **Source Qualifier (SQ)**  
   - Read data from source transaction table.

2. **Expression Transformation**  
   - Parse year and month from transaction date:  
     ```text
     out_year  = TO_CHAR(TO_DATE(txn_date,'MM/DD/YYYY HH24:MI'),'YYYY')
     out_month = TO_CHAR(TO_DATE(txn_date,'MM/DD/YYYY HH24:MI'),'MM')
     ```

3. **Sorter Transformation**  
   - Sort by year and month to group similar rows together.

4. **Aggregator Transformation**  
   - Aggregate amount (`SUM(amt)`) grouped by year and month.

5. **Lookup Transformation (parallel)**  
   - Lookup against target table using year and month.  
   - Ensure only one matching row is returned (disable multiple matches).

6. **Joiner Transformation (Left Outer Join)**  
   - Master: Aggregated pipeline.  
   - Detail: Lookup pipeline.  
   - Combine aggregated data with lookup results.

7. **Expression Transformation**  
   - Create flag column to decide insert vs update:  
     ```text
     ifinsert = IIF(ISNULL(lkp_col), 1, 0)
     ```

8. **Router Transformation**  
   - Group 1 (Insert): `ifinsert = 1`  
   - Group 2 (Update): `ifinsert = 0`

9. **Update Strategy Transformation**  
   - Insert group → DD_INSERT  
   - Update group → DD_UPDATE

---

### Key Insight
- Lookup + Joiner ensures we know whether a record already exists.  
- Expression + Router cleanly split the flow into insert vs update.  
- Update Strategy applies SCD Type 1 logic: overwrite existing rows, insert new ones.

---

### Interview Tip
> “I implemented SCD Type 1 by parsing year/month, aggregating amounts, and then using Lookup + Joiner to check target records. With Expression and Router, I flagged rows for insert or update, and applied Update Strategy accordingly. This ensures the target always reflects the latest aggregated state.”
