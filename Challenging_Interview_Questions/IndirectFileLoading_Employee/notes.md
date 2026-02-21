# Mapping: m_IndirectFileLoading_Employee

## ❓ Question
How can we load multiple `.csv` source files into a target using **Indirect File Loading** in Informatica?

---

## ⚙️ Strategy
1. **Create Control File**  
   - Prepare a text file (e.g., `emp_file_list.txt`) containing the list of all `.csv` files to be loaded.  
   - Example:
     ```
     emp_hr.csv
     emp_it.csv
     emp_finance.csv
     ```

2. **Source Definition**  
   - Point the flat file source to the control file (`emp_file_list.txt`).  
   - Ensure all listed `.csv` files share the same structure.

3. **Workflow Manager Settings**  
   - In the session properties, set **Source File Type = Indirect**.  
   - Provide the full path of the control file in the **Source File Name** property.

4. **Mapping Flow**  
   - SQ (Flat File Source) → Transformations (if needed) → Target.  
   - Informatica will automatically read each `.csv` file listed in the control file and load them into the target.

---

## 🎯 Key Insight
Indirect file loading allows you to process multiple source files in a single session run.  
Instead of creating separate mappings or sessions for each file, you simply maintain a control file listing all sources.

---

## 📝 Interview Tip
> “I used Indirect File Loading to process multiple employee `.csv` files. The control file listed all source files, and Informatica automatically loaded them sequentially. This approach reduces maintenance overhead and makes the ETL more scalable.”
