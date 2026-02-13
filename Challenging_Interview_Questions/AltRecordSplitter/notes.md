## Question: Send alternate records into two different target files

### Example Input
A  
B  
C  
D  
E  

---

### Expected Output
- **File 1**: A, C, E  
- **File 2**: B, D  

---

### Strategy
1. **Source Qualifier (SQ)**  
   - Read data from the source.

2. **Sequence Generator (SG)**  
   - Start value = 1, Increment = 1.  
   - Assigns a sequence number to each row.

A → 1  
B → 2  
C → 3  
D → 4  
E → 5  

---

3. **Router Transformation**  
- Define two groups based on sequence number:  
  - **Odd Group**: `MOD(seq_val, 2) = 1` → goes to File 1.  
  - **Even Group**: `MOD(seq_val, 2) = 0` → goes to File 2.

4. **Target Files**  
- Connect Odd group → Target File 1.  
- Connect Even group → Target File 2.

---

### Mapping Name
**OddEvenRouter**  
(Descriptive, professional, and directly reflects the logic used.)

