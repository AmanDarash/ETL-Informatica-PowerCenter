# Mapping: m_TC_EmployeeDeptSplit

## ❓ Problem Statement
Split employee data into **department‑wise target files** using **Transaction Control Transformation**.  
For example:
- `HR.csv` → contains all HR employees  
- `IT.csv` → contains all IT employees  
- `Finance.csv` → contains all Finance employees  

---

## ⚙️ Strategy

1. **Source Qualifier (SQ)**  
   - Read data from two tables: `EMPLOYEE` and `DEPARTMENT`.  
   - Join by `Dept_ID` to get department names.

2. **Sorter Transformation**  
   - Sort rows by `Department_Name`.  
   - Ensures employees are grouped department‑wise before transaction control logic.

3. **Expression Transformation**  
   - Create variables and output ports:  
     ```text
     v_ifChange (V) = IIF(pre_Dept_id = Dept_id, 0, 1)
     pre_Dept_id (V) = Dept_id
     IfChange (O)    = v_ifChange
     file_name (O)   = Department_Name || '.csv'
     ```
   - `IfChange` flags when the department changes.  
   - `file_name` dynamically generates the target file name.

4. **Transaction Control Transformation (TC)**  
   - Define commit condition in properties:  
     ```text
     IIF(IfChange = 1, TC_COMMIT_BEFORE, TC_CONTINUE_TRANSACTION)
     ```
   - Commits data whenever the department changes, so each department’s rows go into a separate file.

5. **Target Definition**  
   - Flat file target.  
   - Configure session properties to use `file_name` port for dynamic file naming.  
   - Each department’s employees are written into their own `.csv` file.

---

## 📂 Flow
SQ(Employee + Department) → Sort(Department_Name) → Expression → Transaction Control → Target Flat Files

---

## 🎯 Key Insight
- **Transaction Control** allows you to break data into multiple transactions dynamically.  
- By committing when the department changes, you ensure each department’s employees are written into separate files.  
- This is more efficient than creating separate mappings or sessions for each department.

---

## 📝 Interview Tip
> “I used Transaction Control Transformation to split employee data into department‑wise files. The Expression transformation flags when the department changes and generates dynamic file names. Transaction Control commits before each change, so every department’s employees are grouped into their own file. This shows how Transaction Control can be used for file partitioning, not just database commits.”
