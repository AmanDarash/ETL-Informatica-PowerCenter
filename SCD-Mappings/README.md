# Slowly Changing Dimensions (SCD) Mappings

This folder contains Informatica mappings that implement **SCD Type 1** and **SCD Type 2** strategies.  
These are common interview and real-world scenarios for handling changes in dimension tables.

---

## 📌 SCD Type 1
- **Definition**: Overwrites existing records with new values (no history maintained).
- **Use Case**: Correcting data errors or when historical tracking is not required.
- **Mappings Included**:
  - `m_SCD1_AggByYearMonth` → Aggregates transaction data by year/month and applies SCD1 logic (update if exists, insert if not).
  - Other examples: `m_SCD1_UpdateInsertBasic`

---

## 📌 SCD Type 2
- **Definition**: Preserves history by creating new records when changes occur, often with effective dates or versioning.
- **Use Case**: Tracking historical changes in attributes (e.g., employee department changes).
- **Mappings Included**:
  - `m_SCD2_EffectiveDate` → Maintains start/end dates for each version of a record.
  - `m_SCD2_FlagBased` → Uses current flag to mark active record and preserve history.

---

## ✅ Best Practices
- Always define clear surrogate keys for dimension tables.
- Use **Update Strategy** for inserts/updates.
- Handle nulls and defaults carefully in Expression transformations.
- Document assumptions (e.g., max rank, date formats) for clarity.
