# Slowly Changing Dimensions (SCD) Type 2 Mappings

This folder contains Informatica mappings that implement **SCD Type 2** logic.  
SCD Type 2 is used when changes in dimension attributes must be tracked historically, preserving multiple versions of a record.

---

## 📌 What is SCD Type 2?
- **Definition**: Inserts a new record when an attribute changes, while expiring the old record.  
- **Use Case**: Tracking historical changes in attributes such as customer balances, employee departments, or product prices.  
- **Behavior**: Maintains history with effective dates and a current flag.

---

## 📂 Mappings Included
- **m_SCD2_CustomerBalanceHistory**  
  - Source: Bank Transaction table.  
  - Aggregates balances per customer.  
  - Uses Lookup + Joiner to check existing dimension records.  
  - Router splits rows into Insert, Update, or Drop.  
  - Insert pipeline adds new records with `End_Date = 9999-12-31` and `Current_Flag = Y`.  
  - Update pipeline expires old records (`End_Date = SYSDATE`, `Current_Flag = N`) and inserts new ones.  
  - Drop group discards unchanged balances.

---

## ⚙️ Strategy Overview
1. **Source Qualifier (SQ)** → Read data from source.  
2. **Sorter** → Sort by `Customer_ID` and `Transaction_Date`.  
3. **Aggregator** → Aggregate balances, capture latest transaction date.  
4. **Lookup** → Fetch current active record from target dimension.  
5. **Joiner (Master Outer Join)** → Aggregator as Detail, Lookup as Master.  
6. **Expression** → Flag rows for insert/update.  
7. **Router** → Split into Insert, Update, Drop.  
8. **Update Strategy** → Apply DD_INSERT or DD_UPDATE accordingly.

---

## 🎯 Interview Tip
Recruiters often ask:
- *“How do you implement SCD Type 2 in Informatica?”*  
- *“When would you use Type 2 instead of Type 1?”*

Answer:
> “SCD Type 2 preserves history by expiring old records and inserting new ones with effective dates and a current flag. I use Lookup + Joiner to check existing records, Router to split rows into insert/update/drop, and Update Strategy to apply changes. This ensures the dimension table maintains full historical tracking.”

---

## ✅ Best Practices
- Always define surrogate keys for dimension tables.  
- Use `9999-12-31` as the open‑ended date for current records.  
- Ensure Lookup returns only one active record (Current_Flag = Y).  
- Document assumptions (e.g., date formats, aggregation logic) for clarity.  
