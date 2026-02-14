# SCD Type 1 Mappings

This folder contains Informatica mappings that implement **Slowly Changing Dimension (SCD) Type 1** logic.  
SCD Type 1 is used when changes in dimension attributes should **overwrite existing values** without preserving history.

---

## 📌 What is SCD Type 1?
- **Definition**: Updates existing records with new values if they already exist, otherwise inserts new records.  
- **Use Case**: Correcting errors or when historical tracking is not required.  
- **Behavior**: No history is maintained — the dimension table always reflects the latest state.

---

## 📂 Mappings Included
- **m_SCD1_AggByYearMonth**  
  - Aggregates transaction data by year/month.  
  - Uses Lookup + Joiner to check target table for existing records.  
  - Router + Update Strategy split rows into **insert** (new records) and **update** (existing records).  
- **m_SCD1_UpdateInsertBasic**  
  - Simple example of SCD Type 1 logic with insert/update handling.

---

## ⚙️ Strategy Overview
1. **Source Qualifier (SQ)** → Read data from source.  
2. **Expression** → Parse year/month or other attributes.  
3. **Aggregator** → Aggregate measures if required.  
4. **Lookup** → Check target table for existing dimension records.  
5. **Joiner** → Combine source pipeline with lookup results.  
6. **Expression** → Flag rows as insert/update (`IIF(ISNULL(lkp_col),1,0)`).  
7. **Router** → Split into insert vs update groups.  
8. **Update Strategy** → Apply DD_INSERT or DD_UPDATE accordingly.  

---

## 🎯 Interview Tip
Recruiters often ask:  
- *“How do you implement SCD Type 1 in Informatica?”*  
- *“When would you use Type 1 instead of Type 2?”*

Answer:  
> “SCD Type 1 overwrites existing records without keeping history. I use Lookup + Router + Update Strategy to decide whether to insert or update. This is ideal when correcting data errors or when history is not required.”

---

## ✅ Best Practices
- Always define clear surrogate keys for dimension tables.  
- Ensure Lookup returns only one matching row (disable multiple matches).  
- Handle nulls carefully in Expression transformations.  
- Document assumptions (e.g., aggregation logic, date parsing) for clarity.
