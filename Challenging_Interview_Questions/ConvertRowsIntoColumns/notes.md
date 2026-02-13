## Question: Convert rows into columns (pivot)

### Input
10,a  
10,b  
10,c  
20,d  
20,f  
20,g  


### Expected Output
10,a,b,c  
20,d,f,g  

---

### Mapping Name
**m_RowsToColumnsPivot**

---

### Strategy
1. **Source Qualifier (SQ)** → Read data from source.
2. **Sorter (SRT)** → Sort by `field1` so similar groups are together.
3. **Rank Transformation**  
   - Group by `field1`.  
   - Rank on `field2`.  
   - Assign rank values (1, 2, 3…).
4. **Expression Transformation**  
   - Create separate columns based on rank:  
     ```text
     col1 = IIF(rnk = 1, field2, NULL)
     col2 = IIF(rnk = 2, field2, NULL)
     col3 = IIF(rnk = 3, field2, NULL)
     ```
   - Each row now has only one populated column depending on its rank.
5. **Aggregator Transformation**  
   - Group by `field1`.  
   - Take `MAX(col1), MAX(col2), MAX(col3)` to collapse multiple rows into one.  
   - This produces pivoted output.
6. **Target** → Insert pivoted rows into target.

---

### Key Insight
- **Rank** ensures each value within a group gets a position.  
- **Expression** maps rank to a column.  
- **Aggregator** collapses rows into a single row per group, effectively pivoting data.  
- This approach is scalable if you know the maximum number of values per group (e.g., 3 or 4).

---

### Interview Tip
> “I used Rank to assign positions within each group, Expression to map ranks into columns, and Aggregator to collapse rows. This simulates a pivot operation in Informatica, converting rows into columns.”
