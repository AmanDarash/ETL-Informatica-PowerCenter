## Question: Add an extra column containing the salary of the previous row employee

### Input
A 1000
B 2000
C 3000


### Expected Output
A 1000 null
B 2000 1000
C 3000 2000


---

### Mapping Name
**m_AddPreviousRowSalary**

### Special Transformation
**RowShiftChampion**  
- Purpose: Captures the salary of the previous row using variable port evaluation order.  
- Why “Champion”? Because it solves a tricky problem (previous row logic) elegantly, and stands out as the hero of this mapping.

---

### Strategy
1. **Source Qualifier (SQ)** → Read data from source.
2. **Expression Transformation (RowShiftChampion)**  
   - Define variable ports in evaluation order:
     ```text
     cal_pre_sal (v) = IIF(ISNULL(pre_sal), NULL, pre_sal)
     pre_sal      (v) = sal
     out_pre_sal  (o) = cal_pre_sal
     ```
   - Explanation:
     - `pre_sal` holds the current row’s salary.
     - `cal_pre_sal` captures the previous row’s salary before `pre_sal` is updated.
     - `out_pre_sal` outputs the shifted value.
3. **Target** → Insert into target table with both current salary and previous row salary.

---

### Key Insight
- Variable ports in Expression are evaluated **top‑to‑bottom**.  
- This evaluation order allows us to simulate a **lag function**: the previous row’s salary is carried forward into the next row.  
- This is a creative solution that many developers assume is impossible in Expression, but works when ports are chained correctly.

---

### Interview Tip
> “I used Expression Transformation variables to simulate a lag function. By carefully ordering variable ports, I captured the previous row’s salary and passed it forward. This shows how algorithmic thinking can extend Informatica beyond its usual row‑by‑row limits.”
