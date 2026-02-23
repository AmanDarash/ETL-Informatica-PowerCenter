# Mapping: m_SCD2_CustomerBalanceHistory

## ❓ Problem Statement
You have a Bank Transaction table with `Customer_ID`, `Transaction_Date`, and `Transaction_Amount`.  
The requirement is to maintain a **Customer Balance Dimension** using **SCD Type 2** logic:
- If a customer is new → Insert a new record.
- If a customer’s balance changes → Expire the old record and insert a new one.
- If balance is unchanged → Do nothing.

The dimension table (`CUSTOMER_BALANCE_DIM`) contains:
- `Customer_ID`
- `Balance`
- `Start_Date`
- `End_Date`
- `Current_Flag` (Y/N)

---

## ⚙️ Mapping Strategy

1. **Source Qualifier (SQ)**  
   - Read data from the Bank Transaction table.

2. **Sorter Transformation**  
   - Sort by `Customer_ID` and `Transaction_Date`.

3. **Aggregator Transformation**  
   - Group by `Customer_ID`.  
   - Calculate aggregated balance (`SUM(Transaction_Amount)`).  
   - Use `MAX(Transaction_Date)` to capture the latest transaction date.

4. **Lookup Transformation**  
   - Lookup against `CUSTOMER_BALANCE_DIM` using `Customer_ID`.  
   - Return only the current active record (`Current_Flag = 'Y'`).  
   - Ensure no multiple matches.

5. **Joiner Transformation**  
   - **Join Type**: Master Outer Join.  
   - **Master Pipeline**: Lookup.  
   - **Detail Pipeline**: Aggregator.  
   - Ensures all aggregated balances flow through, even if Lookup doesn’t find a match.

6. **Expression Transformation**  
   - Create flags:  
     ```text
     insert_or_update = IIF(ISNULL(lkp_customer_id), 1, 0)
     if_update        = IIF(lkp_balance = agg_balance, 0, 1)
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
- **Update Strategy**: DD_UPDATE (connect by `Customer_ID` and active record flag)

- **Expression 2 (Insert new record)**:  
  - `End_Date = '9999-12-31'`  
  - `Current_Flag = 'Y'`  
- **Update Strategy**: DD_INSERT

---

## 🔧 Blockages Faced & How I Solved Them

- **Issue 1: Precision mismatch in balance comparison**  
  - Initially, `BALANCE` and `total_amount` were stored as `DOUBLE`.  
  - Floating‑point precision caused equality checks to fail.  
  - **Fix**: Converted both to `INTEGER` (or `NUMBER(15,2)` for finance) before comparison.

- **Issue 2: Multiple matches in Lookup**  
  - Lookup sometimes returned more than one record for a customer.  
  - **Fix**: Added filter `Current_Flag = 'Y'` to ensure only active record is fetched.

- **Issue 3: Unnecessary updates for unchanged balances**  
  - Router was sending unchanged rows to Update pipeline.  
  - **Fix**: Added condition `lkp_balance = agg_balance` → drop group.

---

## 🎯 Key Insight
- **Master Outer Join** ensures all aggregated balances flow through.  
- **Router** cleanly splits rows into Insert, Update, or Drop.  
- **Update Strategy** applies SCD Type 2 logic: expire old records, insert new ones, preserve history.  
- **Datatype discipline** (INTEGER/NUMBER) prevents floating‑point mismatches.

---

## 📝 Interview Tip
> “For SCD Type 2, I aggregate balances per customer, then use Lookup + Joiner to check existing records. Router splits rows into Insert, Update, or Drop. Insert pipeline adds new customers with open‑ended dates. Update pipeline expires old records and inserts new ones with updated balance. Drop group ensures unchanged balances don’t cause redundant updates. I faced issues with floating‑point mismatches and multiple lookup matches, but solved them by enforcing datatypes and filtering only active records. This way, history is preserved efficiently.”
