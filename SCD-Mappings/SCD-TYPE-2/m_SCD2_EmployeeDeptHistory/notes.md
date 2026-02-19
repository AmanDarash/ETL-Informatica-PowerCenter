# Mapping: m_SCD2_EmployeeDeptHistory

## ❓ Problem Statement
Track changes in employee department assignments using **SCD Type 2** logic:
- If an employee is new → Insert a new record.
- If an employee’s department changes → Expire the old record and insert a new one.
- If the department is unchanged → Do nothing.

The dimension table (`EMP_DEPT_DIM`) contains:
- `Employee_ID`
- `Department`
- `Start_Date`
- `End_Date`
- `Current_Flag` (Y/N)

---

## ⚙️ Strategy

1. **Source Qualifier (SQ)**  
   - Read data from the Employee source table.

2. **Sorter Transformation**  
   - Sort by `Employee_ID` and `Effective_Date`.

3. **Aggregator Transformation**  
   - Group by `Employee_ID`.  
   - Use `MAX(Effective_Date)` to capture the latest department assignment.

4. **Lookup Transformation**  
   - Lookup against `EMP_DEPT_DIM` by `Employee_ID`.  
   - Return only the current active record (`Current_Flag = 'Y'`).  
   - Ensure no multiple matches.

5. **Joiner Transformation**  
   - **Join Type**: Master Outer Join.  
   - **Master Pipeline**: Lookup.  
   - **Detail Pipeline**: Aggregator.  
   - Ensures all employee records flow through, even if Lookup doesn’t find a match.

6. **Expression Transformation**  
   - Create flags:  
     ```text
     insert_or_update = IIF(ISNULL(lkp_employee_id), 1, 0)
     if_update        = IIF(lkp_department = agg_department, 0, 1)
     ```

7. **Router Transformation**  
   - **Group 1 (Insert)**: `insert_or_update = 1`  
   - **Group 2 (Update)**: `if_update = 1 AND insert_or_update = 0`  
   - **Group 3 (Drop)**: Rows with no change are discarded.

---

## 📂 Pipelines

### Insert Pipeline
- **Expression**:  
  - `End_Date = '9999-12-31'`  
  - `Current_Flag = 'Y'`  
- **Update Strategy**: DD_INSERT

### Update Pipeline
- **Expression 1 (Expire old record)**:  
  - `End_Date = SYSDATE`  
  - `Current_Flag = 'N'`  
- **Update Strategy**: DD_UPDATE (connect by `Employee_ID` and active record flag)

- **Expression 2 (Insert new record)**:  
  - `End_Date = '9999-12-31'`  
  - `Current_Flag = 'Y'`  
- **Update Strategy**: DD_INSERT

---

## 🎯 Key Insight
- **Master Outer Join** ensures all employee records flow through.  
- **Router** splits rows into Insert, Update, or Drop.  
- **Update Strategy** applies SCD Type 2 logic: expire old records, insert new ones, preserve history of department changes.

---

## 📝 Interview Tip
> “For Employee Department history, I use SCD Type 2 to track department changes. Lookup + Joiner checks existing records, Router splits rows into Insert, Update, or Drop. Insert pipeline adds new employees with open‑ended dates. Update pipeline expires old records and inserts new ones when department changes. Drop group ensures unchanged assignments don’t cause redundant updates.”
